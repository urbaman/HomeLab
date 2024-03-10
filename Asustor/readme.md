# Docker environment setup

## To Do

- [x] Create new domain urbaman.cloud
- [x] Setup the domain on cloudflare
- [x] Create directory paths
- [ ] Install Traefik (uninstall nginx)
- [ ] Try to proxy both Asustor itself and Portainer
- [ ] Install all other apps

## Cloudflare domain setup

- Create an A record, and a CNAME for every service to proxy
- Use Full (strict) if you have proper Letsencrypt certs on the server (do it)
- SSL:
  - Always Use HTTPS: ON
  - HTTP Strict Transport Security (HSTS): Enable
  - Minimum TLS Version: 1.2
  - Opportunistic Encryption: ON
  - TLS 1.3: ON
  - Automatic HTTPS Rewrites: ON
  - Certificate Transparency Monitoring: ON
- Firewall/WAF
  - Security Level: High
  - Bot Fight Mode: ON
  - Challenge Passage: 30 Minutes
  - Browser Integrity Check: ON
- Speed (check each setting for your apps)
  - Brotli: ON
  - Cloudflare Fonts: ON
  - Early Hints: ON
  - Rocket Loader: ON
  - Minification: ON for all
- Cache
  - Standard level
  - TTL: 1h

## Directories

### Docker compose

- \volume1\home\nasadmin\docker-compose
  - \volume1\home\nasadmin\docker-compose\services
  - \volume1\home\nasadmin\docker-compose\secrets (root:root, 600)
  - \volume1\home\nasadmin\docker-compose\.env (root:root, 600)
  - \volume1\home\nasadmin\docker-compose\...

### Docker containers data

- \volume1\Docker
  - \volume1\Docker\configs
    - \volume1\Docker\configs\app1
    - \volume1\Docker\configs\app2
    - \volume1\Docker\configs\...
  - \volume1\Docker\data
    - \volume1\Docker\data\app1
    - \volume1\Docker\data\app2
    - \volume1\Docker\data\...
  - \volume1\Docker\logs
    - \volume1\Docker\logs\app1
    - \volume1\Docker\logs\app2
    - \volume1\Docker\logs\...
