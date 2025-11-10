# Homelab Ansible

Ansible playbooks and roles for homelab infrastructure management.

## Overview

This repository contains Ansible automation for configuring and maintaining homelab services. It is designed to work with [Semaphore](https://www.ansible-semaphore.com/), an Ansible web UI and automation platform.

## Structure

```
.
├── roles/              # Ansible roles
│   └── snmpd/         # SNMP daemon installation and configuration
├── site.yml           # Main playbook
└── ansible.cfg        # Ansible configuration
```

## Roles

### snmpd

Installs and configures the SNMP daemon for monitoring. Supports both SNMPv2c and SNMPv3.

See [roles/snmpd/README.md](roles/snmpd/README.md) for detailed documentation.

## Usage with Semaphore

### Prerequisites

Semaphore must be configured with the following environment variables for secrets:

- `SNMP_USER` - SNMPv3 username (required if SNMPv3 is enabled)
- `SNMP_PASS` - SNMPv3 authentication password (required if SNMPv3 is enabled)
- `SNMP_COMMUNITY` - SNMPv2c read-only community string (required if SNMPv2c is enabled)

### Running Playbooks

1. In Semaphore, create a new project pointing to this repository
2. Create a task using the `site.yml` playbook
3. Select your target inventory (host group should be named `Hosts`)
4. Ensure environment variables are set in the Semaphore environment
5. Run the task

## Local Testing

If you want to test locally without Semaphore:

```bash
# Set environment variables
export SNMP_USER="your_snmp_user"
export SNMP_PASS="your_snmp_password"

# Run the playbook
ansible-playbook -i your_inventory site.yml
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
