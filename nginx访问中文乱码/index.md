# Nginx访问中文乱码



```bash
upstream you.domainName.com {
server 127.0.0.1:8080;
}

server {
  listen 80;
  server_name you.domainName.com;
  charset utf-8;  ##添加这段

  location /examples {
    return 403;
  }
}
```

修改后，重启Nginx服务。
