# AWS Web Platform

Automation for provisioning and configuring a minimal web platform on AWS using Terraform and Ansible.

The setup is suitable for hosting PHP-based web applications such as WordPress.

Work in progress.

## Local testing

Note: The local environment uses Docker and is fully ephemeral. All changes are reset after `docker compose down`.

### 1. Setup Python environment

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
ansible-galaxy collection install -r ansible/collections/requirements.yml
```

### 2. Start local test container

```bash
docker compose -f local/compose.yml up -d --build
```

### 3. Configure PHP version

The PHP version is configurable and supports versions from 7.4 to 8.5.

By default, the version is defined in:

`ansible/inventories/local/group_vars/all.yml`

```bash
php_version: "8.3"
```

To use a different version, update this value before running the playbook. _Note:_ Changing the PHP version requires resetting the local environment.

### 4. Configure site settings

Site settings such as domain and system user are defined in:

`ansible/inventories/local/group_vars/all.yml`

```bash:
site:
  domain: "example.com"
  user: "site"
```

Update these values before running the playbook if needed.

### 5. Run Ansible playbook

```bash
source .venv/bin/activate
ansible-playbook -i ansible/inventories/local/inventory.ini ansible/playbook.yml
```

### 6. Verify in browser

Open `http://localhost:8080` in your browser.

### 7. Reset environment

```bash
docker compose -f local/compose.yml down --rmi local
```
