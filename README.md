# Complete NGINX Cookbook notes
Complete NGINX Cookbook notes（ZH-CN）

Complete NGINX Cookbook 首先这本说是可以免费下载的。
https://www.nginx.com/resources/library/complete-nginx-cookbook/

正题

我们都清楚 O’Reilly Cookbook 类型的书籍的风格，主要行文风格都是提出问题，给出答案并解决问题。

这本说主要分为三部分讲解：

Part I: Load Balancing and HTTP Caching（负载平衡和HTTP缓存）
Part II: Security and Access （安全和访问）
Part III: Deployment and Operations （部署和运营）

相对于开发者而言，我们更多的只需要了解第一部分。

第一部分：



## 第一章 High-Performance Load Balancing （高性能负载平衡）

1.1 HTTP Load Balancing（http负载均衡）

```
upstream backend {
    server 10.10.12.45:80 weight=1;
    server app.example.com:80 weight=2;
}
server {
    location / {
        proxy_pass http://backend;
    }
}

```
你把当前请求负载到多个server上，同时server可以指定权重（weight）。

更多配置可以访问 https://docs.w3cub.com/nginx/stream/ngx_stream_upstream_module/#upstream 


1.2 TCP Load Balancing （TCP负载均衡）

```
stream {
    upstream mysql_read {
        server read1.example.com:3306 weight=5;
        server read2.example.com:3306;
        server 10.10.12.34:3306 backup;
    }
    server {
        listen 3306;
        proxy_pass mysql_read;
    }
}
```

1.3 Load-Balancing Methods （负载均衡方法）

The following load-balancing methods are available for upstream HTTP, TCP, and UDP pools:

五种方法（指令名称）：

Round robin （ weight=x）
Least connections （least_conn）
Least time （least_time）
Generic hash （hash）
IP hash （ip_hash）

阅读： https://docs.w3cub.com/nginx/stream/ngx_stream_upstream_module/


1.4 Connection Limiting （连接数限制）

```
upstream backend {
    zone backends 64k;
    queue 750 timeout=30s;
    server webserver1.example.com max_conns=25;
    server webserver2.example.com max_conns=15;
}

```


## 第二章 Intelligent Session Persistence  （智能会话持久性）


2.1 Sticky Cookie （粘性Cookie）

You need to bind a downstream client to an upstream server

sticky cookie 指令

```
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
    sticky cookie
           affinity
           expires=1h
           domain=.example.com
           httponly
           secure
           path=/;
}

```

2.2 Sticky Learn

You need to bind a downstream client to an upstream server by using an existing cookie.

sticky learn 指令


```
upstream backend {
    server backend1.example.com:8080;
    server backend2.example.com:8081;
    sticky learn
            create=$upstream_cookie_cookiename
            lookup=$cookie_cookiename
            zone=client_sessions:2m;
}
```

2.3 Sticky Routing

提供一个映射修正处理

```
map $cookie_jsessionid $route_cookie {
    ~.+\.(?P<route>\w+)$ $route;
}
map $request_uri $route_uri {
    ~jsessionid=.+\.(?P<route>\w+)$ $route;
}
upstream backend {
    server backend1.example.com route=a;
    server backend2.example.com route=b;
    sticky route $route_cookie $route_uri;
}
```



2.4 Connection Draining

You need to gracefully remove servers for maintenance or other reasons while still serving sessions.

```
curl 'http://localhost/upstream_conf?upstream=backend&id=1&drain=1'
```



## 第五章 Massively Scalable Content Caching （大规模可扩展的内容缓存）


5.1 Caching Zones （缓存区）

You need to cache content and need to define where the cache is stored.



Use the proxy_cache_path directive to define shared memory cache
zones and a location for the content:

```
proxy_cache_path /var/nginx/cache
                keys_zone=CACHE:60m
                levels=1:2
                inactive=3h
                max_size=20g;
proxy_cache CACHE;
```


5.2 Caching Hash Keys

You need to control how your content is cached and looked up.

Use the proxy_cache_key directive, along with variables to define
what constitutes a cache hit or miss:

```
proxy_cache_key "$host$request_uri $cookie_user";
```

5.3 Cache Bypass


Use the proxy_cache_bypass directive with a nonempty or nonzero
value. One way to do this is by setting a variable within location
blocks that you do not want cached to equal 1:

```
proxy_cache_bypass $http_cache_bypass;

```

The configuration tells NGINX to bypass the cache if the HTTP
request header named cache_bypass is set to any value that is not 0.


5.4 Cache Performance (性能)


```
location ~* \.(css|js)$ {
  expires 1y;
  add_header Cache-Control "public";
}
```

5.5 Purging （）

```
map $request_method $purge_method {
    PURGE 1;
    default 0;
}
server {
    ...
    location / {
        ...
        proxy_cache_purge $purge_method;
    }
}

```



