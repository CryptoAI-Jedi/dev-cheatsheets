# Tailscale Cheatsheet

> WireGuard mesh VPN: every device gets a stable 100.x address and a name, NAT is somebody else's problem, and "internal service" finally means something. The backbone the nginx, hardening, and rsync sheets already lean on.

---

## Table of Contents
- [Mental Model](#mental-model)
- [Install & Bring Up](#install--bring-up)
- [Status & Diagnostics](#status--diagnostics)
- [MagicDNS & Naming](#magicdns--naming)
- [Tailscale SSH](#tailscale-ssh)
- [Serve & Funnel](#serve--funnel)
- [Exit Nodes](#exit-nodes)
- [Subnet Router](#subnet-router)
- [ACL Basics](#acl-basics)
- [Keys & Maintenance](#keys--maintenance)
- [Stack Integration](#stack-integration)
- [Tips & Gotchas](#tips--gotchas)

---

## MENTAL MODEL

```text
Data plane    → WireGuard, peer-to-peer, encrypted end-to-end
Control plane → Tailscale's coordination server (key exchange, ACLs, DNS)
Addresses     → Each node gets a stable IP in 100.64.0.0/10 (CGNAT space)
Fallback      → Can't punch through NAT? Traffic relays via DERP (slower, still encrypted)
No inbound firewall rules needed — everything is established outbound.
```

---

## INSTALL & BRING UP

```bash
curl -fsSL https://tailscale.com/install.sh | sh    # Official installer (adds apt repo)
sudo tailscale up                                   # Prints auth URL — open, approve, done
sudo tailscale up --hostname=vps-hermes             # Set the tailnet name at join
sudo tailscale up --operator=$USER                  # Let your user run tailscale without sudo
sudo tailscale up --auth-key=tskey-auth-xxxxx       # Headless/scripted join (see Keys)
```

```bash
sudo tailscale set --hostname=new-name    # Change settings AFTER up — see Gotchas
tailscale down                            # Disconnect (config kept)
tailscale logout                          # Disconnect + invalidate node key
```

```text
⚠️ Re-running `tailscale up` RESETS any flag you don't re-specify (it
errors and tells you which). For changing one setting on a live node,
`tailscale set` is the right tool.
```

---

## STATUS & DIAGNOSTICS

```bash
tailscale status                 # All peers: IPs, OS, connection state
tailscale ip -4                  # This node's tailnet IPv4
tailscale ip -4 zafra-asus       # A peer's tailnet IPv4
tailscale ping vps-hermes        # Tailnet-layer ping — shows DIRECT vs DERP relay
tailscale netcheck               # NAT type, nearest DERP, UDP reachability
tailscale whois 100.71.128.46    # Which device/user owns this tailnet IP
tailscale version
systemctl status tailscaled      # It's a systemd service like any other
journalctl -u tailscaled -e      # Daemon logs (see systemd sheet)
```

---

## MAGICDNS & NAMING

```text
With MagicDNS on (default for new tailnets), machine names just resolve:

ssh jedi@zafra-asus              → No IP, no /etc/hosts, no ssh config needed
curl http://vps-hermes:8501      → Internal dashboards by name
Full name: <machine>.<tailnet-name>.ts.net

Rename: admin console → Machines → rename, or --hostname / tailscale set
```

```text
💡 Names beat IPs in scripts and configs — a rebuilt node keeps its name
even when its tailnet IP changes.
```

---

## TAILSCALE SSH

Tailscale terminates SSH itself: authentication is your tailnet identity + ACLs — no authorized_keys to distribute or rotate for tailnet-internal access.

```bash
sudo tailscale up --ssh          # On the server (re-specify your other up flags!)
ssh user@vps-hermes              # From any tailnet device — just works
tailscale status                 # Server shows it's advertising SSH
```

```text
💡 Where it shines: many boxes, no key sprawl, ACL-controlled access.
Where it doesn't replace the SSH sheet: public-facing SSH, non-tailnet
CI, and the day tailscaled itself is down — keep a fallback path
(see Gotchas + VPS Hardening sheet).
```

---

## SERVE & FUNNEL

### serve — share a local port with your TAILNET (HTTPS included)

```bash
tailscale serve 3000             # https://<machine>.<tailnet>.ts.net → localhost:3000
tailscale serve --bg 3000        # Same, backgrounded (persists)
tailscale serve status           # What's currently shared
tailscale serve reset            # Clear all serve/funnel config
```

### funnel — expose a local port to the PUBLIC INTERNET

```bash
tailscale funnel 3000            # Public https://<machine>.<tailnet>.ts.net → localhost:3000
tailscale funnel --bg 3000
```

```text
⚠️ Funnel = internet-facing. The moment it's on, treat the service like
anything public: auth, rate limits, updates (see nginx + hardening sheets).
Both need HTTPS enabled for the tailnet; funnel additionally requires the
funnel node attribute in your ACL policy (admin console prompts you).
💡 serve is the zero-config alternative to nginx for tailnet-internal
dashboards — TLS cert provisioning is automatic.
```

---

## EXIT NODES

Route ALL of a device's traffic through another tailnet node (VPN-out-through-my-VPS).

```bash
# On the node that will carry traffic — IP forwarding first:
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
sudo sysctl -p /etc/sysctl.d/99-tailscale.conf

sudo tailscale up --advertise-exit-node     # Then APPROVE in the admin console
```

```bash
# On the client:
sudo tailscale set --exit-node=vps-hermes
sudo tailscale set --exit-node=                      # Stop using it
sudo tailscale set --exit-node-allow-lan-access      # Keep local LAN reachable meanwhile
```

---

## SUBNET ROUTER

Expose a whole LAN (e.g. mining-site equipment) to the tailnet through one node:

```bash
# On the gateway node (IP forwarding sysctls as above):
sudo tailscale up --advertise-routes=192.168.1.0/24   # Approve in admin console

# On Linux clients that should see those routes:
sudo tailscale set --accept-routes
```

---

## ACL BASICS

Policy is HuJSON in the admin console (Access Controls). Default = everyone reaches everything; tighten by tagging servers:

```json
{
  "tagOwners": {
    "tag:server": ["autogroup:admin"]
  },
  "acls": [
    // Members can reach servers only on SSH + HTTPS:
    { "action": "accept", "src": ["autogroup:member"], "dst": ["tag:server:22,443"] }
  ]
}
```

```bash
sudo tailscale up --advertise-tags=tag:server    # Tag a node at join time
```

```text
💡 ACLs are default-DENY once the file defines any rule — the console's
preview panel shows exactly who can reach what before you save. Start
tightening only after everything is named and tagged.
```

---

## KEYS & MAINTENANCE

```text
⚠️ NODE KEYS EXPIRE (default ~180 days) — an unattended VPS silently
drops off the tailnet when its key lapses. For servers: admin console
→ Machines → ⋯ → Disable key expiry. Do this the day you join them.
```

```bash
# Auth keys (admin console → Settings → Keys) for scripted joins:
sudo tailscale up --auth-key=tskey-auth-xxxxx
# Reusable  → many machines, same key
# Ephemeral → node auto-removes when it goes offline (containers, CI)
```

```bash
sudo tailscale update            # Update the client (non-store installs)
# Or just apt — the installer added the repo:
sudo apt update && sudo apt install tailscale
```

---

## STACK INTEGRATION

How the rest of the repo already uses the tailnet:

```text
ufw        → sudo ufw allow in on tailscale0        (VPS Hardening sheet)
fail2ban   → ignoreip = 127.0.0.1/8 100.64.0.0/10   (VPS Hardening sheet)
nginx      → allow 100.64.0.0/10; deny all;          (nginx sheet — dashboard containment)
Services   → bind to the tailnet IP or 127.0.0.1, never 0.0.0.0 (Docker sheet)
SSH        → tailnet-only endgame: close public 22   (VPS Hardening sheet)
rsync      → pull backups over tailnet names: rsync -az vps-hermes:/opt/ ... (rsync sheet)
```

---

## TIPS & GOTCHAS

- **`tailscale up` vs `tailscale set`** — `up` re-applies the FULL flag set; anything omitted is reset (it errors and lists what you dropped). `set` changes one thing. Scripts should always pass complete flags to `up`.
- **Key expiry is the #1 unattended-server outage** — disable it on every VPS the day it joins. A key that expires during your vacation takes the box off the tailnet until someone re-auths.
- **`tailscale ping` before blaming the network** — `via DERP` means NAT traversal failed and you're relayed (slower); check UDP 41641 egress. `direct` means the mesh is doing its job.
- **MagicDNS can hijack /etc/resolv.conf** — on VPS boxes with their own resolver setup, `sudo tailscale set --accept-dns=false` keeps Tailscale out of DNS while keeping the mesh.
- **Exit node "accepts but no traffic flows"** — you skipped the IP-forwarding sysctls, or never approved it in the admin console. Both are required.
- **Funnel is not serve** — one exposes to your tailnet, the other to the planet. Read the command twice; audit with `tailscale serve status`.
- **100.64.0.0/10 is borrowed CGNAT space** — if your ISP or cloud provider also uses CGNAT internally, address collisions are possible (rare, confusing). `tailscale netcheck` output helps spot the overlap.
- **Tailnet-only SSH needs a break-glass path** — if tailscaled crashes or expires, SSH-over-tailnet dies with it. Keep the provider's web console credentials handy, or leave hardened public SSH (keys-only + fail2ban) as the fallback per the hardening sheet.
- **Ephemeral keys for containers** — Docker/CI nodes joined with ephemeral keys clean themselves up; reusable keys leave zombie machines in the console.

---
*Last Updated: 2026-07*
