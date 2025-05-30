# Ansible Inventory Guide for Beginners

## Introduction

Ansible inventory is a simple configuration file that tells Ansible which servers to manage. Think of it as your contact list for servers - it contains the addresses and details of all the machines you want to control with Ansible.

## Basic Inventory File Structure

### Simple INI Format (`inventory/hosts.ini`)

```ini
# Web Servers
[webservers]
web1.example.com
web2.example.com
web3.example.com

# Database Servers
[databases]
db1.example.com
db2.example.com

# Load Balancers
[loadbalancers]
lb1.example.com

# All servers combined
[production:children]
webservers
databases
loadbalancers
```

### With IP Addresses and Connection Details

```ini
# Web Servers
[webservers]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11
web3 ansible_host=192.168.1.12

# Database Servers  
[databases]
db1 ansible_host=192.168.1.20
db2 ansible_host=192.168.1.21

# Load Balancer
[loadbalancers]
lb1 ansible_host=192.168.1.30

# Common settings for all servers
[production:children]
webservers
databases
loadbalancers

[production:vars]
ansible_user=admin
ansible_ssh_private_key_file=~/.ssh/server_key
```

## Running Commands with Inventory: Ping Examples

### 1. Ping a Single Server

```bash
ansible -i inventory/hosts.ini web1 -m ping
```

This checks if the server named `web1` is reachable.

**Expected Output:**
```
web1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### 2. Ping a Group of Servers

```bash
ansible -i inventory/hosts.ini webservers -m ping
```

This pings all servers in the webservers group (web1, web2, web3).

**Expected Output:**
```
web1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
web2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
web3 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### 3. Ping All Servers

```bash
ansible -i inventory/hosts.ini all -m ping
```

This pings every server in your inventory file.

### 4. Ping Multiple Groups

```bash
# Ping web servers and databases
ansible -i inventory/hosts.ini webservers:databases -m ping

# Ping everything except load balancers
ansible -i inventory/hosts.ini all:!loadbalancers -m ping
```

## Simple Inventory Examples

### Example 1: Home Lab Setup

```ini
# Home Lab Servers
[raspberry-pi]
pi1 ansible_host=192.168.0.100
pi2 ansible_host=192.168.0.101
pi3 ansible_host=192.168.0.102

[ubuntu-servers]
server1 ansible_host=192.168.0.110
server2 ansible_host=192.168.0.111

[all:vars]
ansible_user=pi
ansible_ssh_password=raspberry
```

### Example 2: Small Business Setup

```ini
# Office Servers
[file-servers]
fileserver ansible_host=10.0.1.5

[mail-servers]
mailserver ansible_host=10.0.1.10

[backup-servers]
backup ansible_host=10.0.1.15

[office:children]
file-servers
mail-servers
backup-servers

[office:vars]
ansible_user=administrator
```

### Example 3: Development Environment

```ini
# Development Servers
[dev-web]
dev-web1 ansible_host=172.16.1.10
dev-web2 ansible_host=172.16.1.11

[dev-db]
dev-db1 ansible_host=172.16.1.20

[staging]
staging-server ansible_host=172.16.2.10

[development:children]
dev-web
dev-db

[development:vars]
ansible_user=developer
ansible_become=yes
```

## YAML Format (Alternative)

Some people prefer YAML format, which looks like this:

```yaml
all:
  children:
    webservers:
      hosts:
        web1:
          ansible_host: 192.168.1.10
        web2:
          ansible_host: 192.168.1.11
        web3:
          ansible_host: 192.168.1.12
    databases:
      hosts:
        db1:
          ansible_host: 192.168.1.20
        db2:
          ansible_host: 192.168.1.21
  vars:
    ansible_user: admin
```

## Common Connection Settings

### Basic Variables You'll Use

```ini
# Server connection details
ansible_host=192.168.1.10          # IP address of the server
ansible_port=22                    # SSH port (default is 22)
ansible_user=ubuntu                # Username to connect with

# Authentication methods
ansible_ssh_private_key_file=~/.ssh/my_key    # Use SSH key
ansible_ssh_password=mypassword               # Use password (not recommended)

# Become (sudo) settings
ansible_become=yes                 # Enable sudo
ansible_become_user=root          # Become root user
ansible_become_password=sudopass  # Sudo password
```

## Testing Your Inventory

### 1. Check if Ansible can read your inventory

```bash
# List all hosts
ansible-inventory -i inventory/hosts.ini --list

# Show details for a specific host
ansible-inventory -i inventory/hosts.ini --host web1

# Show inventory structure
ansible-inventory -i inventory/hosts.ini --graph
```

### 2. Test basic connectivity

```bash
# Ping all servers
ansible -i inventory/hosts.ini all -m ping

# Get basic facts about servers
ansible -i inventory/hosts.ini all -m setup --limit web1
```

### 3. Run simple commands

```bash
# Check uptime on all web servers
ansible -i inventory/hosts.ini webservers -m command -a "uptime"

# Check disk space
ansible -i inventory/hosts.ini all -m command -a "df -h"

# Check memory usage
ansible -i inventory/hosts.ini all -m command -a "free -h"
```

## Directory Structure for Beginners

Keep it simple at first:

```
my-ansible-project/
├── inventory/
│   └── hosts.ini
├── playbooks/
│   └── basic-setup.yml
└── ansible.cfg
```

Later, as you grow:

```
my-ansible-project/
├── inventory/
│   ├── production/
│   │   └── hosts.ini
│   └── development/
│       └── hosts.ini
├── playbooks/
├── roles/
└── group_vars/
```

## Common Beginner Mistakes and Solutions

### 1. Can't connect to servers

**Check:**
- Is the IP address correct?
- Can you SSH manually? `ssh user@192.168.1.10`
- Are SSH keys set up properly?
- Is the username correct?

### 2. Permission denied errors

**Solutions:**
```ini
# Add these to your inventory
ansible_become=yes
ansible_become_password=your_sudo_password
```

### 3. Python not found errors

**Solution:**
```ini
# Add this to your inventory
ansible_python_interpreter=/usr/bin/python3
```

## Quick Start Checklist

1. **Create inventory file** with your server details
2. **Test connectivity** with `ansible all -m ping`
3. **Try basic commands** like `ansible all -m command -a "whoami"`
4. **Start with simple playbooks** once ping works
5. **Add more servers** and groups as needed

## Next Steps

Once you're comfortable with basic inventory:
- Learn about group variables (`group_vars/`)
- Try dynamic inventory for cloud servers
- Use inventory plugins for advanced setups
- Organize multiple environments (dev, staging, production)

Remember: Start simple! You can always make your inventory more complex as you learn more about Ansible.
