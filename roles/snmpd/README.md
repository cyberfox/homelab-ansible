# SNMPD Role

Installs and configures the SNMP daemon (snmpd) for network monitoring. Supports both SNMPv2c and SNMPv3 protocols.

## Description

This role:
- Installs the SNMP daemon package (OS-specific)
- Configures SNMPv2c with read-only community access (optional)
- Configures SNMPv3 with authentication (optional)
- Manages the snmpd service

## Requirements

- Target system must be Debian/Ubuntu or RHEL/CentOS/Rocky
- Root/sudo access on target hosts

## Role Variables

### Required (when protocols are enabled)

These must be provided via environment variables in Semaphore:

| Variable | Environment Variable | Description | Required When |
|----------|---------------------|-------------|---------------|
| `snmp_user` | `SNMP_USER` | SNMPv3 username | `snmp_enable_v3` is true |
| `snmp_pass` | `SNMP_PASS` | SNMPv3 authentication password | `snmp_enable_v3` is true |
| `snmp_ro_community` | `SNMP_COMMUNITY` | SNMPv2c read-only community string | `snmp_enable_v2c` is true |

### Optional (with defaults)

| Variable | Default | Description |
|----------|---------|-------------|
| `snmpd_pkg` | `snmpd` (Debian) or `net-snmp` (RHEL) | Package name for SNMP daemon |
| `snmp_agent_address` | `udp:0.0.0.0:161` | Address and port for SNMP agent to listen on |
| `snmp_syslocation` | `Homelab` | System location field in SNMP |
| `snmp_syscontact` | `ops@example.com` | System contact field in SNMP |
| `snmp_enable_v2c` | `false` | Enable SNMPv2c protocol |
| `snmp_enable_v3` | `true` | Enable SNMPv3 protocol |

## Dependencies

None.

## Example Playbook

```yaml
- name: Configure SNMP monitoring
  hosts: monitoring_targets
  become: true
  roles:
    - snmpd
```

### Enabling SNMPv2c

Set in your playbook or inventory:

```yaml
- name: Configure SNMP with v2c
  hosts: monitoring_targets
  become: true
  vars:
    snmp_enable_v2c: true
    snmp_enable_v3: false
  roles:
    - snmpd
```

Ensure `SNMP_COMMUNITY` environment variable is set in Semaphore.

### Enabling Both Protocols

```yaml
- name: Configure SNMP with both protocols
  hosts: monitoring_targets
  become: true
  vars:
    snmp_enable_v2c: true
    snmp_enable_v3: true
  roles:
    - snmpd
```

Ensure all three environment variables (`SNMP_USER`, `SNMP_PASS`, `SNMP_COMMUNITY`) are set.

## Security

- SNMPv3 uses SHA-512 for authentication
- Configuration file is created with 0600 permissions (root read/write only)
- Secrets are never logged (`no_log: true`)
- SNMPv2c has limited view access (systemonly)
- SNMPv3 has full MIB tree access with authentication

## Testing

To verify SNMP is working:

```bash
# SNMPv3
snmpwalk -v3 -u <username> -l authNoPriv -a SHA-512 -A <password> localhost

# SNMPv2c
snmpwalk -v2c -c <community> localhost
```

## License

MIT

## Author

Homelab Admin
