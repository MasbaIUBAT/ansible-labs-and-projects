Docker-Based Ansible Automation Lab

A local Ansible practice environment built with Docker Compose.

This repository creates one Ansible control node and four managed Ubuntu Linux nodes. It supports SSH key authentication, inventory management, ad-hoc commands, playbooks, Ansible Vault, and repeatable configuration tasks.

Architecture
Windows Host
└── Docker Desktop
    ├── ansible-control
    ├── node01
    ├── node02
    ├── node03
    └── node04

ansible-control runs Ansible. It connects to node01 through node04 over SSH with an ED25519 key pair.

Skills Practised
Dockerfile and Docker Compose
Multi-container Docker networking
SSH key-based authentication
Ansible inventory and configuration
Ad-hoc commands
Ansible modules: ping, command, shell, setup, copy, lineinfile, file, user, and apt
Playbooks and idempotency
Ansible Vault and protected configuration files
Project Structure
.
├── docker/
│   ├── control/Dockerfile
│   ├── managed-node/Dockerfile
│   └── keys/                     # Local only, excluded from Git
├── inventory/hosts.ini
├── playbooks/
│   ├── deploy-message.yml
│   └── deploy-vault-config.yml
├── files/ansible-message.txt
├── evidence/
├── ansible.cfg
├── docker-compose.yml
└── .gitignore
Prerequisites
Windows 10 or Windows 11
Docker Desktop with the Linux engine running
OpenSSH Client, including ssh-keygen
PowerShell
Start the Lab After Cloning

Open PowerShell in the repository folder.

1. Generate a local SSH key pair
New-Item -ItemType Directory -Force -Path .\docker\keys | Out-Null
ssh-keygen -t ed25519 -f ".\docker\keys\ansible_lab_key"

Press Enter twice when asked for a passphrase. Do not commit either key file.

2. Build and start the containers
docker compose up -d --build
docker compose ps

Expected containers:

ansible-control
node01
node02
node03
node04
3. Verify the Ansible inventory
docker exec -e ANSIBLE_CONFIG=/workspace/ansible.cfg ansible-control ansible-inventory --graph
4. Test connectivity
docker exec -e ANSIBLE_CONFIG=/workspace/ansible.cfg ansible-control ansible managed_nodes -m ansible.builtin.ping

Each node should return:

SUCCESS
"ping": "pong"
Common Ad-Hoc Commands

Check hostnames:

docker exec -e ANSIBLE_CONFIG=/workspace/ansible.cfg ansible-control ansible managed_nodes -m ansible.builtin.command -a "hostname"

Run shell commands:

docker exec -e ANSIBLE_CONFIG=/workspace/ansible.cfg ansible-control ansible managed_nodes -m ansible.builtin.shell -a "hostname && uptime"

Collect operating system facts:

docker exec -e ANSIBLE_CONFIG=/workspace/ansible.cfg ansible-control ansible managed_nodes -m ansible.builtin.setup -a "filter=ansible_distribution*"

Collect network facts:

docker exec -e ANSIBLE_CONFIG=/workspace/ansible.cfg ansible-control ansible managed_nodes -m ansible.builtin.setup -a "filter=ansible_default_ipv4"
Playbook Practice

Run the message deployment playbook:

docker exec -e ANSIBLE_CONFIG=/workspace/ansible.cfg ansible-control ansible-playbook /workspace/playbooks/deploy-message.yml

Run it a second time. A correct idempotent result shows:

changed=0
unreachable=0
failed=0
Ansible Vault Practice

Create a local secrets file. Do not use real credentials in a practice repository.

app_database_password: example-password
api_token: example-token

Encrypt the file:

docker exec -it -e ANSIBLE_CONFIG=/workspace/ansible.cfg ansible-control ansible-vault encrypt /workspace/playbooks/secrets.yml

Run the Vault-based playbook:

docker exec -it -e ANSIBLE_CONFIG=/workspace/ansible.cfg ansible-control ansible-playbook /workspace/playbooks/deploy-vault-config.yml --ask-vault-pass

The playbook deploys /etc/ansible-lab/app.env with permission 0600. It uses no_log: true so secret values do not appear in terminal output.

Security Notes
docker/keys/ is excluded from Git because it contains an SSH private key.
playbooks/secrets.yml is excluded from Git because it contains sensitive values, even when encrypted.
Never place real passwords, API tokens, or private keys in a public repository.
Use --ask-vault-pass, a protected password file, or a CI/CD secret for Vault passwords.
Stop the Lab

Stop containers but keep them available for the next session:

docker compose stop

Start them again:

docker compose start

Remove containers and the Docker network:

docker compose down
Verified Results

The lab successfully demonstrated:

SSH key authentication from the Ansible control node to four managed nodes
Inventory discovery of four target nodes
Successful Ansible ping responses from all nodes
File deployment and line management with idempotent playbook execution
Linux user and directory creation
Package installation with the apt module
Encrypted variable loading with Ansible Vault
Protected configuration files with root ownership and 0600 permissions