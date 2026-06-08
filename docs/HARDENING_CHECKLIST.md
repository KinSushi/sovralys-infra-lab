# Hardening Checklist

This checklist documents the minimum hardening baseline for the personal Ubuntu/KVM infrastructure lab.

It is not a production security baseline. It is a portfolio-quality operational checklist showing disciplined infrastructure hygiene for DataOps, MLOps and IT production roles.

---

## Access control

| Check | Expected state | Evidence command |
|---|---|---|
| SSH key authentication | Enabled | `sudo grep -E "PubkeyAuthentication|PasswordAuthentication" /etc/ssh/sshd_config` |
| Password SSH login | Disabled or not used | `sudo grep PasswordAuthentication /etc/ssh/sshd_config` |
| Root SSH login | Disabled | `sudo grep PermitRootLogin /etc/ssh/sshd_config` |
| Admin access | Private network or tunnel only | `tailscale status` |
| SSH keys | Never committed | `.gitignore`, `SECURITY.md` |

---

## Network exposure

| Check | Expected state | Evidence command |
|---|---|---|
| UFW enabled | Active | `sudo ufw status verbose` |
| RDP public exposure | Blocked | `sudo ss -tulpn | grep 3389` |
| noVNC public exposure | Blocked / tunnel-only | `sudo ss -tulpn | grep 6080` |
| Jupyter public exposure | Blocked / private only | `sudo ss -tulpn | grep jupyter` |
| Unexpected listeners | Reviewed | `sudo ss -tulpn` |

---

## Host runtime

| Check | Expected state | Evidence command |
|---|---|---|
| Disk usage | Below operational threshold | `df -h` |
| Memory pressure | Reviewed | `free -h` |
| Load average | Reviewed | `uptime` |
| Failed services | None unexplained | `systemctl --failed` |
| Recent errors | Reviewed | `journalctl -p 3 -xb --no-pager` |

---

## Virtualization

| Check | Expected state | Evidence command |
|---|---|---|
| VM state known | Running / stopped intentionally | `sudo virsh list --all` |
| VM autostart policy known | Explicitly documented | `sudo virsh dominfo windows10` |
| VM snapshots/backups | Documented | backup inventory |
| Host/VM separation | Maintained | `docs/ADR-001-host-vm-separation.md` |

---

## Secrets and public documentation

| Check | Expected state |
|---|---|
| `.env` files | Never committed |
| SSH private keys | Never committed |
| Public IPs / hostnames | Replaced with placeholders |
| Licenses | Kept outside GitHub |
| Screenshots | Sanitized before publication |
| Logs | Sanitized before publication |

---

## Data / MLOps readiness

| Check | Expected state |
|---|---|
| Docker services | Host-only unless explicitly justified |
| Jupyter | Host-level, private access only |
| PostgreSQL demos | Docker Compose, synthetic data only |
| MLflow demos | Private/tunnel-only access |
| Vector DB demos | Synthetic documents only |
| Runtime docs | Linked from `docs/DATA_MLOPS_RUNTIME_STACK.md` |

---

## Release checklist

Before updating public documentation:

- [ ] no secrets;
- [ ] no real IPs or hostnames;
- [ ] no license values;
- [ ] no raw private logs;
- [ ] no screenshots with identifiers;
- [ ] no public admin endpoint described as open;
- [ ] all commands use placeholders where needed;
- [ ] public-safety note included if relevant.
