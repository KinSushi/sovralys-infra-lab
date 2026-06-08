# Roadmap — sovralys-infra-lab

This roadmap keeps the infrastructure lab aligned with the broader Data / MLOps / Swiss banking portfolio.

## Phase 1 — Stabilize public portfolio layer

| Task | Status | Outcome |
|---|---:|---|
| Reposition README around Data / ML infrastructure | Done | Recruiter-readable project purpose |
| Add `.gitignore` for secrets, backups and runtime files | Done | Safer public repository hygiene |
| Add `SECURITY.md` | Done | Clear public documentation rules |
| Add `PORTFOLIO.md` | Done | Interview-ready explanation of the project |
| Sanitize public docs | In progress | No real IPs, secrets, licenses or private identifiers |

## Phase 2 — Improve operational documentation

| Task | Priority | Outcome |
|---|---:|---|
| Add architecture decision records | High | Clear reasoning behind host/VM/network choices |
| Add recovery runbook | High | Repeatable steps after VM or service failure |
| Add service health checklist | High | Standardized daily/weekly checks |
| Add backup verification checklist | Medium | Backups are verified, not merely created |
| Add monitoring notes | Medium | Prepare future Prometheus/Grafana or lightweight checks |

## Phase 3 — Connect to DataOps/MLOps projects

| Task | Priority | Outcome |
|---|---:|---|
| Host `banking-dataops-monitoring` locally | High | PostgreSQL + Python + dashboard environment |
| Add Docker Compose patterns | High | Reusable container setup for data projects |
| Add Jupyter-to-production workflow note | Medium | Bridge notebooks, scripts and scheduled jobs |
| Add MLflow local tracking option | Medium | Prepare future MLOps portfolio repo |
| Add CI/CD notes | Medium | Document how local infra maps to GitHub Actions |

## Phase 4 — Bank-compatible hardening path

| Topic | Target standard |
|---|---|
| Secrets | No secrets in Git, rotation documented, local `.env` only |
| Access | No public admin ports; private network or SSH tunnel only |
| Logs | Operational logs retained locally, sanitized before publication |
| Backups | Backup, restore and verification procedures documented |
| Change management | Significant changes documented with ADRs and commit history |
| Incident management | Failures documented with root cause, impact and corrective action |

## Non-goals

This repository is not intended to become a production trading system, a cloud certification lab, or a complete enterprise platform. Its role is to prove hands-on operational understanding and to support the later DataOps/MLOps portfolio.
