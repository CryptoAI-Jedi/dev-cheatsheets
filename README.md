# dev-cheatsheets

A personal, field-ready reference library for CLI tools, infrastructure, APIs, and blockchain development. Built for fast lookup during SRE/DevOps/Web3 work.

---

## Contents

| File | Description |
|------|-------------|
| [ansible_cheatsheet.md](./ansible_cheatsheet.md) | Inventory, playbooks, roles, modules, vault, ad-hoc commands, and templates |
| [aws_cli_cheatsheet.md](./aws_cli_cheatsheet.md) | EC2, S3, IAM, VPC, CloudWatch, SSM, and JMESPath output shaping |
| [bash_scripting_cheatsheet.md](./bash_scripting_cheatsheet.md) | Variables, loops, conditionals, functions, arrays, traps, getopts, and script patterns |
| [curl_api_testing_cheatsheet.md](./curl_api_testing_cheatsheet.md) | HTTP methods, headers, auth, JSON-RPC for ETH/Solana, and AI/LLM API examples |
| [docker_cheatsheet.md](./docker_cheatsheet.md) | Images, containers, volumes, networking, Compose, Dockerfile patterns, and cleanup |
| [git_cheatsheet.md](./git_cheatsheet.md) | Branching, rebasing, stash, reflog, bisect, undoing, and recovery |
| [jq_cheatsheet.md](./jq_cheatsheet.md) | JSON parsing, filtering, transformation, NDJSON log streams, and blockchain RPC examples |
| [kubernetes_cheatsheet.md](./kubernetes_cheatsheet.md) | kubectl, pods, deployments, services, namespaces, configmaps, secrets, and debugging |
| [linux_arch_cheatsheet.md](./linux_arch_cheatsheet.md) | pacman, AUR, yay, nftables, systemd-boot, recovery, and rolling-release discipline |
| [linux_centos_rhel_cheatsheet.md](./linux_centos_rhel_cheatsheet.md) | dnf/yum, firewalld, SELinux, LVM, NetworkManager (RHEL/CentOS specific) |
| [linux_debian_ubuntu_cheatsheet.md](./linux_debian_ubuntu_cheatsheet.md) | apt, ufw, systemd, networking, and CLI troubleshooting (Debian/Ubuntu specific) |
| [nginx_cheatsheet.md](./nginx_cheatsheet.md) | Server blocks, reverse proxy, SSL/TLS, load balancing, access control, and rate limiting |
| [prometheus_grafana_cheatsheet.md](./prometheus_grafana_cheatsheet.md) | PromQL queries, alert rules, exporters, Grafana dashboards, and containment |
| [python_cheatsheet.md](./python_cheatsheet.md) | Data types, comprehensions, functions, OOP, file I/O, error handling, and stdlib |
| [python_environments_packaging_cheatsheet.md](./python_environments_packaging_cheatsheet.md) | venv, pip, pyproject.toml, lockfiles, pipx, uv, and VPS deployment |
| [regex_cheatsheet.md](./regex_cheatsheet.md) | Core syntax, quantifiers, groups, lookaheads, flavors (BRE/ERE/PCRE), Python re, grep/sed/awk |
| [rsync_backup_cheatsheet.md](./rsync_backup_cheatsheet.md) | Delta sync over SSH, filters, --link-dest snapshots, pull-based backups, restic/rclone |
| [sql_cheatsheet.md](./sql_cheatsheet.md) | SELECT, JOIN, GROUP BY, subqueries, indexes, transactions, and common patterns |
| [ssh_cheatsheet.md](./ssh_cheatsheet.md) | Config, ProxyJump, port forwarding, keepalives, multiplexing, and escape sequences |
| [ssh_gpg_keys_cheatsheet.md](./ssh_gpg_keys_cheatsheet.md) | SSH & GPG key lifecycle, agent, encrypt/sign, backup/revoke, and Git commit signing |
| [systemd_journalctl_cheatsheet.md](./systemd_journalctl_cheatsheet.md) | Unit files, service management, timers, journalctl filtering, and log persistence |
| [tailscale_cheatsheet.md](./tailscale_cheatsheet.md) | Mesh VPN, MagicDNS, Tailscale SSH, serve/funnel, exit nodes, ACLs, and key expiry |
| [terraform_cheatsheet.md](./terraform_cheatsheet.md) | Init, plan, apply, state management, modules, variables, and provider config |
| [tmux_cheatsheet.md](./tmux_cheatsheet.md) | Sessions, windows, panes, keybindings, copy mode, mouse mode, scripting, and config |
| [vim_cheatsheet.md](./vim_cheatsheet.md) | Modes, motions, operators, text objects, registers, macros, and Neovim |
| [vps_hardening_cheatsheet.md](./vps_hardening_cheatsheet.md) | ufw, fail2ban, unattended-upgrades, tailnet-first patterns, and audit checklists |

---

## Usage

Clone this repo and reference locally:

```bash
git clone git@github.com:CryptoAI-Jedi/dev-cheatsheets.git
cd dev-cheatsheets
```

Search across all sheets:

```bash
grep -rn "ssh-keygen" .
grep -rn "jq.*select" .
grep -rn "firewall-cmd" .
```

Open a sheet in the terminal:

```bash
cat tmux_cheatsheet.md | less
```

---

## Stack Context

These sheets are optimized for:

- **OS:** Ubuntu 24 LTS (primary VPS), Debian, CentOS/RHEL, Arch
- **Blockchain:** Solana, Base (x402/EVM), Bitcoin, Ethereum
- **Tools:** Docker, Kubernetes, Prometheus, Grafana, Ansible, Terraform, Postman
- **Languages:** Bash, Python, Rust (learning), Vyper (learning)
- **Projects:** `node-health-monitor`, `wallet-ops-sentinel`, `mining-ops-toolkit`, Base x402 marketplace

---

## Maintenance

```bash
# Pull latest on your VPS
cd ~/dev-cheatsheets && git pull

# Add a new cheatsheet
git add new_tool_cheatsheet.md
git commit -m "Add new_tool cheatsheet"
git push origin main
```

---

## License

Personal use. Fork freely.
