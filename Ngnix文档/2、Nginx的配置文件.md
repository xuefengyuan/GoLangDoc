# Nginx的配置文件

[TOC]

## 一、Nginx的默认配置文件>

> 1. 配置文件由指令与指令块构成
> 2. 每条指令以 ; 号结尾，指令与参数间以空格符分隔
> 3. 指令块以{ } 大括号将多条指令组织在一起
> 4. include语句允许组合多个配置文件以提升可维护性
> 5. 使用#符号添加注释，提高可读性
> 6. 使用$符号使用变量
> 7. 部分指令的参数支持正则表达式

```shell
worker_processes  1;

events {
    worker_connections  1024;
}

http { # 配置块 跟大扩号之间要有空格
    include       mime.types; # 属性配置，引入文件
    default_type  application/octet-stream; # 属性配置

    sendfile        on; # 启用sendfile
  
    keepalive_timeout  65; # 长连接的保活时间65秒

    server { # 服务
        listen       80; # 端口
        server_name  localhost; # 域名配置，ip

        location / { # 请求地址
            root   html; # 静态界面地址
            index  index.html index.htm; # index界面(首页)
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
       
    }
 
}
```











































