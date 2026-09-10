# Lab 01: Install Ansible and Verify Local Functionality

## Overview

In this lab, I installed Ansible in a Killercoda Ubuntu environment, checked the installed version, and tested local module execution.

## Lab Environment

| Component | Value |
| --- | --- |
| Platform | Killercoda |
| Course | ProLUG Ansible Intro |
| Hostname | controlplane |
| Operating system | Ubuntu 24.04.4 LTS |
| User | root |
| Python | 3.12.3 |
| Ansible Core | 2.16.3 |
| Ansible community package | 9.2.0 |
| Jinja | 3.1.2 |

## Objectives

- Check the operating system, current user, and Python version.
- Install Ansible Core.
- Verify the installed Ansible version.
- Run a local test with the built-in ping module.
- Understand the command options and test output.

## Step 1: Check the Environment

Check the Linux distribution:

```bash
cat /etc/os-release
```

The output identified Ubuntu 24.04.4 LTS.

Check the current user:

```bash
whoami
```

Output:

```text
root
```

Check Python:

```bash
python3 --version
```

Output:

```text
Python 3.12.3
```

Check whether Ansible is installed:

```bash
ansible --version
```

Initially, the terminal reported:

```text
Command 'ansible' not found, but can be installed with:
apt install ansible-core
```

## Step 2: Install Ansible

Refresh the package index:

```bash
apt update
```

After the update completes successfully, install Ansible Core:

```bash
apt install -y ansible-core
```

The `-y` option automatically accepts installation confirmation prompts.

This session used the root account, so `sudo` was unnecessary.

The installation log included:

```text
Setting up ansible-core (2.16.3-0ubuntu2) ...
Setting up ansible (9.2.0+dfsg-0ubuntu5) ...
```

Both packages were installed in this environment. Ansible Core provides the main runtime and built-in modules. The larger Ansible package includes additional collections.

Their version numbers are different because they are separate packages.

These installation commands do not pin a version. A later installation may receive different versions from the configured repositories.

## Step 3: Verify the Installation

Run:

```bash
ansible --version
```

Observed output:

```text
ansible [core 2.16.3]
  config file = None
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.12.3 (main, Jun 19 2026, 12:46:00) [GCC 13.3.0] (/usr/bin/python3)
  jinja version = 3.1.2
  libyaml = True
```

### What the Output Means

| Output | Meaning |
| --- | --- |
| `core 2.16.3` | Installed Ansible Core version |
| `config file = None` | No Ansible configuration file was in use |
| `executable location` | Location of the Ansible command |
| `python version` | Python used to run Ansible |
| `jinja version` | Installed Jinja template engine version |

`config file = None` was not an error. The local test worked with default settings and explicit command options.

## Step 4: Test Local Module Execution

Run:

```bash
ansible localhost -i 'localhost,' -c local -m ansible.builtin.ping
```

### Command Explanation

| Part | Purpose |
| --- | --- |
| `ansible` | Runs an ad hoc task |
| `localhost` | Selects the current machine as the target |
| `-i 'localhost,'` | Supplies a one-host inventory directly |
| `-c local` | Runs locally without SSH |
| `-m ansible.builtin.ping` | Selects the built-in ping module |

The comma after `localhost` tells the inventory parser that this is a host list rather than a file path.

### Actual Result

```text
localhost | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

### Result Explanation

- `SUCCESS`: The module executed successfully.
- `discovered_interpreter_python`: Ansible detected `/usr/bin/python3` for module execution.
- `changed: false`: The test reported no configuration change.
- `ping: pong`: The module returned its expected success response.

Ansible's ping module checks whether it can execute a small Python-based test on the target. It is not an ICMP network ping.

Because this command used `-c local`, it did not test remote SSH connectivity.

## Step 5: Explore Command Help

Run:

```bash
ansible --help
```

Use the help output to review options such as `-i`, `-c`, and `-m`.

The help text includes:

```text
Some actions do not make sense in Ad-Hoc (include, meta, etc)
```

This is an informational note, not a failed test. Some actions are intended for playbook execution rather than standalone ad hoc tasks.

For further reading, the installed module documentation is available through:

```bash
ansible-doc ansible.builtin.ping
```

## What I Achieved

- Identified the lab operating system, user, and Python version.
- Installed Ansible and checked its Core version.
- Successfully executed the built-in ping module locally.
- Understood the inventory, connection, and module options in the test command.
- Interpreted the success response and change status.

## Scope

This lab verifies installation and local module execution.

It does not demonstrate remote SSH authentication, a persistent inventory file, playbook execution, or production deployment.

## Next Lab

Set up an Ansible inventory file and test connectivity to managed nodes.

## References

- [Ansible Installation Guide](https://docs.ansible.com/projects/ansible/latest/installation_guide/intro_installation.html)
- [Ansible Built-in Ping Module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/ping_module.html)
- [Ubuntu Ansible Core Package](https://packages.ubuntu.com/noble/ansible-core)