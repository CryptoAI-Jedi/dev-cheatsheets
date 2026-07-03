# SSH & GPG Keys Cheatsheet

> Key lifecycle for both worlds: SSH keys (server auth, Git push) and GPG keys (encryption, signing, Git commit verification) — generate, deploy, protect, back up, rotate, revoke.

> Connection usage (tunnels, ProxyJump, config deep-dive, sshd hardening) lives in the [SSH Cheatsheet](ssh_cheatsheet.md). This sheet is about the **keys themselves**.

---

## Table of Contents
- [SSH vs GPG at a Glance](#ssh-vs-gpg-at-a-glance)
- [SSH Keys](#ssh-keys)
- [GPG Keys](#gpg-keys)
- [Git Commit Signing](#git-commit-signing)
- [Common Workflows](#common-workflows)
- [Tips & Gotchas](#tips--gotchas)

---

## SSH vs GPG AT A GLANCE

| Purpose                 | SSH Key            | GPG Key          |
| ----------------------- | ------------------ | ---------------- |
| Remote server login     | ✓ Primary use      | ✗                |
| Git commit signing      | ✓ (GitHub support) | ✓ Classic method |
| File encryption         | ✗                  | ✓ Primary use    |
| File signing            | Limited            | ✓ Primary use    |
| Key exchange/publishing | authorized_keys    | Keyservers       |
| Identity verification   | Fingerprint        | Web of Trust     |
| Passphrase protection   | Optional           | Recommended      |

---

## SSH KEYS

### Generate a key pair

```bash
ssh-keygen -t ed25519 -C "your@email.com"               # Recommended (modern)
ssh-keygen -t rsa -b 4096 -C "your@email.com"           # RSA fallback (legacy compat)
ssh-keygen -t ed25519 -f ~/.ssh/id_deploy -C "deploy"   # Custom filename

# Output: ~/.ssh/id_ed25519 (private) + ~/.ssh/id_ed25519.pub (public)
# NEVER share the private key. Only distribute .pub
```

### Permissions (CRITICAL — wrong perms = silent auth failure)

```bash
chmod 700 ~/.ssh                     # SSH directory
chmod 600 ~/.ssh/id_ed25519          # Private key (owner read/write only)
chmod 644 ~/.ssh/id_ed25519.pub      # Public key (readable)
chmod 600 ~/.ssh/authorized_keys     # Authorized keys file
chmod 600 ~/.ssh/config              # SSH config file
```

### Deploy public key to a server

```bash
ssh-copy-id user@host                              # Preferred method
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@host     # Specify key
# Manual (when ssh-copy-id is unavailable):
cat ~/.ssh/id_ed25519.pub | ssh user@host "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

### authorized_keys (server side — one public key per line)

```bash
echo "ssh-ed25519 AAAA... comment" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

```bash
# Restrict what a key can do (prefix options on its line):
command="/usr/bin/rsync" ssh-ed25519 AAAA...     # Only this command runs
from="192.168.1.0/24" ssh-ed25519 AAAA...        # Only from these IPs
restrict,command="/usr/bin/backup.sh" ssh-ed25519 AAAA...   # restrict = no forwarding/PTY/X11
```

### SSH agent (type your passphrase once, not per connection)

```bash
eval "$(ssh-agent -s)"              # Start agent
ssh-add ~/.ssh/id_ed25519           # Add key
ssh-add -l                          # List loaded keys
ssh-add -D                          # Remove all keys
ssh-add -t 3600 ~/.ssh/id_ed25519   # Add with 1h expiry
```

```bash
# Auto-start in ~/.bashrc or ~/.zshrc:
if [ -z "$SSH_AUTH_SOCK" ]; then
    eval "$(ssh-agent -s)"
    ssh-add ~/.ssh/id_ed25519
fi
```

### Key management

```bash
ls -la ~/.ssh/                            # List all keys
cat ~/.ssh/id_ed25519.pub                 # View public key
ssh-keygen -y -f ~/.ssh/id_ed25519        # Derive public from private
ssh-keygen -l -f ~/.ssh/id_ed25519        # Show fingerprint
ssh-keygen -lv -f ~/.ssh/id_ed25519.pub   # Visual fingerprint (randomart)
ssh-keygen -p -f ~/.ssh/id_ed25519        # Change passphrase
```

### known_hosts (server fingerprints)

```bash
ssh-keygen -R hostname     # Remove a bad/changed host entry
ssh-keygen -R 1.2.3.4      # Works with IPs too
```

### Keys in ~/.ssh/config

Full config reference is in the [SSH Cheatsheet](ssh_cheatsheet.md) — the key-relevant bits:

```bash
Host github
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_github

Host *
    AddKeysToAgent yes     # Auto-load keys into agent on first use
    IdentitiesOnly yes     # Only offer the configured key (fixes "Too many auth failures")
```

```bash
# Usage after config:
git clone git@github:CryptoAI-Jedi/repo.git
```

### Test authentication

```bash
ssh -v user@host           # Verbose auth debug (-vvv for full detail)
ssh -T git@github.com      # Test GitHub SSH auth
ssh -T git@gitlab.com      # Test GitLab SSH auth
```

---

## GPG KEYS

### Generate a GPG key

```bash
gpg --full-generate-key            # Interactive (recommended)
# gpg 2.2 (Ubuntu 22.04): choose "RSA and RSA", 4096 bits, 2y expiry (renewable)
# gpg 2.3+: default is ed25519 — just take the defaults

gpg --gen-key                      # Minimal prompts (name + email only)
gpg --expert --full-generate-key   # ed25519/cv25519 on gpg 2.2 ("ECC and ECC")
```

### List keys

```bash
gpg --list-keys                        # Public keys (shorthand: gpg -k)
gpg --list-secret-keys                 # Private keys (shorthand: gpg -K)
gpg --list-keys --keyid-format LONG    # Show full key ID
```

```text
Example output:
pub   ed25519 2026-01-01 [SC]
      ABCD1234EFGH5678IJKL9012MNOP3456QRST7890
uid   [ultimate] Alice <alice@example.com>
sub   cv25519 2026-01-01 [E]
```

### Export keys

```bash
# Public key (share this with others):
gpg --export --armor alice@example.com > alice_public.asc
gpg --export --armor KEYID > key.asc

# Private key (BACKUP — keep secure!):
gpg --export-secret-keys --armor alice@example.com > alice_private.asc

# Subkeys only:
gpg --export-secret-subkeys --armor KEYID > subkeys.asc
```

### Import keys

```bash
gpg --import alice_public.asc      # Import someone's public key
gpg --import alice_private.asc     # Restore from backup
gpg --keyserver keyserver.ubuntu.com --search-keys user@example.com
gpg --recv-keys KEYID              # Fetch from default keyserver
```

### Trust a key

```bash
gpg --edit-key alice@example.com
# > trust      → Set trust level
# > 5          → Ultimate (your own keys only)
# > 4          → Full
# > quit

gpg --sign-key alice@example.com   # Sign (certify) their key
```

### Encrypt & decrypt

```bash
# Encrypt for a recipient (they decrypt with their private key):
gpg --encrypt --recipient alice@example.com file.txt   # → file.txt.gpg
gpg -e -r alice@example.com file.txt                   # Shorthand

# Multiple recipients:
gpg -e -r alice@example.com -r bob@example.com file.txt

# ASCII-armored output (paste-friendly):
gpg --encrypt --armor -r alice@example.com file.txt    # → file.txt.asc

# Symmetric (password only, no keys):
gpg --symmetric file.txt                               # Shorthand: gpg -c

# Decrypt:
gpg --decrypt file.txt.gpg                # Auto-detects key, prints to stdout
gpg -d file.txt.gpg > output.txt          # Redirect to file
gpg --output output.txt --decrypt file.txt.gpg
```

### Sign & verify

```bash
gpg --detach-sign file.txt                 # → file.txt.sig (detached signature)
gpg --detach-sign --armor file.txt         # → file.txt.asc (ASCII)
gpg --sign --encrypt -r alice@example.com file.txt   # Sign + encrypt in one step
gpg --clearsign file.txt                   # Readable text + inline signature

# Verify:
gpg --verify file.txt.sig file.txt
gpg --verify file.txt.asc
```

### Delete keys

```bash
gpg --delete-key alice@example.com               # Public key
gpg --delete-secret-key alice@example.com        # Private key (asks first)
gpg --delete-secret-and-public-key KEYID         # Both
```

### Keyserver operations

```bash
gpg --keyserver keys.openpgp.org --send-keys KEYID        # Publish (modern default)
gpg --keyserver keyserver.ubuntu.com --send-keys KEYID    # Publish (Ubuntu pool)
gpg --keyserver keys.openpgp.org --recv-keys KEYID        # Fetch
gpg --refresh-keys                                        # Update all from server
```

### GPG agent (avoid re-entering passphrase)

```bash
gpgconf --launch gpg-agent            # Start agent
gpg-connect-agent reloadagent /bye    # Reload after config changes
```

```bash
# ~/.gnupg/gpg-agent.conf
default-cache-ttl 3600      # Cache passphrase 1h after last use
max-cache-ttl 86400         # Hard cap: re-prompt after 24h regardless
```

```bash
# Required in CLI/SSH environments — without it, pinentry fails silently:
echo 'export GPG_TTY=$(tty)' >> ~/.bashrc
```

### Backup, restore & revoke

```bash
# BACKUP (store offline / encrypted drive):
gpg --export --armor > all_public_keys.asc
gpg --export-secret-keys --armor > all_private_keys.asc
gpg --export-ownertrust > ownertrust.txt      # Cleaner than copying trustdb.gpg

# RESTORE:
gpg --import all_public_keys.asc
gpg --import all_private_keys.asc
gpg --import-ownertrust ownertrust.txt
```

```bash
# Revocation certificate — your kill switch if the key is compromised.
# gpg ≥2.1 auto-generates one at key creation:
ls ~/.gnupg/openpgp-revocs.d/
# Or generate manually (do this early, store offline):
gpg --gen-revoke KEYID > revoke.asc

# REVOKE (import the cert, then broadcast):
gpg --import revoke.asc
gpg --keyserver keys.openpgp.org --send-keys KEYID
```

---

## GIT COMMIT SIGNING

### Option A — GPG key (classic)

```bash
gpg --list-secret-keys --keyid-format LONG   # Find YOURKEYID (sec ed25519/YOURKEYID)

git config --global user.signingkey YOURKEYID
git config --global commit.gpgsign true
git config --global tag.gpgsign true

git commit -S -m "Signed commit"             # Sign one commit manually
git log --show-signature                     # Verify signatures in history
git verify-commit HEAD                       # Verify one commit
```

```bash
# Add to GitHub: copy this output →
gpg --export --armor YOURKEYID
# GitHub → Settings → SSH and GPG keys → New GPG key → paste
```

```text
⚠️ "gpg failed to sign the data" → missing GPG_TTY.
Fix in the GPG Agent section above.
```

### Option B — SSH key (simpler, GitHub-supported)

```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
# GitHub → Settings → SSH and GPG keys → New SIGNING key → paste .pub
```

```text
💡 The same SSH key can be added to GitHub twice — once as an
authentication key, once as a signing key. They are separate entries.
```

---

## COMMON WORKFLOWS

### New machine setup

```bash
mkdir -p ~/.ssh && chmod 700 ~/.ssh
ssh-keygen -t ed25519 -C "me@email.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub            # → add to GitHub + servers
ssh -T git@github.com                # Verify GitHub auth
gpg --full-generate-key              # Create GPG key
git config --global user.signingkey YOURKEYID
git config --global commit.gpgsign true
echo 'export GPG_TTY=$(tty)' >> ~/.bashrc
```

### Rotate SSH keys

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_new
ssh-copy-id -i ~/.ssh/id_ed25519_new.pub user@host
# Test the NEW key works, THEN remove the old line from authorized_keys
ssh-keygen -R oldhost                # Clean known_hosts if needed
```

### Secure file send

```bash
gpg --encrypt --armor -r recipient@email.com secret.txt
# Send secret.txt.asc over ANY channel — only the recipient can decrypt
```

---

## TIPS & GOTCHAS

- **SSH key silently ignored** — it's permissions 95% of the time. `~/.ssh` = 700, private keys/config/authorized_keys = 600. Debug with `ssh -vvv`.
- **"gpg failed to sign the data" in Git** — `GPG_TTY` isn't set. One-line fix in the GPG Agent section; the #1 GPG-in-terminal failure.
- **keys.openpgp.org strips identities** — it won't serve your name/email with the key until you verify the email with them. `--recv-keys` by email fails until then; fetch by KEYID.
- **Set a GPG expiry date** — an expired key is recoverable (extend it with `--edit-key` → `expire`); a compromised eternal key is not. 1–2 years, renewable.
- **The revocation cert IS the key's off switch** — anyone holding `revoke.asc` can kill your key. Store it offline, separate from backups of the key itself.
- **ed25519 vs RSA** — ed25519 everywhere unless a legacy system forces RSA; smaller, faster, no parameter-choice footguns.
- **One key per purpose beats one key everywhere** — separate keys for GitHub, prod servers, and deploy automation limit blast radius and make rotation painless (see `id_deploy` pattern above).

---
*Last Updated: 2026-07*
