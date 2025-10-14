# Home Assistant Docker Stack with Traefik

This Docker Compose stack sets up Home Assistant with Traefik reverse proxy support for both public and private domain access, plus optional Cloudflared tunnel and Matter Server for Matter device support.

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
- **Matter Server**: Accessible on port 5580 for Home Assistant Matter integration

## Matter Server Integration

The stack includes a Matter Server that allows Home Assistant to connect to and manage Matter devices.

### Setup in Home Assistant:
1. **Access Home Assistant** via your domain
2. **Go to Settings** → **Devices & Services**
3. **Add Integration** → Search for "Matter (BETA)"
4. **Server URL**: Use `matter-server:5580` (internal Docker network)
5. **Complete setup** to start adding Matter devices

### Matter Server Features:
- **Automatic Discovery**: Discovers Matter devices on your network
- **Thread Support**: Full Thread network support for Matter devices
- **Persistent Storage**: Matter data stored in Docker volume
- **Integration Ready**: Pre-configured to work with Home Assistant

### Matter Device Setup:
1. **Commission devices** through Home Assistant's Matter integration
2. **Use Thread**: Enable Thread border router in Home Assistant if needed
3. **Device Control**: All Matter devices will appear in Home Assistant automatically

## Reverse Proxy Configuration

After starting the stack, you'll need to configure Home Assistant to work properly with Traefik as a reverse proxy:

### Method 1: Environment Variables (Automatic)
The Docker Compose stack includes environment variables that should automatically configure Home Assistant for reverse proxy usage:
- `HASS_HTTP_TRUSTED_PROXY_*` - Trusts Docker and private networks
- `HASS_HTTP_USE_X_FORWARDED_FOR=true` - Enables forwarded header support

### Method 2: Configuration File (Manual)
If you see reverse proxy errors in the logs, add this to your Home Assistant `configuration.yaml`:

```yaml
# HTTP Configuration for Reverse Proxy (Traefik)
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 172.16.0.0/12    # Docker networks
    - 192.168.0.0/16   # Private networks
    - 10.0.0.0/8       # Private networks
    - 127.0.0.1        # Localhost
    - ::1              # IPv6 localhost
```

### How to Edit Configuration:
1. **Access Home Assistant** via your domain
2. **Go to Settings** → **Add-ons** → **File editor** (install if needed)
3. **Edit** `configuration.yaml` and add the HTTP section above
4. **Restart** Home Assistant

### Common Reverse Proxy Errors:
- `"A request from a reverse proxy was received from X.X.X.X, but your HTTP integration is not set-up for reverse proxies"`
- **Solution**: Add the HTTP configuration above to trust your Traefik proxy

## Troubleshooting

1. **Check container status**: `docker-compose ps`
2. **View logs**: `docker-compose logs homeassistant`
3. **Verify networks**: `docker network ls`
4. **Check Traefik configuration**: Ensure Traefik is properly configured with the required entrypoints and certificate resolver
5. **Reverse proxy errors**: See "Reverse Proxy Configuration" section above