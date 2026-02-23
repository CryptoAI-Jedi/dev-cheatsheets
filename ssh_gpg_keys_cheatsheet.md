# ssh_gpg_keys_cheatsheet

---

## SSH KEY BASICS

### Generate a Key Pair
ssh-keygen -t ed25519 -C "your@email.com"          → Recommended (modern)
ssh-keygen -t rsa -b 4096 -C "your@email.com"      → RSA fallback (legacy compat)
ssh-keygen -t ed25519 -f ~/.ssh/id_deploy -C "deploy key"  → Custom filename

# Output: ~/.ssh/id_ed25519 (private) + ~/.ssh/id_ed25519.pub (public)
# NEVER share the private key. Only distribute .pub

---

### Key Permissions (CRITICAL)
chmod 700 ~/.ssh                   → SSH directory
chmod 600 ~/.ssh/id_ed25519        → Private key (owner read/write only)
chmod 644 ~/.ssh/id_ed25519.pub    → Public key (readable)
chmod 600 ~/.ssh/authorized_keys   → Authorized keys file
chmod 600 ~/.ssh/config            → SSH config file

# Wrong permissions = SSH will refuse to use the key (silent failure)

---

### Copy Public Key to Server
ssh-copy-id user@host                              → Preferred method
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@host     → Specify key
cat ~/.ssh/id_ed25519.pub | ssh user@host "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

---

### SSH Config File (~/.ssh/config)
# Makes "ssh myserver" work instead of "ssh -i ~/.ssh/key -p 2222 user@1.2.3.4"

Host myserver
  HostName 1.2.3.4
  User ubuntu
  Port 2222
  IdentityFile ~/.ssh/id_ed25519

Host vps-prod
  HostName vps.example.com
  User admin
  IdentityFile ~/.ssh/id_prod
  ServerAliveInterval 60
  ServerAliveCountMax 3

Host github
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_github

Host *
  AddKeysToAgent yes
  IdentitiesOnly yes

# Usage after config:
ssh myserver
ssh vps-prod
git clone git@github:CryptoAI-Jedi/repo.git

---

### Connect & Test
ssh user@host                      → Basic connect
ssh -p 2222 user@host              → Custom port
ssh -i ~/.ssh/key.pem user@host    → Specific key
ssh -v user@host                   → Verbose (debug)
ssh -vvv user@host                 → Extra verbose (TLS/auth detail)
ssh -T git@github.com              → Test GitHub SSH auth
ssh -T git@gitlab.com              → Test GitLab SSH auth

---

### SSH Agent (avoid re-entering passphrase)
eval "$(ssh-agent -s)"             → Start agent
ssh-add ~/.ssh/id_ed25519          → Add key to agent
ssh-add -l                         → List loaded keys
ssh-add -D                         → Remove all keys from agent
ssh-add -t 3600 ~/.ssh/id_ed25519  → Add key with 1hr timeout

# Auto-start agent in ~/.bashrc or ~/.zshrc:
if [ -z "$SSH_AUTH_SOCK" ]; then
  eval "$(ssh-agent -s)"
  ssh-add ~/.ssh/id_ed25519
fi

---

### Key Management
ls -la ~/.ssh/                     → List all keys
cat ~/.ssh/id_ed25519.pub          → View public key
ssh-keygen -y -f ~/.ssh/id_ed25519 → Derive public from private
ssh-keygen -l -f ~/.ssh/id_ed25519 → Show key fingerprint
ssh-keygen -lv -f ~/.ssh/id_ed25519.pub  → Visual fingerprint (randomart)
ssh-keygen -p -f ~/.ssh/id_ed25519 → Change passphrase
ssh-keygen -R hostname             → Remove host from known_hosts

---

### authorized_keys
~/.ssh/authorized_keys             → One public key per line
# Add manually:
echo "ssh-ed25519 AAAA... comment" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# Restrict a key (in authorized_keys):
command="/usr/bin/rsync" ssh-ed25519 AAAA...   → Restrict to one command
from="192.168.1.0/24" ssh-ed25519 AAAA...     → Restrict by IP

---

### known_hosts
~/.ssh/known_hosts                 → Stores server fingerprints
# Clear a bad/changed host entry:
ssh-keygen -R hostname
ssh-keygen -R 1.2.3.4

---

### SSHD Server Config (/etc/ssh/sshd_config)
Port 22                            → Change port (security through obscurity)
PermitRootLogin no                 → Disable root login
PasswordAuthentication no          → Force key-only auth (IMPORTANT)
PubkeyAuthentication yes           → Enable key auth
AuthorizedKeysFile .ssh/authorized_keys
AllowUsers alice bob               → Whitelist users
MaxAuthTries 3
ClientAliveInterval 300            → Keepalive every 5min
ClientAliveCountMax 2

# After changes:
systemctl restart sshd
sshd -t                            → Test config before restart

---

### Port Forwarding / Tunneling
# Local forward: access remote service locally
ssh -L 8080:localhost:80 user@host        → Map local 8080 → remote 80
ssh -L 5432:db.internal:5432 user@bastion → Access remote DB locally

# Remote forward: expose local port on remote
ssh -R 9090:localhost:3000 user@host      → Remote sees local :3000 as :9090

# Dynamic SOCKS5 proxy
ssh -D 1080 user@host                     → Use as SOCKS5 proxy
# Then configure browser to use localhost:1080

# Persistent tunnel (with autossh)
autossh -M 0 -f -N -L 8080:localhost:80 user@host

---

### Jump Hosts / Bastion
ssh -J user@bastion user@target    → Jump through bastion
# Or in config:
Host target
  ProxyJump user@bastion.example.com

---

### SCP / SFTP
scp file.txt user@host:/path/       → Copy to remote
scp user@host:/path/file.txt .      → Copy from remote
scp -r ./dir user@host:/path/       → Recursive copy
scp -P 2222 file.txt user@host:/path/ → Custom port
sftp user@host                      → Interactive SFTP session
rsync -avz -e ssh ./dir user@host:/path/  → Efficient sync

---

## GPG KEY BASICS

### Generate a GPG Key
gpg --full-generate-key             → Interactive (recommended)
# Choose: RSA and RSA (or Ed25519), 4096 bits, no expiry or 2yr expiry

gpg --gen-key                       → Minimal prompts (name + email only)
gpg --expert --full-generate-key    → Expert mode (ed25519, cv25519)

---

### List Keys
gpg --list-keys                     → List public keys
gpg --list-secret-keys              → List private/secret keys
gpg --list-keys --keyid-format LONG → Show full key ID
gpg -K                              → Shorthand for list-secret-keys
gpg -k                              → Shorthand for list-keys

# Example output:
# pub   ed25519 2026-01-01 [SC]
#       ABCD1234EFGH5678IJKL9012MNOP3456QRST7890
# uid   [ultimate] Alice <alice@example.com>
# sub   cv25519 2026-01-01 [E]

---

### Export Keys
# Export public key (share this with others)
gpg --export --armor alice@example.com > alice_public.asc
gpg --export --armor KEYID > key.asc

# Export private key (BACKUP — keep secure!)
gpg --export-secret-keys --armor alice@example.com > alice_private.asc

# Export subkeys only
gpg --export-secret-subkeys --armor KEYID > subkeys.asc

---

### Import Keys
gpg --import alice_public.asc      → Import someone's public key
gpg --import alice_private.asc     → Restore from backup
gpg --keyserver keyserver.ubuntu.com --search-keys email@example.com
gpg --recv-keys KEYID               → Fetch from default keyserver

---

### Trust a Key
gpg --edit-key alice@example.com
> trust                             → Set trust level
> 5                                 → Ultimate (your own keys)
> 4                                 → Full
> quit

gpg --sign-key alice@example.com   → Sign (certify) their key

---

### Encrypt & Decrypt
# Encrypt for recipient (they decrypt with private key)
gpg --encrypt --recipient alice@example.com file.txt
gpg -e -r alice@example.com file.txt    → Shorthand
# Output: file.txt.gpg

# Encrypt for multiple recipients
gpg -e -r alice@example.com -r bob@example.com file.txt

# Encrypt + armor (ASCII output)
gpg --encrypt --armor -r alice@example.com file.txt
# Output: file.txt.asc

# Symmetric encryption (password only, no key)
gpg --symmetric file.txt
gpg -c file.txt                     → Shorthand
# Output: file.txt.gpg

# Decrypt
gpg --decrypt file.txt.gpg          → Auto-detect key
gpg -d file.txt.gpg > output.txt    → Output to file
gpg --output output.txt --decrypt file.txt.gpg

---

### Sign & Verify
# Sign a file (creates .sig detached signature)
gpg --detach-sign file.txt          → file.txt.sig
gpg --detach-sign --armor file.txt  → file.txt.asc (ASCII)

# Sign + encrypt in one step
gpg --sign --encrypt -r alice@example.com file.txt

# Clear-sign (readable text + signature)
gpg --clearsign file.txt            → file.txt.asc

# Verify signature
gpg --verify file.txt.sig file.txt
gpg --verify file.txt.asc

---

### Delete Keys
gpg --delete-key alice@example.com         → Delete public key
gpg --delete-secret-key alice@example.com  → Delete private key
gpg --delete-secret-and-public-key KEYID   → Delete both

---

### Key Server Operations
gpg --keyserver keyserver.ubuntu.com --send-keys KEYID   → Publish
gpg --keyserver keys.openpgp.org --recv-keys KEYID       → Fetch
gpg --refresh-keys                                        → Update all from server

---

### Git Commit Signing with GPG
# Get your key ID
gpg --list-secret-keys --keyid-format LONG
# Look for: sec  rsa4096/YOURKEYID

# Configure Git globally
git config --global user.signingkey YOURKEYID
git config --global commit.gpgsign true
git config --global tag.gpgsign true

# Sign a specific commit manually
git commit -S -m "Signed commit message"

# Verify a signed commit
git log --show-signature
git verify-commit HEAD

# Add GPG key to GitHub
gpg --export --armor YOURKEYID      → Copy output
# GitHub → Settings → SSH and GPG keys → New GPG key → Paste

# Troubleshoot "secret key not available"
export GPG_TTY=$(tty)               → Add to ~/.bashrc / ~/.zshrc
echo "export GPG_TTY=\$(tty)" >> ~/.bashrc

---

### Git Commit Signing with SSH Key (GitHub alternative to GPG)
# Configure Git to use SSH for signing
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true

# Add signing key to GitHub
# GitHub → Settings → SSH and GPG keys → New signing key → Paste .pub

---

### Backup & Restore GPG Keys
# BACKUP (store offline/encrypted drive)
gpg --export --armor > all_public_keys.asc
gpg --export-secret-keys --armor > all_private_keys.asc
# Also backup: ~/.gnupg/trustdb.gpg

# RESTORE
gpg --import all_public_keys.asc
gpg --import all_private_keys.asc

# Export revocation certificate (do this at key creation)
gpg --gen-revoke KEYID > revoke.asc
# Store safely — use to invalidate key if compromised

---

### Revoke a Key
gpg --import revoke.asc            → If pre-generated
# Or generate + import now:
gpg --gen-revoke KEYID | gpg --import
gpg --keyserver keyserver.ubuntu.com --send-keys KEYID  → Broadcast revocation

---

### GPG Agent (avoid re-entering passphrase)
gpgconf --launch gpg-agent        → Start GPG agent
gpg-connect-agent reloadagent /bye → Reload agent

# ~/.gnupg/gpg-agent.conf
default-cache-ttl 3600
max-cache-ttl 86400

echo "export GPG_TTY=\$(tty)" >> ~/.bashrc   → Fix pinentry issues

---

### Environment Variable
export GPG_TTY=$(tty)             → Required in many CLI environments
# Add to ~/.bashrc or ~/.zshrc permanently

---

## QUICK REFERENCE: SSH vs GPG

| Purpose                    | SSH Key              | GPG Key              |
|----------------------------|----------------------|----------------------|
| Remote server login        | ✓ Primary use        | ✗                    |
| Git commit signing         | ✓ (GitHub support)   | ✓ Classic method     |
| File encryption            | ✗                    | ✓ Primary use        |
| File signing               | Limited              | ✓ Primary use        |
| Key exchange/publishing    | authorized_keys      | Keyservers           |
| Identity verification      | Fingerprint          | Web of Trust         |
| Passphrase protection      | Optional             | Recommended          |

---

## COMMON WORKFLOWS

### New Machine Setup
mkdir -p ~/.ssh && chmod 700 ~/.ssh
ssh-keygen -t ed25519 -C "me@email.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub           → Add to GitHub/servers
ssh -T git@github.com               → Verify GitHub
gpg --full-generate-key             → Create GPG key
git config --global user.signingkey YOURKEYID
git config --global commit.gpgsign true
export GPG_TTY=$(tty)              → Add to ~/.bashrc

### Rotate SSH Keys
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_new
ssh-copy-id -i ~/.ssh/id_ed25519_new.pub user@host
# Test new key works, then remove old from authorized_keys
ssh-keygen -R oldhost               → Clean known_hosts if needed

### Secure File Send
gpg --encrypt --armor -r recipient@email.com secret.txt
# Send secret.txt.asc over any channel — only recipient can decrypt
