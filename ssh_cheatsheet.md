# SSH Cheatsheet

> Secure remote shell, tunnels, and file transfer. The transport layer under every VPS session, git push, and rsync. Master the config file and you type 80% less.

---

## Table of Contents
- [Basics](#basics)
- [~/.ssh/config (Client Config)](#sshconfig-client-config)
- [Keys](#keys)
- [SSH Agent](#ssh-agent)
- [ProxyJump (bastion / jump hosts)](#proxyjump-bastion--jump-hosts)
- [Port Forwarding (tunnels)](#port-forwarding-tunnels)
- [Keepalives & Reliability](#keepalives--reliability)
- [Multiplexing (connection reuse)](#multiplexing-connection-reuse)
- [File Transfer (scp / rsync / sftp)](#file-transfer-scp--rsync--sftp)
- [Escape Sequences](#escape-sequences)
- [Server-Side Hardening (sshd_config)](#server-side-hardening-sshd_config)
- [Tips & Gotchas](#tips--gotchas)

---

## BASICS

```bash
ssh user@host                     # Connect
ssh -p 2222 user@host             # Non-default port
ssh -i ~/.ssh/id_ed25519 user@host   # Explicit key
ssh user@host 'uptime'            # Run command, exit
ssh -t user@host 'htop'           # Force TTY (needed for interactive programs)
ssh -v user@host                  # Verbose (add -vv / -vvv to debug auth failures)
```

### tmux auto-attach on connect

```bash
ssh -t user@vps 'tmux attach -t main || tmux new -s main'
```

---

## ~/.ssh/config (CLIENT CONFIG)

Define hosts once, then `ssh vps1` instead of retyping user/port/key/jump every time.

```bash
# ~/.ssh/config  (chmod 600)

Host vps1
    HostName 203.0.113.10
    User jedi
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes            # Only offer THIS key (see Gotchas)

Host *.internal
    User ops
    ProxyJump bastion             # Wildcards apply to matching hosts

Host *
    ServerAliveInterval 60        # Global defaults go LAST
    ServerAliveCountMax 3

Include config.d/*                # Split large configs into files
```

```text
Tokens: %h = hostname, %r = remote user, %p = port
Precedence: FIRST matching value wins — put specific Host blocks
above wildcard/global blocks.
```

---

## KEYS

```bash
ssh-keygen -t ed25519 -C "jedi@arch-laptop"    # Generate (ed25519 = modern default)
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@host # Install key on server
ssh-keygen -lf ~/.ssh/id_ed25519.pub           # Show fingerprint
ssh-keygen -p -f ~/.ssh/id_ed25519             # Change/add passphrase
ssh-keygen -R hostname                         # Remove host from known_hosts (after server rebuild)
```

### Required permissions (auth silently fails otherwise)

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/config ~/.ssh/id_* ~/.ssh/authorized_keys
chmod 644 ~/.ssh/*.pub
```

---

## SSH AGENT

Holds decrypted keys in memory — type your passphrase once per boot, not per connection.

```bash
eval "$(ssh-agent -s)"            # Start agent (if not running)
ssh-add ~/.ssh/id_ed25519         # Add key
ssh-add -l                        # List loaded keys
ssh-add -D                        # Remove all keys
ssh-add -t 3600 ~/.ssh/id_ed25519 # Add with 1h expiry
```

### Agent forwarding

```bash
ssh -A user@host                  # Forward agent (or ForwardAgent yes in config)
# Then from host you can ssh/git-pull onward using your LOCAL keys.
```

```text
⚠️ Only forward the agent to hosts you trust — root on that host can
use (not read) your keys while you're connected. Prefer ProxyJump,
which never exposes the agent to the intermediate host.
```

---

## PROXYJUMP (bastion / jump hosts)

Reach hosts that are only accessible through an intermediate box. Traffic is end-to-end encrypted — the jump host can't read it.

```bash
ssh -J user@bastion user@target             # One jump
ssh -J u1@jump1,u2@jump2 user@target        # Chained jumps
scp -J user@bastion file.txt user@target:~/ # Works with scp too
```

### Config equivalent (the right way)

```bash
Host bastion
    HostName 198.51.100.5
    User jedi

Host target
    HostName 10.0.0.12            # Private IP, resolvable FROM the bastion
    User jedi
    ProxyJump bastion             # Now plain `ssh target` just works
```

### Legacy equivalent (pre-OpenSSH 7.3 servers)

```bash
Host target
    ProxyCommand ssh -W %h:%p bastion
```

```text
💡 On a Tailscale tailnet, ProxyJump is usually unnecessary — the mesh
gives you a direct encrypted path. Keep it for hosts outside the tailnet.
```

---

## PORT FORWARDING (tunnels)

```text
-L  Local forward   → a LOCAL port reaches a service on/behind the remote
-R  Remote forward  → a REMOTE port reaches a service on/behind your machine
-D  Dynamic (SOCKS) → route arbitrary traffic through the remote
```

### Local (-L): pull remote services to localhost

```bash
ssh -L 8080:localhost:80 user@vps        # http://localhost:8080 → vps port 80
ssh -L 5432:db.internal:5432 user@bastion  # Reach private DB through bastion
ssh -fN -L 3000:localhost:3000 user@vps  # -f background, -N no shell (pure tunnel)
```

### Remote (-R): expose your local service on the remote

```bash
ssh -R 9000:localhost:3000 user@vps      # vps:9000 → your local port 3000
```

```text
⚠️ Remote forwards bind to 127.0.0.1 on the server by default.
To expose publicly: set `GatewayPorts yes` in sshd_config (server side)
and bind explicitly: ssh -R 0.0.0.0:9000:localhost:3000 user@vps
```

### Dynamic (-D): SOCKS5 proxy

```bash
ssh -fN -D 1080 user@vps                 # Point browser/apps at socks5://localhost:1080
```

### Kill a background tunnel

```bash
pgrep -af "ssh -fN"                      # Find it
kill <pid>                               # Or use multiplexing: ssh -O exit host
```

---

## KEEPALIVES & RELIABILITY

Fixes silent session death behind NAT/firewalls ("broken pipe" after idling).

```bash
# ~/.ssh/config (client side)
Host *
    ServerAliveInterval 60        # Ping server every 60s
    ServerAliveCountMax 3         # Drop after 3 missed replies (3 min)
```

```bash
# /etc/ssh/sshd_config (server side equivalent)
ClientAliveInterval 60
ClientAliveCountMax 3
```

```text
💡 Keepalives keep the TCP session alive — tmux keeps your WORK alive
when the session dies anyway. Use both.
```

---

## MULTIPLEXING (connection reuse)

Reuse one authenticated connection for all subsequent ssh/scp/rsync/git to the same host — follow-up connections are near-instant.

```bash
# ~/.ssh/config
Host *
    ControlMaster auto
    ControlPath ~/.ssh/sockets/%r@%h-%p
    ControlPersist 10m            # Keep master alive 10 min after last use
```

```bash
mkdir -p ~/.ssh/sockets           # Socket dir must exist
ssh -O check host                 # Is a master connection running?
ssh -O exit host                  # Close the master (and its tunnels)
```

---

## FILE TRANSFER (scp / rsync / sftp)

```bash
scp file.txt user@host:/path/            # Upload
scp user@host:/path/file.txt .           # Download
scp -r dir/ user@host:/path/             # Recursive
scp -P 2222 file user@host:/path/        # Port is -P (capital) for scp
```

### rsync (resumable, incremental — prefer for anything big)

```bash
rsync -avz --progress src/ user@host:/dest/       # Archive, compress, show progress
rsync -avz -e "ssh -p 2222" src/ user@host:/dest/ # Non-default port
rsync -avz --delete src/ user@host:/dest/         # Mirror (deletes extras — careful)
rsync -avzn src/ user@host:/dest/                 # -n = dry run FIRST
```

```text
💡 Trailing slash matters in rsync: `src/` copies contents of src,
`src` copies the directory itself into dest.
```

---

## ESCAPE SEQUENCES

Typed at the START of a line inside a live session (press Enter first if unsure).

```text
~.    → Kill a frozen session (the lifesaver)
~C    → Open command line — add/remove forwards without reconnecting (e.g. -L 8080:localhost:80)
~#    → List forwarded connections
~~    → Send a literal ~
~?    → List all escape sequences
```

```text
⚠️ Nested sessions: each extra ~ targets one hop deeper — `~~.` kills
the inner session, `~.` kills the outer one.
```

---

## SERVER-SIDE HARDENING (sshd_config)

Quick hits for a fresh VPS — `/etc/ssh/sshd_config`:

```bash
PasswordAuthentication no         # Keys only (confirm key login works FIRST)
PermitRootLogin no                # Or prohibit-password at minimum
ClientAliveInterval 60
ClientAliveCountMax 3
```

```bash
sudo sshd -t                      # Validate config BEFORE restarting
sudo systemctl restart ssh        # Ubuntu/Debian (sshd on RHEL/Arch)
```

```text
⚠️ Keep your current session open while testing changes — verify a NEW
connection works before you disconnect, or you've locked yourself out.
```

---

## TIPS & GOTCHAS

- **"Too many authentication failures"** — the agent is offering every loaded key; the server drops you before the right one. Fix: `IdentitiesOnly yes` + explicit `IdentityFile` in the host block.
- **Auth works with `-i` but not from config** — permissions. `~/.ssh` must be 700, keys and config 600 (server side: `authorized_keys` 600, home dir not group-writable).
- **Host key changed after a server rebuild** — don't blindly delete `known_hosts`; remove one entry: `ssh-keygen -R hostname`.
- **First-connect prompts in scripts** — `-o StrictHostKeyChecking=accept-new` accepts new hosts but still fails on *changed* keys (safer than `no`).
- **Tunnels die with the shell** — use `-fN` for pure tunnels, or multiplexing + `ControlPersist` so tunnels outlive individual sessions.
- **`ssh host` hangs at connect** — try `ssh -vvv`; commonly DNS on the server (`UseDNS no` server-side) or a dead ControlMaster socket (delete it from `~/.ssh/sockets/`).
- **scp vs rsync** — scp for one-off small files; rsync for anything big, repeated, or resumable.
- **Agent forwarding vs ProxyJump** — default to ProxyJump for reaching internal hosts; forward the agent only when you must run git/ssh *on* the remote box.

---
*Last Updated: 2026-07*
