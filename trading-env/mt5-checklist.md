# MT5 Pre-Live Checklist

Professional checklist before going live with an EA. Every block must pass before moving to the next stage.

**Standard protocol:** Backtest → Forward → Monte-Carlo → Demo → Micro-lot → Gradual live

---

## Block 1 — Statistical Validation (mandatory)

- [ ] Backtest Every Tick Real Ticks, tick quality > 99%
- [ ] Forward test: 70% optimization / 30% validation — EA must remain profitable, otherwise immediate rejection
- [ ] Massive Monte-Carlo: minimum 500 simulations

---

## Block 2 — Strategic Validation

- [ ] Understand **why** the EA wins (edge identified and explained) — an unexplained EA is dangerous
- [ ] Multi-period test: different market cycles, crises, extreme volatility
- [ ] Multi-symbol test if applicable

---

## Block 3 — Risk Management

- [ ] Max drawdown < 20–30% of total capital
- [ ] Position sizing as % of capital (never arbitrary fixed lot)
- [ ] Kill-switch: EA must automatically stop if drawdown exceeds defined threshold

---

## Block 4 — Technical Validation

- [ ] 2–4 weeks test on demo account or real micro-lot
- [ ] Latency verification, average slippage, connection stability
- [ ] CPU, RAM, reboot, network interruption monitoring during this period

---

## Block 5 — Operational Security

- [ ] Automatic backups: EA, parameters, history, MT5 logs
- [ ] Continuous monitoring: email/Telegram alerts, detailed logs
- [ ] Documented recovery plan: server crash, EA bug, connection loss

---

## Block 6 — Psychological Validation

- [ ] Worst-case acceptance: trader must be ready to sustain the expected maximum drawdown, otherwise the strategy does not match their profile
- [ ] No manual intervention: EA must be able to operate without emotional decisions

---

## Professional Monte-Carlo Workflow

Monte-Carlo does not optimize. It tests statistical robustness under uncertain conditions.

> A backtest shows what happened. Monte-Carlo shows thousands of possible futures — it simulates real uncertainty.

**5 simulation types:**

1. **Tick randomization** — simulates micro-variations and execution latency
2. **Spread variation** — simulates real broker conditions and news (crucial for scalping)
3. **Random slippage** — simulates network latency and high volatility
4. **Trade order permutation** — tests timing dependency
5. **Statistical bootstrap** — estimates real maximum drawdown and future distribution

**Professional validation criteria:**
- ≥ 70% of simulations remain profitable
- Max drawdown < 2× historical drawdown
- Median profit factor > 1.2
- No excessive dependency on a single trade

**Real impact:** without Monte-Carlo, real EA failure rate ≈ 80%. With serious Monte-Carlo: < 20%.
