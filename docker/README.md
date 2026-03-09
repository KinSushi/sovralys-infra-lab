# Docker — Host-Only Isolation Strategy

**Rule: Docker runs on the Ubuntu host only. Never inside the Windows VM.**

---

## Why Not in the VM

Root cause documented in `../kvm-setup/lessons-learned.md` (Incident 1 and Incident 5):

Docker Desktop for Windows requires Hyper-V or WSL2, both of which need hardware virtualization features exposed to the guest OS. KVM does not expose these by default (nested virtualization). Installing Docker Desktop in the VM caused a complete performance collapse — the installer completed silently but Docker launched virtualization layers in a loop, saturating CPU.

**Alternative (nested virtualization — not adopted):** It is technically possible to enable nested virtualization so that Docker Desktop works in the VM. Not adopted because:
- Significant performance overhead (VM inside VM inside OVH hypervisor = 3 layers)
- Stability risks for MT5 24/7
- Docker is more useful on Linux for Jedha ML/Data Engineering workloads

---

## Installation on Ubuntu Host

```bash
sudo apt install -y docker.io docker-compose
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world  # Test
```

On native Linux Docker: no virtualization constraints — uses kernel namespaces and cgroups directly. Maximum performance, no workarounds.

---

## Tool Allocation

| Tool | Machine | Notes |
|---|---|---|
| Docker / docker-compose | Ubuntu host | Native Linux, full performance |
| Anaconda | Ubuntu host | Native Python environment |
| Jupyter | Ubuntu host | Systemd service, port 8888 |
| Python (pyenv) | Ubuntu host | Version management |
| PostgreSQL | Ubuntu host | Data persistence |
| VS Code | Windows VM | MQL5 development |
| MetaTrader 5 | Windows VM | 24/7 trading, isolation required |
| Git | Both | Sync and versioning |
