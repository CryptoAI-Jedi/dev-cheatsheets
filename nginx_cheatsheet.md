# nginx Cheatsheet

> Web server, reverse proxy, load balancer, and access-control layer. On a VPS it's the front door: TLS terminates here, rate limits live here, and internal dashboards hide behind it.

---

## Table of Contents
- [Service Management](#service-management)
- [File Locations (Debian/Ubuntu)](#file-locations-debianubuntu)
- [nginx.conf Structure](#nginxconf-structure)
- [Static Site (server block)](#static-site-server-block)
- [Reverse Proxy](#reverse-proxy)
- [Load Balancer](#load-balancer)
- [SSL / HTTPS (Let's Encrypt)](#ssl--https-lets-encrypt)
- [Location Blocks](#location-blocks)
- [Access Control (allow / deny)](#access-control-allow--deny)
- [Rate Limiting](#rate-limiting)
- [Basic Auth](#basic-auth)
- [Security Headers](#security-headers)
- [Common Directives](#common-directives)
- [Logs & Debugging](#logs--debugging)
- [Tips & Gotchas](#tips--gotchas)

---

## SERVICE MANAGEMENT

```bash
sudo systemctl start nginx      # Start
sudo systemctl stop nginx       # Stop
sudo systemctl restart nginx    # Full restart (brief downtime)
sudo systemctl reload nginx     # Reload config (zero downtime)
systemctl status nginx          # Check status
sudo systemctl enable nginx     # Start on boot
```

```bash
sudo nginx -t                   # Test config syntax — ALWAYS before reload
sudo nginx -T                   # Test + print full merged config
sudo nginx -s reload            # Send reload signal to master
sudo nginx -s stop              # Fast stop
nginx -v                        # Version
nginx -V                        # Version + compiled modules
```

---

## FILE LOCATIONS (Debian/Ubuntu)

```text
/etc/nginx/nginx.conf          → Main config
/etc/nginx/sites-available/    → Available virtual hosts
/etc/nginx/sites-enabled/      → Active virtual hosts (symlinked)
/etc/nginx/conf.d/             → Drop-in config files
/var/log/nginx/access.log      → Access log
/var/log/nginx/error.log       → Error log
/var/www/html/                 → Default web root
/run/nginx.pid                 → PID file
```

### Enable a site

```bash
sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

---

## NGINX.CONF STRUCTURE

```nginx
user www-data;
worker_processes auto;              # Match CPU cores
pid /run/nginx.pid;

events {
    worker_connections 1024;        # Max connections per worker
}

http {
    sendfile on;
    tcp_nopush on;
    keepalive_timeout 65;
    gzip on;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log warn;

    include /etc/nginx/sites-enabled/*;
}
```

---

## STATIC SITE (server block)

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/example;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

## REVERSE PROXY

```nginx
# In http {} context — proper WebSocket handling (close when no upgrade):
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;   # via map above
    }
}
```

```text
⚠️ proxy_pass trailing slash changes behavior — see Tips & Gotchas.
```

---

## LOAD BALANCER

```nginx
upstream backend {
    server 10.0.0.1:8080;
    server 10.0.0.2:8080 max_fails=3 fail_timeout=30s;   # Eject after 3 failures
    server 10.0.0.3:8080 backup;                         # Used only if others fail
}

upstream backend_weighted {
    server 10.0.0.1:8080 weight=3;    # Gets 3x more traffic
    server 10.0.0.2:8080 weight=1;
}

upstream backend_ip_hash {
    ip_hash;                          # Sticky sessions by client IP
    server 10.0.0.1:8080;
    server 10.0.0.2:8080;
}

upstream backend_least {
    least_conn;                       # Route to least-busy backend
    server 10.0.0.1:8080;
    server 10.0.0.2:8080;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

---

## SSL / HTTPS (Let's Encrypt)

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d example.com -d www.example.com
sudo certbot renew --dry-run             # Test auto-renewal
sudo certbot renew                       # Manual renew
systemctl list-timers | grep certbot     # Auto-renewal timer (installed by certbot)
```

### Manual SSL config

```nginx
server {
    listen 443 ssl http2;            # nginx ≥1.25: use `http2 on;` on its own line
    server_name example.com;

    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_session_cache shared:SSL:10m;    # Reuse TLS handshakes

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Redirect HTTP → HTTPS

```nginx
server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}
```

---

## LOCATION BLOCKS

```nginx
location / { }                    # Match all (catch-all)
location = /exact { }             # Exact match (highest priority)
location ^~ /static/ { }          # Prefix match (stop regex search)
location ~ \.php$ { }             # Case-sensitive regex
location ~* \.(jpg|png|gif)$ { }  # Case-insensitive regex
```

```text
Priority order:  =  >  ^~  >  ~ and ~* (first match in file order)  >  plain prefix
```

---

## ACCESS CONTROL (allow / deny)

Contain internal services — dashboard reachable only from the tailnet/localhost:

```nginx
location /dashboard/ {
    allow 100.64.0.0/10;      # Tailscale CGNAT range
    allow 127.0.0.1;
    deny all;                 # Everyone else → 403

    proxy_pass http://127.0.0.1:8501;
}
```

### Combine IP allowlist with basic auth

```nginx
location /admin/ {
    satisfy any;              # Pass EITHER check (default `all` requires BOTH)
    allow 100.64.0.0/10;
    deny all;
    auth_basic "Restricted";
    auth_basic_user_file /etc/nginx/.htpasswd;
}
```

---

## RATE LIMITING

```nginx
http {
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;   # 10m ≈ 160k IPs
}

server {
    location /api/ {
        limit_req zone=api burst=20 nodelay;   # Absorb bursts of 20, no queuing delay
        limit_req_status 429;                  # Default is 503 — 429 is honest
    }
}
```

---

## BASIC AUTH

```bash
sudo apt install apache2-utils
sudo htpasswd -c /etc/nginx/.htpasswd alice   # -c CREATES/OVERWRITES the file — first user only
sudo htpasswd /etc/nginx/.htpasswd bob        # Add users WITHOUT -c
```

```nginx
server {
    location /admin/ {
        auth_basic "Restricted";
        auth_basic_user_file /etc/nginx/.htpasswd;
    }
}
```

---

## SECURITY HEADERS

```nginx
add_header X-Frame-Options "SAMEORIGIN";
add_header X-Content-Type-Options "nosniff";
add_header Referrer-Policy "strict-origin-when-cross-origin";
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
add_header X-XSS-Protection "1; mode=block";   # Legacy — modern browsers ignore it; harmless
server_tokens off;                             # Hide nginx version in headers/errors
```

```text
⚠️ add_header does NOT inherit if a child block adds its own — see Gotchas.
```

---

## COMMON DIRECTIVES

```nginx
root /var/www/html;                       # Document root
index index.html index.php;               # Default files
try_files $uri $uri/ /index.html;         # SPA fallback
return 301 https://$host$request_uri;     # Redirect
rewrite ^/old(.*)$ /new$1 permanent;      # Rewrite URL
error_page 404 /404.html;
error_page 500 502 503 504 /50x.html;
client_max_body_size 20m;                 # Upload limit (default 1m → 413 errors)
```

### Timeouts

```nginx
proxy_connect_timeout 60s;
proxy_send_timeout 60s;
proxy_read_timeout 60s;      # Raise for long-polling / streaming backends
```

### Buffer sizes

```nginx
proxy_buffer_size 128k;
proxy_buffers 4 256k;
```

### File serving

```nginx
expires 30d;                          # Cache static assets 30 days
add_header Cache-Control "public";
gzip on;
gzip_types text/plain text/css application/json application/javascript;
gzip_min_length 1000;
```

---

## LOGS & DEBUGGING

```bash
tail -f /var/log/nginx/access.log        # Live access log
tail -f /var/log/nginx/error.log         # Live error log — read THIS first on 5xx
grep "error" /var/log/nginx/error.log
```

```bash
# HTTP status code distribution:
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn
# Top client IPs:
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head
# Top requested paths:
awk '{print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head
```

---

## TIPS & GOTCHAS

- **`proxy_pass` trailing slash is a behavior switch** — `proxy_pass http://127.0.0.1:3000;` (no slash) forwards the FULL request path; `proxy_pass http://127.0.0.1:3000/;` (slash) STRIPS the matched location prefix first. The #1 source of mysterious 404s behind a proxy.
- **`add_header` inheritance trap** — a single `add_header` inside a `location` block silently discards ALL `add_header` lines inherited from `server`/`http`. Repeat your security headers in any block that adds its own, and use `always` so they apply to error responses too.
- **413 Request Entity Too Large** — default `client_max_body_size` is 1m. Any upload endpoint needs it raised (server or location level).
- **First server block is the accidental default** — nginx routes unmatched Host headers to the first `server` in config order. Pin an explicit catch-all: `listen 80 default_server;` with `return 444;` (drop connection) so bots scanning your IP never hit a real vhost.
- **`nginx -t` passes ≠ site works** — it validates syntax, not logic. It won't catch a wrong `proxy_pass` port or missing upstream. Check `error.log` after reload.
- **Symlink sites with absolute paths** — `ln -s ../sites-available/x` style relative links break depending on cwd; always use full `/etc/nginx/sites-available/...` paths.
- **Avoid `if` inside `location`** — famously buggy ("if is evil"). For redirects use `return`; for conditionals use `map` in the `http` context.
- **502 Bad Gateway checklist** — backend actually listening? (`ss -tlnp`) Right port in `proxy_pass`? Backend bound to 127.0.0.1 vs 0.0.0.0? SELinux/AppArmor denials in `error.log`?

---
*Last Updated: 2026-07*
