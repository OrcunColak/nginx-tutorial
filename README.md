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