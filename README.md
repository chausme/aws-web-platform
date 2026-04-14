# AWS Web Platform

Automation for provisioning and configuring a minimal web platform on AWS using Terraform and Ansible.

The setup is suitable for hosting PHP-based web applications such as WordPress.

Work in progress.

## Local testing

### 1. Setup Python environment

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
ansible-galaxy collection install -r ansible/collections/requirements.yml
```

### 2. Add local environment variables

Copy the example `local/.env.example` and update values as needed. This file defines environment variables used by Docker services such as MySQL credentials.

```bash
cp local/.env.example local/.env
```

### 3. Start local test container

```bash
docker compose -f local/compose.yml up -d --build
```

### 4. Run server configuration playbook

```bash
.venv/bin/ansible-playbook -i ansible/inventories/local/inventory.yml ansible/server.yml
```

### 5. Define site configuration

Site settings such as PHP version, domain, system user and application type are defined in:

`ansible/group_vars/web.yml`

```bash
site:
  domain: "example.com"
  user: "siteuser"
  app: "wordpress" # supported: "php", "wordpress"
  php_version: "8.3"
```

- `php` provisions a generic PHP site
- `wordpress` provisions a WordPress site and installs WP-CLI

The PHP version is configurable and supports versions from 7.4 to 8.5. To use a different version, update this value before running the site playbook.

### 6. Run site setup playbook

```bash
.venv/bin/ansible-playbook -i ansible/inventories/local/inventory.yml ansible/site.yml
```

### 7. Verify in browser

Open `http://localhost:8080` in your browser.

Alternatively, map test domains in the system hosts file and access sites using their domain names, for example: `http://example.com:8080`. This ensures nginx can correctly match virtual hosts based on the request hostname.

### 8. Review inside the container

```bash
docker exec -it ansible_target /bin/bash
```

### 9. Reset environment

Ansible target only

```bash
docker compose -f local/compose.yml down --rmi local
```

Ansible target and MySQL data

```bash
docker compose -f local/compose.yml down --rmi local --volumes
```

## Example scenarios

Configuration changes are applied by updating variables and re-running playbooks.

### Initial site setup

Provision a new site using the current configuration:

```bash
.venv/bin/ansible-playbook -i ansible/inventories/local/inventory.yml ansible/site.yml
```

This will:

- create the system user and directories
- install PHP and required packages
- configure PHP-FPM pool
- configure and enable nginx site

### Change domain for an existing site

To change the domain for an existing site, update the `site.domain` value in `ansible/group_vars/web.yml` and then run the site playbook again:

```bash
.venv/bin/ansible-playbook -i ansible/inventories/local/inventory.yml ansible/site.yml --tags nginx
```

This will:

- update nginx configuration to use the new domain
- remove old nginx site configuration for the previous domain

### Change PHP version for an existing site

To change the PHP version for an existing site, update the `site.php_version` value in `ansible/group_vars/web.yml` and then run the site playbook again:

```bash
.venv/bin/ansible-playbook -i ansible/inventories/local/inventory.yml ansible/site.yml --tags php
```

This will:

- install the new PHP version if not already installed
- create or update the PHP-FPM pool configuration
- remove old PHP-FPM pool configurations for this site from other PHP versions
- update nginx configuration to use the new PHP-FPM socket

### Add another site

To add another site, define a new site variable set in `ansible/group_vars/web.yml` and then run the site playbook again:

```bash
.venv/bin/ansible-playbook -i ansible/inventories/local/inventory.yml ansible/site.yml
```

This will:

- create the system user and directories
- install PHP and required packages if not already installed
- configure PHP-FPM pool
- configure and enable nginx site

## Notes

- Site system users are treated as unique and persistent:
    - they cannot be changed for an existing site
    - they cannot be reused across multiple sites
- In the local Docker environment, all data inside the Ansible target container is ephemeral and reset after container removal. MySQL data is persisted in a Docker volume, but can be removed using the `--volumes` flag
