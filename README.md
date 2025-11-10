# Homelab Ansible

Ansible playbooks and roles for homelab infrastructure management.

## Overview

This repository contains Ansible automation for configuring and maintaining homelab services. It is designed to work with [Semaphore](https://www.ansible-semaphore.com/), an Ansible web UI and automation platform.

## Structure

```
.
├── roles/                    # Ansible roles
│   ├── snmpd/               # SNMP daemon installation and configuration
│   ├── portainer_agent/     # Portainer agent container deployment
│   └── portainer_register/  # Automatic Portainer environment registration
├── site.yml                 # SNMP deployment playbook
├── portainer.yml            # Portainer agent deployment and registration playbook
├── requirements.yml         # Ansible collection dependencies
└── ansible.cfg              # Ansible configuration
```

## Roles

### snmpd

Installs and configures the SNMP daemon for monitoring. Supports both SNMPv2c and SNMPv3.

See [roles/snmpd/README.md](roles/snmpd/README.md) for detailed documentation.

### portainer_agent

Deploys the Portainer agent container on Docker hosts, enabling remote management via Portainer server.

See [roles/portainer_agent/README.md](roles/portainer_agent/README.md) for detailed documentation.

### portainer_register

Automatically registers Docker hosts with Portainer server using the Portainer REST API. Eliminates the need to manually add environments through the Portainer web UI.

See [roles/portainer_register/README.md](roles/portainer_register/README.md) for detailed documentation.

## Playbooks

- **site.yml** - Installs and configures SNMP daemon on target hosts
- **portainer.yml** - Deploys Portainer agent container and automatically registers it with Portainer server

## Usage with Semaphore

### Prerequisites

**Required Ansible Collections:**

Install required collections before running playbooks:

```bash
ansible-galaxy collection install -r requirements.yml
```

Semaphore should do this automatically if configured properly.

**Environment Variables:**

Semaphore must be configured with the following environment variables for secrets:

**For SNMP (site.yml):**
- `SNMP_USER` - SNMPv3 username (required if SNMPv3 is enabled)
- `SNMP_PASS` - SNMPv3 authentication password (required if SNMPv3 is enabled)
- `SNMP_COMMUNITY` - SNMPv2c read-only community string (required if SNMPv2c is enabled)

**For Portainer (portainer.yml):**
- `PORTAINER_API_TOKEN` - Portainer API access token (required for automatic registration)
  - Generate in Portainer: User menu → My account → Access tokens → Add access token

**For Docker Hosts:**

- Docker must be pre-installed on target hosts before running the Portainer playbook
- The `portainer.yml` playbook automatically detects Docker availability and gracefully skips hosts without Docker
- Safe to run against mixed inventories (hosts with and without Docker)

### Running Playbooks

1. In Semaphore, create a new project pointing to this repository
2. Create a task using the desired playbook (`site.yml` or `portainer.yml`)
3. Select your target inventory (host group should be named `Hosts`)
4. Ensure environment variables are set in the Semaphore environment (if required by playbook)
5. Run the task

## Local Testing

If you want to test locally without Semaphore:

```bash
# Install required collections
ansible-galaxy collection install -r requirements.yml

# For SNMP playbook - set environment variables
export SNMP_USER="your_snmp_user"
export SNMP_PASS="your_snmp_password"

# Run the SNMP playbook
ansible-playbook -i your_inventory site.yml

# For Portainer playbook - set API token
export PORTAINER_API_TOKEN="your_portainer_api_token"

# Run the Portainer agent playbook (deploys agent and registers with Portainer)
ansible-playbook -i your_inventory portainer.yml
```

## Adding New Playbooks

When adding new playbooks:

1. Create the playbook YAML file in the root directory
2. Create any required roles in the `roles/` directory
3. Update this README with the new playbook information
4. Ensure proper secret management using environment variables

## Security

- All secrets are loaded from environment variables, never committed to the repository
- SNMP configuration files are created with restrictive permissions (0600)
- Use `no_log: true` for tasks handling sensitive data

## License

MIT
