# Portainer Register Role

Automatically registers Docker hosts with Portainer server using the Portainer REST API.

## Description

This role:
- Authenticates with Portainer using an API token
- Checks if the environment already exists
- Registers the agent with Portainer if not already present
- Supports idempotent registration (skip if already exists)
- Configures environment name, tags, and group assignment

## Requirements

- Portainer server must be accessible from the Ansible control node
- A valid Portainer API access token
- The Portainer agent must be running on the target host (see `portainer_agent` role)
- Python `requests` library (usually included with Ansible)

**Note:** When using the `portainer.yml` playbook, hosts without Docker are automatically detected and gracefully skipped. This role will only run on hosts where Docker is available.

## Obtaining a Portainer API Token

You need to create an API access token in Portainer:

1. Log into your Portainer web interface at https://portainer.vixy.ai
2. Click on your **username** in the top right → **My account**
3. Scroll to **Access tokens** section
4. Click **+ Add access token**
5. Enter a description (e.g., "Ansible automation")
6. Click **Create access token**
7. **Copy the token immediately** - it won't be shown again!
8. Store it securely (you'll set it as `PORTAINER_API_TOKEN` environment variable)

## Role Variables

### Required (via environment variable)

| Variable | Environment Variable | Description |
|----------|---------------------|-------------|
| `portainer_api_token` | `PORTAINER_API_TOKEN` | Portainer API access token |

### Optional (with defaults)

| Variable | Default | Description |
|----------|---------|-------------|
| `portainer_url` | `https://portainer.vixy.ai` | Portainer server URL |
| `portainer_api_version` | `api` | API path prefix |
| `portainer_agent_port` | `9001` | Port where the agent is listening |
| `portainer_agent_use_tls` | `false` | Whether agent uses TLS |
| `portainer_environment_name` | `{{ inventory_hostname }}` | Name for the environment in Portainer |
| `portainer_agent_url` | Auto-detected | Agent connection URL (tcp://host:port) |
| `portainer_group_id` | `""` | Group ID to assign environment to |
| `portainer_tags` | `[]` | List of tag IDs to apply |
| `portainer_skip_if_exists` | `true` | Skip registration if environment exists |

## Dependencies

None.

## Example Playbook

### Basic Usage

```yaml
- name: Deploy and register Portainer agent
  hosts: docker_hosts
  become: true
  roles:
    - portainer_agent      # Deploy the agent first
    - portainer_register   # Then register it
```

### Custom Environment Names

```yaml
- name: Register with custom naming
  hosts: docker_hosts
  become: true
  vars:
    portainer_environment_name: "prod-{{ inventory_hostname }}"
  roles:
    - portainer_agent
    - portainer_register
```

### Assigning to Group and Tags

First, you need to find the Group ID and Tag IDs from Portainer:

```bash
# Get groups
curl -H "X-API-Key: YOUR_TOKEN" https://portainer.vixy.ai/api/endpoint_groups

# Get tags
curl -H "X-API-Key: YOUR_TOKEN" https://portainer.vixy.ai/api/tags
```

Then use them in your playbook:

```yaml
- name: Register with group and tags
  hosts: docker_hosts
  become: true
  vars:
    portainer_group_id: 1
    portainer_tags: [1, 2]  # Tag IDs from API
  roles:
    - portainer_agent
    - portainer_register
```

### Force Re-registration

```yaml
- name: Force registration even if exists
  hosts: docker_hosts
  become: true
  vars:
    portainer_skip_if_exists: false
  roles:
    - portainer_agent
    - portainer_register
```

## Usage with Semaphore

### Prerequisites

1. **Generate Portainer API token** (see instructions above)

2. **Add environment variable in Semaphore:**
   - Go to your Semaphore project → **Environment**
   - Add environment variable:
     - Name: `PORTAINER_API_TOKEN`
     - Value: `<your-api-token>`
     - Mark as **Secret**

3. **Update your playbook** to include the `portainer_register` role

### Running in Semaphore

1. Create/update task using `portainer.yml` playbook
2. Ensure `PORTAINER_API_TOKEN` is set in environment
3. Run the task
4. The agents will be automatically registered!

## Verification

After running the role, verify in Portainer:

1. Log into https://portainer.vixy.ai
2. Go to **Environments**
3. You should see your hosts listed with status "Up"

You can also verify via API:

```bash
curl -H "X-API-Key: YOUR_TOKEN" https://portainer.vixy.ai/api/endpoints
```

## Idempotency

The role is idempotent:
- First run: Registers the agent
- Subsequent runs: Detects agent already exists and skips registration
- Set `portainer_skip_if_exists: false` to force re-registration

## Troubleshooting

### API token not working

Check token is valid:
```bash
curl -H "X-API-Key: YOUR_TOKEN" https://portainer.vixy.ai/api/status
```

### Environment variable not set

Error: `PORTAINER_API_TOKEN environment variable must be set`

Solution: Ensure the environment variable is set in Semaphore or your shell:
```bash
export PORTAINER_API_TOKEN="your-token-here"
```

### SSL certificate issues

If you get SSL verification errors and your Portainer uses self-signed certificates, you can disable validation (not recommended for production):

Add to tasks/main.yml:
```yaml
validate_certs: false
```

### Agent not reachable

Error: Portainer can't connect to agent

Solutions:
- Verify agent container is running: `docker ps | grep portainer_agent`
- Check firewall allows port 9001
- Verify `portainer_agent_url` is correct
- Test connectivity: `curl http://localhost:9001`

### Using different IP address

By default, the role uses `ansible_default_ipv4.address`. To override:

```yaml
vars:
  portainer_agent_url: "tcp://192.168.1.100:9001"
```

## Security Considerations

- **API Token**: Treat as a password. Never commit to repository. Use Semaphore secrets or Ansible Vault.
- **Token Permissions**: Create a dedicated API token with minimal required permissions
- **Token Rotation**: Regularly rotate API tokens
- **HTTPS**: Always use HTTPS for Portainer server (already configured)
- **Network Security**: Ensure only Portainer server can reach agent port 9001

## Advanced Usage

### Using with Ansible Vault

Instead of environment variables, you can use Ansible Vault:

```yaml
# vault.yml (encrypted with ansible-vault)
portainer_api_token: "your-secret-token"
```

```yaml
# playbook
- name: Deploy with vault
  hosts: docker_hosts
  become: true
  vars_files:
    - vault.yml
  roles:
    - portainer_agent
    - portainer_register
```

### Dynamic Environment Names

```yaml
vars:
  portainer_environment_name: "{{ ansible_hostname }}-{{ ansible_distribution | lower }}"
```

### Conditional Registration

```yaml
- name: Only register production hosts
  hosts: docker_hosts
  become: true
  roles:
    - portainer_agent
    - role: portainer_register
      when: "'production' in group_names"
```

## API Reference

This role uses the Portainer API endpoints:

- **GET /api/endpoints** - List all environments
- **POST /api/endpoints** - Create new environment

For full API documentation, see:
https://docs.portainer.io/api/access

## License

MIT

## Author

Homelab Admin
