# NGINX的访问限制

[TOC]

## 一、Nguix访问限制的方式

| 模块                   | 功能                   |
| ---------------------- | ---------------------- |
| http_access_module     | 基于IP访问的限制       |
| http_auth_basic_module | 基于用户的信任登录限制 |

## 二、http_access_module IP访问的限制

### 1、允许访问

> Syntas: allow address | CIDR | unix: | all ;
>
> Default: --
>
> Content: http, server, location, limit_except

### 2、不允许访问

> Syntas: deny address | CIDR | unix: | all ;
>
> Default: --
>
> Content: http, server, location, limit_except

### 3、IP访问限制配置

```shell
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/log/host.access.log  main;

    location / {
        root   /opt/app/code;
        index  index.html index.htm;
    }
# 允许访问的配置
    location ~ ^/admin.html {
        root   /opt/app/code;
        deny 222.128.189.17; # 不允许访问的ip配置
        allow all; # 允许其它ip访问
        index  index.html index.htm;
    }
    
# 不允许访问的配置
    location ~ ^/admin.html {
        root   /opt/app/code;
        allow 222.128.189.0/24; # 允许访问的IP段
        deny all; # 不允许其它的ip访问
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504 404  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

}
```

### 4、http_access_module局限性

> 1. 采用别的HTTP头信息控制访问，如：HTTP_X_FORWARD_FOR
> 2. 结合geo模块
> 3. 通过HTTP自定义变量传递

## 三、http_auth_basic_module 用户登录访问的限制

> Syntas: auth_basic string | off ;
>
> Default: auth_basic  off;
>
> Content: http, server, location, limit_except



> Syntas: auth_basic_user_file file ;
>
> Default:--
>
> Content: http, server, location, limit_except

### 1、用户登录访问限制的配置文件

```shell
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/log/host.access.log  main;

    location / {
        root   /opt/app/code;
        index  index.html index.htm;
    }

    location ~ ^/admin.html {
        root   /opt/app/code;
        auth_basic "Auth access test!input your passward!";
        auth_basic_user_file /etc/nginx/auth_conf; # 密码文件
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504 404  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

}
```

### 2、http_auth_basic_module 的局限性

> 1. 用户信息依赖文件方式
> 2. 操作管理机械化，效率低下
>
> 解决方案
>
> 1. Nginx结合LUA实现高效验证
> 2. Nginx和LDAP打通，利用nginx-auth-ldap模块



