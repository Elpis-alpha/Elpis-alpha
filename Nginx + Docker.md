The docker compose file

```yaml
name: nginx-overseer

services:
  nginx:
    image: nginx:alpine
    volumes:
      - ./ngx/etc/nginx:/etc/nginx
      - ./ngx/www:/var/www/html
      - ./ngx/etc/selfsignedssl:/etc/selfsignedssl
      - ./ngx/etc/letsencrypt:/etc/letsencrypt
    ports:
      - "80:80"
      - "443:443"
    restart: unless-stopped

  certbot-domain-name:
    image: certbot/certbot
    volumes:
      - ./ngx/etc/nginx:/etc/nginx
      - ./ngx/www:/var/www/html
      - ./ngx/etc/letsencrypt:/etc/letsencrypt
    entrypoint: >
      sh -c "certbot certonly --webroot -w /var/www/html
      --email mail@gmail.com --agree-tos --non-interactive
      --domains domain-name.com --domains www.domain-name.com --domains api.domain-name.com"

# use "docker compose up nginx --build -d" to start the nginx part
# use "docker compose run --rm certbot-domain-name" to start the certbot part
```

The conf file

```nginx
events {
  worker_connections 1024;
}

http {
  # HTTP
  server {
    listen 80;
    server_name domain-name.com www.domain-name.com;

    location /.well-known/acme-challenge/ {
      root /var/www/html;
      try_files $uri =404;
    }

    location / {
      return 301 https://$host$request_uri;
    }
  }
  server {
    listen 80;
    server_name api.domain-name.com;

    location /.well-known/acme-challenge/ {
      root /var/www/html;
      try_files $uri =404;
    }

    location / {
      return 301 https://$host$request_uri;
    }
  }

  # HTTPS
  server {
    listen 443 ssl;
    server_name domain-name.com www.domain-name.com;

    ssl_certificate /etc/letsencrypt/live/domain-name.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/domain-name.com/privkey.pem;

    location /.well-known/acme-challenge/ {
      root /var/www/html;
      try_files $uri =404;
    }

    location / {
      proxy_pass http://<IP_ADDRESS>:<PORT>;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
      client_max_body_size 50M;
    }
  }
  server {
    listen 443 ssl;
    server_name api.domain-name.com;

    ssl_certificate /etc/letsencrypt/live/domain-name.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/domain-name.com/privkey.pem;

    location /.well-known/acme-challenge/ {
      root /var/www/html;
      try_files $uri =404;
    }

    location / {
      proxy_pass http://<IP_ADDRESS>:<PORT>;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
      client_max_body_size 50M;


      # WebSocket support
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";

      proxy_read_timeout 86400;
      proxy_send_timeout 86400;
      proxy_buffering off;
    }
  }
}
```
