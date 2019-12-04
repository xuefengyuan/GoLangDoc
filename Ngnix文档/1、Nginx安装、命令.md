# Nginx的安装和基本命令

[TOC]

## 一、下载安装

> Mainline version 主线版本新功能基本都在主线版本中
>
> Stable version 稳定版本

下载地址：[Nginx](http://nginx.org/en/download.html)

### 1.1、下载Nginx

```shell
# 关闭防火墙规则(生产环境不可以这样搞)
# 查看防火墙规则
$ iptables -L 
$ iptables -t nat -L
# 关闭规则
$ iptables -F
$ iptables -t nat -F

# 关闭SELinux
$ getenforce
$ setenforce 0


# 下载
$ cd /data
$ wget http://nginx.org/download/nginx-1.16.1.tar.gz
# 解压
$ tar -zxvf nginx-1.16.1.tar.gz
 
# 下载编译的依赖环境
$ yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel pcre pcre-devel

```

### 1.2、编译安装Nginx

```shell
# 1、查看编译参数
$ cd nginx-1.16.1
$ ./configure --help|more

# 2、编译到指定目录下，使用默认编译方式
$ ./configure --prefix=/data/nginx
# 3、使用make进行编译
$ make
# 编译后的可执行二进制文件放在objs目录下
$ ls -l
# 4、使用make install安装，安装完成后可以进入安装目录查看安装后的文件
$ make install
# 5、进入安装目录查看
$ cd /data/nginx
# 6、查看文件
$ ll
# conf 配置文件目录
# access.log 和error.log日志目录
# sbin 二进制执行文件目录
```

### 1.3、Nginx启动

```shell
# nginx 帮助命令
$ cd /data/nginx/sbin
$ ./nginx -h
# 加载指定的配置文件启动
$ ./nginx -c /data/nginx/conf/nginx.conf
# 查看nginx是否启动
$ ps -ef | grep nginx

# nginx启动后外部访问不了的话，需要开放端口(默认是80，修改成自己开放的端口)
$ firewall-cmd --zone=public --add-port=80/tcp --permanent
$ firewall-cmd --reload
```

### 1.4、Nginx基本命令

```shell
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
$ ./nginx -s quit
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
$ ./nginx -s reopen

# 查看Nginx版本信息
$ ./nginx -v
# 查看完整的Nginx配置信息(编译参数)(上面是小写，下面是大写)
$ ./nginx -V
```

### 1.5、Nginx安装目录

```shell
nginx
├── client_body_temp
├── conf # 配置文件目录
├── fastcgi_temp
├── html # 默认静态界面目录
├── logs # 日志目录
├── proxy_temp
├── sbin # 二进制文件目录
│   └── nginx
├── scgi_temp
└── uwsgi_temp

```











































