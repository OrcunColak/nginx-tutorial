# nginx.conf

1. Global Block
2. Events Block
3. HTTP Block
4. Server Block
5. Location Block

# Common Configuration Examples

1. Reverse Proxy
   Reverse proxying is primarily done using the proxy_pass directive to forward requests to a backend server.

```
server {
    listen 8090;
    server_name www.example.com;

    location / {
        proxy_pass http://app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
# Backend server configuration
upstream app {
    server 10.10.10.100:8090;
    server 10.10.10.101:8090;
}
```

2. Use ip_hash (Sticky per Client IP) for web sockets
WebSockets require the Upgrade and Connection headers to be properly set when proxying. When
```
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
```
so this tellx nginx to use a map block that sets the value of a new variable $connection_upgrade depending on the presence of $http_upgrade:
- If $http_upgrade is non-empty, set $connection_upgrade to "upgrade" (needed for WebSockets).
- If $http_upgrade is empty, set $connection_upgrade to "close" (for normal HTTP requests).
```
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

upstream websocket_backend {
    ip_hash;  # Ensures requests from the same client IP go to the same server
    server app1:8080;
    server app2:8080;
}

server {
    listen 80;
    location /chat {
        proxy_pass http://websocket_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
    }
}
```
