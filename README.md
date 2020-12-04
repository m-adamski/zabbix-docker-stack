# Zabbix - Docker Stack

Configuration file to run the Docker Compose stack of the Zabbix monitoring tool

## NGINX Reverse Proxy

In this configuration we would like to proxy all requests through NGINX

```
server {
    listen 80;
    listen [::]:80;

    server_name zabbix.example.com;
    server_tokens off;

    if ($host = zabbix.example.com) {
        return 301 https://$host$request_uri;
    }

    return 404;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2 ipv6only=on;

    server_name zabbix.example.com;
    server_tokens off;

    # Configuration of the Content Security Policy
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Frame-Options DENY;
    add_header Strict-Transport-Security "max-age=31536000 includeSubDomains" always;

    # SSL Configuration
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:10m;
    ssl_session_tickets off;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    ssl_stapling on;
    ssl_stapling_verify on;

    # SSL Certificate
    ssl_certificate /etc/ssl/zabbix.example.com.pem;
    ssl_certificate_key /etc/ssl/zabbix.example.com.key;

    location / {
        proxy_redirect     off;
        proxy_set_header   Host $host;
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
        proxy_set_header   X-Forwarded-Host $server_name;
        proxy_set_header   X-Forwarded-Port $server_port;
        proxy_pass         http://localhost:8080;
    }

    error_log /var/log/nginx/zabbix.example.com_error.log;
    access_log /var/log/nginx/zabbix.example.com_access.log;
}
```

There is also an option to generate a self-signed certificate and place it on the zabbix-web volume (default /var/lib/docker/volumes/zabbix-web.ssl/_data). Three files are required:

* dhparam.pem
* ssl.crt
* ssl.key

After restarting the container, SSL should be available on port 8443.