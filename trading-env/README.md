# Trading Environment — MT5 24/7 + Distributed Agents

MetaTrader 5 running 24/7 in the Windows VM as MASTER node, with the local PC acting as distributed worker farm via Tailscale.

---

## Architecture

```
[Windows VM — MT5 MASTER]
  Local agents: 4-8 cores
        |
        | Tailscale VPN (100.G.H.I)
        |
[Local PC — MT5 WORKERS]
  Remote agents: additional cores
        |
        +--► Optimization speed x3 to x6
```

MetaTrader 5 running 24/7 in the Windows VM as MASTER node, with the local PC acting as distributed worker farm via Tailscale.

---

## Architecture

```
[Windows VM — MT5 MASTER]
  Local agents: 4-8 cores
        |
        | Tailscale VPN (100.G.H.I)
        |
[Local PC — MT5 WORKERS]
  Remote agents: additional cores
        |
        +--► Optimization speed x3 to x6
```

The VM always runs as MASTER (launches optimizations, stores results). The local PC acts as a worker farm via Tailscale.

---

## MT5 24/7 Configuration

**Autostart at Windows boot:**
```
shell:startup -> create shortcut to terminal64.exe
```

**Disable sleep (admin CMD in VM):**
```cmd
powercfg /change standby-timeout-ac 0
powercfg /change monitor-timeout-ac 0
powercfg /change hibernate-timeout-ac 0
reg add "HKCU\Control Panel\Desktop" /v ScreenSaveActive /t REG_SZ /d 0 /f
```

---

## Distributed Agents Configuration

**MT5 agent ports (VM):** MetaTester-N → 192.168.122.X:200N (ports 2000-2007 for 8 vCPUs)

**Open ports in Windows firewall (VM):**
```cmd
netsh advfirewall firewall add rule name="MT5 Agents" dir=in action=allow protocol=TCP localport=2000-2030
```

**Configure Agent Manager on local PC:**
```
C:\Program Files\MetaTrader 5\Tester\AgentManager.exe
```
For each agent: Status = Running, Allow remote connections = enabled, Password = set, Port = 2000, 2001...

**Add remote agents in MT5 (on VM Master):**
```
Tools → Options → Agents → Add → Remote Agent
  IP = 100.G.H.I (Tailscale IP of local PC)
  Port = 2000 (repeat for 2001, 2002...)
  Password = (set in Agent Manager)
```

---

## Performance Settings

| Setting | Recommendation |
|---|---|
| Number of agents | CPU threads − 1 (never use 100% of cores) |
| CPU priority | LOW or BELOW NORMAL (Agent Manager) |
| CPU Affinity | Enable — assign 1 core per agent (avoids context switching, up to 40% gain) |
| Tick cache | Check "Use Local Cache" in Strategy Tester (−30 to −70% test time) |
| Genetic algorithm | Always enable for initial optimization (×10 to ×100 combination reduction) |
| Data type | OHLC for exploration → Every Tick Real Ticks for final validation only |
| Disk I/O | SSD NVMe only, never network disk, Tester folder on local disk |
| CPU temperature | Monitor (HWMonitor on local PC) — thermal throttling = invisible losses |

---

## Professional Optimization Workflow

1. **Fast OHLC optimization** (genetic algorithm) — find parameter zones
2. **Filter best parameters**
3. **Validate with Every Tick Real Ticks**
4. **Massive Monte-Carlo** (500–2000 simulations)
5. **Forward test** (70% optimization / 30% validation)

Typical gain with this method: >80% reduction in total optimization time.

---

## Multi-EA Portfolio

Individual EAs don't win durably — intelligent diversification across EAs does.

**Portfolio structure targets:**
- 5 to 15 EAs running simultaneously
- Average inter-EA correlation < 0.4
- Portfolio Sharpe Ratio target > 1.5

**4 diversification axes:**

| Axis | Examples |
|---|---|
| Strategy | Trend following, mean reversion, breakout, carry |
| Timeframe | M1, M5, M15, H1, H4, D1 — never concentrate on a single TF |
| Market | Forex pairs, indices (NAS100, SPX500), commodities (XAUUSD, XAGUSD) |
| Entry logic | Price action, indicator-based, statistical, order flow |

> Correlation < 0.4 between EAs means drawdowns don't overlap. A portfolio of 10 low-correlation EAs with individual Sharpe 0.8 can produce a portfolio Sharpe > 1.5 through diversification alone.

---

## Scaling

| Setup | Agents | Acceleration |
|---|---|---|
| VM only | 4–8 | 1× |
| VM + local PC | 12–20 | ~3× |
| + 1 OVH dedicated | 30–40 | ~6× |
| Full cluster | 50–200 | ~15–30× |

**Cost hierarchy (optimal):** Local PC (~€2/core/month amortized) → dedicated bare-metal (~€5/core/month) → cloud VPS (~€11/core/month) → MQL5 Cloud Network (~€40/core/month, short bursts only)
