# Security Policy

This repository is a public documentation and infrastructure-learning repository. It must never contain real credentials, private addresses, license keys, production hostnames or operational secrets.

## Public documentation rules

| Rule | Required practice |
|---|---|
| No secrets | Never commit private keys, API tokens, passwords, `.env` files or credential exports |
| No real host exposure | Use placeholders such as `YOUR_SERVER_IP`, `YOUR_USER`, `TAILSCALE_HOSTNAME` |
| No license values | Keep license and activation material outside the repository |
| No raw backup images | Do not commit VM images, database dumps, archives or private logs |
| No open admin ports | Administrative access should be documented as tunnel-only or private-network-only |
| Sanitized screenshots | Blur hostnames, usernames, IPs, account IDs and machine identifiers |

## Files intentionally ignored

The `.gitignore` blocks common sensitive material: SSH keys, certificates, `.env`, token files, local backups, VM images, Terraform state, logs and private folders.

## Local pre-commit checklist

Before every public commit:

```bash
# Review changed files
git status --short
git diff --cached

# Search for common secret patterns
grep -RInE "(password|passwd|secret|token|api[_-]?key|private[_-]?key|BEGIN .*PRIVATE KEY)" . \
  --exclude-dir=.git \
  --exclude-dir=.venv \
  --exclude=SECURITY.md

# Search for accidental real IPv4 addresses, then manually review results
grep -RInE "([0-9]{1,3}\.){3}[0-9]{1,3}" . \
  --exclude-dir=.git \
  --exclude=SECURITY.md
```

## If sensitive data is committed

1. Remove the file immediately from the working tree.
2. Rotate the exposed secret or credential. Deleting a Git commit is not enough.
3. Rewrite history only if necessary and only after understanding the impact.
4. Re-check the repository with a secret scanner before continuing.

## Scope

This repository documents a personal infrastructure lab. It is not a production security baseline and does not represent any employer, client or financial institution.
