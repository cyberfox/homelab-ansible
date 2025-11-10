# Portainer Agent Role

Deploys the Portainer agent container on Docker hosts to enable management via Portainer server.

## Description

This role:
- Verifies Docker is installed and running
- Installs Python Docker library (required for Ansible Docker module)
- Deploys the Portainer agent container
- Configures the agent to expose port 9001
- Mounts Docker socket and volumes directory
- Ensures the container restarts automatically

## Requirements

- Docker must be installed and running on target hosts
- Ansible collection: `community.docker`
- Root/sudo access on target hosts

**Note:** When using the `portainer.yml` playbook, hosts without Docker are automatically detected and gracefully skipped. The role will only run on hosts where Docker is available.

### Installing Required Collections

```bash
ansible-galaxy collection install community.docker
```

Or add to `requirements.yml`:

```yaml
collections:
  - name: community.docker
```

## Role Variables

### Optional (with defaults)

| Variable | Default | Description |
|----------|---------|-------------|
| `portainer_agent_image` | `portainer/agent:latest` | Docker image for Portainer agent |
| `portainer_agent_container_name` | `portainer_agent` | Name of the Docker container |
| `portainer_agent_port` | `9001` | Host port to expose the agent on |
| `portainer_agent_restart_policy` | `always` | Container restart policy |
| `portainer_agent_state` | `started` | Desired container state (started/stopped) |
| `portainer_agent_volumes` | See defaults/main.yml | List of volume mounts |

## Dependencies

This role requires the `community.docker` Ansible collection.

## Example Playbook

```yaml
- name: Deploy Portainer agent on Docker hosts
  hosts: docker_hosts
  become: true
  roles:
    - portainer_agent
```

### Custom Port Configuration

```yaml
- name: Deploy Portainer agent on custom port
  hosts: docker_hosts
  become: true
  vars:
    portainer_agent_port: 9002
  roles:
    - portainer_agent
```

### Using Specific Agent Version

```yaml
- name: Deploy specific Portainer agent version
  hosts: docker_hosts
  become: true
  vars:
    portainer_agent_image: "portainer/agent:2.19.4"
  roles:
    - portainer_agent
```

## Post-Deployment

After running this role, you need to manually add the agents to your Portainer server:

1. Log into your Portainer web interface
2. Go to **Environments** → **Add environment**
3. Select **Agent** as the environment type
4. Enter the agent URL: `http://<host-ip>:9001`
5. Click **Connect**

### Future Automation

The Portainer API can be used to automate adding agents:

```bash
# Example API call to add an environment (requires API token)
curl -X POST "https://portainer.example.com/api/endpoints" \
  -H "X-API-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "Name": "docker-host-1",
    "EndpointCreationType": 1,
    "URL": "tcp://192.168.1.100:9001",
    "TLS": false
  }'
```

This could be implemented in a future role or playbook.

## Verification

To verify the agent is running:

```bash
# Check container status
docker ps | grep portainer_agent

# Check agent is listening
netstat -tlnp | grep 9001

# Test agent endpoint
curl http://localhost:9001
```

## Security Considerations

- The agent has access to the Docker socket (full Docker API access)
- Port 9001 should only be accessible from your Portainer server
- Consider using firewall rules to restrict access:
  ```bash
  # Example: Allow only from Portainer server
  ufw allow from <portainer-server-ip> to any port 9001
  ```
- Consider using TLS for agent communication in production

## Troubleshooting

### Container not starting

Check Docker logs:
```bash
docker logs portainer_agent
```

### Python Docker library missing

The role should install it automatically, but you can install manually:
```bash
# Debian/Ubuntu
apt-get install python3-docker

# RHEL/CentOS
dnf install python3-docker
```

### Port already in use

Change the port in your playbook:
```yaml
vars:
  portainer_agent_port: 9002
```

## License

MIT

## Author

Homelab Admin
