# Ansible Cheatsheet

> Agentless configuration management over SSH: describe the desired state in YAML, run it against your inventory, get idempotent results. Same playbook, one host or fifty.

---

## Table of Contents
- [Core Concepts](#core-concepts)
- [Installation & Config](#installation--config)
- [Inventory](#inventory)
- [Ad-hoc Commands](#ad-hoc-commands)
- [Playbook Structure](#playbook-structure)
- [Running Playbooks](#running-playbooks)
- [Common Modules](#common-modules)
- [Variables & Facts](#variables--facts)
- [Conditionals & Loops](#conditionals--loops)
- [Roles](#roles)
- [Ansible Vault](#ansible-vault)
- [Debugging](#debugging)
- [Tips & Gotchas](#tips--gotchas)

---

## CORE CONCEPTS

```text
Control Node   → Machine running Ansible (your workstation/VPS)
Managed Node   → Remote host being configured
Inventory      → List of managed hosts (file or dynamic)
Playbook       → YAML file defining tasks to run
Task           → Single action (install pkg, copy file, run cmd)
Module         → Built-in function for a task (apt, copy, service, etc.)
Role           → Reusable, structured collection of tasks
Handler        → Task triggered by notify (e.g. restart nginx)
Fact           → Auto-collected host info (IP, OS, RAM, etc.)
Variable       → Dynamic values used in playbooks
Vault          → Encrypted secrets storage
```

---

## INSTALLATION & CONFIG

```bash
pipx install --include-deps ansible   # Isolated install (see Python env sheet)
pip install ansible                   # Or into a project venv
ansible --version                     # Verify install
```

### Project config — ./ansible.cfg (overrides /etc/ansible/ansible.cfg)

```ini
[defaults]
inventory         = ./inventory
remote_user       = ubuntu
private_key_file  = ~/.ssh/id_ed25519
host_key_checking = False        ; Disable SSH fingerprint check (lab/dev only)
```

---

## INVENTORY

### Static inventory (hosts or inventory.ini)

```ini
[webservers]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[dbservers]
db1 ansible_host=192.168.1.20 ansible_user=admin

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

### YAML inventory (inventory.yml)

```yaml
all:
  children:
    webservers:
      hosts:
        web1:
          ansible_host: 192.168.1.10
    dbservers:
      hosts:
        db1:
          ansible_host: 192.168.1.20
```

```bash
ansible-inventory --list             # Show parsed inventory
ansible-inventory --graph            # Visual host tree
```

```text
💡 Tailnet hosts work beautifully as inventory — MagicDNS names
(ansible_host: vps-hermes) instead of IPs (see Tailscale sheet).
```

---

## AD-HOC COMMANDS

```bash
ansible all -m ping                              # Ping all hosts (SSH + Python check)
ansible webservers -m ping                       # Ping group
ansible all -m command -a "uptime"               # Run command
ansible all -m shell -a "df -h | grep /dev/sda"  # Shell command (supports pipes)
ansible all -m setup                             # Gather all facts
ansible all -m setup -a "filter=ansible_os_family"   # Filter facts
ansible webservers -m apt -a "name=nginx state=present" -b   # Install pkg
ansible webservers -m service -a "name=nginx state=restarted" -b
ansible all -m copy -a "src=file.txt dest=/tmp/file.txt"
ansible all -m file -a "path=/tmp/testdir state=directory"
ansible all -m user -a "name=alice state=present" -b
ansible webservers -m reboot -b                  # Reboot hosts
```

### Flags

```text
-i inventory         → Specify inventory file
-u user              → Remote user
-b                   → Become (sudo)
--become-user=root   → Specify become user
-k                   → Prompt for SSH password
-K                   → Prompt for sudo password
--check              → Dry run
--diff               → Show file diffs
-v / -vv / -vvv      → Verbosity levels
```

---

## PLAYBOOK STRUCTURE

```yaml
# site.yml
---
- name: Configure web servers
  hosts: webservers
  become: true
  vars:
    http_port: 80
    app_user: www-data

  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
        update_cache: true

    - name: Copy config file
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart nginx

    - name: Ensure nginx is running
      service:
        name: nginx
        state: started
        enabled: true

  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

```text
💡 Modern style prefers fully-qualified module names —
ansible.builtin.apt instead of apt. Short names still resolve; FQCN
avoids collisions once collections enter the picture.
```

---

## RUNNING PLAYBOOKS

```bash
ansible-playbook site.yml                      # Run playbook
ansible-playbook site.yml -i inventory.ini     # Specify inventory
ansible-playbook site.yml --syntax-check       # Validate without connecting
ansible-playbook site.yml --check              # Dry run
ansible-playbook site.yml --check --diff       # Dry run + show what WOULD change
ansible-playbook site.yml --tags "install"     # Run tagged tasks only
ansible-playbook site.yml --skip-tags "debug"  # Skip tagged tasks
ansible-playbook site.yml -l webservers        # Limit to host/group
ansible-playbook site.yml -e "var=value"       # Extra variable
ansible-playbook site.yml -v                   # Verbose output
```

---

## COMMON MODULES

### Package management

```yaml
apt:
  name: nginx
  state: present        # present | absent | latest
  update_cache: true

dnf:
  name: httpd
  state: present
```

### File operations

```yaml
copy:
  src: local_file.txt
  dest: /remote/path/file.txt
  owner: root
  mode: '0644'

template:                 # Jinja2 — variables render into the file
  src: config.j2
  dest: /etc/app/config.conf

file:
  path: /tmp/mydir
  state: directory        # directory | absent | touch
  mode: '0755'

fetch:                    # Pull files FROM remote hosts
  src: /remote/file.txt
  dest: ./backups/
```

### Services

```yaml
service:
  name: nginx
  state: started          # started | stopped | restarted | reloaded
  enabled: true
```

### Commands

```yaml
command:                  # No shell features (safer)
  cmd: /usr/bin/myapp --init
  creates: /var/lib/myapp/.initialized   # Skip if file exists → idempotence

shell:                    # Pipes/redirects allowed (use sparingly)
  cmd: "echo hello > /tmp/test.txt"
```

### Users, cron, config lines, git

```yaml
user:
  name: alice
  groups: sudo
  shell: /bin/bash
  state: present

cron:
  name: "backup job"
  minute: "0"
  hour: "2"
  job: "/usr/local/bin/backup.sh"

lineinfile:
  path: /etc/ssh/sshd_config
  regexp: '^PasswordAuthentication'
  line: 'PasswordAuthentication no'
  state: present

git:
  repo: https://github.com/user/repo.git
  dest: /opt/myapp
  version: main
```

---

## VARIABLES & FACTS

```yaml
# In playbook
vars:
  app_port: 8080
  app_name: myapp

# From files
vars_files:
  - vars/main.yml
```

```text
Auto-loaded by path:
group_vars/webservers.yml    → Vars for a group
group_vars/all.yml           → Vars for everything
host_vars/web1.yml           → Vars for one host
```

```yaml
- name: Print app port
  debug:
    msg: "App running on port {{ app_port }}"
```

### Useful facts

```text
ansible_os_family             → "Debian" / "RedHat"
ansible_distribution          → "Ubuntu" / "CentOS"
ansible_default_ipv4.address  → Primary IP
ansible_hostname              → Short hostname
ansible_memtotal_mb           → Total RAM in MB
```

---

## CONDITIONALS & LOOPS

```yaml
- name: Install on Debian only
  apt:
    name: nginx
  when: ansible_os_family == "Debian"

- name: Skip if already configured
  shell: setup.sh
  when: not config_file.stat.exists
```

```yaml
- name: Install packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - curl
    - git

- name: Create users
  user:
    name: "{{ item.name }}"
    groups: "{{ item.group }}"
  loop:
    - { name: alice, group: sudo }
    - { name: bob, group: www-data }
```

---

## ROLES

```bash
ansible-galaxy init my_role            # Create role structure
```

```text
roles/my_role/
├── tasks/main.yml
├── handlers/main.yml
├── templates/
├── files/
├── vars/main.yml
├── defaults/main.yml
└── meta/main.yml
```

```yaml
# Use roles in a playbook
- hosts: webservers
  roles:
    - my_role
    - { role: nginx, http_port: 8080 }
```

```bash
ansible-galaxy install geerlingguy.nginx           # Community role
ansible-galaxy collection install community.general
```

---

## ANSIBLE VAULT

```bash
ansible-vault create secrets.yml        # Create encrypted file
ansible-vault edit secrets.yml          # Edit encrypted file
ansible-vault view secrets.yml          # View encrypted file
ansible-vault encrypt existing.yml      # Encrypt existing file
ansible-vault decrypt existing.yml      # Decrypt file
ansible-vault rekey secrets.yml         # Change password
```

```bash
ansible-playbook site.yml --ask-vault-pass
ansible-playbook site.yml --vault-password-file ~/.vault_pass

# Encrypt a single value (paste result into any vars file):
ansible-vault encrypt_string 'mypassword' --name 'db_password'
```

---

## DEBUGGING

```yaml
- name: Print variable
  debug:
    var: ansible_os_family

- name: Print message
  debug:
    msg: "Value is {{ my_var }}"

- name: Register output
  command: whoami
  register: result

- name: Show registered output
  debug:
    var: result.stdout

- name: Pause for inspection
  pause:
    prompt: "Check state, then press Enter"
```

---

## TIPS & GOTCHAS

- **`--check --diff` before every real run** — the dry-run diff is Ansible's `terraform plan`; skipping it is how "configure nginx" becomes "restart production."
- **Idempotence is the contract** — `command`/`shell` break it unless you add `creates:`/`removes:` or `changed_when:`. If a second run reports changes, the playbook is lying about state.
- **Handlers run once, at the end** — three tasks notifying "Restart nginx" produce ONE restart after all tasks finish; a failed task before the end means it never fires (`--force-handlers` exists).
- **YAML indentation is the syntax** — two spaces, never tabs; most "module not found"/parse errors are indentation. `--syntax-check` and `ansible-lint` catch it mechanically.
- **`host_key_checking = False` is for labs** — on real fleets, pre-populate known_hosts (see SSH sheet) instead of disabling the protection.
- **`shell` vs `command`** — `command` can't use pipes/redirects and is safer; reach for `shell` only when you need shell features, and quote carefully.
- **Vault the value, not the repo hygiene** — `encrypt_string` lets secrets live inline in otherwise-plaintext vars files; either way, the vault password file itself never gets committed.

---
*Last Updated: 2026-07*
