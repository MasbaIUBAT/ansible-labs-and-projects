# Lab 04: Ansible Roles, Handlers, and Node.js Deployment

## Overview

This lab deploys a simple Node.js web application to a managed Docker container through Ansible.

The project runs on Windows. Docker Compose creates one Ansible control node and four managed Linux nodes. Ansible connects through SSH and applies configuration from reusable roles.

## Lab Environment

| Component                     | Value                                  |
| ----------------------------- | -------------------------------------- |
| Host operating system         | Windows                                |
| Project directory             | `D:\ansible`                           |
| Control node                  | `ansible-control`                      |
| Managed nodes                 | `node01`, `node02`, `node03`, `node04` |
| Node.js deployment target     | `node01`                               |
| Application port              | `3000`                                 |
| Managed-node operating system | Ubuntu 24.04                           |

## Architecture

```text
Windows host: D:\ansible
        |
        | Docker bind mount: /workspace
        v
ansible-control container
        |
        | SSH key authentication
        v
node01   node02   node03   node04
  |
  | Node.js application on port 3000
  v
nodejs-demo
```

## Project Files

```text
D:\ansible
├── ansible.cfg
├── docker-compose.yml
├── inventory/
│   └── hosts.ini
├── playbooks/
│   ├── apply-common.yml
│   └── deploy-nodejs.yml
├── roles/
│   ├── common/
│   │   └── tasks/main.yml
│   └── nodejs_app/
│       ├── defaults/main.yml
│       ├── tasks/main.yml
│       ├── handlers/main.yml
│       └── templates/app.js.j2
└── lab04-ansible-roles-nodejs-handler.md
```

## Inventory

The inventory defines four managed nodes.

File: `inventory/hosts.ini`

```ini
[managed_nodes]
node01 ansible_host=node01 ansible_user=ansible
node02 ansible_host=node02 ansible_user=ansible
node03 ansible_host=node03 ansible_user=ansible
node04 ansible_host=node04 ansible_user=ansible

[managed_nodes:vars]
ansible_ssh_private_key_file=/root/.ssh/ansible_lab_key
ansible_python_interpreter=/usr/bin/python3
```

View the inventory:

```powershell
docker exec -e ANSIBLE_CONFIG=/workspace/ansible.cfg ansible-control ansible-inventory --graph
```

Test Ansible connectivity:

```powershell
docker exec -e ANSIBLE_CONFIG=/workspace/ansible.cfg ansible-control ansible managed_nodes -m ansible.builtin.ping
```

All four nodes returned:

```text
ping: pong
```

## Common Role

The `common` role prepares Linux servers for application deployment.

File: `roles/common/tasks/main.yml`

The role ensures that each managed node has:

* `acl`
* `curl`
* `git`
* `ca-certificates`
* `appadmin` application user
* `/opt/apps` application base directory

Run the common role:

```powershell
docker exec -e ANSIBLE_CONFIG=/workspace/ansible.cfg ansible-control ansible-playbook /workspace/playbooks/apply-common.yml
```

The role was tested twice. The second execution returned `changed=0` on all four nodes. This confirms idempotency.

Idempotency means Ansible does not make unnecessary changes when the target server already matches the required configuration.

## Node.js Application Role

The `nodejs_app` role deploys a Node.js web application to `node01`.

```text
roles/nodejs_app/
├── defaults/main.yml
├── tasks/main.yml
├── handlers/main.yml
└── templates/app.js.j2
```

### Default Variables

File: `roles/nodejs_app/defaults/main.yml`

```yaml
---
nodejs_app_name: nodejs-demo
nodejs_app_dir: /opt/apps/nodejs-demo
nodejs_app_port: 3000
nodejs_app_message: Hello from node01. Managed by Ansible!
```

These variables make the role reusable. You can change the application name, application directory, port, or message without changing the task logic.

### Tasks

File: `roles/nodejs_app/tasks/main.yml`

The role performs these tasks:

1. Installs Node.js and npm.
2. Installs PM2 globally.
3. Creates `/opt/apps/nodejs-demo`.
4. Deploys the Node.js application from a Jinja2 template.
5. Notifies the handler if the application file changes.

PM2 keeps the Node.js application running as the `appadmin` user.

### Jinja2 Application Template

File: `roles/nodejs_app/templates/app.js.j2`

The template creates a small Node.js HTTP server.

Ansible replaces these variables during deployment:

```text
{{ nodejs_app_port }}
{{ nodejs_app_message }}
{{ nodejs_app_name }}
```

The application listens on port `3000`.

### Handler

File: `roles/nodejs_app/handlers/main.yml`

```yaml
---
- name: Restart Node.js application
  ansible.builtin.shell: |
    /usr/local/bin/pm2 restart "{{ nodejs_app_name }}" || /usr/local/bin/pm2 start app.js --name "{{ nodejs_app_name }}"
  args:
    chdir: "{{ nodejs_app_dir }}"
  become: true
  become_user: appadmin
```

The handler does not run during every playbook execution.

It runs only when Ansible changes `app.js`.

* On the first deployment, PM2 starts the application.
* On later application changes, PM2 restarts the existing application.

## Deployment Playbook

File: `playbooks/deploy-nodejs.yml`

```yaml
---
- name: Deploy Node.js demo application
  hosts: node01
  become: true

  roles:
    - common
    - nodejs_app
```

Run the deployment:

```powershell
docker exec -e ANSIBLE_CONFIG=/workspace/ansible.cfg ansible-control ansible-playbook /workspace/playbooks/deploy-nodejs.yml
```

## Verification

The deployed application was tested from the Ansible control container.

```powershell
docker exec ansible-control python3 -c "import urllib.request; print(urllib.request.urlopen('http://node01:3000').read().decode())"
```

Verified response:

```text
Hello from node01. Managed by Ansible!
```

This confirms that:

* Ansible deployed the application file.
* PM2 started the Node.js process.
* The control node can reach `node01` over the Docker network.
* The application responds on port `3000`.

## Issue Resolved

The first handler attempt failed because Ansible needed to switch from `root` to the unprivileged `appadmin` user.

The managed node did not contain the `acl` package. Ansible needed this package to manage permissions for temporary files during privilege escalation.

The fix was to add `acl` to the package list in the `common` role.

After the fix:

* The handler ran successfully.
* PM2 started the Node.js application.
* The application returned the expected HTTP response.

## Skills Practiced

* Docker and Docker Compose
* Linux containers
* Ansible inventory
* SSH key authentication
* Ansible ad-hoc commands
* Ansible playbooks
* Ansible roles
* Default variables
* Jinja2 templates
* Ansible handlers
* Privilege escalation with `become`
* PM2 process management
* Node.js application deployment
* Idempotent automation
* Git and GitHub

## GitHub Safety

The project excludes sensitive files from GitHub:

```text
docker/keys/
playbooks/secrets.yml
```

SSH private keys and Ansible Vault secrets must never be committed to a public repository.
