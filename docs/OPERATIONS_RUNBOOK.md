# Operations Runbook

This runbook documents the minimum operational checks for the personal Ubuntu/KVM infrastructure lab.

It is written as a portfolio-quality document: short, repeatable, and focused on how an operator investigates and recovers systems.

## 1. Daily health checks

```bash
# Host uptime and load
uptime
free -h
df -h

# VM state
sudo virsh list --all

# Services
sudo systemctl status jupyter --no-pager
sudo systemctl status novnc --no-pager

# Firewall
sudo ufw status verbose
```

Expected result:

- Ubuntu host reachable over private access path.
- Windows VM state known: running, shut off or intentionally paused.
- Jupyter service state known.
- noVNC not publicly exposed.
- Disk usage below operational threshold.

## 2. VM recovery checklist

```bash
# Check VM list
sudo virsh list --all

# Graceful shutdown if running
sudo virsh shutdown windows10

# Force stop only if the VM is stuck
sudo virsh destroy windows10

# Start VM again
sudo virsh start windows10

# Inspect VM details
sudo virsh dominfo windows10
```

Recovery notes:

1. Prefer graceful shutdown before force stop.
2. Record timestamps and symptoms before acting.
3. If the VM repeatedly crashes, stop new workload changes and review recent installs.
4. Check whether host-level CPU, RAM or disk pressure explains the incident.

## 3. Service recovery checklist

```bash
# Restart service
sudo systemctl restart jupyter

# Review status
sudo systemctl status jupyter --no-pager

# Review recent logs
sudo journalctl -u jupyter -n 100 --no-pager
```

Root-cause questions:

- Did the service fail after a configuration change?
- Did disk space, memory or port binding cause the failure?
- Did a package upgrade change runtime behavior?
- Is the failure reproducible?

## 4. Network access checklist

```bash
# Tailscale status
tailscale status

# Host routes
ip route

# Listening ports
sudo ss -tulpn

# Firewall
sudo ufw status numbered
```

Security expectation:

- No administrative service should be intentionally exposed on a public interface.
- Administrative access should use private networking or SSH tunnels.
- Any public exposure must be explicit, documented and justified.

## 5. Backup verification checklist

Backup creation is not enough. A backup has value only if it can be restored.

Minimum checklist:

1. Identify backup source and target.
2. Confirm backup timestamp.
3. Confirm file size is plausible.
4. Confirm checksum when applicable.
5. Perform test restore on non-critical path when possible.
6. Document restore result.

Example checksum:

```bash
sha256sum backup-file.tar.gz
```

## 6. Incident note template

```markdown
# Incident: <short title>

## Date / time

YYYY-MM-DD HH:MM local time

## Impact

What stopped working? Which workload was affected?

## Symptoms

Observed errors, logs, resource pressure or user-visible behavior.

## Root cause

Technical explanation after investigation.

## Resolution

Steps taken to restore service.

## Preventive action

What will be changed to avoid recurrence?

## Evidence

Commands, logs, screenshots or config references.
```

## 7. Change-management principle

Every non-trivial infrastructure change should answer three questions before implementation:

1. What does this change improve?
2. What can break?
3. How do I roll back?

This keeps the lab aligned with production-oriented DataOps and MLOps practices.
