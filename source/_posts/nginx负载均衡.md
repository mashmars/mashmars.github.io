---
title: nginx负载均衡
date: 2019-08-19 14:16:29
tags:
    - nginx
categories: nginx
---
温故而知新
### nginx支持以下负载均衡机制:
* 循环 -对应用程序服务器的请求以循环方式分发
* 最少连接(least_conn) -下一个请求被分配给活动连接数最少的服务器
* ip-hase -基础客户端ip地址 保持session连接 也叫 粘滞 持久
* 加权 -数值越大 分配的连接数越多

#### 默认的负载均衡配置
```
http {
    upstream mysites {
        server 47.52.152.239:81;
        server 47.52.152.239;
    }

    server {
        listen 80;
        server_name localhost;
        location / {
            #root /www/xx
            #index index.html
            proxy_pass http://mysites;
        }
    }
}
```

#### 最小连接
```
http {
    upstream mysites {
        least_conn;
        server 47.52.152.239:81;
        server 47.52.152.239;
    }

    server {
        listen 80;
        server_name localhost;
        location / {
            #root /www/xx
            #index index.html
            proxy_pass http://mysites;
        }
    }
}
```

#### 会话持久性
```
http {
    upstream mysites {
        ip_hash;
        server 47.52.152.239:81;
        server 47.52.152.239;
    }

    server {
        listen 80;
        server_name localhost;
        location / {
            #root /www/xx
            #index index.html
            proxy_pass http://mysites;
        }
    }
}
```
#### 加权
```
http {
    upstream mysites {
        server 47.52.152.239:81 weight = 3;
        server 47.52.152.239;
    }

    server {
        listen 80;
        server_name localhost;
        location / {
            #root /www/xx
            #index index.html
            proxy_pass http://mysites;
        }
    }
}
```

更多参数请访问 [nginx doc](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#server "nginx文档")


