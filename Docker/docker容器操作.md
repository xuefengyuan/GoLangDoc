[TOC]

## 三、Docker容器

### 1）：查看容器

```shell
# 命令格式：
docker ps
# 命令演示：
# 查看已启动的容器
$ docker ps
# 查看全部容器，包括未启动的
$ docker ps -a

# CONTAINER ID 容器ID
# IMAGE 基于那个镜像
# COMMAND 运行镜像使用了哪些命令？
# CREATED多久前创建时间
# STATUS 开启还是关闭
# PORTS端口号
# NAMES容器名称默认是随机的
#注意：
# 管理docker容器可以通过名称，也可以通过ID
# ps是显示正在运行的容器， -a是显示所有运行过的容器，包括已经不运行的容器
```

### 2）：创建待启动容器

```shell
# 命令格式：
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
docker create [参数命令] 依赖镜像 [容器内命令] [命令参数]
# 命令参数(OPTIONS)：查看更多
-t, --tty 分配一个伪TTY，也就是分配虚拟终端
-i, --interactive 即使没有连接，也要保持STDIN打开
--name 为容器起名，如果没有指定将会随机产生一个名称
# 命令参数（COMMAND\ARG）:
COMMAND 表示容器启动后，需要在容器中执行的命令，如ps、ls 等命令
ARG 表示执行 COMMAND 时需要提供的一些参数，如ps 命令的 aux、ls命令的-a等等
# 命令演示：创建容器（附上ls命令和a参数）
$ docker create -it --name ubuntu-test1 ubuntu ls -al
```

### 3）：启动容器

- 启动容器有三种方式
  1. 启动待启动或已关闭容器
  2. 基于镜像新建一个容器并启动
  3. 守护进程方式启动docker

#### 3.1）：启动创建好的容器

```shell
# 命令格式：
docker start [容器名称]或[容器ID]
# 命令参数(OPTIONS)：
-a, --attach 将当前shell的 STDOUT/STDERR 连接到容器上
-i, --interactive 将当前shell的 STDIN连接到容器上
# 命令演示：启动上面创建的容器
$ docker start -a ubuntu-test1
```

#### 3.2）：创建新容器并启动

```shell
# 命令格式：
docker run [命令参数] [镜像名称][执行的命令]
命令参数(OPTIONS)：
-t, --tty 分配一个伪TTY，也就是分配虚拟终端
-i, --interactive 即使没有连接，也要保持STDIN打开
--name 为容器起名，如果没有指定将会随机产生一个名称
-d, --detach 在后台运行容器并打印出容器ID
--rm 当容器退出运行后，自动删除容器
# 启动一个镜像输出内容并删除容器
$ docker run --rm --name nginx-test1 nginx /bin/echo "hello docker"
# 注意：
docker run 其实 是两个命令的集合体 docker create + docker start
```

#### 3.3）：守护进程方式启动容器<常用的方式>

```shell
# 命令格式：
docker run -d [image_name] command ...
# 命令演示：守护进程方式启动容器
$ docker run -d --name nginx-test2 nginx
```

### 4）：容器暂停

```shell
# 命令格式：
docker pause [容器名称]或[容器ID]
# 暂停容器
$ docker pause nginx-test2
```

### 5）：容器取消暂停

```shell
# 命令格式：
docker unpause [容器名称]或[容器ID]
# 恢复容器
$ docker unpause nginx-test2
```

### 6）：重启容器

```shell
# 命令格式：
docker restart [容器名称]或[容器ID]
# 命令参数(OPTIONS)：
-t, --time int 重启前，等待的时间，单位秒(默认 10s)
# 恢复容器
$ docker restart -t 20 nginx-test2
```

### 7）：关闭容器

```shell
# 命令格式：
docker stop [容器名称]或[容器ID]
# 关闭容器:
$ docker stop nginx-test2
```

### 8）：终止容器

```shell
# 命令格式：
docker kill [容器名称]或[容器ID]
# 终止容器
$ docker kill nginx-test2
```

### 9）：删除容器

删除容器有三种方法： **正常删除 -- 删除已关闭的 强制删除 -- 删除正在运行的 强制批量删除 -- 删除全部的容器**

#### 9.1）：正常删除容器

```shell
# 命令格式：
docker rm [容器名称]或[容器ID]
# 删除已关闭的容器:
$ docker rm nginx-test2
```

#### 9.2）：强制删除运行容器

```shell
# 命令格式：
docker rm -f [容器名称]或[容器ID]
# 删除正在运行的容器
$ docker rm -f nginx-test2
```

#### 9.3）：批量删除容器

```shell
# 命令格式：
docker rm -f $(docker ps -a -q)
# 按照执行顺序$（）， 获取到现在容器的id然后进行删除
$ docker rm -f $(docker ps -a -q)
```

### 10）：进入容器

- 进入容器有三种方法：
  1. 创建容器的同时进入容器
  2. 手工方式进入容器
  3. 生产方式进入容器

#### 10.1）：创建并进入容器

> <font color=red>注意：需要进入的容器，要先启动才可以进入</font>

```shell
# 命令格式：
docker run --name [container_name] -it [docker_image] /bin/bash
# 命令演示：
$ docker run -it --name nginx-test1 nginx /bin/bash
# 进入容器后
root@ddf8345076af:/# echo "hello world"
hello world
root@ddf8345076af:/# exit
exit
# docker 容器启动命令参数详解：
# --name:给容器定义一个名称
# -i:则让容器的标准输入保持打开。
# -t:让docker分配一个伪终端,并绑定到容器的标准输入上
# /bin/bash:执行一个命令
```

**创建并进入Ubuntu容器**

```shell
$ docker run -dit --name ubuntu01 ubuntu /bin/bash
```

#### 10.2）：手工方式进入容器

```shell
# 命令格式：
docker exec -it 容器id /bin/bash
# 效果演示：
$ docker exec -it ddf8345076af /bin/bash
$ docker exec -it ubuntu-test1 /bin/bash
```

#### 10.3）：生产方式进入容器

生产中常用的进入容器方法是使用脚本，脚本内容如下

```shell
# 创建一个脚本文件
vim docker_in.sh
```

脚本文件内容

```shell
#!/bin/bash
#定义进入仓库函数
docker_in(){
    NAME_ID=$1
    PID=$(docker inspect --format {{.State.Pid}} $NAME_ID)
    nsenter --target $PID --mount --uts --ipc --net --pid
}
docker_in $1
```

直接执行的话脚本是没有执行权限的所以需要赋值权限

```shell
# 赋权执行
$ chmod +x docker_in.sh
# 进入指定的容器，并测试,容器id和容器名称都可以
$ sudo ./docker_in.sh b3fbcba852fd
$ sudo ./docker_in.sh nginx-test1
```

注意：

```shell
-bash: ./docker_in.sh: /bin/bash^M: 解释器错误: 没有那个文件或目录
这个问题大多数是因为脚本文件在windows下编辑过。windows下，每一行的结尾是\n\r，而在linux下
文件的结尾是\n，那么在windows下编辑过的文件在linux下打开看的时候每一行的结尾就会多出来一个字
符\r,用cat -A docker_in.sh时可以看到这个\r字符被显示为^M，这时候只需要删除这个字符就可以了。
可以使用命令 sed -i 's/\r$//' docker_in.sh
```

### 11）：退出容器

```shell
# 方法一：
exit
# 方法二：
Ctrl + D
```

### 12）：基于容器创建镜像

#### 12.1）：方式一：

```shell
# 命令格式
docker commit -m '改动信息' -a "作者信息" [container_id][new_image:tag]
# 命令演示：
# 1/进入一个容器，创建文件后并退出:
$ sudo ./docker_in.sh d74fff341687
$ sudo ./docker_in.sh nginx-test1
mkdir /hello
mkdir /world
ls
exit
# 2、创建一个镜像: (用容器id和容器名称都可以创建镜像)
$ docker commit -m 'mkdir /hello /world ' -a "ybd" d74fff341687 nginx:v0.2
$ docker commit -m 'mkdir /hello /world ' -a "ybd" nginx-test1 nginx:v0.2
# 查看镜像:
$ docker images
# 启动一个容器
$ docker run -itd nginx:v0.2 /bin/bash
$ docker run -it --name nginx-test2 nginx:v0.2 /bin/bash
# 进入容器进行查看
$ sudo ./docker_in.sh nginx-test2
ls
```

#### 12.2：）方式二

```shell
# 命令格式：
docker export [容器id] > 模板文件名.tar
# 命令演示：
# 创建镜像:
$ docker export ae63ab299a84 > nginx.tar
$ docker export nginx-test1 > nginx-test.tar
# 导出的时候也可以指定镜像tar包存放位置，不指定则默认在当前执行命令的目录下
$ docker export nginx-test1 > ./images/nginx-test.tar
# 导入镜像:
$ cat nginx.tar | docker import - panda-test
# 查看镜像
$ docker images
```

### 13）：容器其它命令

#### 13.1）：查看容器运行日志

```shell
# 命令格式：
docker logs [容器id]
# 命令效果：
docker logs 7c5a24a68f96
```

#### 13.2）：查看容器详细信息

```shell
# 命令格式
docker inspect [容器id]
# 命令效果：
# 查看容器全部信息:
$ docker inspect 930f29ccdf8a
# 查看容器网络信息:
$ docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' 容器id
```

#### 13.3）：查看容器端口信息

```shell
# 命令格式：
docker port [容器id]
# 命令效果：
$ docker port 930f29ccdf8a
# 没有效果没有和宿主机关联
```

#### 13.4）：容器重命名

```shell
# 命令格式：
docker rename [容器id]或[容器名称] [容器新名称]
# 命令效果：
$ docker rename 930f29ccdf8a newname
```



## 四、Docker数据管理

### 1）：数据卷之目录

```shell
# 命令格式：
docker run -itd --name [容器名字] -v [宿主机目录]:[容器目录][镜像名称] [命令(可选)]
# 命令演示：
# 创建测试文件:
echo "file1" > tmp/file1.txt
# 启动一个容器，挂载数据卷:
docker run -itd --name test-nginx -v /home/darry/testdocker/:/dockertest/ nginx
# 注意宿主机目录需要绝对路径
# 测试效果
docker exec -it 4470d3ff34b8 /bin/bash
# 执行相关的目录操作命令
```

### 2）：数据卷容器

#### 2.1）：创建一个数据圈容器

```shell
# 命令格式：
docker create -v [容器数据卷目录] --name [容器名字][镜像名称] [命令(可选)]
# 执行效果
docker create -v /data --name v1-nginx nginx
```

#### 2.2）：创建两个容器，同时挂载数据卷容器

```shell
# 命令格式：
docker run --volumes-from [数据卷容器id/name] -tid --name [容器名字][镜像名称] [命令(可
选)]
# 执行效果：
# 创建 v1-test1 容器:
docker run --volumes-from d4bf68189dc4 -tid --name v1-test1 nginx /bin/bash
# 创建 v1-test2 容器:
docker run --volumes-from d4bf68189dc4 -tid --name v1-test2 nginx /bin/bash
```

#### 2.3）：确认卷容器共享

```shell
# 1、先进v1-test1容器
docker exec -it v1-test1 /bin/bash
# 在v1-test里面进行相应的操作
# 2、先进v1-test2容器
docker exec -it v1-test2 /bin/bash
# 在v2-test里面进行v1-test操作的验证
```

#### 3）：数据备份

数据备份方案： 

> 1. 创建一个挂载数据卷容器的容器
> 2. 挂载宿主机本地目录作为备份数据卷
> 3. 将数据卷容器的内容备份到宿主机本地目录挂载的数据卷中
> 4. 完成备份操作后销毁刚创建的容器

```shell
# 命令格式：
docker run --rm --volumes-from [数据卷容器id/name] -v [宿主机目录]:[容器目录][镜像名称]
# [备份命令]
# 命令演示：
# 创建备份目录:
mkdir /home/darry/testdocker/dockerdata
# 创建备份的容器:
docker run --rm --volumes-from v1-nginx -v /home/darry/testdocker/dockerdata/:/dockerdata/ nginx tar zcPf /dockerdata/data.tar.gz /data
#验证操作:
ls /backup
zcat /backup/data.tar.gz
```

### 