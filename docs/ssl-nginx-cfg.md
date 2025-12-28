### SSL-updated Nginx config file chatjn.barryonweb.com

```bash
server {
  server_name chatjn.barryonweb.com;

  # Proxy all API requests to Spring Boot backend (port 8081)
  location /api/ {
    proxy_pass http://127.0.0.1:8081;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }

  listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/chatjn.barryonweb.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/chatjn.barryonweb.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
server {
  if ($host = chatjn.barryonweb.com) {
    return 301 https://$host$request_uri;
  } # managed by Certbot
  listen 80;
    server_name chatjn.barryonweb.com;
    return 404; # managed by Certbot
}
```