# Ansible Cheat Sheet

## Table of Contents

1. [Basics](#basics)
2. [Inventory](#inventory)
3. [Modules](#modules)
4. [Playbooks](#playbooks)
5. [Roles](#roles)
6. [Variables](#variables)
7. [Handlers](#handlers)
8. [Loops](#loops)
9. [Conditionals](#conditionals)
10. [Facts & Templates](#facts--templates)
11. [Tags](#tags)
12. [Vault & Security](#vault--security)
13. [Debugging](#debugging)
14. [Ad-Hoc Module Examples](#common-ad-hoc-module-examples)
15. [Homelab Use Cases](#homelab-use-cases)
16. [Advanced Features](#advanced-features)

    * Dynamic Inventory
    * Advanced Variables
    * Jinja2 Filters
    * Role Dependencies
    * Handlers & Notifications
    * Blocks, Rescue & Always
    * Async & Polling
    * Error Handling & Retry
    * Custom Modules
    * Plugins
    * Lookup & Register
    * Performance Optimization

---

## Basics

```bash
# Check Ansible version
ansible --version

# Ping all hosts in inventory
ansible all -m ping

# Run ad-hoc command
ansible all -m command -a "uptime"

# Run shell command
ansible webservers -m shell -a "df -h"
```

---

## Inventory

```ini
# hosts.ini
[webservers]
web01.example.com
web02.example.com

[dbservers]
db01.example.com
db02.example.com

[webservers:vars]
ansible_user=admin
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

```bash
# Test inventory
ansible all -i hosts.ini -m ping
```

---

## Modules

```bash
# File module
ansible all -m file -a "path=/tmp/testfile state=touch mode=0644"

# Package modules
ansible all -m apt -a "name=nginx state=latest"
ansible all -m yum -a "name=httpd state=present"

# Service module
ansible all -m service -a "name=nginx state=started enabled=yes"

# Copy module
ansible all -m copy -a "src=/local/file dest=/remote/file mode=0644"

# Template module
ansible all -m template -a "src=template.j2 dest=/etc/config.conf"
```

---

## Playbooks

```yaml
- name: Install Nginx and start service
  hosts: webservers
  become: yes
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
    - name: Start Nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

```bash
# Run playbook
ansible-playbook -i hosts.ini playbook.yml

# Dry run
ansible-playbook -i hosts.ini playbook.yml --check

# Extra vars
ansible-playbook -i hosts.ini playbook.yml -e "package=nginx"
```

---

## Roles

```bash
# Create role
ansible-galaxy init webserver
```

```yaml
# Use role in playbook
- hosts: webservers
  roles:
    - webserver
```

---

## Variables

```yaml
package_name: nginx
service_name: nginx

# Use variables
- name: Install package
  apt:
    name: "{{ package_name }}"
    state: present
```

```bash
# Extra vars
ansible-playbook playbook.yml -e "package_name=httpd"
```

---

## Handlers

```yaml
handlers/main.yml:
- name: restart nginx
  service:
    name: nginx
    state: restarted

# Trigger from task
- name: Update config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: restart nginx
```

---

## Loops

```yaml
- name: Install multiple packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - git
    - curl
```

---

## Conditionals

```yaml
- name: Install httpd only on RedHat
  yum:
    name: httpd
    state: present
  when: ansible_os_family == "RedHat"
```

---

## Facts & Templates

```bash
# Gather facts
ansible all -m setup

# Filter facts
ansible all -m setup -a "filter=ansible_distribution*"
```

```yaml
# Template example
server_name {{ inventory_hostname }}
os_version {{ ansible_distribution }} {{ ansible_distribution_version }}
```

---

## Tags

```yaml
- name: Install Web Stack
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
      tags: nginx
```

```bash
# Run specific tag
ansible-playbook playbook.yml --tags nginx
```

---

## Vault & Security

```bash
# Encrypt secrets
ansible-vault encrypt secrets.yml

# Edit
ansible-vault edit secrets.yml

# Run playbook with vault
ansible-playbook playbook.yml --ask-vault-pass
```

---

## Debugging

```yaml
- name: Debug variable
  debug:
    var: package_name
```

```bash
# Verbose output
ansible-playbook playbook.yml -vvv
```

---

## Common Ad-Hoc Module Examples

```bash
# Disk usage
ansible all -m shell -a "df -h"

# Restart service
ansible all -m service -a "name=nginx state=restarted"

# User management
ansible all -m user -a "name=testuser state=present groups=sudo"

# Fetch remote file
ansible all -m fetch -a "src=/etc/hosts dest=./hosts/{{ inventory_hostname }}/"
```

---

## Homelab Use Cases

```yaml
# Deploy Docker Compose
- hosts: docker_hosts
  become: yes
  tasks:
    - name: Copy compose file
      copy:
        src: docker-compose.yml
        dest: /home/admin/docker-compose.yml

    - name: Deploy stack
      shell: docker-compose -f /home/admin/docker-compose.yml up -d
```

```yaml
# Update multiple VMs
- hosts: vms
  tasks:
    - name: Upgrade system
      apt:
        upgrade: dist
        update_cache: yes

    - name: Reboot if needed
      reboot:
        msg: "Reboot initiated by Ansible"
```

---

## Advanced Features

### Dynamic Inventory

```yaml
plugin: aws_ec2
regions:
  - us-east-1
filters:
  tag:Environment: dev
```

```bash
ansible-inventory -i inventory_aws.yml --list
```

### Advanced Variables

```yaml
# Host vars
hostvars['web01'].ansible_eth0.ipv4.address

# Variable precedence
# extra vars > playbook vars > role defaults
```

### Jinja2 Filters

```yaml
{{ mylist | join(',') }}
{{ "hello world" | capitalize }}
{{ variable | default('fallback') }}
```

### Role Dependencies

```yaml
# meta/main.yml
dependencies:
  - role: common
  - role: logging
    vars:
      log_level: debug
```

### Blocks, Rescue & Always

```yaml
block:
  - name: Install package
    apt: name=nginx state=present
rescue:
  - debug: msg="Failed"
always:
  - file: path=/tmp/tempfile state=absent
```

### Async & Polling

```yaml
- shell: /usr/bin/long_script.sh
  async: 3600
  poll: 0
```

### Error Handling & Retry

```yaml
- apt:
    name: nginx
    state: present
  retries: 3
  delay: 5
  until: result is success
```

### Custom Modules

```bash
ansible all -m my_module -a "param1=value1 param2=value2"
```

### Plugins

```yaml
callback_whitelist = profile_tasks,json
ansible_connection: ssh
ansible_ssh_common_args: "-o StrictHostKeyChecking=no"
```

### Lookup & Register

```yaml
- debug: msg="{{ lookup('file','localfile.txt') }}"
- shell: cat /etc/hosts
  register: hosts_file
- debug: var=hosts_file.stdout_lines
```

### Performance Optimization

```yaml
- hosts: all
  serial: 5
```

```bash
ansible-playbook playbook.yml --limit webservers -f 20
```
