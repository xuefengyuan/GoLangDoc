# Nginx 初识

[TOC]

![架构图](https://img-blog.csdnimg.cn/20190116152033544.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3J1c3NlbGxfdGFv,size_16,color_FFFFFF,t_70)

## 一、Nginx的主本三个应用场景

### 1、静态资源服务

通过本地文件系统提供服务

### 2、反向代理服务

> - Nginx强大的性能
> - 缓存
> - 负载均衡

### 3、PAI服务

## 二、Nginx的优点

> 1. 高并发，高性能
> 2. 可扩展性好
> 3. 高可靠性
> 4. 热部署
> 5. BSD许可证

## 三、Nginx的四个主要组成部分

### 1、Nginx 二进制可执行文件

​	由各模块源码编译出的一个文件

### 2、Nginx.conf配置文件

​	控制Nginx的行为

### 3、access.log访问日志

​	记录每一条http请求信息

### 4、error.log错误日志

​	定位异常问题

## 四、Nginx发行版本

### 1、开源版

​	nginx.org

> Mainline version 主线版本新功能基本都在主线版本中
>
> Stable version 稳定版本

### 2、商业版

​	nginx.com

## 五、Nginx安装编译

### 1、下载Nginx

```shell
$ wget http://nginx.org/download/nginx-1.14.2.tar.gz
$ chmod 766 nginx-1.14.2.tar.gz
$ tar -xzf nginx-1.14.2.tar.gz
# 复制nginx语法文件
$ cd nginx-1.14.2
$ sudo cp -r contrib/vim/* ~/.vim/
# 可能会报不是目录，需要新建一个
$ sudo mkdir ~/.vim
```

### 2、编译Nginx

```shell
# 查看编译的哪些参数
$ ./configure --help|more
# 编译前，需要先解决依赖环境库
$ sudo apt-get install build-essential -y
$ sudo apt-get install openssl libssl-dev -y
$ sudo apt-get install libpcre3 libpcre3-dev -y
$ sudo apt-get install zlib1g-dev

# 编译到指定目录下，使用默认编译方式
$ ./configure --prefix=/home/darry/nginx

# 输出如下信息，表示编译成功
Configuration summary
  + using system PCRE library
  + OpenSSL library is not used
  + using system zlib library

  nginx path prefix: "/home/darry/nginx"
  nginx binary file: "/home/darry/nginx/sbin/nginx"
  nginx modules path: "/home/darry/nginx/modules"
  nginx configuration prefix: "/home/darry/nginx/conf"
  nginx configuration file: "/home/darry/nginx/conf/nginx.conf"
  nginx pid file: "/home/darry/nginx/logs/nginx.pid"
  nginx error log file: "/home/darry/nginx/logs/error.log"
  nginx http access log file: "/home/darry/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"

# 使用make进行编译
$ make
# 编译后的可执行二进制文件放在objs目录下
$ ls -l
总用量 4672
-rw-rw-r-- 1 darry darry   17474 4月  20 12:04 autoconf.err
-rw-rw-r-- 1 darry darry   39523 4月  20 12:04 Makefile
-rwxrwxr-x 1 darry darry 4642480 4月  20 12:10 nginx
-rw-rw-r-- 1 darry darry    5345 4月  20 12:10 nginx.8
-rw-rw-r-- 1 darry darry    6974 4月  20 12:04 ngx_auto_config.h
-rw-rw-r-- 1 darry darry     657 4月  20 12:04 ngx_auto_headers.h
-rw-rw-r-- 1 darry darry    5725 4月  20 12:04 ngx_modules.c
-rw-rw-r-- 1 darry darry   44288 4月  20 12:10 ngx_modules.o
drwxrwxr-x 9 darry darry    4096 4月  20 12:04 src

```

### 3、Nginx安装

```shell
# 使用make install安装，安装完成后可以进入安装目录查看安装后的文件
$ make install
# 进入安装目录查看 
$ cd /home/darry/nginx
$ ls -l 
drwxrwxr-x 2 darry darry 4096 4月  20 12:15 conf
drwxr-xr-x 2 darry darry 4096 4月  20 12:15 html
drwxrwxr-x 2 darry darry 4096 4月  20 12:15 logs
drwxrwxr-x 2 darry darry 4096 4月  20 12:15 sbin

# conf 配置文件目录
# access.log 和error.log日志目录
# sbin 二进制执行文件目录
```

## 六、Nginx配置文件的通用语法

### 1、配置指令说明

> 1. 配置文件由指令与指令块构成
> 2. 每条指令以 ; 号结尾，指令与参数间以空格符分隔
> 3. 指令块以{ } 大括号将多条指令组织在一起
> 4. include语句允许组合多个配置文件以提升可维护性
> 5. 使用#符号添加注释，提高可读性
> 6. 使用$符号使用变量
> 7. 部分指令的参数支持正则表达式

### 2、配置参数：时间单位

| ms   | milliseconds | 毫秒 | d    | days           | 天   |
| ---- | ------------ | ---- | ---- | -------------- | ---- |
| s    | seconds      | 秒   | w    | weeks          | 周   |
| m    | minutes      | 分   | M    | months, 30days | 月   |
| h    | hours        | 时   | y    | years, 365days | 年   |

### 3、配置参数：空间单位

|      | bytes     | 不加表示字节 |
| ---- | --------- | ------------ |
| k/K  | kilobytes | 1024 byte    |
| m/M  | megabytes | 1204 k/K     |
| g/G  | gigabytes | 1024 m/M     |

### 4、http配置的指令块

#### 4.1、http

>  http{ } 标识里面所有指令都属于http模块

#### 4.2、upstream

> 上游服务模块，一般用来做转发

#### 4.3、server

> 对应一个域名，或者一组域名

#### 4.4、location

> 对应一个url表达式

## 七、Nginx 基本操作命令

```shell
# nginx 帮助命令
$ ./nginx -h

nginx version: nginx/1.15.8
Usage: nginx [-?hvVtTq] [-s signal] [-c filename] [-p prefix] [-g directives]

Options:
  -?,-h         : this help
  -v            : show version and exit
  -V            : show version and configure options then exit
  -t            : test configuration and exit
  -T            : test configuration, dump it and exit
  -q            : suppress non-error messages during configuration testing
  -s signal     : send signal to a master process: stop, quit, reopen, reload
  -p prefix     : set prefix path (default: /home/darry/nginx/)
  -c filename   : set configuration file (default: conf/nginx.conf)
  -g directives : set global directives out of configuration file
# 加载指定的配置文件
$ sudo ./nginx -c ../nginx.conf
# 指定配置指令
$ ./nginx -g  
# 指定运行目录
$ ./nginx -p 
# 测试配置文件是否有语法错误
$ ./nginx -t

# nginx 信号指令

# 重载配置文件
$ ./nginx -s reload
# 立刻停止服务
$ ./nginx -s stop
# 优雅的停止服务
$ ./nginx -s quid
# 重新开始记录日志文件
$ ./nginx -s rcopen
# 平滑的重启nginx
$ ps -ef | grep ngix
$ kill -HUP 进程id
# nginx 热部署，把编译后的nginx执行文件拷贝到进程中所执行的nginx所在目录下
$ kill -USR2 进程id
# 热部署后，新的master进程启动，老的worker进程还在，
# worker进程会慢慢的过渡到新的master进程，并且不在对端口进行监听
# 通知老的进程优雅的关闭
$ kill -WINCH 进程id
# 日志切割，先把之前的日志文件进行备份，需要注意的是nginx的日志所在目录
$ mv access.log bak.log
$ sudo ../sbin/nginx -s reopen

```





















































































































