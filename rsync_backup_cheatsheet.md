# rsync & Backup Patterns Cheatsheet

> Delta-transfer file sync over SSH, plus the backup patterns built on it: mirrors, hardlink snapshots, pull-based VPS backups, and when to graduate to restic. Rule zero: a backup you haven't restored from is a hypothesis.

---

## Table of Contents
- [Mental Model](#mental-model)
- [Core Usage](#core-usage)
- [Remote Sync over SSH](#remote-sync-over-ssh)
- [Include / Exclude Filters](#include--exclude-filters)
- [Mirroring & Delete Safety](#mirroring--delete-safety)
- [Incremental Snapshots (--link-dest)](#incremental-snapshots---link-dest)
- [Big Transfers & Resume](#big-transfers--resume)
- [Verify & Audit](#verify--audit)
- [Backup Patterns](#backup-patterns)
- [Beyond rsync (restic / rclone)](#beyond-rsync-restic--rclone)
- [Tips & Gotchas](#tips--gotchas)

---

## MENTAL MODEL

```text
rsync copies only what CHANGED (delta transfer) — safe to run repeatedly.
Local:  rsync -a src/ dst/
Push:   rsync -a src/ user@host:/dst/
Pull:   rsync -a user@host:/src/ dst/
```

### The trailing slash (the #1 rsync mistake)

```text
rsync -a src/ dst/   → Copies the CONTENTS of src into dst
rsync -a src  dst/   → Copies the DIRECTORY src into dst (→ dst/src/)
```

```text
💡 Read `src/` as "what's inside src". Destination slash doesn't matter.
```

---

## CORE USAGE

```text
rsync -a src/ dst/              → Archive mode: recursive + perms, times, owners, symlinks
rsync -av src/ dst/             → + verbose (list files as they move)
rsync -avz src/ dst/            → + compress in transit (remote links)
rsync -avP src/ dst/            → + progress bars AND keep partial files (resume)
rsync -avn src/ dst/            → DRY RUN — shows what WOULD happen
rsync -avh src/ dst/            → Human-readable sizes
rsync -a --stats src/ dst/      → Transfer summary (files, bytes, speedup)
```

```text
⚠️ -a = -rlptgoD. It does NOT include -H (hardlinks), -A (ACLs),
-X (xattrs). Preserving OWNERSHIP requires running the receiving
side as root — see Gotchas.
```

---

## REMOTE SYNC OVER SSH

Your `~/.ssh/config` aliases, keys, and ProxyJump all apply automatically (see [SSH Cheatsheet](ssh_cheatsheet.md)).

```bash
rsync -avz ./project/ vps1:/opt/project/        # Push (vps1 = ssh config alias)
rsync -avz vps1:/opt/agent/data/ ./data/        # Pull
rsync -avz -e "ssh -p 2222" src/ user@host:/dst/    # Non-default port
rsync -avz -e "ssh -J bastion" src/ user@target:/dst/   # Through a jump host
```

### Read/write root-owned files on the remote

```bash
rsync -avz --rsync-path="sudo rsync" vps1:/etc/nginx/ ./nginx-backup/
# Requires passwordless sudo for rsync on the remote (visudo):
#   deploy ALL=(ALL) NOPASSWD: /usr/bin/rsync
```

---

## INCLUDE / EXCLUDE FILTERS

```bash
rsync -av --exclude '.git/' --exclude 'node_modules/' src/ dst/
rsync -av --exclude '*.pyc' --exclude '__pycache__/' --exclude '.venv/' src/ dst/
rsync -av --exclude-from '.rsync-exclude' src/ dst/     # Patterns from file
rsync -av --filter=':- .gitignore' src/ dst/            # Respect .gitignore rules
```

### Copy ONLY certain files, keeping the tree (include-then-exclude)

```bash
# Only markdown files, preserving directory structure:
rsync -av --include='*/' --include='*.md' --exclude='*' vault/ vault-md-only/
# Order matters: dirs allowed in, .md allowed in, everything else excluded
```

---

## MIRRORING & DELETE SAFETY

```bash
rsync -av --delete src/ dst/            # TRUE MIRROR — deletes dst files not in src
rsync -avn --delete src/ dst/           # ALWAYS dry-run a --delete first
rsync -av --delete --delete-excluded src/ dst/   # Also delete excluded files from dst
rsync -av --delete --max-delete=50 src/ dst/     # Abort if >50 deletions (fat-finger fuse)
```

### Trash instead of destroy

```bash
rsync -av --delete \
  --backup --backup-dir="/backups/trash-$(date +%F)" \
  src/ dst/
# Deleted/overwritten files land in the dated trash dir instead of vanishing
```

---

## INCREMENTAL SNAPSHOTS (--link-dest)

Time-machine-style: each snapshot looks complete, but unchanged files are hardlinks to the previous one — N snapshots cost ~1 full copy + changes.

```bash
#!/bin/bash
SRC="vps1:/opt/agent/data/"
DEST="/backups/agent"
DATE=$(date +%Y-%m-%d_%H%M)

rsync -a --delete --link-dest="$DEST/latest" "$SRC" "$DEST/$DATE/"
ln -sfn "$DEST/$DATE" "$DEST/latest"

# Prune snapshots older than 30 days:
find "$DEST" -maxdepth 1 -type d -mtime +30 -exec rm -rf {} +
```

```text
💡 First run: `latest` doesn't exist yet — rsync warns and does a full
copy. That's the seed; every run after is incremental.
💡 Restore = plain copy from any dated dir. No tooling needed.
💡 Deleting any snapshot is safe — hardlinks keep shared files alive
until the LAST reference goes.
```

---

## BIG TRANSFERS & RESUME

```bash
rsync -avP src/ dst/                          # --partial: interrupted files resume
rsync -avP --partial-dir=.rsync-partial src/ dst/   # Keep partials out of sight
rsync -av --append-verify big.iso vps1:/data/ # Resume a single huge file in place
rsync -avz --bwlimit=5m src/ vps1:/dst/       # Cap at 5 MB/s (don't starve the link)
rsync -av --timeout=60 vps1:/src/ dst/        # Give up on a dead connection
```

```text
💡 Skip -z for already-compressed data (video, archives, images) —
it burns CPU for ~0% gain. rsync auto-skips common suffixes, but for
media trees just drop the flag.
```

---

## VERIFY & AUDIT

```bash
rsync -avni src/ dst/           # Itemized dry run — per-file change codes
rsync -avnc src/ dst/           # Checksum compare (slow, thorough) — silence = identical
rsync -ac src/ dst/             # Transfer using checksums, not size+mtime
```

```text
Itemize codes (first chars): > transfer  c change/create  . attribute-only
                             + new  * deletion (with --delete)
```

---

## BACKUP PATTERNS

### Pull, don't push (VPS → local)

```text
The backup machine PULLS from the VPS — the VPS holds no credentials to
reach your backups. A compromised server can't encrypt/delete what it
can't log into. (Pairs with the VPS Hardening sheet.)
```

```bash
# On the local/backup machine, scheduled nightly:
rsync -az --delete --link-dest=/backups/vps1/latest \
  vps1:/opt/ /backups/vps1/$(date +%F)/
```

### Schedule with a systemd timer (see systemd sheet for full anatomy)

```bash
# backup.service → Type=oneshot, ExecStart=/usr/local/bin/backup.sh
# backup.timer   → OnCalendar=*-*-* 03:00:00, Persistent=true
systemctl list-timers | grep backup
journalctl -u backup.service --since yesterday   # Backups log like everything else
```

### Docker volume backup

```bash
# Cold path (container stopped) — rsync the volume's host dir:
sudo rsync -a /var/lib/docker/volumes/agent_data/_data/ /backups/agent_data/
# Hot path — the tar-through-a-container method (see Docker sheet)
```

### Vault snapshot before risky operations

```bash
rsync -a --delete --link-dest=~/snapshots/vault/latest \
  ~/SecondBrain/ ~/snapshots/vault/$(date +%F_%H%M)/
ln -sfn ~/snapshots/vault/$(date +%F_%H%M) ~/snapshots/vault/latest
# Syncthing propagates mistakes to every device with equal enthusiasm —
# snapshots are the undo button it doesn't have.
```

### The restore drill (do this quarterly)

```bash
rsync -av /backups/agent/latest/ /tmp/restore-test/
diff -r /tmp/restore-test/ /opt/agent/data/ | head    # Spot-check
# Can the service actually START from restored data? That's the real test.
```

---

## BEYOND RSYNC (restic / rclone)

### restic — encrypted, deduplicated, snapshot-native

When backups leave machines you control (cloud, offsite), rsync's plaintext mirroring stops being enough:

```bash
restic init --repo /backups/restic                # Create repo (sets password)
restic -r /backups/restic backup ~/SecondBrain    # Snapshot
restic -r /backups/restic snapshots               # List
restic -r /backups/restic restore latest --target /tmp/restore
restic -r /backups/restic forget --keep-daily 7 --keep-weekly 4 --prune
restic -r /backups/restic check                   # Verify repo integrity
```

### rclone — rsync for cloud storage

```bash
rclone config                                     # Interactive remote setup
rclone sync ~/vault remote:vault --dry-run        # ALWAYS dry-run sync first
rclone sync ~/vault remote:vault -P
rclone ls remote:vault
```

```text
3-2-1 rule: 3 copies, 2 different media, 1 offsite.
Typical stack: live data → link-dest snapshots (local, fast restore)
→ restic to offsite/cloud (encrypted, disaster case).
```

---

## TIPS & GOTCHAS

- **Trailing slash, again** — `src/` = contents, `src` = the directory itself. When in doubt, `-n` dry-run and read the paths it prints.
- **`--delete` without `-n` first is how directories die** — the flag does exactly what it says to the DESTINATION. Dry-run, read, then run. `--max-delete` is the cheap insurance.
- **Ownership silently changes on non-root pulls** — files land owned by the local user. For faithful system backups, run the pull as root or accept uid drift; for app data it usually doesn't matter.
- **Don't rsync a live database** — you'll copy a file mid-write and back up corruption. Dump first: `sqlite3 app.db ".backup /backups/app.db"` or `pg_dump`, then rsync the dump.
- **Timers/cron need absolute paths** — no interactive PATH, no ssh-agent. Use full binary paths and a dedicated key (`-e "ssh -i /home/user/.ssh/id_backup"`), restricted on the server via `authorized_keys` options (see Keys sheet).
- **`-z` over a fast LAN can be SLOWER** — compression becomes the bottleneck. Reserve it for internet links.
- **Interrupted transfers leave debris** — without `--partial`, progress is discarded; with it, partial files sit in the tree. `--partial-dir=.rsync-partial` gives resume AND cleanliness.
- **rsync ≠ versioning** — a mirror faithfully replicates yesterday's ransomware and today's fat-finger. Only `--link-dest` snapshots or restic give you a *past* to return to.
- **Test restores, not backups** — green backup logs prove writing works. Only a restore proves the thing you'll actually need works.

---
*Last Updated: 2026-07*
