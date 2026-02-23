# ansible_cheatsheet

# CORE CONCEPTS

---

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

---

## INSTALLATION & CONFIG

pip install ansible                   → Install via pip
ansible --version                     → Verify install

# Default config file: /etc/ansible/ansible.cfg

# Project-level: ./ansible.cfg

[defaults]
inventory      = ./inventory
remote_user    = ubuntu
private_key_file = ~/.ssh/id_ed25519
host_key_checking = False            → Disable SSH fingerprint check

---

## INVENTORY

# Static inventory (hosts or inventory.ini)

[webservers]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[dbservers]
db1 ansible_host=192.168.1.20 ansible_user=admin

[all:vars]
ansible_python_interpreter=/usr/bin/python3

# YAML inventory (inventory.yml)

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

ansible-inventory --list             → Show parsed inventory
ansible-inventory --graph            → Visual host tree

---

## AD-HOC COMMANDS

ansible all -m ping                              → Ping all hosts
ansible webservers -m ping                       → Ping group
ansible all -m command -a "uptime"               → Run command
ansible all -m shell -a "df -h | grep /dev/sda" → Shell command (supports pipes)
ansible all -m setup                             → Gather all facts
ansible all -m setup -a "filter=ansible_os_family"  → Filter facts
ansible webservers -m apt -a "name=nginx state=present" -b   → Install pkg
ansible webservers -m service -a "name=nginx state=restarted" -b
ansible all -m copy -a "src=file.txt dest=/tmp/file.txt"
ansible all -m file -a "path=/tmp/testdir state=directory"
ansible all -m user -a "name=alice state=present" -b
ansible webservers -m reboot -b                  → Reboot hosts

# Flags

- i inventory → Specify inventory file
-u user → Remote user
-b → Become (sudo)
--become-user=root → Specify become user
-k → Prompt for SSH password
-K → Prompt for sudo password
--check → Dry run
--diff → Show file diffs
-v / -vv / -vvv → Verbosity levels

---

## PLAYBOOK STRUCTURE

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

---

## RUNNING PLAYBOOKS

ansible-playbook site.yml                      → Run playbook
ansible-playbook site.yml -i inventory.ini     → Specify inventory
ansible-playbook site.yml --check              → Dry run
ansible-playbook site.yml --diff               → Show changes
ansible-playbook site.yml --tags "install"     → Run tagged tasks only
ansible-playbook site.yml --skip-tags "debug"  → Skip tagged tasks
ansible-playbook site.yml -l webservers        → Limit to host/group
ansible-playbook site.yml -e "var=value"       → Extra variable
ansible-playbook site.yml -v                   → Verbose output

---

## COMMON MODULES

# Package management

apt:
name: nginx
state: present | absent | latest
update_cache: true

yum:
name: httpd
state: present

# File operations

copy:
src: local_file.txt
dest: /remote/path/file.txt
owner: root
mode: '0644'

template:
src: config.j2
dest: /etc/app/config.conf

file:
path: /tmp/mydir
state: directory | absent | touch
mode: '0755'

fetch:
src: /remote/file.txt
dest: ./backups/

# Services

service:
name: nginx
state: started | stopped | restarted | reloaded
enabled: true

# Commands

command:
cmd: /usr/bin/myapp --init
creates: /var/lib/myapp/.initialized   → Skip if file exists

shell:
cmd: "echo hello > /tmp/test.txt"

# Users

user:
name: alice
groups: sudo
shell: /bin/bash
state: present

# Cron

cron:
name: "backup job"
minute: "0"
hour: "2"
job: "/usr/local/bin/backup.sh"

# Lineinfile

lineinfile:
path: /etc/ssh/sshd_config
regexp: '^PasswordAuthentication'
line: 'PasswordAuthentication no'
state: present

# Git

git:
repo: [https://github.com/user/repo.git](https://github.com/user/repo.git)
dest: /opt/myapp
version: main

---

## VARIABLES & FACTS

# In playbook

vars:
app_port: 8080
app_name: myapp

# In vars file

vars_files:

- vars/main.yml

# Group vars (auto-loaded)

group_vars/webservers.yml
group_vars/all.yml

# Host vars (auto-loaded)

host_vars/web1.yml

# Using variables

- name: Print app port
debug:
msg: "App running on port {{ app_port }}"

# Accessing facts

ansible_os_family           → "Debian" / "RedHat"
ansible_distribution        → "Ubuntu" / "CentOS"
ansible_default_ipv4.address → Primary IP
ansible_hostname            → Short hostname
ansible_memtotal_mb         → Total RAM in MB

---

## CONDITIONALS & LOOPS

# When (conditional)

- name: Install on Debian only
apt:
name: nginx
when: ansible_os_family == "Debian"
- name: Skip if already configured
shell: [setup.sh](http://setup.sh/)
when: not config_file.stat.exists

# Loop

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
name: "{{ [item.name](http://item.name/) }}"
groups: "{{ item.group }}"
loop:
    - { name: alice, group: sudo }
    - { name: bob, group: www-data }

---

## ROLES

ansible-galaxy init my_role            → Create role structure

# Role structure

roles/my_role/
├── tasks/main.yml
├── handlers/main.yml
├── templates/
├── files/
├── vars/main.yml
├── defaults/main.yml
└── meta/main.yml

# Use role in playbook

- hosts: webservers
roles:
    - my_role
    - { role: nginx, http_port: 8080 }

# Install community role

ansible-galaxy install geerlingguy.nginx
ansible-galaxy collection install community.general

---

## ANSIBLE VAULT

ansible-vault create secrets.yml        → Create encrypted file
ansible-vault edit secrets.yml          → Edit encrypted file
ansible-vault view secrets.yml          → View encrypted file
ansible-vault encrypt existing.yml      → Encrypt existing file
ansible-vault decrypt existing.yml      → Decrypt file
ansible-vault rekey secrets.yml         → Change password

# Use vault in playbook

ansible-playbook site.yml --ask-vault-pass
ansible-playbook site.yml --vault-password-file ~/.vault_pass

# Encrypt single value

ansible-vault encrypt_string 'mypassword' --name 'db_password'

---

## DEBUGGING

- name: Print variable
debug:
var: ansible_os_family
- name: Print message
debug:
msg: "Value is {{ my_var }}"
- name: Pause for inspection
pause:
prompt: "Check state, then press Enter"
- name: Register output
command: whoami
register: result
- name: Show registered output
debug:
var: result.stdout