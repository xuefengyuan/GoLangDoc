# NGINX 模块

[TOC]

## 一、安装编译参数详解

> Nginx的官方模块



| 编译选项                        | 作用                   |
| ------------------------------- | ---------------------- |
| --with-http_stub_status_module  | Nginx的客户端状态      |
| --with-http-random_index_module | 目录中选择一个随机主页 |
| --with-http_sub_module          | HTTP内容替换           |

### 1.1、http_stub_status_module 的配置

> Syntas: stub_status
>
> Default: --
>
> Content: server, location (标识是在哪个层级下面，这里有两个)

```shell
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
 
    sendfile        on;

    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;
        # 这里配置 http_stub_status_module 模块
		location /mystatus {
			stub_status;
		}
        location / {
            root   html;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

### 1.2、http-random_index_module 的配置

> 很少用到，对于想展示多个不同主页可以用这个配置
>
> Syntax: random_index    on|off;
>
> Default: random_index  off;
>
> Context : locaton

```shell
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
 
    sendfile        on;

    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;
       
        location / {
            root   html;
            # 这里配置 http-random_index_module 模块
            random_index  on; # 打开
            #index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

### 1.3、http_sub_module 的配置

> 用于对请求的HTTP内容进行替换
>
> Syntax: sub_filter string replacement;
>
> Default: --
>
> Context : http, serer, locaton
>
> ==============================
>
> Syntax: sub_filter_last_modified   on | of;
>
> Default: sub_filter_last_modified   off;
>
> Context : http, serer, locaton
>
> ==============================
>
> 这个表示是否是全部替换
>
> Syntax: sub_filter_once   on | off;
>
> Default: sub_filter_once  on;
>
> Context : http, serer, locaton

```shell
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
 
    sendfile        on;

    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;
       
        location / {
            root   html;           
            index  index.html index.htm;
            # 这里配置 http_sub_module 的配置
            sub_filter '要替换的字符串' '替换后的字符串';
            # 这里off表示替换全部，默认只替换一次
            sub_filter_once off;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

