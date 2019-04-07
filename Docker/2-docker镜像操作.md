[TOC]

## 二、Docker相关服务操作

### 1、服务的基本操作命令

```shell
启动
systemctl start docker
停止
systemctl stop docker
重启
systemctl restart docker
查看状态
systemctl status docker ps
```

### 2、Docker镜像管理

#### 1）：搜索镜像

```shell
# 命令格式：
docker search [镜像名称]
# 命令演示：
$  docker search ubuntu

# NAME：名称
# DESCRIPTION：基本功能描述
# STARS：下载次数
# OFFICIAL：官方
#AUTOMATED：自动的运行
```

#### 2）：获取镜像

```shell
# 命令格式：
docker pull [镜像名称]
# 命令演示：
docker pull ubuntu
docker pull nginx

# 注释：
# 获取的镜像在哪里？
# /var/lib/docker 目录下
# 由于权限的原因我们需要切换root用户
# 那我们首先要重设置root用户的密码：
$ sudo passwd root
# 这样就可以设置root用户的密码了。
# 之后就可以自由的切换到root用户了
$ su
# 输入root用户的密码即可。
# 当然，如果想从root用户切换回一般用户，则可使用 su -val(一般用户名)
# 而当你再次切回到root用户，则只需要键入exit,再次输入exit则回到最初的用户下
# 操作下面的文件可以查看相关的镜像信息
vim /var/lib/docker/image/overlay2/repositories.json

```

#### 3）：查看镜像

```shell
# 查看当前安装的镜像，下面命令都可以查看镜像

$ docker images
$ docker image ls
$ docker images ubuntu

# REPOSITORY：镜像的名称
# TAG ：镜像的版本标签
# IMAGE ID：镜像id
# CREATED：镜像是什么时候创建的
# SIZE：大小
```

#### 4）：镜像重命名

```shell
# 命令格式：
$ docker tag [老镜像名称]:[老镜像版本][新镜像名称]:[新镜像版本]
# 命令演示：
$ docker tag nginx:latest panda-nginx:v1.0
```

#### 5）：删除镜像

```shell
# 命令格式：
$ docker rmi [命令参数][镜像ID]
$ docker rmi [命令参数][镜像名称]:[镜像版本]
$ docker image rm [命令参数][镜像]
# 命令演示：
$ docker rmi 3fa822599e10
$ docker rmi mysql:latest
# 注意：
如果一个image_id存在多个名称，那么应该使用 名称:版本 的格式删除镜像
# 命令参数(OPTIONS)：
-f, --force 强制删除
$ docker image rmi -f --force 3fa822599e10
# 批量删除镜像
$ docker image rm -f --force $(docker image ls -a -q)
```

#### 6）：镜像导出

```shell
# 命令格式：
$ docker save [命令参数][导出镜像名称][本地镜像镜像]
# 命令参数(OPTIONS)：
-o, --output string 指定写入的文件名和路径
# 导出镜像
$ docker save -o nginx.tar nginx
# 导出到指定目录
$ docker save -o ./images/nginx.tar nginx
```

#### 7）：导入镜像

```shell
# 导入镜像命令格式：
$ docker load [命令参数][被导入镜像压缩文件的名称]
$ docker load < [被导入镜像压缩文件的名称]
$ docker load --input [被导入镜像压缩文件的名称]
# 命令参数(OPTIONS)：
-i, --input string 指定要打入的文件，如没有指定，默认是STDIN
# 为了更好的演示效果，我们先将nginx的镜像删除掉
$ docker rmi nginx:v1.0
$ docker rmi nginx
# 导入镜像文件：
$ docker load < nginx.tar
# 从指定目录导入
$ docker load < ./images/nginx.tar 
# 注意：
如果发现导入的时候没有权限需要使用chmod命令修改镜像文件的权限
```

#### 8）：查看镜像历史

```shell
# 查看镜像命令格式：
$ docker history [镜像名称]:[镜像版本]
$ docker history [镜像ID]
# 我们获取到一个镜像，想知道他默认启动了哪些命令或者都封装了哪些系统层，那么我们可以使用docker history这条命令来获取我们想要的信息
$ docker history sswang-nginx:v1.0

$ docker history 2bcb04bdb83f
# IMAGE：编号
# CREATED：创建的
# CREATED BY ：基于那些命令创建的
# SIZE：大小
# COMMENT：评论
```

#### 9）：查看镜像详细信息

```shell
# 命令格式：
$ docker image inspect [命令参数] [镜像名称]:[镜像版本]
$ docker inspect [命令参数] [镜像ID]
# 查看镜像详细信息：
$ docker inspect nginx
```

