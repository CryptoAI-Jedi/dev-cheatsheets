# dev-cheatsheets

A personal, field-ready reference library for CLI tools, infrastructure, APIs, and blockchain development. Built for fast lookup during SRE/DevOps/Web3 work.

---

## Contents

| File | Description |
|------|-------------|
| [ansible_cheatsheet.md](./ansible_cheatsheet.md) | Inventory, playbooks, roles, modules, vault, ad-hoc commands, and templates |
| [aws_cli_cheatsheet.md](./aws_cli_cheatsheet.md) | EC2, S3, IAM, VPC, Lambda, CloudWatch, and credential management |
| [bash_scripting_cheatsheet.md](./bash_scripting_cheatsheet.md) | Variables, loops, conditionals, functions, arrays, traps, getopts, and script patterns |
| [curl_api_testing_cheatsheet.md](./curl_api_testing_cheatsheet.md) | GET/POST/PUT/DELETE, headers, auth, file upload, JSON-RPC for ETH/Solana/Base |
| [docker_cheatsheet.md](./docker_cheatsheet.md) | Images, containers, volumes, networking, Compose, Dockerfile patterns, and cleanup |
| [git_cheatsheet.md](./git_cheatsheet.md) | Full Git workflow — branching, rebasing, stash, worktrees, submodules, hooks, and recovery |
| [jq_cheatsheet.md](./jq_cheatsheet.md) | JSON parsing, filtering, transformation, CSV output, and blockchain RPC examples |
| [kubernetes_cheatsheet.md](./kubernetes_cheatsheet.md) | kubectl, pods, deployments, services, namespaces, configmaps, secrets, and RBAC |
| [linux_arch_cheatsheet.md](./linux_arch_cheatsheet.md) | pacman, AUR, yay, nftables, systemd-boot, rolling release tips |
| [linux_centos_rhel_cheatsheet.md](./linux_centos_rhel_cheatsheet.md) | dnf/yum, firewalld, SELinux, NetworkManager (RHEL/CentOS specific) |
| [linux_debian_ubuntu_cheatsheet.md](./linux_debian_ubuntu_cheatsheet.md) | apt, ufw, systemd, users, networking (Debian/Ubuntu specific) |
| [nginx_cheatsheet.md](./nginx_cheatsheet.md) | Server blocks, reverse proxy, SSL/TLS, load balancing, logging, and rate limiting |
| [prometheus_grafana_cheatsheet.md](./prometheus_grafana_cheatsheet.md) | PromQL queries, metrics, alerting rules, exporters, and Grafana dashboard setup |
| [python_cheatsheet.md](./python_cheatsheet.md) | Data types, comprehensions, functions, OOP, file I/O, error handling, and stdlib |
| [regex_cheatsheet.md](./regex_cheatsheet.md) | Core syntax, quantifiers, groups, lookaheads, Python re module, grep, sed, awk patterns |
| [sql_cheatsheet.md](./sql_cheatsheet.md) | SELECT, JOIN, GROUP BY, subqueries, indexes, transactions, and common patterns |
| [ssh_gpg_keys_cheatsheet.md](./ssh_gpg_keys_cheatsheet.md) | Key generation, ssh-agent, config, tunneling, GPG encrypt/sign, Git commit signing |
| [systemd_journalctl_cheatsheet.md](./systemd_journalctl_cheatsheet.md) | Unit files, service management, timers, journalctl filtering, and log persistence |
| [terraform_cheatsheet.md](./terraform_cheatsheet.md) | Init, plan, apply, state management, modules, variables, and provider config |
| [tmux_cheatsheet.md](./tmux_cheatsheet.md) | Sessions, windows, panes, keybindings, copy mode, scripting, and .tmux.conf config |
| [vim_cheatsheet.md](./vim_cheatsheet.md) | Modes, motions, operators, search/replace, macros, registers, splits, and config |

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
