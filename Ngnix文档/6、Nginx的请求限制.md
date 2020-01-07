# NGINX的请求限制

[TOC]

## 一、模块说明

| 模块              | 功能         |
| ----------------- | ------------ |
| limit_conn_module | 连接频率限制 |
| limit_req_module  | 请求频率限制 |
|                   |              |



## 二、HTTP协议的连接与请求

| HTTP协议版本 | 连接关系        |
| ------------ | --------------- |
| HTTP1.0      | TCP不能复用     |
| HTTP1.1      | 顺序性TCP复用   |
| HTTP2.0      | 多路复用TCP复用 |

> HTTP请求建立在一次TCP连接基础上
>
> 一次TCP请求至少产生一次HTTP请求



## 三、limit_conn_module 连接频率限制配置

> 这个配置相当于开辟一个空间
>
> Syntas: limit_conn_zone key zone=name:size;
>
> Default: --
>
> Content: http
>
> ================================
>
> 这里是使用上面开辟的空间
>
> Syntas: limit_conn zone number;
>
> Default: --
>
> Content: http, server, location
>
> 这两个配置要结合使用



## 四、limit_req_module 请求频率限制的配置

> 这个配置相当于开辟一个空间
>
>  rate=rate 请求限制：一般为秒
>
> Syntas: limit_req_zone key zone=name:size rate=rate;
>
> Default: --
>
> Content: http
>
> ================================
>
> 这里是使用上面开辟的空间
>
> Syntas: limit_req zone=name [burst=number] [nodelay];
>
> Default: --
>
> Content: http, server, location
>
> 这两个配置要结合使用



## 五、配置文件

```shell

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
    # 连接频率限制
	limit_conn_zone $binary_remote_addr zone=conn_zone:1m;
	# 请求频率限制
	limit_req_zone $binary_remote_addr zone=req_zone:1m rate=1r/s; # 1秒只允许一次请求
    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            #limit_conn conn_zone 1; # 连接限制(只允许一个ip连接进来)
            # 下面的是请求限制
            #limit_req zone=req_zone burst=3 nodelay; # 超过3个后放到下一次执行，只放三个
            #limit_req zone=req_zone burst=3;
            #limit_req zone=req_zone;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
     
    }

}
```







