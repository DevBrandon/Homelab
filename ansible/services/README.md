# Services

This directory contains Docker Compose service definitions for the homelab.

## Available Services

- **portainer** - Container management UI (port 9000)
- **monitoring** - Prometheus + Node Exporter (port 9090)
- **homeassistant** - Home automation platform (port 8123)
- **pihole** - DNS ad-blocker (ports 53, 80, 443)
- **heimdall** - Application dashboard (ports 8080, 8443)
- **wireguard** - VPN server with UI (ports 51820/udp, 5000)

## Configuration

Services are deployed based on the `services_to_deploy` variable in group_vars.

### Example: Deploy all services to infra group

Edit `inventories/prod/group_vars/infra.yml`:

```yaml
services_to_deploy:
  - portainer
  - monitoring
  - homeassistant
  - pihole
  - heimdall
  - wireguard
```

### Example: Deploy specific services to apps group

Edit or create `inventories/prod/group_vars/apps.yml`:

```yaml
services_to_deploy:
  - portainer
  - homeassistant
```

## Service-Specific Configuration

### PiHole

Set the web password via environment variable or `.env` file:
- Default: empty (no password)
- Recommended: Set `WEBPASSWORD` in a `.env` file

### WireGuard

Update these environment variables:
- `SERVERURL` - Your public domain/IP
- `PEERS` - Number of client configs to generate
- `WGUI_USERNAME` / `WGUI_PASSWORD` - UI credentials

### HomeAssistant

Runs with privileged mode for hardware access. Configure via web UI at port 8123.

## Adding New Services

1. Create a new directory: `services/your-service/`
2. Add `docker-compose.yml` in that directory
3. Add the service name to `services_to_deploy` in group_vars
4. Run `ansible-playbook site.yml`

## Service Management

Services are deployed to `/opt/services/` on each host. You can manage them directly:

```bash
# SSH to host
ssh pi@pi-1

# Navigate to service
cd /opt/services/portainer

# View logs
docker compose logs -f

# Restart service
docker compose restart

# Update service
docker compose pull
docker compose up -d
```
