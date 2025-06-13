# Ansible Playbooks for Server Monitoring

This repository contains Ansible playbooks to automate server monitoring, including:

- Cloning monitoring scripts from GitHub
- Running scripts on remote servers
- Collecting output locally
- Creating and managing directories remotely

---

## Inventory Setup

Refer to the `01_Inventory.md` for details on how to configure your Ansible inventory.

---

## Run Monitoring Script on Remote Servers

 Use this command:

```bash
ansible-playbook -i inventory/hosts.ini playbooks/run_server_monitor.yml
```

---

## What the Playbook Does

- Copies `server_monitor.sh` to each server
- Executes it remotely
- Fetches output to `monitor_reports/` locally
- Supports sudo and multi-host setups

---

## Prerequisites

1. Clone the GitHub repo containing the script:

```bash
git clone https://github.com/DevOpswithAnkita/linux-server-monitor.git
```

2. Confirm the script exists at:
   `linux-server-monitor/server_monitor.sh`
3. Your Ansible inventory file should be set up and tested.

---

## Sample Playbook: `run_server_monitor.yml`

```yaml
- name: Run server_monitor.sh on all servers
  hosts: all-prod-servers
  become: yes
  gather_facts: no

  vars:
    local_report_dir: "../monitor_reports"

  tasks:
    - name: Copy monitoring script to remote servers
      copy:
        src: ../linux-server-monitor/server_monitor.sh
        dest: /tmp/server_monitor.sh
        mode: '0755'

    - name: Run monitoring script and save output to file
      shell: /tmp/server_monitor.sh > /tmp/{{ inventory_hostname }}.txt
      args:
        executable: /bin/bash

    - name: Create local report directory
      delegate_to: localhost
      run_once: true
      become: no
      file:
        path: "{{ local_report_dir }}"
        state: directory
        mode: '0755'

    - name: Fetch report from remote server to local
      fetch:
        src: /tmp/{{ inventory_hostname }}.txt
        dest: "{{ local_report_dir }}/"
        flat: yes
```

---

## More Example Tasks

### 
    Create a Directory on Remote Server

```yaml
- name: Create a temp directory on remote servers
  file:
    path: /tmp/my-temp-dir
    state: directory
    mode: '0755'
```

###   Run a Command Using Shell

```yaml
- name: Check disk usage on remote server
  shell: df -h > /tmp/disk_usage.txt
  args:
    executable: /bin/bash
```

###   Print a Message

```yaml
- name: Print Hello World
  debug:
    msg: "Hello from Ansible!"
```

---

## Module Reference Summary

| Section | What It Does |
|--------|---------------|
| `- name:` | Human-readable description of what the task is doing. |
| `hosts:` | Defines which servers to run this playbook on. |
| `become:` | Whether to use sudo (admin privileges). |
| `vars:` | Defines variables used in tasks. |
| `tasks:` | List of actions to perform on the servers. |
| `copy:` | Transfers files from your computer to the servers. |
| `shell:` | Runs shell commands on remote machines. |
| `file:` | Manages files or directories (create/delete/modify). |
| `fetch:` | Brings files from the servers to your local machine. |
| `debug:` | Prints messages in the terminal during execution. |
| `delegate_to:` | Runs a task locally instead of on the remote server. |
| `run_once:` | Ensures a task runs just one time, not on every host. |

---

## Run the Playbook

```bash
ansible-playbook -i inventory/hosts.ini playbooks/run_server_monitor.yml
```

---

## Result

After running the playbook:

- Reports from all remote servers will be stored locally in the `monitor_reports/` directory.
- Each file will be named using the server hostname (e.g., `web1.txt`, `db1.txt`).

---

## Output Example

Each file might contain info like:

```
Hostname: web1
Uptime: 5 days
CPU Load: 0.23
Memory Usage: 45%
Disk Usage: / 70%
```

---

## Tips

- You can customize `server_monitor.sh` to collect different types of data.
- Schedule this playbook in a cron job for regular reporting.
- Extend it with Ansible email modules to send reports via email.

---

## Resources

- [Monitoring Script Repo](https://github.com/DevOpswithAnkita/linux-server-monitor)
-  [Ansible Docs](https://docs.ansible.com/)
