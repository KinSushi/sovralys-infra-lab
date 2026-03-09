# VM Operations — virsh Cheat Sheet

Daily reference for managing the Windows 10 VM.

---

## VM State Management

```bash
sudo virsh list --all
sudo virsh start windows10
sudo virsh shutdown windows10    # graceful shutdown (preferred)
sudo virsh destroy windows10     # force stop (frozen VM only)
sudo virsh reboot windows10
```

---

## Resource Management

> ⚠️ RAM and CPU changes require the VM to be shut down first.

**RAM — increase to 20 GB:**
```bash
sudo virsh shutdown windows10
sudo virsh setmaxmem windows10 20971520 --config
sudo virsh setmem windows10 20971520 --config
sudo virsh start windows10
sudo virsh dominfo windows10 | grep memory
```

**CPU topology — 8 cores:**
```bash
sudo virsh shutdown windows10
sudo virsh dumpxml windows10 > /home/$USER/windows10.xml
sudo sed -i 's/<cpu mode="host-passthrough".*\/>/<cpu mode="host-passthrough" check="none" migratable="on"><topology sockets="1" cores="8" threads="1"\/><\/cpu>/' /home/$USER/windows10.xml
sudo virsh define /home/$USER/windows10.xml
sudo virsh start windows10
```

---

## Snapshots

```bash
sudo virsh snapshot-create-as windows10 "snapshot-name"
sudo virsh snapshot-list windows10
sudo virsh snapshot-revert windows10 "snapshot-name"
sudo virsh snapshot-delete windows10 "snapshot-name"

# Update snapshot (delete + recreate)
sudo virsh snapshot-delete windows10 "snapshot-initial"
sudo virsh snapshot-create-as windows10 "snapshot-initial"
```

> ⚠️ Snapshots live on the same disk as the VM. A disk failure takes them all. Always maintain an external .qcow2 copy. See `../backups/README.md`.

---

## Monitoring

```bash
sudo virt-top         # VM resources in real-time (quit: q)
htop                  # host resources
df -h                 # disk space
ls -lh /var/lib/libvirt/images/
```

---

## Service Management

```bash
sudo systemctl status novnc
sudo systemctl restart novnc
sudo systemctl status jupyter
sudo systemctl restart jupyter
```

---

## Quick Status Check After Reboot

Run this block after any server reboot to verify all systems:

```bash
sudo virsh list --all
sudo systemctl status novnc
sudo systemctl status jupyter
sudo virsh dominfo windows10 | grep Autostart
df -h
```

**Expected results:**
- VM: `running`
- Services: `active (running)`
- Autostart: `enable`
- Disk: < 80% usage
