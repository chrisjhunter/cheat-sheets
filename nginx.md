# Nginx Cheat Sheet

## Service management
```bash
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx        # reload config without dropping connections
sudo nginx -t                       # test config syntax before reloading
sudo nginx -s reload                # alternative reload method
sudo systemctl status nginx
```

## Key file locations (Debian/Ubuntu)
```
/etc/nginx/nginx.conf              # main config
/etc/nginx/sites-available/        # available server blocks
/etc/nginx/sites-enabled/          # symlinked = active
/var/log/nginx/access.log
/var/log/nginx/error.log
```

## Enabling a site
```bash
sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

## Basic reverse proxy
```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## Static file serving
```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/example;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

## HTTPS / TLS (with certbot-issued certs)
```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
}

server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;   # redirect HTTP -> HTTPS
}
```
```bash
sudo certbot --nginx -d example.com               # obtain + auto-configure cert
sudo certbot renew --dry-run                        # test renewal
```

## Load balancing
```nginx
upstream backend {
    least_conn;                       # or default round-robin, ip_hash
    server 10.0.0.1:8080;
    server 10.0.0.2:8080;
    server 10.0.0.3:8080 backup;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

## Caching
```nginx
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=mycache:10m max_size=1g;

server {
    location / {
        proxy_cache mycache;
        proxy_cache_valid 200 10m;
        add_header X-Cache-Status $upstream_cache_status;
        proxy_pass http://backend;
    }
}
```

## Rate limiting
```nginx
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;

server {
    location /api/ {
        limit_req zone=mylimit burst=20 nodelay;
        proxy_pass http://backend;
    }
}
```

## Useful one-liners
```bash
sudo nginx -T                                            # dump full effective config
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head  # top IPs
awk '{print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head  # top requested paths
grep " 5[0-9][0-9] " /var/log/nginx/access.log                                  # 5xx errors
curl -I http://localhost                                                          # quick local health check
sudo lsof -i :80                                                                    # what's using port 80
```
