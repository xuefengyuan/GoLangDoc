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















































































































