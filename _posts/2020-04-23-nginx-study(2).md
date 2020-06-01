---
layout: post
title: 'Nginx'
categories: nginx
tags: nginx linux
author: thorleishen
---

* content
{:toc}
## 学习路线

部署nginx服务器，nginx配置域名等参数



## 遇到的问题

1. ssl安全证书



#### 问题一：ssl安全证书

```nginx
upstream tornados{
    server 0.0.0.0:8000;
    server 0.0.0.0:8002;
    server 0.0.0.0:8003;
    server 0.0.0.0:8004;
    server 0.0.0.0:8005;
}
#proxy_next_upstream error; # 重试机制, 请求服务器发生错误或超时时会尝试到下一台服务器
#proxy_next_upstream_tries 0; # 重试机制次数，默认为0表示不限制
server{
        listen       8080;
        server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;
  
    location / {
        proxy_pass http://tornados;
        proxy_ssl_server_name on;
        proxy_ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Cookie $http_cookie;
#       proxy_cookie_domain "~[0-9]{2}\.[0-9]{3}\.[0-9]{2}\.[0-9]{3}$" 127.0.0.1;
        proxy_cookie_path / /;
        proxy_connect_timeout 500s;
        proxy_read_timeout 500s;
        proxy_send_timeout 500s;

        if ($request_method = 'OPTIONS') {
        add_header 'Access-Control-Allow-Origin' '$http_origin';
      
upstream tornados{
    server 127.0.0.1:8000;
    server 127.0.0.1:8002;
    server 127.0.0.1:8003;
    server 127.0.0.1:8004;
    server 127.0.0.1:8005;
}
proxy_next_upstream error;
server{
        listen 80;
        server_name api.apptutti.cn;
        return 301 https://api.apptutti.cn$request_uri;
}
server {
    listen       443 ssl;
    server_name  api.apptutti.cn;

    # ssl on; # nginx1.5版本之后换成listen 443 ssl;
    ssl_certificate  /etc/ssl/certs/3387844_api.apptutti.cn.pem;
    ssl_certificate_key /etc/ssl/certs/3387844_api.apptutti.cn.key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        proxy_pass http://tornados;
        proxy_ssl_server_name on;
        proxy_ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Cookie $http_cookie;
        proxy_connect_timeout 500s;
        proxy_read_timeout 500s;
        proxy_send_timeout 500s;
        proxy_cookie_domain api.apptutti.cn .apptutti.cn;
        #proxy_cookie_domain "" .apptutti.cn;
        proxy_cookie_path / /;

        if ($request_method = 'OPTIONS') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        #
        # Custom headers and headers various browsers *should* be OK with but aren't
        #
        add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
        #
        # Tell client that this pre-flight info is valid for 20 days
        #
        add_header 'Access-Control-Max-Age' 1728000;
        add_header 'Content-Type' 'text/plain; charset=utf-8';
        add_header 'Content-Length' 0;
        return 204;
        }
        if ($request_method = 'POST') {
                add_header 'Access-Control-Allow-Origin' '$http_origin';
                add_header 'Access-Control-Allow-Credentials' 'true';
                add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
                add_header 'Access-Control-Allow-Headers' 'Cookie,Set-Cookie,DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
                add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
        }
        if ($request_method = 'GET') {
                add_header 'Access-Control-Allow-Origin' '$http_origin';
                add_header 'Access-Control-Allow-Credentials' 'true';
                add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
                add_header 'Access-Control-Allow-Headers' 'Cookie,Set-Cookie,DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
                add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
        }

        root   /usr/local/dataCollection;
        index  index.html index.htm;
    }
        
    location /api/ {
          proxy_pass http://tornados/;
          # 注: 有个/，当用用户请求domain/api/时，会代理到tornados服务器
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```



## 总结

nginx反向代理，可以解决跨域问题，第一：通过location /***/；第二：通过配置request header参数；当前配置还解决http重定向到https，加上ssl安全证书问题；当然还有其他参数配置解决一些实际问题。