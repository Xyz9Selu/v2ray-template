# V2Ray Template

A template for quickly setting up V2Ray/XRay proxy service with web frontend.

## Project Structure

```
.
├── client/                 # Client-side configuration
│   ├── config.json        # Client configuration (from template)
│   └── docker-compose.yaml # Client docker compose file
├── server/                # Server-side configuration
│   ├── config.json        # Server configuration (from template)
│   └── docker-compose.yaml # Server docker compose file
├── web/                   # Web frontend (DokuWiki)
│   └── docker-compose.yaml # Web docker compose file
└── nginx/                 # Nginx configuration
    └── sites-enabled/     # Nginx site configurations
        ├── dokuwiki      # DokuWiki site configuration
        └── v2ray         # V2Ray proxy configuration
```

## Setup Instructions

### Server Side

1. Copy `server/config.json.template` to `server/config.json`
2. Update the configuration with your settings:
   - Replace `<your-user-id>` with your UUID
   - Replace `<your-vless-server-host>` with your domain
   - Replace `<your-vless-server-path>` with your WebSocket path

3. Start the server:
```bash
cd server
docker-compose up -d
```

### Client Side

1. Copy `client/config.json.template` to `client/config.json`
2. Update the configuration with the same settings as server:
   - Replace `<your-user-id>` with your UUID
   - Replace `<your-vless-server-host>` with your domain
   - Replace `<your-vless-server-path>` with your WebSocket path

3. Start the client:
```bash
cd client
docker-compose up -d
```

### Web Frontend

1. Update the environment variables in `web/docker-compose.yaml`:
   - DOKUWIKI_USERNAME
   - DOKUWIKI_PASSWORD
   - DOKUWIKI_FULL_NAME
   - DOKUWIKI_WIKI_NAME
   - DOKUWIKI_EMAIL

2. Start the web service:
```bash
cd web
docker-compose up -d
```

### Nginx Setup

1. Install Nginx if not already installed:
```bash
sudo apt update
sudo apt install nginx
```

2. Copy the Nginx configuration file:
```bash
sudo cp nginx/sites-enabled/v2ray /etc/nginx/sites-enabled/
```

3. Update the configuration file with your domain:
   - In `nginx/sites-enabled/v2ray`:
     - Replace `<your-vless-server-host>` with your domain
     - Replace `<your-vless-server-path>` with your WebSocket path
     - The default configuration includes:
       - Port 80 for HTTP
       - Proxy pass to DokuWiki on port 13080
       - WebSocket proxy for V2Ray on port 33685

4. Test and reload Nginx:
```bash
sudo nginx -t
sudo systemctl reload nginx
```

### SSL Certificate Setup with Certbot

1. Install Certbot and Nginx plugin:
```bash
sudo apt install certbot python3-certbot-nginx
```

2. Obtain SSL certificate:
```bash
sudo certbot --nginx -d your-domain.com
```

3. Certbot will automatically:
   - Obtain SSL certificate
   - Update Nginx configuration
   - Set up auto-renewal

4. Verify auto-renewal:
```bash
sudo systemctl status certbot.timer
sudo certbot renew --dry-run
```

### VLESS URL Format

After setting up the server, you can generate a VLESS URL for clients to import:

```
vless://<user-id>@<domain>:443?encryption=none&security=tls&type=ws&host=<domain>&path=/<ws-path>#<remarks>
```

Example:
```
vless://41c5b71b-4797-425b-8c27-fc1d47453c27@example.com:443?encryption=none&security=tls&type=ws&host=example.com&path=/websocket#MyProxy
```

Parameters explained:
- `<user-id>`: Your UUID from config.json
- `<domain>`: Your server domain
- `<ws-path>`: WebSocket path from config.json
- `<remarks>`: Optional remarks for this connection

## Notes

- All services are configured to restart automatically unless stopped manually
- Make sure to keep your configuration files secure and never commit them to version control
- The web frontend is optional and can be disabled if not needed
- SSL certificates will auto-renew if Certbot is properly configured
- Remember to update your firewall settings to allow HTTPS traffic (port 443)
