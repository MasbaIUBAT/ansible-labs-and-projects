# Lab 02: Configure Ansible Inventory and Test Connectivity

## Overview

In this lab, I created an Ansible inventory file, added a managed node, verified the inventory structure, and tested connectivity from the Ansible control node to the managed node.

## Lab Environment

| Component | Details |
|---|---|
| Platform | Killercoda |
| Control node | controlplane |
| Managed node | node01 |
| Managed node IP | 172.30.2.2 |
| Remote user | root |
| Ansible Core | 2.16.3 |
| Python | 3.12.3 |

## Objectives

- Understand the difference between `/etc/hosts` and an Ansible inventory
- Create an INI-format inventory file
- Add a managed node to an inventory group
- Validate the inventory structure
- Test network and Ansible connectivity
- Run a command on the managed node

## Step 1: Identify the Managed Node

I checked the Linux hosts file:

```bash
cat /etc/hosts
```

Relevant entry:

```text
172.30.2.2 node01
```

The `/etc/hosts` file maps the hostname `node01` to its IP address. It is a Linux hostname resolution file, while `inventory.ini` tells Ansible which hosts to manage.

## Step 2: Create the Inventory File

I created `inventory.ini` with the following content:

```ini
[managed_nodes]
node01 ansible_host=172.30.2.2 ansible_user=root
```

Inventory details:

- `managed_nodes` is the inventory group.
- `node01` is the inventory hostname.
- `ansible_host` defines the target IP address.
- `ansible_user` defines the SSH user.

## Step 3: Validate the Inventory

```bash
ansible-inventory -i inventory.ini --graph
```

Output:

```text
@all:
  |--@ungrouped:
  |--@managed_nodes:
  |  |--node01
```

This output confirms that Ansible found `node01` inside the `managed_nodes` group.

## Step 4: Test Network Reachability

```bash
ping -c 2 node01
```

Result:

```text
2 packets transmitted, 2 received, 0% packet loss
```

This test confirmed basic network reachability between `controlplane` and `node01`.

## Step 5: Test Ansible Connectivity

```bash
ansible managed_nodes -i inventory.ini -m ansible.builtin.ping
```

Output:

```text
node01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

The `SUCCESS` and `pong` responses confirm that Ansible connected to `node01` and executed the ping module through Python.

The Ansible ping module is different from the normal Linux `ping` command. Linux ping tests network reachability with ICMP. The Ansible ping module tests the Ansible connection and Python execution on the managed node.

## Step 6: Run a Remote Command

```bash
ansible managed_nodes -i inventory.ini -m ansible.builtin.command -a "hostname"
```

Output:

```text
node01 | CHANGED | rc=0 >>
node01
```

The command ran successfully on the managed node. The return code `rc=0` confirms successful execution.

## Step 7: Confirm the Target Hosts

```bash
ansible managed_nodes -i inventory.ini --list-hosts
```

Output:

```text
hosts (1):
  node01
```

This confirmed that the `managed_nodes` group contains one target host.

## Files

```text
02-inventory-and-connectivity/
├── README.md
├── inventory.ini
└── verification.txt
```

## Skills Practised

- Ansible inventory management
- Inventory groups and host variables
- Linux hostname resolution
- Network connectivity testing
- Ansible connectivity testing
- Remote command execution

## Result

The control node successfully connected to `node01`. Ansible discovered Python at `/usr/bin/python3` and executed a remote command successfully.

## References

- [Ansible inventory documentation](https://docs.ansible.com/projects/ansible/latest/inventory_guide/intro_inventory.html)
- [Ansible ping module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/ping_module.html)