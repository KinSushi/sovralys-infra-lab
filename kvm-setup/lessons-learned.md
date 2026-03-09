# Lessons Learned — Real Incidents & Post-Mortems

> Anyone can follow a tutorial. Debugging a production crash at 2am is what actually builds infra skills.

This is the most valuable file in this repository. Every incident here was real, happened in production, and cost time. Each one has a documented root cause and solution.

---

## Incident 1 — VM Performance Collapse (February 2026)

**What happened:** After a successful VM installation and MT5 setup, the full Jedha Bootcamp stack was installed inside the Windows VM:
- Docker Desktop (625 MB, installed without error)
- Anaconda (full distribution)
- VS Code with extensions
- Jupyter Notebook

The VM became unusable within hours. Symptoms: severe lag on all operations, RDP connection barely responsive, MT5 backtests stopped, Windows interface freezing.

**Root causes (3 cumulative):**

1. Docker Desktop requires Hyper-V or WSL2 — both require hardware virtualization features exposed to the guest OS. A KVM VM does not expose these by default (nested virtualization must be explicitly enabled on the host). Docker Desktop installed silently but launched virtualization layers in a loop consuming CPU.

2. Anaconda Navigator on autostart — launched at Windows boot, consuming ~800 MB RAM and intense disk I/O for update checks.

3. Resource saturation — VM configured with 16 GB RAM and 4 vCPUs. Docker + Anaconda + VS Code + MT5 competing = total saturation.

**What was tried (without success):**
- Close Docker Desktop via system tray icon → process persisted in background
- Disable Anaconda Navigator from startup → partial improvement, insufficient
- Increase RAM via `virsh setmem` → no effect, CPU was the bottleneck

**Solution — Restore from backup:**
```bash
sudo virsh destroy windows10
sudo cp /path/to/windows10_backup_YYYYMMDD.qcow2 \
        /var/lib/libvirt/images/windows10.qcow2
sudo virsh start windows10
```

**Architectural fix applied after restore:**

| Tool | Machine | Reason |
|---|---|---|
| Docker | Ubuntu host only | Native Linux, no nested virt issues |
| Anaconda + Jupyter | Ubuntu host only | Better performance, native Unix |
| VS Code | Windows VM only | MQL5 development |
| MT5 + backtesting | Windows VM only | Isolation, 24/7 stability |
| Git | Both machines | Sync and versioning |

This separation completely eliminated the conflicts. The VM has been stable since.

---

## Incident 2 — explorer.exe Disappeared — Black Desktop (February 20, 2026)

**What happened:** On RDP connection to the VM, the desktop was entirely black. The explorer.exe process had disappeared from the task list. MT5 continued running in the background (backtest in progress, not interrupted).

**Diagnosis:**
- `explorer.exe` absent → no graphical interface
- MT5 active → Windows kernel functional
- RDP session still active → Task Manager accessible

**Solution applied:**

Step 1 — Restart explorer.exe:
```
Ctrl + Shift + Esc → File → Run new task → explorer.exe → check "Create with administrator privileges" → OK
```

Step 2 — Disable sleep (admin CMD):
```cmd
powercfg /change standby-timeout-ac 0
powercfg /change monitor-timeout-ac 0
powercfg /change hibernate-timeout-ac 0
```

Step 3 — Verify:
```cmd
powercfg /query SCHEME_CURRENT SUB_SLEEP
```
Expected result: `0x00000000` for both "Sleep after" and "Hibernate after" (AC).

**Recommendations from this incident:**
- Install QEMU Guest Agent in the Windows VM to restart the desktop without going through Task Manager
- Verify after each Windows reboot that sleep settings are still 0 (they can reset depending on the active power plan)
- Regularly back up MT5 profiles (templates, experts, parameters)

---

## Incident 3 — Missing VirtIO Driver During Windows Installation

**What happened:** First VM creation attempt used `bus=virtio` for the primary disk. The Windows installer showed no disk — "No drives found."

**Root cause:** Windows does not contain VirtIO drivers natively. Without the driver loaded during installation, it literally cannot see the virtual disk.

**Fix — two-phase approach:**

Phase 1 — Install with SATA (natively recognized by Windows):
```bash
--disk path=windows10.qcow2,format=qcow2,bus=sata
```

Phase 2 — Load VirtIO driver during installation:
```
"Where to install Windows?" screen → "Load driver" → Uncheck "Hide drivers incompatible with this computer's hardware" → Navigate to E:\viostor\w10\amd64 → Install "Red Hat VirtIO SCSI controller"
```

---

## Incident 4 — Frozen SSH Terminal / Commands Without Response

**What happened:** During initial configuration, commands stopped returning output. `ls`, `df -h` — nothing.

**Root cause:** SSH session silently frozen — common during long operations (ISO transfer, `apt install`) on unstable connections.

**Fix:** Close the PowerShell window entirely → open a new window → reconnect.

**Prevention — use tmux for long operations:**
```bash
sudo apt install tmux
tmux new -s main
# Launch long operations inside
# Reconnect: tmux attach -t main
```

---

## Incident 5 — Docker Desktop Silently Failed to Install in VM

**What happened:** Docker Desktop installed successfully (625 MB, no errors), but `docker --version` returned "command not found."

**Root cause:** Docker Desktop requires Hyper-V or WSL2. In a KVM VM, neither is available by default. The installer completed but Docker could not start its engine.

**PowerShell note encountered (unrelated, but useful):**
```powershell
# This fails in Windows PowerShell (works in bash)
python --version && docker --version

# Correct PowerShell equivalent
python --version; docker --version
```

---

## Structural Lessons

1. **Never install Docker Desktop, Hyper-V or WSL2 in a KVM VM** without explicitly enabling nested virtualization on the host — and even then, stability is debatable.

2. **Backup before every major installation.** The 20 minutes to create a snapshot saved hours of reinstallation.

3. **Tool isolation by machine is an architectural decision, not a comfort choice.** Mixed environments create conflicts that are hard to debug.

4. **A snapshot is not a backup.** Snapshots live on the same disk. A disk failure takes both. Real backup = `.qcow2` copy on separate storage.

5. **Run `hostname` at the start of every command session in a multi-machine environment.** Cost: 0 seconds. Savings: hours of confusion.
