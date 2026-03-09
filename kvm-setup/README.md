# KVM Setup — Full Installation Guide

KVM/QEMU hypervisor on Ubuntu 24.04 hosting a Windows 10 Pro VM for MetaTrader 5 24/7 trading.

---

## Section 0 — Prepare the Windows 10 ISO (on local PC)

Before any server-side work, create the ISO on your local Windows PC.

Download `MediaCreationTool_22H2.exe` from Microsoft, then:
- Launch → Accept → "Create installation media (USB/DVD/ISO)"
- Language: English International, Edition: Windows 10, Architecture: 64-bit
- Choose "ISO file" → save as `Windows.iso`

Transfer to server (send to home directory first — permissions):

```bash
scp -i YOUR_SSH_KEY_PATH YOUR_LOCAL_PATH\Windows.iso $USER@YOUR_SERVER_IP:/home/$USER/
```

Then move from SSH:

```bash
sudo mv /home/$USER/Windows.iso /var/lib/libvirt/images/
```

---

## Section 1 — Install KVM/QEMU

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst virt-manager
sudo systemctl enable --now libvirtd
sudo systemctl status libvirtd
sudo usermod -aG libvirt,kvm $USER
newgrp libvirt
```

**Verify:** `sudo virsh list --all` should return without error.

---

## Section 2 — Prepare VM Disk and VirtIO Drivers

```bash
sudo mkdir -p /var/lib/libvirt/images
sudo qemu-img create -f qcow2 /var/lib/libvirt/images/windows10.qcow2 80G
sudo wget -O /var/lib/libvirt/images/virtio-win.iso \
  https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso
```

---

## Section 3 — Create the VM

> ⚠️ Use `bus=sata` for the initial installation. VirtIO requires loading drivers during install — SATA is recognized natively by Windows.

```bash
sudo virt-install \
  --name windows10 \
  --ram 16384 \
  --vcpus 4 \
  --os-variant win10 \
  --disk path=/var/lib/libvirt/images/windows10.qcow2,format=qcow2,bus=sata \
  --disk path=/var/lib/libvirt/images/virtio-win.iso,device=cdrom,bus=sata \
  --cdrom /var/lib/libvirt/images/Windows.iso \
  --network network=default,model=virtio \
  --graphics vnc,listen=0.0.0.0,port=5900 \
  --noautoconsole \
  --boot cdrom,hd
```

---

## Section 4 — Access the VM During Installation (noVNC)

```bash
sudo apt install -y novnc websockify
websockify --web=/usr/share/novnc/ 6080 localhost:5900 &
```

Access via SSH tunnel from local PC:

```bash
ssh -i YOUR_SSH_KEY_PATH -L 6080:localhost:6080 $USER@YOUR_SERVER_IP -N
# Then: http://localhost:6080/vnc.html
```

**During Windows installation:**
1. Choose "Custom installation"
2. "Load driver" → uncheck "Hide drivers incompatible with this computer's hardware"
3. Navigate to `E:\viostor\w10\amd64` → install VirtIO SCSI storage driver
4. Proceed with installation on the virtual disk

**After installation — network driver:**
1. File Explorer → CD Drive (E:) virtio-win → `NetKVM\w10\amd64`
2. Right-click `netkvm.inf` → Install
3. Restart the VM

---

## Section 5 — Post-Installation Configuration

**Enable RDP in the Windows VM:**
```
Right-click "This PC" → Properties → Remote settings → Allow remote connections
```

**Autostart VM:**
```bash
sudo virsh autostart windows10
sudo virsh dominfo windows10 | grep Autostart  # → Autostart: enable
```

**Block kernel updates (prevent surprise reboots):**
```bash
sudo apt-mark hold linux-image-generic linux-headers-generic
```

**Firewall:**
```bash
sudo ufw allow OpenSSH
sudo ufw enable
```

**noVNC as systemd service** (`/etc/systemd/system/novnc.service`):

```ini
[Unit]
Description=noVNC Service
After=network.target libvirtd.service

[Service]
Type=simple
User=$USER
ExecStart=/usr/bin/websockify --web=/usr/share/novnc/ 6080 localhost:5900
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable novnc
sudo systemctl start novnc
sudo ufw deny 6080  # Close public port — SSH tunnel access only
```

**Disable sleep in VM (Windows admin CMD):**
```cmd
powercfg /change standby-timeout-ac 0
powercfg /change monitor-timeout-ac 0
powercfg /change hibernate-timeout-ac 0
reg add "HKCU\Control Panel\Desktop" /v ScreenSaveActive /t REG_SZ /d 0 /f
```

**Windows 24/7 configuration:**
- Disable automatic updates: Settings → Windows Update → Advanced Options → disable "Automatically restart"
- MT5 autostart: paste `shell:startup` in Explorer → copy MT5 desktop shortcut → paste into Startup folder
- In MT5: Tools → Options → Expert Advisors → check "Allow automated trading"

---

## Final State

| Component | Status |
|---|---|
| KVM/libvirt | ✅ Active |
| Windows 10 Pro VM | ✅ Running |
| VirtIO drivers | ✅ Installed (disk + network) |
| VM Autostart | ✅ Enabled |
| noVNC service | ✅ Active (systemd) |
| UFW firewall | ✅ Active |
| Kernel updates | ✅ Held |
| Windows sleep | ✅ Disabled |
