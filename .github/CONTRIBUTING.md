# Contributing

This is a personal documentation repository. Contributions are not expected but are welcome if you spot an error.

## How to Report an Issue

Open a GitHub Issue with:

1. **Document concerned** — file path (e.g., `kvm-setup/lessons-learned.md`)
2. **Error observed** — what is incorrect or outdated
3. **Proposed correction** — with source or reference

## Commit Convention

If submitting a PR, use [Conventional Commits](https://www.conventionalcommits.org/):

| Prefix | Use case |
|---|---|
| `docs(kvm):` | Changes to kvm-setup/ files |
| `docs(networking):` | Changes to networking/ files |
| `docs(trading):` | Changes to trading-env/ files |
| `fix(networking):` | Correction of a documented procedure |
| `feat(backups):` | New backup procedure |
| `chore:` | Maintenance (typos, formatting) |

## Scope

This repo documents a real production setup. All procedures are verified against actual sessions. Do not submit theoretical alternatives without testing — this is an operational reference, not a wiki.
