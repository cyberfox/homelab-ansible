# NetData Role

Installs [NetData](https://www.netdata.cloud/) for real-time infrastructure monitoring.

## Architecture

```
All child hosts ──stream──> monitoring.yiff.org (parent) ──> NetData Cloud
```

- **Parent node** accepts metric streams from all children and syncs to NetData Cloud
- **Child nodes** install NetData locally and stream metrics to the parent (no cloud connection)

## Configuration

All secrets are passed via environment variables (following repo conventions).

### Required environment variables

| Variable | Description |
|---|---|
| `NETDATA_CLAIM_TOKEN` | Cloud claim token (parent only) |
| `NETDATA_CLAIM_ROOMS` | Cloud room IDs (parent only) |
| `NETDATA_STREAM_API_KEY` | Shared API key for parent-child streaming |

### Optional overrides

```yaml
netdata_bind_to: "0.0.0.0:19999"       # Listening address
netdata_memory_mode: "ram"              # none, ram, save, max
netdata_parent_host: "monitoring.yiff.org"  # Parent address (children)
netdata_parent_port: 19999              # Parent port (children)
netdata_auto_updates: true              # Enable daily auto-updates
```

## Usage

### 1. Set environment variables

```bash
export NETDATA_CLAIM_TOKEN="your-claim-token"
export NETDATA_CLAIM_ROOMS="room-id-1,room-id-2"
export NETDATA_STREAM_API_KEY="your-streaming-api-key"
```

### 2. Run the playbook

```bash
ansible-playbook site.yml -i inventory.yml --private-key ~/.ssh/ansible_ed25519
```

### 3. Verify

```bash
# Check parent is accepting streams
ansible netdata_parent -i inventory.yml --private-key ~/.ssh/ansible_ed25519 \
  -m shell -a "netdatacli stream-status"

# Check child is connected
ansible netdata_children -i inventory.yml --private-key ~/.ssh/ansible_ed25519 \
  -m shell -a "netdatacli stream-status"
```

## Inventory groups

Your inventory should define these groups:

```yaml
all:
  children:
    netdata_parent:
      hosts:
        monitoring.yiff.org:
    netdata_children:
      hosts:
        proxmox-1.yiff.org:
        # ... all other hosts
```
