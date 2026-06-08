# Monitoring Baseline

This document defines lightweight monitoring checks for the personal Ubuntu/KVM infrastructure lab.

The goal is to show production-oriented habits: know what is running, where logs are, what ports are exposed, and how to detect abnormal host or VM behavior.

---

## Daily checks

```bash
# Host health
uptime
free -h
df -h

# Failed services
systemctl --failed

# VM state
sudo virsh list --all

# Docker state
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Firewall
sudo ufw status verbose
```

---

## Weekly checks

```bash
# Listening ports
sudo ss -tulpn

# Recent system errors
sudo journalctl -p 3 -xb --no-pager

# Disk consumers
sudo du -h --max-depth=1 /var /home 2>/dev/null | sort -h

# Service logs
sudo journalctl -u jupyter -n 100 --no-pager
sudo journalctl -u novnc -n 100 --no-pager
```

---

## VM monitoring

```bash
# VM list
sudo virsh list --all

# VM details
sudo virsh dominfo windows10

# VM CPU / memory live view
virt-top
```

If `virt-top` is unavailable:

```bash
sudo apt install virt-top
```

---

## Docker monitoring

```bash
# Container state
docker ps -a

# Resource usage
docker stats --no-stream

# Logs for one container
docker logs --tail 100 <container_name>

# Disk usage
docker system df
```

---

## Network monitoring

```bash
# Tailscale mesh
tailscale status

# Local interfaces
ip addr

# Routes
ip route

# Listening sockets
sudo ss -tulpn

# Firewall
sudo ufw status numbered
```

Expected principle:

- administrative services should not be exposed publicly;
- private network or SSH tunnel access is preferred;
- public documentation should not reveal real hosts or IPs.

---

## Data / MLOps service checks

Future services should define their own checks, but the host-level pattern is:

| Service type | Example check |
|---|---|
| PostgreSQL | `pg_isready -h localhost -p 5432` |
| MLflow | `curl http://localhost:5000/health` if configured |
| FastAPI | `curl http://localhost:8000/health` |
| Streamlit | `curl http://localhost:8501` |
| Vector DB | service-specific health endpoint |

All ports should remain private or tunnel-only unless there is a documented reason.

---

## Incident triggers

Create an incident note if any of these happen:

- disk usage exceeds threshold;
- VM crashes or hangs;
- administrative service becomes publicly reachable;
- Jupyter, Docker or PostgreSQL fails to start;
- backup restore test fails;
- unexpected process listens on a public interface;
- repeated authentication failures appear in logs.

Use `docs/OPERATIONS_RUNBOOK.md` for the incident note template.
