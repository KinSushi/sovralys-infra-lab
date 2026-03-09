# Networking — Tailscale Mesh VPN + RDP Access

Three machines on different networks connected via Tailscale WireGuard mesh.

---

## Why Tailscale

The setup involves 3 machines on different networks:
- Local PC behind a home NAT (potentially CGNAT)
- OVH server with public IP but ports not to be exposed
- Windows VM with no public IP (QEMU NAT)

Tailscale (WireGuard-based) creates an encrypted peer-to-peer mesh between all 3. Each machine gets a stable 100.x.x.x IP. No router config, no exposed ports.

---

## Network Map

| Machine | Role | Public IP | Tailscale IP |
|---|---|---|---|
| Local PC (Windows 10) | Control / development | Dynamic (home network) | 100.G.H.I |
| Ubuntu Host (OVH) | Hypervisor / ML workloads | YOUR_SERVER_IP | 100.A.B.C |
| Windows VM | MT5 / trading / agents | 192.168.122.X (NAT) | 100.D.E.F |

---

## Installation

**Ubuntu Host:**
```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
# Opens auth link in terminal — open in browser, authenticate
```

**Windows (local PC and VM):**
1. Download from https://tailscale.com/download/windows
2. Install → sign in with the same account
3. Verify: `tailscale status` in PowerShell

**Verify connectivity:**
```bash
tailscale status
# Should show all 3 nodes as "online"
```

---

## RDP Access to the VM

**Architecture:**
```
[PC Local] --SSH tunnel--> [Ubuntu Host] --NAT--> [Windows VM]
localhost:3389           port forward           port 3389 RDP
```

**File `ovh-rdp.bat` (on local PC desktop):**
```batch
@echo off
start "" "C:\Windows\System32\OpenSSH\ssh.exe" -i YOUR_SSH_KEY_PATH -L 3389:192.168.122.X:3389 $USER@YOUR_SERVER_IP -N
timeout /t 5
mstsc /v:localhost:3389
```

- Line 1: opens SSH tunnel in background
- Line 2: waits 5 seconds for tunnel establishment
- Line 3: launches Remote Desktop connected to localhost:3389

**Manual tunnel if needed:**
```bash
ssh -i YOUR_SSH_KEY_PATH -L 3389:192.168.122.X:3389 $USER@YOUR_SERVER_IP -N
mstsc /v:localhost:3389
```

---

## noVNC (Browser Access)

Port 6080 is closed publicly (UFW deny). Always use SSH tunnel.

```bash
# From local PC
ssh -i YOUR_SSH_KEY_PATH -L 6080:localhost:6080 $USER@YOUR_SERVER_IP -N
# Then: http://localhost:6080/vnc.html
```

---

## Port Security

| Port | Status | Access method |
|---|---|---|
| 22 (SSH) | Open (UFW allow OpenSSH) | Direct SSH with key |
| 3389 (RDP) | Closed publicly | SSH tunnel only |
| 5900 (VNC) | Internal only | SSH tunnel only |
| 6080 (noVNC) | Closed (UFW deny) | SSH tunnel only |
| 8888 (Jupyter) | Closed publicly | SSH tunnel or direct SSH |
