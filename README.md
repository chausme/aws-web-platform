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

### 3. Run server configuration playbook

```bash
.venv/bin/ansible-playbook -i ansible/inventories/local/inventory.yml ansible/server.yml
```

### 4. Define site configuration

Site settings such as PHP version, domain, system user and application type are defined in:

`ansible/group_vars/web.yml`

```bash
php_version: "8.3"
site:
  domain: "example.com"
  user: "site"
  app: "wordpress" # supported: "php", "wordpress"
```

- `php` provisions a generic PHP site
- `wordpress` provisions a WordPress site and installs additional tooling such as WP-CLI

The PHP version is configurable and supports versions from 7.4 to 8.5. To use a different version, update this value before running the playbook. _Note:_ Changing the PHP version requires resetting the local environment.

### 5. Run site setup playbook

```bash
.venv/bin/ansible-playbook -i ansible/inventories/local/inventory.yml ansible/site.yml
```

### 6. Verify in browser

Open `http://localhost:8080` in your browser.

### 7. Reset environment

```bash
docker compose -f local/compose.yml down --rmi local
```
