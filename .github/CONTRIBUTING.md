# Contributing

This is a personal infrastructure documentation repository. Contributions are not expected, but corrections are welcome if they improve accuracy, safety or operational clarity.

## Scope

This repository documents a real personal lab used to practice Linux, virtualization, private networking and Data/MLOps infrastructure foundations.

Accepted contributions:

- correction of documented commands;
- clarification of recovery procedures;
- improved operational checklists;
- security-hardening suggestions;
- typo and formatting fixes;
- additional architecture decision records if based on tested behavior.

Not accepted:

- theoretical alternatives that were not tested;
- public exposure of real hostnames, IPs, users, keys or licenses;
- trading-performance claims;
- production-security claims beyond the personal lab scope.

## Security rule before any PR

Before submitting a change, verify that the diff does not contain:

```text
.env files
API keys
SSH private keys
license keys
real public IP addresses
real private hostnames
credentials.json
token.pickle
raw backup archives
VM disk images
```

Use placeholders instead:

```text
YOUR_SERVER_IP
YOUR_USER
TAILSCALE_HOSTNAME
YOUR_SSH_KEY_PATH
```

## How to report an issue

Open a GitHub Issue with:

1. **Document concerned** — file path, for example `kvm-setup/lessons-learned.md`.
2. **Problem observed** — what is incorrect, outdated or unsafe.
3. **Proposed correction** — command, explanation or reference.
4. **Validation** — whether the correction was tested locally.

## Commit convention

Use Conventional Commits:

| Prefix | Use case |
|---|---|
| `docs(readme):` | README or profile-positioning changes |
| `docs(kvm):` | Changes to `kvm-setup/` files |
| `docs(networking):` | Changes to `networking/` files |
| `docs(docker):` | Changes to Docker isolation notes |
| `docs(runbook):` | Operations runbook and incident procedures |
| `docs(adr):` | Architecture decision records |
| `docs(security):` | Security and secret-hygiene documentation |
| `fix(networking):` | Correction of a documented network procedure |
| `feat(backups):` | New backup or recovery procedure |
| `chore:` | Maintenance, typos or formatting |

## Documentation style

Prefer:

- short operational steps;
- explicit commands;
- clear assumptions;
- tested behavior;
- rollback or recovery notes;
- sanitized examples.

Avoid:

- vague future claims;
- untested enterprise patterns;
- exposing real infrastructure details;
- turning the repo into a general-purpose wiki.

This repository is an operational reference and portfolio artifact. It should remain practical, sanitized and recruiter-readable.
