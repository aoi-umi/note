```nginx
upstream my_server {
  server localhost:3000;
}

server {
  listen              80 default_server;
  server_name         [domain];
  location / {
    root   [file path];
    
    #spa
    try_files $uri $uri/ /index.html;
    index  index.html index.htm;
    #spa end

    #跨域
    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
    #跨域 end
  }
}

server {
  listen              80;
  server_name         [domain];
  location /{
    proxy_pass  http://localhost:3001/;
    proxy_set_header   Host $host:$server_port;
    proxy_set_header   X-Real-IP        $remote_addr;
    proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
  }
}
```