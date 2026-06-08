# Backup and Restore Test Procedure

Backups are useful only if restoration is tested. This document defines a public, sanitized procedure for validating backup integrity in the personal infrastructure lab.

No real backup paths, license values, hostnames or private data should be published here.

---

## Backup inventory template

| Backup ID | Source | Target | Type | Created at | Checksum | Restore tested |
|---|---|---|---|---|---|---|
| BKP-001 | `<source_placeholder>` | `<target_placeholder>` | VM / config / docs | YYYY-MM-DD | SHA256 | Yes / No |

---

## Backup creation checklist

Before creating a backup:

- [ ] identify what is being backed up;
- [ ] stop or quiesce services if required;
- [ ] confirm enough target storage;
- [ ] avoid storing secrets in public paths;
- [ ] document backup date and scope;
- [ ] compute checksum after creation.

Example checksum:

```bash
sha256sum <backup_file>
```

---

## Restore test checklist

A restore test should be performed on a non-critical path.

- [ ] select backup artifact;
- [ ] verify checksum;
- [ ] restore to isolated location;
- [ ] confirm expected files or VM metadata exist;
- [ ] start service or inspect restored data where safe;
- [ ] document result;
- [ ] delete test restore if not needed.

---

## VM backup restore pattern

Generic pattern only; do not publish real image names or host paths.

```bash
# Verify backup checksum
sha256sum <vm_backup_file>

# Copy to test location
cp <vm_backup_file> <test_restore_location>

# Inspect restored file
ls -lh <test_restore_location>

# Optional: inspect VM image metadata
qemu-img info <test_restore_location>
```

---

## Configuration backup pattern

```bash
# Example: archive sanitized configuration files
tar -czf config-backup-YYYYMMDD.tar.gz <config_directory>

# Verify archive
tar -tzf config-backup-YYYYMMDD.tar.gz | head

# Checksum
sha256sum config-backup-YYYYMMDD.tar.gz
```

---

## Restore result template

```markdown
# Restore Test: <backup_id>

## Date

YYYY-MM-DD

## Backup artifact

<sanitized_name>

## Scope

VM / config / docs / service data

## Checksum verified

Yes / No

## Restore location

<sanitized_test_location>

## Result

Success / Failed

## Evidence

Sanitized command output.

## Follow-up actions

- [ ] update backup procedure
- [ ] rotate old backup
- [ ] fix failed restore step
```

---

## Public-safety note

Do not publish:

- real backup file names if they reveal hostnames;
- real paths containing usernames or clients;
- license keys;
- VM images;
- database dumps;
- private logs;
- screenshots with identifiers.
