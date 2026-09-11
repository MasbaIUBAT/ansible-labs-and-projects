# Lab 03: Ansible Ad Hoc Commands, Shell and Setup Module

## What I learned

This lab used Ansible ad hoc commands. An ad hoc command runs one task directly from the terminal. It does not need a playbook.

I used two Ansible modules:

| Module | Purpose |
|---|---|
| `ansible.builtin.shell` | Runs Linux shell commands on managed hosts |
| `ansible.builtin.setup` | Collects system information, also called Ansible facts |

## Lab Hosts

The `hosts` inventory file contained two servers:

```ini
[servers]
controlplane
node01
```

- `controlplane` is the machine where I typed Ansible commands.
- `node01` is a separate managed node.
- The `servers` group targets both hosts.

## Basic Ad Hoc Command Format

```bash
ansible TARGET -i INVENTORY -m MODULE -a "ARGUMENTS"
```

Example:

```bash
ansible servers -i hosts -m ansible.builtin.shell -a "hostname && uptime"
```

## 1. Test Ansible Connectivity

```bash
ansible servers -i hosts -m ansible.builtin.ping
```

Result: Both `controlplane` and `node01` returned `SUCCESS` and `pong`.

This confirmed that Ansible could connect to both hosts and run Python modules.

## 2. Shell Module: Check Hostname and Uptime

```bash
ansible servers -i hosts -m ansible.builtin.shell -a "hostname && uptime"
```

This command ran on both hosts.

- `hostname` displayed the server name.
- `uptime` displayed how long the server had been running.
- `&&` ran the second command only after the first command succeeded.

The shell module supports shell operators such as `|`, `&&`, `>`, and `>>`.

## 3. Shell Module: Check Disk Usage

```bash
ansible servers -i hosts -m ansible.builtin.shell -a "df -h | grep '^/dev/'"
```

Result:

| Host | Root Disk Usage |
|---|---:|
| `node01` | 37% |
| `controlplane` | 63% |

This is useful for checking disk space across multiple Linux servers.

## 4. Setup Module: Check Operating System

```bash
ansible servers -i hosts -m ansible.builtin.setup -a "filter=ansible_distribution*"
```

Result: Both servers used Ubuntu 24.04 with the release name `noble`.

The setup module collected facts only. It did not change server configuration.

## 5. Setup Module: Check Network Information

```bash
ansible servers -i hosts -m ansible.builtin.setup -a "filter=ansible_default_ipv4"
```

Result:

| Host | Default IP Address |
|---|---|
| `node01` | 172.30.2.2 |
| `controlplane` | 172.30.1.2 |

This confirmed that `node01` and `controlplane` are separate lab hosts.

## 6. Setup Module: Check Total Memory

```bash
ansible servers -i hosts -m ansible.builtin.setup -a "filter=ansible_memtotal_mb"
```

Result:

| Host | Total Memory |
|---|---:|
| `node01` | 1903 MB |
| `controlplane` | 2246 MB |

## Shell vs Setup

| Shell module | Setup module |
|---|---|
| Runs a command on a managed host | Collects system facts from a managed host |
| Can use pipes and conditional commands | Returns OS, IP, memory, CPU, Python and other facts |
| Usually reports `CHANGED` after command execution | Usually reports `changed: false` |
| Example: `df -h \| grep '^/dev/'` | Example: `filter=ansible_memtotal_mb` |

## Skills Practised

- Ansible inventory groups
- Ad hoc command syntax
- Remote shell execution
- Disk usage checks on multiple servers
- System fact collection
- Operating system, IP address and memory checks

## References

- https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/shell_module.html
- https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/setup_module.html