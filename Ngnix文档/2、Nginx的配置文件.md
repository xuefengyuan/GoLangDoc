# Nginx的配置文件

[TOC]

## 一、Nginx的默认配置文件

> 1. 配置文件由指令与指令块构成
> 2. 每条指令以 ; 号结尾，指令与参数间以空格符分隔
> 3. 指令块以{ } 大括号将多条指令组织在一起
> 4. include语句允许组合多个配置文件以提升可维护性
> 5. 使用#符号添加注释，提高可读性
> 6. 使用$符号使用变量
> 7. 部分指令的参数支持正则表达式

```shell
#user  nobody; # 设置nginx服务的系统使用用户
worker_processes  1; # 工作进程数

# nginx的错误日志
#error_log  logs/error.log; 
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid; # nginx服务启动时候的pid


events {
    worker_connections  1024; # 每个进程允许最大连接数(最大65535)
    #use 5; # 工作进程数
}


http {  # 配置块 跟大扩号之间要有空格
    include       mime.types; # 属性配置，引入文件
    default_type  application/octet-stream; # 属性配置

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on; # 启用sendfile
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65; # 长连接的保活时间65秒

    #gzip  on;

    server { # 服务
        listen       80; # 外部访问端口
        server_name  localhost; # 域名配置，ip

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / { # 请求地址(默认访问路径)
            root   html; # 静态界面路径
            index  index.html index.htm; # index界面(首页)
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html; # 错误页面
        location = /50x.html {
            root   html;
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


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```



































