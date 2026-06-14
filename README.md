# pantheon-cluster
Private cloud infrastructure built on Raspberry pi 5 &amp; K3s
[Pantheon_Cluster_Dokumentation_GitHub.docx](https://github.com/user-attachments/files/28930877/Pantheon_Cluster_Dokumentation_GitHub.docx)
# 🏛️ Pantheon Cluster

> *ALEA IACTA EST – The die is cast.*

A fully self-hosted private cloud infrastructure built on **Raspberry Pi 5** and **K3s (Kubernetes)** — designed, implemented, and documented from scratch.

![K3s](https://img.shields.io/badge/K3s-v1.35.5-blue?style=flat-square)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Ready-326CE5?style=flat-square)
![Raspberry Pi](https://img.shields.io/badge/Raspberry_Pi-5-C51A4A?style=flat-square)
![Tailscale](https://img.shields.io/badge/VPN-Tailscale-242424?style=flat-square)
![Status](https://img.shields.io/badge/Status-Active-success?style=flat-square)

---

## 📋 Overview

Pantheon is a 2-node Kubernetes cluster running on Raspberry Pi 5 hardware. The goal is complete digital sovereignty — no Google Drive, no iCloud, no third-party cloud. Everything self-hosted, everything under control.

Built as a personal learning project and portfolio piece, Pantheon covers real-world topics: container orchestration, persistent storage, mesh VPN, DNS filtering, and private AI inference.

---

## 🖥️ Hardware

| Node | Role | Hardware | RAM | Storage |
|------|------|----------|-----|---------|
| **Centurion** | Master (control-plane) | Raspberry Pi 5 | 8 GB | 1 TB NVMe SSD (Crucial P310, PCIe Gen4) |
| **Praetor** | Worker | Raspberry Pi 5 | 4 GB | MicroSD (NVMe planned) |

**Centurion** runs in a SunFounder Pironman 5 enclosure with:
- IceCube Tower Cooler with copper heatpipes
- OLED display (real-time system stats)
- RGB lighting
- NVMe PIP board (PCIe passthrough)
- IO HAT+ for GPIO control

---

## 🛠️ Stack

| Service | Purpose | Orchestration |
|---------|---------|---------------|
| **K3s** | Lightweight Kubernetes | — |
| **Pi-Hole** | Network-wide DNS filtering (84k+ domains) | K3s (Praetor) |
| **Ollama + Nexus** | Private AI inference (Llama3 4-bit) | K3s (Centurion) |
| **Open WebUI** | Chat interface for Nexus AI | K3s (Centurion) |
| **Nextcloud** | Private cloud storage & photo sync | K3s (Centurion) |
| **MariaDB** | Database backend for Nextcloud | K3s (Centurion) |
| **Portainer CE** | Container management UI | Docker |
| **Tailscale** | Mesh VPN (WireGuard-based) | System |

---

## 🗺️ Architecture

```
┌─────────────────────────────────────────────┐
│              Pantheon Cluster               │
│                                             │
│  ┌──────────────┐     ┌──────────────────┐  │
│  │  CENTURION   │     │     PRAETOR      │  │
│  │  Master Node │────▶│   Worker Node    │  │
│  │   8GB RAM    │     │    4GB RAM       │  │
│  │  1TB NVMe    │     │                  │  │
│  │              │     │   Pi-Hole DNS    │  │
│  │  Nextcloud   │     │                  │  │
│  │  Ollama/AI   │     │                  │  │
│  │  Open WebUI  │     │                  │  │
│  └──────────────┘     └──────────────────┘  │
│          │                     │            │
│          └─────────────────────┘            │
│               LAN (0.24ms)                  │
└─────────────────────────────────────────────┘
                    │
              Tailscale VPN
                    │
        ┌───────────┴───────────┐
        │                       │
   gfn-Laptop              iPhone 11
   (Windows)               (iOS)
```

---

## 📁 Repository Structure

```
pantheon-cluster/
├── k3s/
│   ├── pihole/
│   │   ├── pihole-namespace.yaml
│   │   ├── pihole-pvc.yaml
│   │   ├── pihole-deployment.yaml
│   │   └── pihole-service.yaml
│   ├── ollama/
│   │   ├── ollama-namespace.yaml
│   │   ├── ollama-pvc.yaml
│   │   ├── ollama-deployment.yaml
│   │   └── ollama-service.yaml
│   └── nextcloud/
│       └── (manifests)
├── docs/
│   └── Pantheon_Cluster_Documentation.docx
├── scripts/
│   └── sysupdate.sh
└── README.md
```

---

## 🚀 Project Timeline

| Date | Phase | Milestone |
|------|-------|-----------|
| 10.05.2026 | Phase I | Raspberry Pi 5 setup, headless OS, Docker, Portainer |
| 11.05.2026 | Phase I | Ollama deployed, Nexus AI model created |
| 12.05.2026 | Phase II | Tailscale Mesh-VPN configured |
| Mid May | Phase II | Terminal dashboard, Pi-Hole DNS filtering |
| 30.05.2026 | Phase I+ | SunFounder Pironman 5 build |
| 01.06.2026 | Phase I+ | Crucial P310 NVMe SSD installed |
| 02.06.2026 | Phase I+ | NVMe boot successful → renamed Goliath → **Centurion** |
| 03.06.2026 | Phase III | K3s cluster online — Centurion + Praetor |
| 13.06.2026 | Phase III | Full Docker → K3s migration, Nextcloud deployed |

---

## 🧠 Key Learnings

- **K3s on ARM**: Kubernetes on Raspberry Pi requires cgroup memory configuration (`cgroup_memory=1 cgroup_enable=memory` in `/boot/firmware/cmdline.txt`)
- **Proxmox ≠ ARM**: Proxmox is x86-only — K3s is the correct choice for Raspberry Pi clusters
- **ClusterIP vs NodePort**: Internal services (Ollama, MariaDB) use ClusterIP; external-facing services (WebUI, Nextcloud) use NodePort
- **Persistent Volumes**: Container storage is ephemeral — PV/PVC required for stateful services
- **Mesh VPN over Port Forwarding**: Tailscale provides zero-config WireGuard VPN without exposing the router
- **DNS Filtering Challenges**: IPv6 DNS leakage bypasses Pi-Hole — solution: disable IPv6 client-side

---

## 🔧 Troubleshooting Log

Real problems solved during the build:

| Problem | Root Cause | Solution |
|---------|-----------|----------|
| Pi-Hole not filtering | ISP router (Speedport) overriding DNS | Disabled IPv6, DHCP moved to Pi-Hole |
| NVMe boot failure | Wrong PARTUUID in cmdline.txt | `sed` to correct PARTUUID |
| NVMe not detected | Missing `dtparam=nvme` in config.txt | Added Pi 5 specific overlay |
| K3s install failed | cgroup memory not enabled | Added cgroup flags to cmdline.txt |
| DPI firewall blocking VPN | Corporate network blocks WireGuard | iPhone USB tethering as workaround |

---

## 🗓️ Roadmap

- [x] 2-node K3s cluster
- [x] Pi-Hole DNS filtering
- [x] Ollama / Nexus private AI
- [x] Nextcloud private cloud
- [ ] Tailscale remote access for all services
- [ ] HTTPS via Cert-Manager
- [ ] Grafana + Prometheus monitoring
- [ ] GitOps with ArgoCD
- [ ] 3rd node: Pi 5 16GB (planned Q4 2026)
- [ ] Synology NAS 16TB + Jellyfin (planned Q1 2027)

---

## 📸 Setup

*Centurion (left, RGB active, Pironman 5) — Praetor (right, aluminium passive) — Switch (background)*

---

## ⚙️ Deployment

> **Note:** Replace all placeholder values (`192.168.x.x`, `<YOUR_PASSWORD>`, `<YOUR_TOKEN>`) with your own before deploying.

```bash
# Clone the repository
git clone https://github.com/marcelzint123/pantheon-cluster.git
cd pantheon-cluster

# Deploy Pi-Hole
kubectl apply -f k3s/pihole/

# Deploy Ollama + Open WebUI
kubectl apply -f k3s/ollama/

# Verify cluster
kubectl get pods --all-namespaces
```

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.

---

*Built with curiosity, persistence, and way too much troubleshooting. 🏛️*
