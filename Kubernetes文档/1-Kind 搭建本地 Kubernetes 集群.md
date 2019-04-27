[TOC]

## 一、Kind安装

系统环境是Ubuntu18.04

内存：8G

CPU：2核4线程

>  如果是单节点，可以降低配置，如果想高可用集群，内存差不多要8G

详细的文档可以看网络的

[使用 Kind 搭建你的本地 Kubernetes 集群](https://zhuanlan.zhihu.com/p/60464867)

根据链接上的方式安装，可能会遇到下载kind失败的问题，可采用科学上网的方式，下载对应的可执行文件然后再后续操作

如：

```shell
# 移动到/usr相关目录下
$ sudo mv kind-linux-amd64 /usr/local/bin/kind
# 添加可执行权限
$ sudo chmod +x /usr/local/bin/kind
# 验证
$ kind --version
kind version 0.2.0
```

## 二、依赖

下载kind可以采用源码编译或者go get方式安装，所以安装了go的环境

Docker，安装完成后记得需要添加到用户组，注意sudo权限问题 [Docker安装文档](https://docs.docker.com/install/)，官方安装可能会比较繁琐

如果需要操作集群，则需要安装 `kubectl` 命令行 [官方文档](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl)

以下安装来自官方：

```shell
$ sudo snap install kubectl --classic

$ kubectl version
Client Version: version.Info{Major:"1", Minor:"14", GitVersion:"v1.14.0", GitCommit:"641856db18352033a0d96dbc99153fa3b27298e5", GitTreeState:"clean", BuildDate:"2019-03-25T15:53:57Z", GoVersion:"go1.12.1", Compiler:"gc", Platform:"linux/amd64"}
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

## 三、搭建单点集群

### 1、搭建集群

这种方式没有成功过，所以要用下面指定配置文件的方式搭建点单集群

```shell
$ kind create cluster --name moelove
```

> 以上命令中， `--name` 是可选参数，如不指定，默认创建出来的集群名字为 `kind`。

### 2、删除集群

```shell
$ kind delete  cluster --name moelove
```

### 3、通过配置文件搭建单点集群

```shell
$ vim kind-config.yaml
```

kind-config.yaml 配置文件内容

```yaml
kind: Cluster
apiVersion: kind.sigs.k8s.io/v1alpha3
kubeadmConfigPatches:
- |
  apiVersion: kubeadm.k8s.io/v1beta1
  kind: ClusterConfiguration
  metadata:
    name: config
  networking:
    serviceSubnet: 10.0.0.0/16
  imageRepository: registry.aliyuncs.com/google_containers
  nodeRegistration:
    kubeletExtraArgs:
      pod-infra-container-image: registry.aliyuncs.com/google_containers/pause:3.1
- |
  apiVersion: kubeadm.k8s.io/v1beta1
  kind: InitConfiguration
  metadata:
    name: config
  networking:
    serviceSubnet: 10.0.0.0/16
  imageRepository: registry.aliyuncs.com/google_containers
nodes:
- role: control-plane
```

执行命令创建节点

```shell
$ kind create cluster --name moelove --config kind-config.yaml

# 输出以下内容表示搭建成功
Creating cluster "moelove" ...
 ✓ Ensuring node image (kindest/node:v1.13.4) 🖼
 ✓ Preparing nodes 📦 
 ✓ Creating kubeadm config 📜 
 ✓ Starting control-plane 🕹️ 
Cluster creation complete. You can now use the cluster with:

export KUBECONFIG="$(kind get kubeconfig-path --name="moelove")"
kubectl cluster-info

# 查看拉取的镜像
$ docker images
# 查看容器
$ coocker ps

```

## 四、通过配置文件搭建高可用集群

```shell
$ vim kind-ha-config.yaml
```

kind-ha-config.yaml 配置文件内容

```yaml
kind: Cluster
apiVersion: kind.sigs.k8s.io/v1alpha3
kubeadmConfigPatches:
- |
  apiVersion: kubeadm.k8s.io/v1beta1
  kind: ClusterConfiguration
  metadata:
    name: config
  networking:
    serviceSubnet: 10.0.0.0/16
  imageRepository: registry.aliyuncs.com/google_containers
  nodeRegistration:
    kubeletExtraArgs:
      pod-infra-container-image: registry.aliyuncs.com/google_containers/pause:3.1
- |
  apiVersion: kubeadm.k8s.io/v1beta1
  kind: InitConfiguration
  metadata:
    name: config
  networking:
    serviceSubnet: 10.0.0.0/16
  imageRepository: registry.aliyuncs.com/google_containers
nodes:
- role: control-plane
- role: control-plane
- role: control-plane
- role: worker
- role: worker
- role: worker
```

执行命令创建节点

```shell
$ kind create cluster --name moelove-ha --config kind-ha-config.yaml

# 出现以下信息，表示高可用集群创建成功
Creating cluster "moelove-ha" ...
 ✓ Ensuring node image (kindest/node:v1.13.4) 🖼
 ✓ Preparing nodes 📦📦📦📦📦📦📦 
 ✓ Starting the external load balancer ⚖️ 
 ✓ Creating kubeadm config 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Joining more control-plane nodes 🎮 
 ✓ Joining worker nodes 🚜 
Cluster creation complete. You can now use the cluster with:

export KUBECONFIG="$(kind get kubeconfig-path --name="moelove-ha")"
kubectl cluster-info

# 添加临时环境变量
$ export KUBECONFIG="$(kind get kubeconfig-path --name="moelove-ha")"
# 查看集群信息
$ kubectl cluster-info

```

## 五、查看相关信息

```shell
$ kubectl cluster-info
$ kubectl get nodes
```







































