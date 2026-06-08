# sovralys-infra-lab

<div align="center">

**Personal Data / ML Infrastructure Lab**

Ubuntu host · KVM/QEMU VM · Private networking · Docker/Jupyter separation · Recovery procedures · Operational documentation

![Ubuntu 24.04](https://img.shields.io/badge/Ubuntu-24.04-E95420?style=flat&logo=ubuntu&logoColor=white)
![KVM/QEMU](https://img.shields.io/badge/KVM%2FQEMU-Hypervisor-2EA043?style=flat)
![Tailscale](https://img.shields.io/badge/Tailscale-Private%20VPN-24292F?style=flat&logo=tailscale&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Host%20Only-2496ED?style=flat&logo=docker&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Data%20Science-F37626?style=flat&logo=jupyter&logoColor=white)
![Windows VM](https://img.shields.io/badge/Windows-VM%20Workload-0078D6?style=flat&logo=windows&logoColor=white)

</div>

---

## Purpose

This repository documents a personal infrastructure lab used to practice the operational foundations required for data engineering, MLOps and future data-platform leadership.

The lab is built around an OVH dedicated server running **Ubuntu 24.04** as the host operating system. It runs a **KVM/QEMU Windows VM** for Windows-only workloads and keeps the **data-science / Docker / Jupyter stack on the Ubuntu host** for stability, performance and cleaner service isolation.

The repository is both:

1. an operational survival reference for maintaining the environment;
2. a public portfolio artifact demonstrating Linux, virtualization, networking, documentation and incident-recovery skills.

---

## Why this matters for Data / MLOps roles

A future Data Lead, MLOps Lead or Data Platform Engineer is not only expected to write notebooks. They must understand the runtime environment behind data products:

| Capability | Why it matters |
|---|---|
| Ubuntu host operations | Real data and ML workloads run on Linux servers, containers and scheduled services |
| VM lifecycle management | Enterprise environments often isolate Windows-only tools, legacy systems or vendor software |
| Private networking | Production systems must avoid exposing administrative ports publicly |
| Docker/Jupyter separation | Data-science tooling must be isolated from fragile desktop/VM workloads |
| Incident post-mortems | Production reliability requires documenting failures, root causes and corrective actions |
| Backup and recovery | Data-platform operations require recoverable systems, not one-off local setups |

This lab is therefore part of the Data/MLOps path: it shows operational maturity before moving to larger banking-data and ML-platform projects.

---

## Architecture

```text
[Local PC — Windows]
        |
        | Private Tailscale VPN mesh
        |
[OVH Dedicated Server — Ubuntu 24.04 Host]
        |
        |-- KVM/QEMU virtualization
        |
        +-- [Windows 10 Pro VM]
        |       |-- Windows-only workload tools
        |       |-- Backtesting / market-data tooling
        |       |-- Git + VS Code + Office tools
        |
        +-- [Ubuntu Host Data/ML Services]
                |-- Docker / docker-compose
                |-- Jupyter / Python environment
                |-- SSH access
                |-- noVNC service, tunnel-only access
                |-- backups and recovery procedures
```

No administrative service is designed to be exposed publicly. Access is routed through private networking and SSH tunnels.

---

## Design principles

| Decision | Rationale |
|---|---|
| Ubuntu host as the stable compute layer | Native Linux environment for data science, Docker, automation and server administration |
| KVM/QEMU for Windows workload isolation | Keeps Windows-only tooling separate from host-level Data/ML services |
| Docker on Ubuntu host only | Avoids nested-virtualization instability inside the Windows VM |
| Jupyter on Ubuntu host only | Keeps the data-science environment close to the Linux runtime used in production |
| Tailscale instead of public port exposure | Private mesh networking with no router configuration and less attack surface |
| RDP / noVNC through tunnel-only access | Administrative interfaces are not meant to be public |
| Post-mortem documentation | Failures are treated as engineering evidence, not hidden mistakes |
| Sanitized public documentation | Public README uses placeholders for addresses, users and secrets |

---

## Operational lessons already documented

| Incident / decision | Lesson |
|---|---|
| Docker Desktop inside Windows VM caused instability | Docker belongs on the Ubuntu host in this architecture |
| Anaconda inside the VM increased resource pressure | Data-science stack moved to the Ubuntu host |
| VM recovery was required after a performance crash | Backup and recovery procedures must exist before production use |
| noVNC exposed by default would be unsafe | Administrative ports must be blocked and tunnel-only |
| Large tick/market datasets exceeded VM-local storage assumptions | Mounting and storage strategy must be designed before processing large data |

---

## Quick reference

```bash
# SSH to the host
ssh -i YOUR_SSH_KEY_PATH YOUR_USER@YOUR_SERVER_IP

# VM management
sudo virsh list --all
sudo virsh start windows10
sudo virsh shutdown windows10

# Check services
sudo systemctl status novnc
sudo systemctl status jupyter

# Firewall review
sudo ufw status verbose
```

---

## Repository structure

```text
sovralys-infra-lab/
|
|-- README.md
|-- .github/
|   +-- CONTRIBUTING.md
|
|-- kvm-setup/
|   |-- README.md              # Full KVM installation from scratch
|   |-- vm-operations.md       # Daily virsh commands
|   +-- lessons-learned.md     # Real incidents and post-mortems
|
|-- networking/
|   |-- README.md              # Tailscale private VPN mesh
|   +-- sshfs-samba-mount.md   # Remote mount strategy for large local datasets
|
|-- docker/
|   +-- README.md              # Docker host-only isolation strategy
|
|-- data-science-env/
|   +-- README.md              # Jupyter / Python stack on Ubuntu host
|
|-- trading-env/
|   |-- README.md              # Windows-only workload environment
|   +-- mt5-checklist.md       # Operational checklist
|
+-- backups/
    +-- README.md              # Backup and recovery strategy
```

---

## Portfolio signal

This repository is intentionally not a polished cloud product. It is an infrastructure lab that proves practical exposure to:

- Linux server administration;
- KVM/QEMU virtualization;
- private networking with Tailscale;
- SSH and tunnel-based access patterns;
- Docker/Jupyter runtime separation;
- incident documentation and recovery;
- operational checklists;
- security-oriented defaults.

For banking-data and MLOps applications, this repo supports the infrastructure layer behind future projects such as:

- `banking-dataops-monitoring`;
- `fraud-mlops-control-tower`;
- `jedha-rncp35288-portfolio`;
- `secure-wealth-rag-assistant`.

---

## Related repositories

- [pty-flights-pricing](https://github.com/KinSushi/pty-flights-pricing) — production Python automation pipeline
- [git-workflow-demo](https://github.com/KinSushi/git-workflow-demo) — Git workflow reference
- [OpenClassroomsProject](https://github.com/KinSushi/OpenClassroomsProject) — web development track RNCP38145

---

<sub>Public documentation is sanitized. Real credentials, private IPs, keys and license values must never be committed.</sub>
