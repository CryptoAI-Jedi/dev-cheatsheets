# nginx_cheatsheet

# SERVICE MANAGEMENT

---

systemctl start nginx              → Start nginx
systemctl stop nginx               → Stop nginx
systemctl restart nginx            → Full restart (brief downtime)
systemctl reload nginx             → Reload config (zero downtime)
systemctl status nginx             → Check status
systemctl enable nginx             → Start on boot
nginx -t                           → Test config syntax
nginx -T                           → Test & print full merged config
nginx -s reload                    → Send reload signal to master
nginx -s stop                      → Fast stop
nginx -v                           → Nginx version
nginx -V                           → Version + compiled modules

---

## FILE LOCATIONS (Debian/Ubuntu)

/etc/nginx/nginx.conf              → Main config
/etc/nginx/sites-available/        → Available virtual hosts
/etc/nginx/sites-enabled/          → Active virtual hosts (symlinked)
/etc/nginx/conf.d/                 → Drop-in config files
/var/log/nginx/access.log          → Access log
/var/log/nginx/error.log           → Error log
/var/www/html/                     → Default web root
/run/nginx.pid                     → PID file

# Enable a site

ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx

---

## NGINX.CONF STRUCTURE

user www-data;
worker_processes auto;              → Match CPU cores
pid /run/nginx.pid;

events {
worker_connections 1024;          → Max connections per worker
}

http {
sendfile on;
tcp_nopush on;
keepalive_timeout 65;
gzip on;

include /etc/nginx/mime.types;
default_type application/octet-stream;

access_log /var/log/nginx/access.log;
error_log  /var/log/nginx/error.log warn;

include /etc/nginx/sites-enabled/*;
}

---

## STATIC SITE (server block)

server {
listen 80;
server_name [example.com](http://example.com/) [www.example.com](http://www.example.com/);
root /var/www/example;
index index.html;

location / {
try_files $uri $uri/ =404;
}
}

---

## REVERSE PROXY

server {
listen 80;
server_name [api.example.com](http://api.example.com/);

location / {
proxy_pass [http://127.0.0.1:3000](http://127.0.0.1:3000/);
proxy_http_version 1.1;
proxy_set_header Host              $host;
proxy_set_header X-Real-IP         $remote_addr;
proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header Upgrade           $http_upgrade;
proxy_set_header Connection        "upgrade";  → For WebSockets
}
}

---

## LOAD BALANCER

upstream backend {
server 10.0.0.1:8080;
server 10.0.0.2:8080;
server 10.0.0.3:8080 backup;     → Used only if others fail
}

upstream backend_weighted {
server 10.0.0.1:8080 weight=3;   → Gets 3x more traffic
server 10.0.0.2:8080 weight=1;
}

upstream backend_ip_hash {
ip_hash;                          → Sticky sessions
server 10.0.0.1:8080;
server 10.0.0.2:8080;
}

server {
listen 80;
location / {
proxy_pass [http://backend](http://backend/);
}
}

---

## SSL / HTTPS (Let's Encrypt)

apt install certbot python3-certbot-nginx
certbot --nginx -d [example.com](http://example.com/) -d [www.example.com](http://www.example.com/)
certbot renew --dry-run              → Test auto-renewal
certbot renew                        → Manual renew

# Auto-renewal (added by certbot)

# /etc/cron.d/certbot or systemd timer

# Manual SSL config

server {
listen 443 ssl;
server_name [example.com](http://example.com/);

ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
ssl_protocols       TLSv1.2 TLSv1.3;
ssl_ciphers         HIGH:!aNULL:!MD5;

location / {
try_files $uri $uri/ =404;
}
}

# Redirect HTTP → HTTPS

server {
listen 80;
server_name [example.com](http://example.com/);
return 301 https://$host$request_uri;
}

---

## LOCATION BLOCKS

location / { }                     → Match all (catch-all)
location = /exact { }              → Exact match (highest priority)
location ^~ /static/ { }          → Prefix match (stop regex search)
location ~ \.php$ { }             → Case-sensitive regex
location ~* \.(jpg|png|gif)$ { }  → Case-insensitive regex

# Priority order: = > ^~ > ~ and ~* > /

---

## COMMON DIRECTIVES

root /var/www/html;                → Document root
index index.html index.php;       → Default files
try_files $uri $uri/ /index.html; → SPA fallback
return 301 https://$host$request_uri;  → Redirect
rewrite ^/old(.*)$ /new$1 permanent;   → Rewrite URL
error_page 404 /404.html;
error_page 500 502 503 504 /50x.html;

# Timeouts

proxy_connect_timeout 60s;
proxy_send_timeout 60s;
proxy_read_timeout 60s;

# Buffer sizes

proxy_buffer_size 128k;
proxy_buffers 4 256k;

# File serving

expires 30d;                       → Cache static assets 30 days
add_header Cache-Control "public"; → Cache-Control header
gzip on;
gzip_types text/plain text/css application/json application/javascript;
gzip_min_length 1000;

---

## SECURITY HEADERS

add_header X-Frame-Options "SAMEORIGIN";
add_header X-XSS-Protection "1; mode=block";
add_header X-Content-Type-Options "nosniff";
add_header Referrer-Policy "strict-origin-when-cross-origin";
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
server_tokens off;                 → Hide nginx version

---

## RATE LIMITING

http {
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
}

server {
location /api/ {
limit_req zone=api burst=20 nodelay;
}
}

---

## BASIC AUTH

apt install apache2-utils
htpasswd -c /etc/nginx/.htpasswd alice     → Create file + user
htpasswd /etc/nginx/.htpasswd bob          → Add user to file

server {
location /admin/ {
auth_basic "Restricted";
auth_basic_user_file /etc/nginx/.htpasswd;
}
}

---

## LOGS & DEBUGGING

tail -f /var/log/nginx/access.log          → Live access log
tail -f /var/log/nginx/error.log           → Live error log
grep "error" /var/log/nginx/error.log      → Filter errors
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c  → HTTP status codes
nginx -t                                   → Always run before reload