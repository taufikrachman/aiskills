# Cloud Deployment (AWS/GCP/VPS)

Deploy Node.js/Next.js apps to production with Nginx, SSL, PM2.

## Rules

### 1. VPS Setup Checklist
```bash
# Initial server setup
apt update && apt upgrade -y
apt install -y nginx certbot python3-certbot-nginx

# Create deploy user
useradd -m -s /bin/bash deploy
mkdir -p /home/deploy/.ssh && chmod 700 /home/deploy/.ssh
cp ~/.ssh/authorized_keys /home/deploy/.ssh/ && chown -R deploy:deploy /home/deploy/.ssh

# Firewall
ufw allow 22/tcp     # SSH
ufw allow 80/tcp     # HTTP
ufw allow 443/tcp    # HTTPS
ufw enable
```

### 2. Nginx Reverse Proxy
```nginx
server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com;
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /api {
        proxy_pass http://localhost:4000;
        proxy_read_timeout 60s;
    }
}
```
- Redirect HTTP → HTTPS. Use Let's Encrypt for free SSL.
- Increase `proxy_read_timeout` for long requests (SSE, file upload).
- Rate limiting: `limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;`.

### 3. Process Manager (PM2)
```bash
pm2 start dist/main.js --name api --instances max
pm2 save
pm2 startup  # Auto-start on reboot
```
- `instances: max` = one process per CPU core.
- `watch: true` for dev, false for production.
- Logs: `pm2 logs api --lines 100`.

### 4. SSL & Domain
- Let's Encrypt via certbot: `certbot --nginx -d example.com`.
- Auto-renewal: certbot renews automatically. Verify: `certbot renew --dry-run`.
- Wildcard SSL: requires DNS challenge (not HTTP).

### 5. Environment Variables
```bash
# /home/deploy/.env
NODE_ENV=production
DATABASE_URL=postgres://user:pass@localhost:5432/db
JWT_SECRET=...
```
Never commit `.env` to git. Copy manually or use CI/CD secrets.

## Anti-Patterns
- ❌ Running app as root
- ❌ No firewall
- ❌ HTTP in production (no SSL)
- ❌ Hardcoded IPs instead of domain names
- ❌ No backup strategy for DB
