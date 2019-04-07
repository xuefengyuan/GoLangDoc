[TOC]

## 五、Dockerfile

### 1、Dockerfile

> 什么是Dockerfile Dockerfile类似于我们的脚本，将我们在上面的docker镜像，使用自动化的方式实
> 现出来。

#### **1.1、Dockerfile的作用** 

> 1. 找一个镜像： ubuntu
> 2. 创建一个容器： docker run ubuntu
> 3. 进入容器： docker exec -it 容器 命令
> 4. 操作： 各种应用配置....
> 5. 构造新镜像： docker commit

#### <font color=red>**1.2、Dockerfile 使用准则** </font>

> <font color=red>1. 大： 首字母必须大写D。</font>
> <font color=red>2. 空： 尽量将Dockerfile放在空目录中。</font>
> <font color=red>3. 单： 每个容器尽量只有一个功能。</font>
> <font color=red>4. 少： 执行的命令越少越好。</font>

#### **1.3、Dockerfile 分为四部分:**

> 1. 基础镜像信息 
> 2. 维护者信息
> 3. 镜像操作指令
> 4. 容器启动时执行指令

#### **1.4、Dockerfile使用命令：**

```shell
# 构建镜像命令格式：
docker build -t [镜像名]:[版本号][Dockerfile所在目录]
# 构建样例：
$ docker build -t nginx:v0.2 /opt/dockerfile/nginx/
# 参数详解：
-t 指定构建后的镜像信息，
/opt/dockerfile/nginx/ 则代表Dockerfile存放位置，如果是当前目录，则用 .(点)表示
```



### 2、Dockerfile快速入门

#### 2.1）：创建Dockerfile专用目录

```shell
# 创建Dockerfile专用目录
$ mkdir -p docker/dockerfile/nginx
$ cd docker/dockerfile/nginx
# 创建Dockerfile文件
$ vim Dockerfile
```

#### 2.2）：dockerfile内容

```shell
# 构建一个基于ubuntu的docker定制镜像
# 基础镜像
FROM ubuntu
# 镜像作者
MAINTAINER ybd fkewksai@163.com
# 执行命令
RUN mkdir hello
RUN mkdir world
RUN sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
RUN sed -i 's/security.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
RUN apt-get update
RUN apt-get install nginx -y
# 对外端口
EXPOSE 80
```

这里只对执行命令进行注释说明

```shell
# 执行命令
RUN mkdir hello  # 创建hello文件夹
RUN mkdir world  # 合建world文件夹
# 设置源
RUN sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
RUN sed -i 's/security.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
# 更新软件源
RUN apt-get update
# 安装nginx
RUN apt-get install nginx -y
```

#### 2.3）：进行构建操作

```shell
# 构建镜像 
# /home/darry/docker/dockerfile/nginx
$ docker build -t ubuntu-nginx:v0.1 .
# 查看新生成镜像
$ docker images
# 查看构建历史
$ docker history 9d27415808b1
$ docker history ubuntu-nginx:v0.1
# 注意：
因为容器没有启动命令，所以肯定访问不了
```

#### 2.4）：优化刚刚的Dockerfile文件

```shell
$ cp -r nginx/ nginx-2/
$ cd nginxg-2
$ vim Dockerfile
```

```shell
# 构建一个基于ubuntu的docker定制镜像
# 基础镜像
FROM ubuntu
# 镜像作者
MAINTAINER ybd fkewksai@163.com
# 执行命令
RUN mkdir hello && mkdir world
RUN sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list && sed -i 's/security.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
RUN apt-get update && apt-get install nginx -y
# 对外端口
EXPOSE 80
```

这里对优化后的执行命令进行注释说明

```shell
# 执行命令，优化后的命令一条命令执行多个操作之间用 && 连接，左右需要空格格开
# 创建两个文件夹，也可以有多个
RUN mkdir hello && mkdir world
# 设置两个源到源文件
RUN sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list && sed -i 's/security.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
# 同时执行更新和安装命令
RUN apt-get update && apt-get install nginx -y
```

#### 2.5：）运行修改好的Dockerfile进行构建

```shell
$ docker build -t ubuntu-nginx:v0.2 .
# 查看镜像
$ docker images
# 深度对比连个镜像的大小
$ docker inspect ubuntu-nginx:v0.1
$ docker inspect ubuntu-nginx:v0.1
# 发现运行第二个构建命令的镜像比第一个小一些。
```

<font color=red>**Dockerfile构建过程：**</font>

> 从基础镜像1运行一个容器A
> 遇到一条Dockerfile指令，都对容器A做一次修改操作
> 执行完毕一条命令，提交生成一个新镜像2
> 再基于新的镜像2运行一个容器B
>
> 遇到一条Dockerfile指令，都对容器B做一次修改操作
> 执行完毕一条命令，提交生成一个新镜像3

<font color=red>**构建过程镜像介绍**</font>

> 构建过程中，创建了很多镜像，这些中间镜像，可以直接使用来启动容器，通过查看容器效果，从侧面能看到每次构建的效果。提供了镜像调试的能力 



### 3、 基础指令详解

#### 3.1）：FROM

```shell
FROM
# 格式：
FROM <image>
FROM <image>:<tag>
# 解释：
# FROM 是 Dockerfile 里的第一条而且只能是除了首行注释之外的第一条指令
# 可以有多个FROM语句，来创建多个image
# FROM 后面是有效的镜像名称，如果该镜像没有在你的本地仓库，那么就会从远程仓库Pull取，如果远程也没有，就报错失败
# 下面所有的 系统可执行指令 在 FROM 的镜像中执行。
```

#### 3.2）：MAINTAINER

```shell
MAINTAINER
# 格式：
MAINTAINER <name>
# 解释：
# 指定该dockerfile文件的维护者信息。类似我们在docker commit 时候使用-a参数指定的信息
```

#### 3.3）：RUN

```shell
RUN
# 格式：
RUN <command> (shell模式)
RUN["executable", "param1", "param2"] (exec 模式)
# 解释：
# 表示当前镜像构建时候运行的命令，如果有确认输入的话，一定要在命令中添加 -y
# 如果命令较长，那么可以在命令结尾使用 \ 来换行
# 生产中，推荐使用上面数组的格式
# 注释：
# shell模式：类似于 /bin/bash -c command
# 举例： RUN echo hello
# exec模式：类似于 RUN["/bin/bash", "-c", "command"]
# 举例： RUN["echo", "hello"]
```

#### 3.4）：EXPOSE

```shell
EXPOSE
# 格式：
EXPOSE <port> [<port>...]
# 解释：
# 设置Docker容器对外暴露的端口号，Docker为了安全，不会自动对外打开端口，如果需要外部提供访问，还需要启动容器时增加-p或者-P参数对容器的端口进行分配。
```



### 4、运行时指令详解

#### 4.1）：CMD

```shell
CMD
# 格式：
    CMD ["executable","param1","param2"] (exec 模式)推荐
    CMD command param1 param2 (shell模式)
    CMD ["param1","param2"] 提供给ENTRYPOINT的默认参数；
# 解释：
    # CMD指定容器启动时默认执行的命令
    # 每个Dockerfile只能有一条CMD命令，如果指定了多条，只有最后一条会被执行
    # 如果你在启动容器的时候使用docker run 指定的运行命令，那么会覆盖CMD命令。
    # 举例： CMD ["/usr/sbin/nginx","-g","daemon off；"]
    "/usr/sbin/nginx" nginx命令
    "-g" 设置配置文件外的全局指令
	"daemon off；" 后台守护程序开启方式 关闭
```

```shell
# CMD指令实践:
$ cp -r nginx-2/ nginx-3/
$ cd nginx-3/
$ vim Dockerfile
    # 修改Dockerfile文件内容：
    # 在上一个Dockerfile文件内容基础上，在EXPOSE之前增加下面一句话：
    $ CMD ["/usr/sbin/nginx","-g","daemon off;"]
    # 构建镜像
    $ docker build -t ubuntu-nginx:v0.3 .
    # 根据镜像创建容器,创建时候，不添加执行命令
    $ docker run -itd --name nginx-01 ubuntu-nginx:v0.3
    # 根据镜像创建容器,创建时候，添加执行命令/bin/bash
    $ docker run --name nginx-02 -itd ubuntu-nginx:v0.3 /bin/bash
    $ docker ps
    # 会发现两个容器的命令是不一样的
```

#### 4.2）：ENTRYPOINT

```shell
ENTRYPOINT
# 格式：
ENTRYPOINT ["executable", "param1","param2"] (exec 模式)
ENTRYPOINT command param1 param2 (shell 模式)
# 解释：
# 和CMD 类似都是配置容器启动后执行的命令，并且不会被docker run 提供的参数覆盖。
# 每个Dockerfile 中只能有一个ENTRYPOINT，当指定多个时，只有最后一个起效。
# 生产中我们可以同时使用ENTRYPOINT 和CMD，
# 想要在docker run 时被覆盖，可以使用"docker run --entrypoint"
```

```shell
# ENTRYPOINT指令实践：
$ cp -r nginx-3/ nginx-4/
$ cd nginx-4/
$ vim Dockerfile
    # 修改Dockerfile文件内容：
    # 在上一个Dockerfile 文件内容基础上，修改末尾的CMD 为ENTRYPOINT：
    ENTRYPOINT ["/usr/sbin/nginx","-g","daemon off;"]
    # 构建镜像
    $ docker build -t ubuntu-nginx:v0.4 .
    # 根据镜像创建容器,创建时候，不添加执行命令
    $ docker run -itd --name nginx-03 ubuntu-nginx:v0.4
    # 根据镜像创建容器,创建时候，添加执行命令/bin/bash
    $ docker run -itd --name nginx-04 ubuntu-nginx:v0.4 /bin/bash
	# 查看ENTRYPOINT是否被覆盖，可以看到命令并没有覆盖
	$ docker ps -a
	# 根据镜像创建容器,创建时候，使用--entrypoint参数，添加执行命令/bin/bash
	$ docker run --entrypoint "/bin/bash" --name nginx-05 -itd ubuntu-nginx:v0.4
	# 查看ENTRYPOINT是否被覆盖，可以看到命令跟之前两个容器不一样了，被覆盖了
	$ docker ps -a
```

#### 4.3）：CMD ENTRYPOINT 综合使用

```shell
# CMD ENTRYPOINT 综合实践
$ cp -r nginx-2/ nginx-5/
$ cd nginx-5
$ vim Dockerfile
	# 修改Dockerfile文件内容：
    # 在上一个Dockerfile文件内容基础上，修改末尾的ENTRYPOINT
    ENTRYPOINT ["/usr/sbin/nginx"]
    CMD ["-g"]
    # 构建镜像
    $ docker build -t ubuntu-nginx:v0.5 .
    # 根据镜像创建容器,创建时候，不添加执行命令
    $ docker run --name nginx-06 -d ubuntu-nginx:v0.5
    # 查看效果
    $ docker ps -a
    # 根据镜像创建容器,创建时候，不添加执行命令，覆盖cmd的参数 -g "daemon off;"
    $ docker run --name nginx-07 -d ubuntu-nginx:v0.5 -g "daemon off;"
    # 查看效果
    $ docker ps -a
# 注释：
# 任何docker run设置的命令参数或者CMD指令的命令，都将作为ENTRYPOINT 指令的命令参数，追加到ENTRYPOINT指令之后
```

Dockerfile脚本内容

```shell
# 构建一个基于ubuntu的docker定制镜像
# 基础镜像
FROM ubuntu
# 镜像作者
MAINTAINER ybd fkewksai@163.com
# 执行命令
RUN mkdir hello && mkdir world
RUN sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list && sed -i 's/security.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
RUN apt-get update && apt-get install nginx -y
ENTRYPOINT ["/usr/sbin/nginx"]
CMD ["-g"]
# 对外端口
EXPOSE 80
```

### 5、文件编辑指令

#### 5.1）：ADD

解释：

> 将指定的<src> 文件复制到容器文件系统中的<dest>
>
> src 指的是宿主机，dest 指的是容器
>
> 所有拷贝到container 中的文件和文件夹权限为0755,uid 和gid 为0
>
> 如果文件是可识别的压缩格式，则docker 会帮忙解压缩

```shell
# ADD
    # 格式：
    ADD <src>... <dest>
    ADD ["<src>",... "<dest>"]
    # 注意：
    # 1、如果源路径是个文件，且目标路径是以/ 结尾， 则docker 会把目标路径当作一个目录，会把源文件拷贝到该目录下;
    # 如果目标路径不存在，则会自动创建目标路径。
    # 2、如果源路径是个文件，且目标路径是不是以/ 结尾，则docker 会把目标路径当作一个文件。
    # 如果目标路径不存在，会以目标路径为名创建一个文件，内容同源文件；
    # 如果目标文件是个存在的文件，会用源文件覆盖它，当然只是内容覆盖，文件名还是目标文件名。
    # 如果目标文件实际是个存在的目录，则会源文件拷贝到该目录下。注意，这种情况下，最好显示的以/ 结尾，以避免混淆。
    # 3、如果源路径是个目录，且目标路径不存在，则docker 会自动以目标路径创建一个目录，把源路径目录    下的文件拷贝进来。
    # 如果目标路径是个已经存在的目录，则docker 会把源路径目录下的文件拷贝到该目录下。
	# 4、如果源文件是个压缩文件，则docker 会自动帮解压到指定的容器目录中。
```

```shell
# ADD实践：
# 1、拷贝普通文件
$ mkdir nginx-6
$ cd nginx-6
$ vim Dockerfile
# Dockerfile文件内容

# 构建一个基于ubuntu的docker定制镜像
# ------- Dockerfile start -------
# 基础镜像
FROM ubuntu
# 镜像作者
MAINTAINER ybd fkewksai@163.com
# 执行命令,这里只是复制一个文件，可以替换成其它的文件测试
ADD ["sources.list","/etc/apt/sources.list"]
RUN apt-get clean
RUN apt-get update
RUN apt-get install nginx -y
# 对外端口
EXPOSE 80
# ------- Dockerfile end -------

# 复制文件到当前目录下
$ sudo cp /etc/apt/sources.list ./
# 构建镜像,上面是添加一个源文件，然后更新源，再安装nginx，有问题的话，换其它文件测试，只是一个添加文件功能
$ docker build -t ubuntu-nginx:v0.6 .
# 根据镜像创建容器,创建时候，不添加执行命令进入容器查看效果
$ docker run --name nginx-08 -it ubuntu-nginx:v0.6

# ==================================

# 2、拷贝压缩文件
tar zcvf this.tar.gz ./*
# Dockerfile文件内容
...
# 执行命令
...
# 增加文件
ADD ["linshi.tar.gz","/nihao/"]
...
# 构建镜像
$ docker build -t ubuntu-nginx:v0.7 .
# 根据镜像创建容器,创建时候，不添加执行命令进入容器查看效果
$ docker run --name nginx-09 -it ubuntu-nginx:v0.7
```

#### 5.2）：COPY

> COPY 指令和ADD 指令功能和使用方式类似。只是COPY 指令不会做自动解压工作。
>
> 单纯复制文件场景，Docker 推荐使用COPY

```shell
# COPY
    # 格式：
    COPY <src>... <dest>
    COPY ["<src>",... "<dest>"]
	# 命令样式 1、从宿主复制文件到容器
	docker cp 本地文件 容器ID:容器存放文件位置
	docker cp ./go1.11.linux-amd64.tar.gz 5a855d2c35dc:/gofile
	# 命令样式 2、从容器复制文件到宿主
	docker cp  容器ID:容器存放文件位置 本地文件
	docker cp  5a855d2c35dc:/etc/apt/sources.list ./
```

```shell
# COPY实践
$ mkdir nginx-7
$ cd nginx-7
$ vim Dockerfile
# 修改Dockerfile文件内容:
# ========== Dockerfile start ================
# 构建一个基于ubuntu的docker定制镜像
# 基础镜像
FROM ubuntu
# 镜像作者
MAINTAINER ybd fkewksai@163.com
# 执行命令
COPY ["index.html","/var/www/html/"]
# 对外端口
EXPOSE 80
# 运行时默认命令
ENTRYPOINT ["/usr/sbin/nginx","-g","daemon off;"]
# ========== Dockerfile end ================

$ vim index.html
index.html 文件内容：

<h1>hello world </h1>
<h1>hello docker </h1>
<h1>hello nginx</h1>

# 构建镜像
$ docker build -t ubuntu-nginx:v0.8 .
# 根据镜像创建容器,创建时候，不添加执行命令
$ docker run --name nginx-10 -itd ubuntu-nginx:v0.8
# 查看nginx-10信息
$ docker inspect nginx-10
# 浏览器访问nginx查看效果
```

#### 5.3：）VOLUME

```shell
# VOLUME
    # 格式：
    VOLUME ["/data"]
    # 解释：
    # VOLUME 指令可以在镜像中创建挂载点，这样只要通过该镜像创建的容器都有了挂载点
    # 通过VOLUME 指令创建的挂载点，无法指定主机上对应的目录，是自动生成的。
    # 举例：
    VOLUME ["/var/lib/tomcat7/webapps/"]
```

```shell
# VOLUME实践
# 修改Dockerfile文件内容：
# 将COPY替换成为VOLUME
$ vim Dockerfile
VOLUME ["/helloworld/"]
# =========================================
# 构建镜像
$ docker build -t ubuntu-nginx:v0.9 .
# 创建数据卷容器
$docker run -itd --name nginx-11 ubuntu-nginx:v0.9
# 查看镜像信息
$ docker inspect nginx-11

# 验证操作
$ docker run -itd --name vc-nginx01 --volumes-from nginx-11 nginx
$ docker run -itd --name vc-nginx02 --volumes-from nginx-11 nginx

# 进入容器1
$ docker exec -it vc-nginx01 /bin/bash
# 进入容器2
$ docker exec -it vc-nginx02 /bin/bash
```



### 6、环境指令

#### 6.1）：ENV

```shell
# E NV
    # 格式：
    ENV <key> <value> （一次设置一个环节变量）
    ENV <key>=<value> ... （一次设置一个或多个环节变量）
# 解释：
# 设置环境变量，可以在RUN 之前使用，然后RUN 命令时调用，容器启动时这些环境变量都会被指定
```

**EVN实践**

```shell
# ENV实践：
# 命令行创建ENV的容器
$ docker run -e NIHAO="helloworld" -itd --name ubuntu-13 ubuntu /bin/bash
# 进入容器ubuntu-13
$ docker exec -it ubuntu-13 /bin/bash
:/# echo $NIHAO

# 修改Dockerfile文件内容：
# 在上一个Dockerfile 文件内容基础上，在RUN 下面增加一个ENV
ENV NIHAO=helloworld
...
# 构建镜像
$ docker build -t ubuntu-nginx:v0.10 .
# 根据镜像创建容器,创建时候，不添加执行命令
$ docker run --name nginx-14 -itd ubuntu-nginx:v0.10
# 进入容器
$ docker exec -it nginx-14 /bin/bash
$ echo $NIHAO
```

#### 6.2）：WORKDIR

```shell
# WORKDIR
    # 格式：
    WORKDIR /path/to/workdir (shell 模式)
    # 解释：
    # 切换目录，为后续的RUN、CMD、ENTRYPOINT 指令配置工作目录。相当于cd
    # 可以多次切换(相当于cd 命令)，
    # 也可以使用多个WORKDIR 指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。例如
    # 举例：
    WORKDIR /a
    WORKDIR b
    WORKDIR c
    RUN pwd
    # 则最终路径为/a/b/c
```

**WORKDIR实践**

```shell
# WORKDIR实践：
# 修改Dockerfile文件内容：
# 在上一个Dockerfile 文件内容基础上，在RUN 下面增加一个WORKDIR
WORKDIR /nihao/test/
RUN ["touch","test1.txt"]
WORKDIR /nihao
RUN ["touch","test2.txt"]
WORKDIR testb
RUN ["touch","test3.txt"]
...
# 构建镜像
$ docker build -t ubuntu-nginx:v0.11 .
# 根据镜像创建容器,创建时候，不添加执行命令
$ docker run --name nginx-16 -itd ubuntu-nginx:v0.11
# 进入镜像
$ docker exec -it nginx-16 /bin/bash
```



### 7 、触发器指令详解

#### 7.1）：触发器指令

```shell
ONBUILD
# 格式：
	ONBUILD [command]
# 解释：
    # 当一个镜像A被作为其他镜像B的基础镜像时，这个触发器才会被执行，
    # 新镜像B在构建的时候，会插入触发器中的指令。
    # 使用场景对于版本控制和方便传输，适用于其他用户。
```

**触发器实践**

```shell
# 编辑Dockerfile
vim Dockerfile
# 内容如下：
# 构建一个基于ubuntu的docker定制镜像
# 基础镜像
FROM ubuntu
# 镜像作者
MAINTAINER ybd fkewksai@163.com
# 执行命令
ADD ["sources.list","/etc/apt/sources.list"]
RUN apt-get clean
RUN apt-get update
RUN apt-get install nginx -y
# 触发器
ONBUILD COPY ["index.html","/var/www/html/"]
# 对外端口
EXPOSE 80
# 运行时默认命令
ENTRYPOINT ["/usr/sbin/nginx","-g","daemon off;"]

# ==========================
# 构建镜像
$ docker build -t ubuntu-nginx:v0.12 .
# 根据镜像创建容器,
$ docker run -p 80 --name nginx-17 -itd ubuntu-nginx:v0.12
$ docker ps
# 查看镜像信息
$ docker inspect ubuntu-nginx:v0.12
# 访问容器页面，是否被更改
$ docker inspect nginx-17
```

```shell
# 构建子镜像Dockerfile文件
FROM ubuntu-nginx:v0.12
MAINTAINER ybd fkewksai@163.com
EXPOSE 80
ENTRYPOINT ["/usr/sbin/nginx","-g","daemon off;"]

# 构建子镜像
$ docker build -t ubuntu-nginx:v0.13 .
# 根据镜像创建容器,
$ docker run -p 80 --name nginx18 -itd ubuntu-nginx:v0.13
# 查看镜像信息
$ docker inspect ubuntu-nginx:v0.13
$ docker ps
# 访问容器页面，是否被更改
```

