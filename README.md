# Home Assistant Docker Stack with Traefik

This Docker Compose stack sets up Home Assistant with Traefik reverse proxy support for both public and private domain access.

## Prerequisites

1. **Docker and Docker Compose** installed on your system
2. **External networks** must exist:
   - `iot` network for IoT devices
   - `traefik_public` network for Traefik connectivity

3. **Traefik** must be running with:
   - Entrypoints: `web` (HTTP) and `websecure` (HTTPS)
   - Certificate resolver: `letsencrypt`

## Setup Instructions

### 1. Configure Domains
Edit the `.env` file and update:
- `PUBLIC_DOMAIN` - Your public domain (e.g., homeassistant.yourdomain.com)
- `PRIVATE_DOMAIN` - Your internal domain (e.g., homeassistant.local)

### 2. Create External Networks
If the external networks don't exist, create them:

```bash
docker network create iot
docker network create traefik_public
```

### 3. Create Required Directories
```bash
mkdir config
mkdir ssl
```

### 4. Start the Stack
```bash
docker-compose up -d
```

## Configuration

### Traefik Labels Explained

**Public Domain (HTTPS):**
- Routes traffic from your public domain via HTTPS
- Uses Let's Encrypt for SSL certificates
- Includes security headers

**Private Domain (HTTP):**
- Routes traffic from your private/internal domain
- Uses HTTP for local network access

### Network Connectivity

- **iot network**: Allows Home Assistant to communicate with IoT devices
- **traefik_public network**: Enables Traefik to route traffic to Home Assistant

## Environment Variables

All configuration is managed through the `.env` file:

| Variable | Description | Default |
|----------|-------------|---------|
| `PREFIX` | Container/volume name prefix | `homeassistant` |
| `HOMEASSISTANT_VERSION` | Home Assistant image version | `latest` |
| `HOMEASSISTANT_PORT` | Internal Home Assistant port | `8123` |
| `EXTERNAL_NETWORK_IOT` | IoT network name | `iot` |
| `EXTERNAL_NETWORK_TRAEFIK` | Traefik network name | `traefik_public` |
| `PUBLIC_DOMAIN` | Public domain for HTTPS access | `homeassistant.example.com` |
| `PRIVATE_DOMAIN` | Private domain for HTTP access | `homeassistant.local` |
| `TRAEFIK_ENTRYPOINT_WEB` | Traefik HTTP entrypoint | `web` |
| `TRAEFIK_ENTRYPOINT_WEB_SECURE` | Traefik HTTPS entrypoint | `websecure` |
| `CONFIG_DIR` | Local config directory path | `./config` |
| `SSL_DIR` | Local SSL directory path | `./ssl` |

## Access Points

- **Public Access**: https://your-public-domain
- **Private Access**: http://your-private-domain:8123

## Troubleshooting

1. **Check container status**: `docker-compose ps`
2. **View logs**: `docker-compose logs homeassistant`
3. **Verify networks**: `docker network ls`
4. **Check Traefik configuration**: Ensure Traefik is properly configured with the required entrypoints and certificate resolver