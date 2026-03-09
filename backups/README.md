# Backups — 3-Layer Strategy

> A snapshot is not a backup. A backup on the same disk is not a backup.

This repo implements 3 independent protection layers.

---

## Architecture

```
Layer 1 — OVH automatic snapshots
  Protects against: instance corruption, accidental deletion
  Does NOT protect against: disk failure, billing issue

Layer 2 — virsh snapshots (on disk, local)
  Protects against: bad manipulation, software crash
  Does NOT protect against: disk failure

Layer 3 — External .qcow2 copy (local PC / external disk)
  Protects against: everything above
  This is the real backup
```

---

## Layer 1 — OVH Snapshot (Web Interface)

```
OVH Manager → Public Cloud → Compute → Instances → Click ⋮ → "Create a snapshot"
Name: ubuntu-host-windows10-vm-YYYYMMDD
```

> ⚠️ Stop active MT5 backtests first — the instance may be temporarily frozen during capture.

**Restore:**
```
OVH Manager → Compute → Instances → Create instance → Image → "Snapshots" → select snapshot
```

Cost: ~€0.01/GB/month. Delete old snapshots to control costs.

---

## Layer 2 — virsh Snapshots (Internal)

```bash
# Before any major operation
sudo virsh snapshot-create-as windows10 "before-DESCRIPTION"
sudo virsh snapshot-list windows10
sudo virsh snapshot-revert windows10 "before-DESCRIPTION"
sudo virsh snapshot-delete windows10 "before-DESCRIPTION"
```

**Rule:** always create a snapshot before installing new software in the VM or modifying configuration.

---

## Layer 3 — External .qcow2 Copy (Real Backup)

**Compress then SCP (recommended method):**

```bash
sudo virsh shutdown windows10
# Wait for shutdown
watch -n2 'virsh list --all'   # Ctrl+C when: shut off

# Compress (saves 30-50% space)
qemu-img convert -O qcow2 -c \
  /var/lib/libvirt/images/windows10.qcow2 \
  /home/$USER/windows10_backup_$(date +%Y%m%d).qcow2

# Export VM config XML (virtual hardware profile)
sudo virsh dumpxml windows10 > /home/$USER/windows10_$(date +%Y%m%d).xml
```

```powershell
# From local PC — download backup
scp -i YOUR_SSH_KEY_PATH \
   $USER@YOUR_SERVER_IP:/home/$USER/windows10_backup_*.qcow2 \
   C:\Backups\

scp -i YOUR_SSH_KEY_PATH \
   $USER@YOUR_SERVER_IP:/home/$USER/windows10_*.xml \
   C:\Backups\
```

Transfer time: 20–40 minutes depending on disk size.

---

## Restore from External Backup

```bash
# Copy backup to server from local PC
scp -i YOUR_SSH_KEY_PATH \
   C:\Backups\windows10_backup_YYYYMMDD.qcow2 \
   $USER@YOUR_SERVER_IP:/var/lib/libvirt/images/windows10.qcow2

sudo virsh define /path/to/windows10_YYYYMMDD.xml
sudo virsh start windows10
```

The saved XML preserves the virtual hardware profile (UUID, MAC address). Windows license reactivates automatically because it sees the same "hardware."

---

## Host Configuration Backup

```bash
sudo virsh dumpxml windows10 > /home/$USER/windows10.xml
sudo cp /etc/systemd/system/novnc.service /home/$USER/
sudo cp /etc/systemd/system/jupyter.service /home/$USER/

tar -czf /home/$USER/stack_config_$(date +%Y%m%d).tar.gz \
  /home/$USER/windows10.xml \
  /home/$USER/novnc.service \
  /home/$USER/jupyter.service
```

---

## Backup Calendar

| Layer | Frequency | Trigger |
|---|---|---|
| OVH snapshot | Monthly | Manual |
| virsh snapshot | Before each major change | Manual |
| External .qcow2 copy | Monthly | Manual (SCP overnight) |
| XML + services config | After each infra change | Manual |

---

## What a .qcow2 Backup Contains

- ✅ Windows 10 Pro (activated)
- ✅ MetaTrader 5 + all EAs
- ✅ Git, VS Code, Anaconda
- ✅ Office 365 (activation tied to VM profile)
- ✅ All data in `C:\Users\$WIN_USER\`
- ✅ All settings and shortcuts
