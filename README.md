# Home Infra

This repository contains the full GitOps-driven IaC setup for my home Kubernetes cluster.  
It is built to be **fully reproducible**, allowing the entire cluster to be easily redeployed from scratch.

---

## Infrastructure

- **Kubernetes**: [Talos Linux](https://www.talos.dev/)  
- **GitOps**: [Argo CD](https://argo-cd.readthedocs.io/)  
- **Secrets Management**: [Doppler](https://www.doppler.com/) + [External Secrets](https://external-secrets.io/)  
- **Automated Dependency Updates**: [Renovate](https://docs.renovatebot.com/)  
- **Volumes + Backups**: [Longhorn](https://longhorn.io/)

---

## Applications

- **Argo CD** – GitOps controller
- **AWX** – Ansible automation platform
- **cert-manager** – Automated TLS certificates
- **Crossplane** – Cloud resource provisioning
- **ESPHome** – IoT device management
- **External Secrets** – Kubernetes secrets from Doppler
- **Grafana** – Metrics visualization
- **Home Assistant** – Home automation hub
- **home-gpt** – Custom AI assistant
- **Homepage** – Dashboard
- **ingress-nginx** – Ingress controller
- **Longhorn** – Distributed storage + backups
- **mediamtx** – Media streaming server
- **metrics-server** – Resource metrics for Kubernetes
- **MQTT** – Message broker
- **n8n** – AI automation workflows
- **Node-RED** – Event-driven home automation
- **Postgres** – Database
- **Tailscale** – Secure networking
- **the-block** – Custom inertial remote

---
