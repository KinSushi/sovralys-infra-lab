# SSHFS + Samba — Mount Local PC Drive in Windows VM

Mount `E:\` from the local PC into the VM as a network drive, without physical data copy.

---

## Problem

MT5 runs in the Windows VM, but the XAUUSD tick dataset (23 years, ~18 GB CSV+JSON) lives on the external drive `E:\` of the local PC. The VM's virtual disk has only ~20 GB free — direct copy is impossible.

## Solution

Mount `E:\` from local PC into Ubuntu host via SSHFS over Tailscale, then reshare to the Windows VM via Samba (SMB). MT5 accesses the data via mapped network drive (Z:\\), with no physical copy.

---

## Architecture

```
[Local PC — Windows 10]
  E:\ (1 TB external, tick data XAUUSD ~18 GB)
  Tailscale: 100.G.H.I
  OpenSSH Server: port 22
        |
        | SSHFS over Tailscale
        |
[Ubuntu Host — OVH]
  /mnt/pc_local  <- E:\ mounted here
  Samba (SMB) gateway: 192.168.122.1
        |
        | SMB (internal QEMU network only)
        | 192.168.122.0/24
        |
[Windows VM]
  Z:\ <- \\\\192.168.122.1\\pc_local
  MT5 reads data from Z:\
```

---

## Step 1 — OpenSSH Server on Local PC (Windows)

```powershell
# Admin PowerShell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
```

**Critical note on `sshd_config`** (`C:\ProgramData\ssh\sshd_config`): for Windows administrator accounts, the public key must be in a specific file:

```
C:\Users\$WIN_USER\.ssh\authorized_keys              <- user file
C:\ProgramData\ssh\administrators_authorized_keys   <- this file is actually read for admins
```

The bottom of `sshd_config` contains (do not modify):
```
Match Group administrators
       AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys
```

---

## Step 2 — SSH Key: Ubuntu → Local PC

The key must be generated for root since SSHFS runs with sudo:

```bash
sudo ssh-keygen -t ed25519 -C "ubuntu-host-to-pc-local" -f /root/.ssh/id_ed25519_pc
# Enter x2 (no passphrase — required for fstab automatic mount)
sudo cat /root/.ssh/id_ed25519_pc.pub  # Copy this public key
```

Deploy key to local PC (from Ubuntu via Tailscale):
```bash
# User file
ssh $WIN_USER@100.G.H.I "mkdir -p C:/Users/$WIN_USER/.ssh && echo 'PASTE_PUBKEY' >> C:/Users/$WIN_USER/.ssh/authorized_keys"

# Administrator file (the one actually used)
ssh $WIN_USER@100.G.H.I "echo 'PASTE_PUBKEY' >> C:/ProgramData/ssh/administrators_authorized_keys"
```

Fix permissions on Windows (required — OpenSSH rejects files with wrong permissions):
```cmd
icacls C:\ProgramData\ssh\administrators_authorized_keys /inheritance:r /grant "SYSTEM:(F)" /grant "Administrators:(F)"
```

**Test:**
```bash
sudo ssh -i /root/.ssh/id_ed25519_pc $WIN_USER@100.G.H.I
# Must connect without password
```

---

## Step 3 — SSHFS Mount (Ubuntu ← E:\ from local PC)

```bash
sudo apt install sshfs -y
sudo mkdir -p /mnt/pc_local

# Under Windows OpenSSH, E:\ is written as /e: (not /e/ as in Linux)
sudo sshfs -o IdentityFile=/root/.ssh/id_ed25519_pc \
  $WIN_USER@100.G.H.I:/e: /mnt/pc_local \
  -o allow_other,reconnect,ServerAliveInterval=15,ServerAliveCountMax=3

ls /mnt/pc_local  # Verify
```

**Persistent mount via `/etc/fstab`:**
```
$WIN_USER@100.G.H.I:/e: /mnt/pc_local fuse.sshfs IdentityFile=/root/.ssh/id_ed25519_pc,allow_other,reconnect,ServerAliveInterval=15,ServerAliveCountMax=3,_netdev 0 0
```
(`_netdev` → systemd waits for network before mounting)

---

## Step 4 — Samba Share (Ubuntu → Windows VM)

```bash
sudo apt install samba -y
```

Add to end of `/etc/samba/smb.conf`:
```ini
[pc_local]
   path = /mnt/pc_local
   browseable = yes
   read only = yes
   guest ok = yes
   force user = $USER
```

```bash
sudo systemctl restart smbd nmbd
sudo systemctl enable smbd nmbd
testparm -s 2>/dev/null  # Validate config
```

---

## Step 5 — Open Samba Ports in UFW (Internal Only)

```bash
# Internal QEMU subnet only — never expose Samba to the Internet
sudo ufw allow from 192.168.122.0/24 to any port 445
sudo ufw allow from 192.168.122.0/24 to any port 139
sudo ufw reload
```

---

## Step 6 — Map as Network Drive in Windows VM

**One-time access** (Win + R): `\\\\192.168.122.1\\pc_local`

**Persistent drive for MT5:**
1. File Explorer in VM
2. Right-click "This PC" → "Map network drive"
3. Drive: Z: | Folder: `\\\\192.168.122.1\\pc_local` | Check "Reconnect at sign-in"

---

## Troubleshooting

```bash
mount | grep pc_local         # SSHFS active?
sudo systemctl status smbd    # Samba running?
ls /mnt/pc_local              # Data accessible?
sudo ufw status | grep 445    # Port open for 192.168.122.0/24?
testparm -s 2>/dev/null       # smb.conf syntax valid?
```

> **Note:** Under Windows OpenSSH, drive letters are written `/e:`, not `/e/` or `/E/`.

---

## Reference

| Component | Address |
|---|---|
| Local PC (Tailscale) | 100.G.H.I |
| Ubuntu Host (Tailscale) | 100.A.B.C |
| Ubuntu Host (QEMU gateway) | 192.168.122.1 |
| Windows VM | 192.168.122.X |
| Samba path (from VM) | \\\\192.168.122.1\\pc_local |
| Mapped drive (MT5) | Z:\ |
