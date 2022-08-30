```nginx
server {
  listen              80 default_server;
  server_name         [domain];
  location / {
    root   [file path];
    try_files $uri $uri/ /index.html;
    index  index.html index.htm;
  }
}

server {
  listen              80;
  server_name         [domain];
  location /{
    proxy_pass  http://localhost:3001/;
    proxy_set_header   Host $host;
    proxy_set_header   X-Real-IP        $remote_addr;
    proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
  }
}
```