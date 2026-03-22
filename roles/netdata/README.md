# NetData Role

Installs [NetData](https://www.netdata.cloud/) for real-time infrastructure monitoring.

## What it does

- Installs NetData via the official kickstart script (stable channel)
- Configures bind address, memory mode, and cloud settings
- Optionally configures streaming to a parent/central node
- Opens firewall port 19999 (if ufw is present)

## Configuration

Override defaults in `host_vars` or `group_vars`:

```yaml
# NetData listening address
netdata_bind_to: "0.0.0.0:19999"

# Memory mode for smaller hosts
netdata_memory_mode: "ram"  # none, ram, save, max

# Disable NetData cloud (default: true for self-hosted)
netdata_disable_cloud: true

# Stream metrics to a central node (optional)
netdata_enable_streaming: true
netdata_parent_host: "monitoring.yiff.org"
netdata_parent_port: 19999
netdata_parent_api_key: "your-api-key-here"
```

## Usage

Add to `site.yml`:

```yaml
- name: Install NetData monitoring
  hosts: all
  become: true
  roles:
    - netdata
```

Or target specific groups:

```yaml
- name: Install NetData on ProxMox hosts
  hosts: proxmox
  become: true
  roles:
    - netdata
```
