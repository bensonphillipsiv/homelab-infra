# homelab infra

This repository contains the full GitOps-driven IaC setup for my homelab infrastructure.  
It is built to be **fully reproducible**, allowing everything to be easily redeployed from scratch.

---

## Primary Infrastructure

- **Kubernetes**: [Talos Linux](https://www.talos.dev/)
- **GitOps**: [Argo CD](https://argo-cd.readthedocs.io/)
- **Secrets Management**: [Doppler](https://www.doppler.com/) + [External Secrets](https://external-secrets.io/)
- **Automated Dependency Updates**: [Renovate](https://docs.renovatebot.com/)
- **Distrbuted Volumes + Backups**: [Longhorn](https://longhorn.io/)
- **Cloud Infra Management**: [Crossplane](https://www.crossplane.io/)

---

## Applications

### Custom Apps

| App | Repo | Description |
|------|---------------|-------------|
| homeGPT | [bensonphillipsiv/homelab-builds](https://github.com/bensonphillipsiv/homelab-builds/tree/main/homeGPT) | Custom AI assistant |
| the-block | [bensonphillipsiv/homelab-builds](https://github.com/bensonphillipsiv/homelab-builds/tree/main/the_block) | Custom inertial remote for triggering home automations and scenes |

### OpenSource Apps

| App | Repo | Description |
|-------------|----------------------|-------------|
| Argo CD | [argoproj/argo-cd](https://github.com/argoproj/argo-cd) | GitOps continuous deployment tool for Kubernetes |
| AWX | [ansible/awx](https://github.com/ansible/awx) | Server for Ansible automation |
| cert-manager | [cert-manager/cert-manager](https://github.com/cert-manager/cert-manager) | Automates TLS certificate management in Kubernetes |
| Crossplane | [crossplane/crossplane](https://github.com/crossplane/crossplane) | Control plane framework for managing cloud infrastructure |
| ESPHome | [esphome/esphome](https://github.com/esphome/esphome) | Simple control of ESP32 devices |
| External Secrets | [external-secrets/external-secrets](https://github.com/external-secrets/external-secrets) | Kubernetes secret management |
| Grafana | [grafana/grafana](https://github.com/grafana/grafana) | Monitoring and observability platform |
| Home Assistant | [home-assistant/core](https://github.com/home-assistant/core) | Open-source home automation platform |
| Homepage | [gethomepage/homepage](https://github.com/gethomepage/homepage) | Personal dashboard |
| ingress-nginx | [kubernetes/ingress-nginx](https://github.com/kubernetes/ingress-nginx) | Ingress controller for Kubernetes using NGINX |
| Longhorn | [longhorn/longhorn](https://github.com/longhorn/longhorn) | Distributed block storage and backups for Kubernetes |
| mediamtx | [bluenviron/mediamtx](https://github.com/bluenviron/mediamtx) | RTSP/RTMP/HTTP media streaming server |
| metrics-server | [kubernetes-sigs/metrics-server](https://github.com/kubernetes-sigs/metrics-server) | Resource metrics API for Kubernetes |
| MQTT | [mqtt/mqtt.github.io](https://github.com/mqtt/mqtt.github.io) | Lightweight, publish-subscribe messaging protocol for small IoT sensors and devices.  |
| n8n | [n8n-io/n8n](https://github.com/n8n-io/n8n) | Workflow AI automation tool |
| Node-RED | [node-red/node-red](https://github.com/node-red/node-red) | Flow-based development tool for home automation |
| Postgres | [postgres/postgres](https://github.com/postgres/postgres) | PostgreSQL Database Management System |
| Tailscale | [tailscale/tailscale](https://github.com/tailscale/tailscale) | VPN based on WireGuard |

---
