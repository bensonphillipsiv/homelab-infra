# terminus

[Terminus](https://github.com/usetrmnl/terminus) is the self-hosted TRMNL server. It is the dashboard source for the Glyph (TRMNL device-API client).

- **URL**: https://terminus.bensonphillips.dev (Let's Encrypt via DNS-01, LAN only)
- **Image**: `ghcr.io/usetrmnl/terminus` (pinned, bumped by Renovate)
- **Pod**: `db-setup` init container (creates role + database `terminus` on `apps/postgres`, runs `hanami db migrate`), `web` (puma on 2300), `worker` (sidekiq)
- **Valkey**: `terminus-valkey`, no persistence (Sidekiq queues and Rack::Attack counters, re-created at Terminus boot)
- **Storage**: PVCs `terminus-uploads` (2Gi) and `terminus-fonts` (500Mi)

---

## Before the first sync: Doppler

Add one key to the Doppler config the `doppler-token` service token points at (the one holding `HOMEASSISTANT_TOKEN`, `CLOUDFLARE_API_KEY`, `TAILSCALE_TOKEN`):

| Doppler key | Value | Why |
|-------------|-------|-----|
| `TERMINUS_APP_SECRET` | output of `openssl rand -hex 64` (128 hex chars; Terminus requires at least 64) | Session/CSRF/cookie key. Must be stable across restarts, otherwise every login and JWT dies on each pod restart. |

```bash
openssl rand -hex 64
```

Nothing else is secret (`DATABASE_URL` has no password under the cluster Postgres' trust auth; Valkey is ClusterIP only). Do not commit the value anywhere.

---

## Deploy and verify

### 1. Watch it come up

```bash
kubectl -n terminus get pods -w
kubectl -n terminus logs deploy/terminus -c db-setup       # role/db creation, then migration output, exit 0
kubectl -n terminus logs deploy/terminus -c web | head -50  # "hanami assets compile", puma listening on 2300
kubectl -n terminus logs deploy/terminus -c worker | grep -i -E "synchron|error" | head
kubectl -n terminus get certificate,externalsecret,pvc
```

- `ExternalSecret terminus` must be `SecretSynced`.
- `Certificate letsencrypt` must be `Ready=True` (DNS-01 takes 1–3 min).

If `db-setup` fails on `CREATE EXTENSION citext` (permission denied because `citext` is not trusted on this build), either run once by hand and delete the pod:

```bash
kubectl -n postgres exec postgresql-0 -- psql -U postgres -c "ALTER ROLE terminus SUPERUSER"
```

or switch `DATABASE_URL` to `postgres://postgres@…/terminus`.

If the migration fails on PostgreSQL 17 (upstream runs 18; **[unverified]** that every migration runs on 17), the fallback is a dedicated `postgres:18-alpine` Deployment in this app.

### 2. Certificate chain as the device sees it

Must be Let's Encrypt, chaining to ISRG Root X1:

```bash
openssl s_client -connect terminus.bensonphillips.dev:443 -servername terminus.bensonphillips.dev </dev/null 2>/dev/null \
  | openssl x509 -noout -subject -issuer
curl -sS -o /dev/null -w '%{http_code}\n' https://terminus.bensonphillips.dev/up     # 200 (or a 30x), never a TLS error
```

### 3. First login

Do this immediately: **the first registered user becomes the admin**. Open https://terminus.bensonphillips.dev, click **Register**, enter email + password. Optionally keep the credentials in Doppler (`TERMINUS_ADMIN_LOGIN` / `TERMINUS_ADMIN_PASSWORD`; not consumed by the cluster).

### 4. Smoke test `/api/setup` from the LAN

Proves TLS + `API_URI` + Rack::Attack together before touching the board. Use a locally administered fake MAC in upper-case hex (the schema rejects lower case) and a real User-Agent (blank UAs are blocklisted):

```bash
curl -sS -i https://terminus.bensonphillips.dev/api/setup \
  -H 'ID: 02:47:4C:59:50:01' \
  -H 'Model: glyph_v1' \
  -H 'FW-Version: 0.1.0' \
  -H 'User-Agent: Glyph/smoke' \
  -H 'Content-Type: application/json'
```

Expected: `HTTP/2 200` with

```json
{"api_key":"<30 chars>","image_url":"https://terminus.bensonphillips.dev/assets/setup.bmp","message":"Welcome to Terminus!","status":200}
```

The `image_url` host must be exactly `https://terminus.bensonphillips.dev`, which proves `API_URI` is right.

| Response | Cause | Fix |
|----------|-------|-----|
| 403 `Forbidden` | Blank-UA blocklist (header missing), or, if the body says Rack::Attack, the source address was not private | Find the client address with `kubectl -n ingress-nginx logs deploy/ingress-nginx-controller \| grep api/setup` and add that subnet to `RACK_ATTACK_ALLOWED_SUBNETS` |
| 503 `Server Error` | Throttle hit (10 `/api/setup` per 3 min per non-safelisted IP) | Same diagnosis as 403 |
| 404 problem details `#device_setup` | Model sync has not run yet (fallback model `og_plus` missing) | Wait a minute after the worker started, check `kubectl -n terminus logs deploy/terminus -c worker`, confirm egress to trmnl.com |
| 422 | A header failed the schema | Check MAC case and `FW-Version` format (`X.Y.Z`) |

Repeat the same curl: the second call returns `"api_key":""` (known device, key redacted). The Glyph firmware relies on this behaviour.

### 5. Delete the fake device

In the UI: Devices → the row with MAC `02:47:4C:59:50:01` → Delete. Or via the API:

```bash
TOKEN=$(curl -sS -X POST https://terminus.bensonphillips.dev/login \
  -H 'Content-Type: application/json' \
  -d '{"login":"<email>","password":"<password>"}' | python3 -c 'import json,sys; print(json.load(sys.stdin)["access_token"])')
curl -sS https://terminus.bensonphillips.dev/api/devices -H "Authorization: Bearer $TOKEN" -H 'Content-Type: application/json' \
  | python3 -c 'import json,sys; [print(d["id"], d["mac_address"], d["label"]) for d in json.load(sys.stdin)["data"]]'
curl -sS -X DELETE https://terminus.bensonphillips.dev/api/devices/<id> -H "Authorization: Bearer $TOKEN" -H 'Content-Type: application/json' -d '{}'
```

- `Authorization: Bearer <token>` vs a bare token: the upstream doc redacts the header; Rodauth's JWT feature accepts both **[unverified, minor]**.
- Also delete the auto-created playlist "Device N" on the Playlists page if it survives the device delete **[unverified whether it cascades]**.

### 6. Create model `glyph_v1`

Models → New in the UI, or:

```bash
curl -sS -X POST https://terminus.bensonphillips.dev/api/models \
  -H "Authorization: Bearer $TOKEN" -H 'Content-Type: application/json' \
  -d '{"model":{"name":"glyph_v1","label":"Glyph v1","description":"Glyph v1, 400x300 1-bit e-paper","mime_type":"image/png","bit_depth":1,"colors":2,"scale_factor":1,"rotation":0,"offset_x":0,"offset_y":0,"width":400,"height":300}}'
```

`default_palette_id` is optional; if the UI insists on a palette, pick the synced 1-bit black/white one. Leave `css` empty (it only matters for framework-rendered extension screens at this size).

### 7. Per device

After the Glyph's first `/api/setup` it appears as model `og_plus`. Then:

- **Devices → the Glyph → Edit**: Model `glyph_v1`, Refresh Rate as wanted (900 s default), Command **Next Screen** (`screen_forward`), Firmware Update **off**, no sleep window (`sleep_start_at` / `sleep_stop_at` empty).
- **Playlists → "Device N"**: set **Manual**, remove the `og_plus` Welcome item, add `glyph_v1` screens (Screens → upload a 400×300 PNG, or an extension rendering for `glyph_v1`).
- **Key recovery later**: tick **Firmware Reset** on the device, then press B on the Glyph.

---

## Notes

- `RACK_ATTACK_ALLOWED_SUBNETS` is empty on purpose: behind ingress-nginx, a LAN client is seen as a private 192.168.x address and is safelisted by default. Add `100.64.0.0/10` only if the UI is driven over Tailscale and the 300 req / 5 min throttle bites.
- Valkey has no persistence. If Valkey alone restarts while Terminus keeps running, the Sidekiq cron schedules are gone until Terminus restarts: `kubectl -n terminus rollout restart deploy/terminus`.
- `FIRMWARE_SYNCHRONIZER=false` (no TRMNL hardware here). Model and font sync stay on; the `og_plus` fallback and the fonts come from them, so egress to trmnl.com is required.
