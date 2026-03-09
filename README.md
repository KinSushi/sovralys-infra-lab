# sovralys-infra-lab

![Ubuntu 24.04](https://img.shields.io/badge/Ubuntu-24.04-E95420?logo=ubuntu&logoColor=white)
![KVM/QEMU](https://img.shields.io/badge/KVM%2FQEMU-Hypervisor-brightgreen)
![Tailscale](https://img.shields.io/badge/Tailscale-VPN-blue?logo=tailscale)
![Windows 10 Pro](https://img.shields.io/badge/Windows%2010-Pro-0078D6?logo=windows)
![MetaTrader 5](https://img.shields.io/badge/MetaTrader-5-FF6600)

Production infrastructure for an algo trader / ML engineer in retraining. OVH dedicated server running Ubuntu 24.04 as hypervisor, hosting a Windows 10 Pro KVM/QEMU VM for MetaTrader 5 24/7 trading, combined with a Data Science stack on the Ubuntu host (Jedha Bootcamp, April 2026).

**Dual purpose:** operational survival reference + public portfolio demonstrating infra/Linux/virtualization skills.

---

## Architecture

```
[Local PC — Windows 10]   Tailscale: 100.G.H.I
        |
        |  Tailscale VPN mesh (WireGuard)
        |
[OVH Dedicated Server — Ubuntu 24.04 Host]   Public IP: YOUR_SERVER_IP   Tailscale: 100.A.B.C
        |
        +--► [KVM/QEMU — Windows 10 Pro VM]
                  Private IP: 192.168.122.X (NAT)
                  Tailscale: 100.D.E.F
                  |-- MetaTrader 5 (24/7 live trading)
                  |-- MT5 Backtest Agents (distributed)
                  |-- Git + VS Code + Anaconda
                  +-- Office 365
```

---

## Design Principles

| Tool / Service | Machine | Reason |
|---|---|---|
| Docker / docker-compose | Ubuntu host only | Nested virtualization unavailable in KVM VM — Docker Desktop caused complete VM crash |
| MT5 + backtesting agents | Windows VM only | Isolation, 24/7 stability |
| Anaconda + Jupyter | Ubuntu host only | Native performance, Unix environment matching Jedha stack |
| RDP access | SSH tunnel via Tailscale | No port exposed publicly |
| noVNC port 6080 | Closed publicly (UFW deny) | SSH tunnel access only |
| Anaconda in VM | **Architectural mistake — corrected** | Was installed in VM, triggered crash with Docker Desktop. See `kvm-setup/lessons-learned.md` |

---

## Quick Reference

```bash
# SSH to server
ssh -i YOUR_SSH_KEY_PATH $USER@YOUR_SERVER_IP

# VM management
sudo virsh list --all
sudo virsh start windows10
sudo virsh shutdown windows10

# RDP (double-click bat file on local PC desktop)
ovh-rdp.bat

# Check all services
sudo virsh list --all
sudo systemctl status novnc
sudo systemctl status jupyter
```

---

## Infrastructure Decisions

| Decision | Justification |
|---|---|
| KVM/QEMU vs VirtualBox | Native kernel integration, better performance, virsh CLI |
| Docker on host only | Nested virtualization unavailable in KVM VM — Docker Desktop provoked complete crash |
| Tailscale vs port forwarding | Works behind NAT/CGNAT, no router config, end-to-end encryption |
| RDP via SSH tunnel | Avoids exposing port 3389 publicly |
| noVNC port 6080 closed | Security — SSH tunnel access only |
| Anaconda in VM | Mistake — caused CPU saturation with Docker Desktop. Fixed: Anaconda on Ubuntu host |
| bus=sata for Windows install | VirtIO requires a driver during install that Windows does not contain natively |
| Kernel updates blocked | Prevents surprise server reboots during backtests |
| SSHFS + Samba vs direct copy | XAUUSD tick dataset ~18 GB — VM disk space insufficient for physical copy |

---

## Timeline

| Date | Milestone |
|---|---|
| Feb 2026 | OVH server provisioned, Ubuntu 24.04 installed |
| Feb 2026 | KVM/QEMU configured, Windows 10 Pro VM created with VirtIO drivers |
| Feb 2026 | RDP + noVNC configured, Tailscale 3-node mesh operational |
| Feb 2026 | MT5 installed, distributed agent cluster configured |
| Feb 2026 | VM performance crash (Docker Desktop + Anaconda + Jedha tools conflict) |
| Feb 2026 | VM restored from backup, Docker isolated to Ubuntu host |
| Feb 2026 | Jupyter stack deployed on Ubuntu host, systemd services configured |
| Mar 2026 | SSHFS + Samba configured — mounting E:\ in VM for XAUUSD tick data |
| Apr 2026 | Jedha Bootcamp starts — infrastructure used for ML workloads |

---

## Repository Structure

```
sovralys-infra-lab/
|
|-- README.md
|-- .github/
|   +-- CONTRIBUTING.md
|
|-- kvm-setup/
|   |-- README.md              <- Full KVM installation from scratch
|   |-- vm-operations.md       <- Daily virsh commands
|   +-- lessons-learned.md     <- Real incidents + post-mortems
|
|-- networking/
|   |-- README.md              <- Tailscale mesh VPN 3 nodes
|   +-- sshfs-samba-mount.md   <- Mount E:\ from local PC into VM via SSHFS+Samba
|
|-- docker/
|   +-- README.md              <- Docker host-only isolation strategy
|
|-- trading-env/
|   |-- README.md              <- MT5 24/7 + distributed agents
|   +-- mt5-checklist.md       <- Professional checklist before going live
|
|-- backups/
|   |-- README.md              <- 3-layer backup strategy
|   +-- windows-key-card.html  <- Windows license card (HTML file, keep as-is)
|
+-- data-science-env/
    +-- README.md              <- Jedha stack on Ubuntu host
```

---

## Related Repositories

- [OpenClassroomsProject](https://github.com/KinSushi/OpenClassroomsProject) — WordPress developer track RNCP38145
- [git-workflow-demo](https://github.com/KinSushi/git-workflow-demo) — Git workflow reference
