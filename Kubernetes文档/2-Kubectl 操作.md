[TOC]

## 一、Kubectl

### 1、Kubectl基本操作

安装Kubectl在之前Kind环境的时候已安装，可以直接使用

```shell
$ kubectl
# 查看所有全局可用的配置项
$ kubectl options
# 查看node节点信息,注意在创建集群后，需要配置一个临时环境变量，不然无法访问
$ kubectl get nodes
# 查看node节点更多信息
$ kubectl get nodes -o wide

```

> 也可以传递 `-o yaml` 或者 `-o json` 得到更加详尽的信息。

```shell
# 使用 -o json 将内容以 JSON 格式输出时，可以配合 jq 进行内容提取
$ kubectl get nodes -o json | jq ".items[] | {name: .metadata.name} + .status.nodeInfo"

# 查看服务端支持的 API 资源及别名和描述等信息。
$ kubectl api-resources
```

### 2、Kubectl Run

```shell
# 基础语法
$ kubectl run NAME --image=image [--env="key=value"] [--port=port] [--replicas=replicas] [--dry-run=bool] [--overrides=inline-json] [--command] -- [COMMAND] [args...] [options]
```

> `NAME` 和 `--image` 是必需项。分别代表此次部署的名字及所使用的镜像，在实际使用时，推荐编写配置文件并通过 `kubectl create` 进行部署

#### 2.1、部署一个Redis

```shell
# 实际操作
$ kubectl run redis --image='redis:alpine'
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/redis created
# 查看结果
$ kubectl get all
ME                         READY   STATUS    RESTARTS   AGE
pod/redis-658d78cf9c-gb9qc   1/1     Running   0          86s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.0.0.1     <none>        443/TCP   58m

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/redis   1/1     1            1           86s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/redis-658d78cf9c   1         1         1       86s

```

### 3、Deployment

`Deployment` 是一种高级别的抽象，允许我们进行扩容，滚动更新及降级等操作。我们使用 `kubectl run redis --image='redis:alpine` 命令便创建了一个名为 `redis` 的 `Deployment`，并指定了其使用的镜像为 `redis:alpine`。

同时 K8S 会默认为其增加一些标签（`Label`）。我们可以通过更改 `get` 的输出格式进行查看。

```shell
# 验证
$ kubectl get deployment.apps/redis -o wide
NAME    READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES         SELECTOR
redis   1/1     1            1           4m12s   redis        redis:alpine   run=redis
$ kubectl get deploy redis -o wide
NAME    READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES         SELECTOR
redis   1/1     1            1           6m31s   redis        redis:alpine   run=redis
```

那么这些 `Label` 有什么作用呢？它们可作为选择条件进行使用。如：

```shell
# 验证测试
$ kubectl get deploy -l run=redis -o wide
NAME    READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES         SELECTOR
redis   1/1     1            1           7m44s   redis        redis:alpine   run=redis
# 由于并没有创建过test，所以查不到任何东西
$ kubectl get deploy -l run=test -o wide
No resources found.
```

我们在应用部署或更新时总是会考虑的一个问题是如何平滑升级，利用 `Deployment` 也能很方便的进行金丝雀发布（Canary deployments）。这主要也依赖 `Label` 和 `Selector`。

`Deployment` 的创建除了使用我们这里提到的方式外，更推荐的方式便是使用 `yaml` 格式的配置文件。在配置文件中主要是声明一种预期的状态，而其他组件则负责协同调度并最终达成这种预期的状态。当然这也是它的关键作用之一，将 `Pod` 托管给下面将要介绍的 `ReplicaSet`。

### 4、ReplicaSet

`ReplicaSet` 是一种较低级别的结构，允许进行扩容。

上面已经提到 `Deployment` 主要是声明一种预期的状态，并且会将 `Pod` 托管给 `ReplicaSet`，而 `ReplicaSet` 则会去检查当前的 `Pod` 数量及状态是否符合预期，并尽量满足这一预期。

<font color=blue>`ReplicaSet` 可以由我们自行创建，但一般情况下不推荐这样去做，因为如果这样做了，那其实就相当于跳过了 `Deployment` 的部分，`Deployment` 所带来的功能或者特性便都使用不到了。</font>

除了 `ReplicaSet` 外，还有一个选择名为 `ReplicationController`，这两者的主要区别更多的在选择器上，后面再做讨论。现在推荐的做法是 `ReplicaSet` 所以不做太多解释。

`ReplicaSet` 可简写为 `rs`，通过以下命令查看：

```shell
$ kubectl get rs -o wide
NAME               DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES         SELECTOR
redis-658d78cf9c   1         1         1       13m   redis        redis:alpine   pod-template-hash=658d78cf9c,run=redis

```

> 在输出结果中，注意到这里除了我们前面看到的 `run=redis` 标签外，还多了一个 `pod-template-hash=3731017676` 标签，这个标签是由 `Deployment controller` 自动添加的，目的是为了防止出现重复，所以将 `pod-template` 进行 hash 用作唯一性标识。

### 5、Service

`Service` 简单点说就是为了能有个稳定的入口访问我们的应用服务或者是一组 `Pod`。通过 `Service`可以很方便的实现服务发现和负载均衡。

```shell
$ kubectl get service -o wide
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   SELECTOR
kubernetes   ClusterIP   10.0.0.1     <none>        443/TCP   82m   <none>
```

通过使用 `kubectl` 查看，能看到主要会显示 `Service` 的名称，类型，IP，端口及创建时间和选择器等。

#### 5.1、Service类型

`Service` 目前有 4 种类型：

- `ClusterIP`： 是 K8S 当前默认的 `Service` 类型。将 service 暴露于一个仅集群内可访问的虚拟 IP 上。
- `NodePort`： 是通过在集群内所有 `Node` 上都绑定固定端口的方式将服务暴露出来，这样便可以通过 `<NodeIP>:<NodePort>` 访问服务了。
- `LoadBalancer`： 是通过 `Cloud Provider` 创建一个外部的负载均衡器，将服务暴露出来，并且会自动创建外部负载均衡器路由请求所需的 `Nodeport` 或 `ClusterIP` 。
- `ExternalName`： 是通过将服务由 DNS CNAME 的方式转发到指定的域名上将服务暴露出来，这需要 `kube-dns` 1.7 或更高版本支持。

#### 5.2、实践

```shell
$ kubectl expose deploy/redis --port=6379 --protocol=TCP --target-port=6379 --name=redis-server  
service/redis-server exposed

$ kubectl get svc -o wide
NAME           TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE    SELECTOR
kubernetes     ClusterIP   10.0.0.1      <none>        443/TCP    101m   <none>
redis-server   ClusterIP   10.0.185.29   <none>        6379/TCP   77s    run=redis

```

通过 `kubectl expose` 命令将 redis server 暴露出来，这里需要进行下说明：

- `port`： 是 `Service` 暴露出来的端口，可通过此端口访问 `Service`。
- `protocol`： 是所用协议。当前 K8S 支持 TCP/UDP 协议，在 1.12 版本中实验性的加入了对 [SCTP 协议](https://link.juejin.im/?target=https%3A%2F%2Fzh.wikipedia.org%2Fzh-hans%2F%25E6%25B5%2581%25E6%258E%25A7%25E5%2588%25B6%25E4%25BC%25A0%25E8%25BE%2593%25E5%258D%258F%25E8%25AE%25AE)的支持。默认是 TCP 协议。
- `target-port`： 是实际服务所在的目标端口，请求由 `port` 进入通过上述指定 `protocol` 最终流向这里配置的端口。
- `name`： `Service` 的名字，它的用处主要在 dns 方面。
- `type`： 是前面提到的类型，如果没指定默认是 `ClusterIP`。

现在的 redis 是使用的默认类型 `ClusterIP`，所以并不能直接通过外部进行访问，要使用 `port-forward` 的方式让它可在集群外部访问。

```shell
# 启动Redis让外部访问
$ kubectl port-forward svc/redis-server 6379:6379
Forwarding from 127.0.0.1:6379 -> 6379
Forwarding from [::1]:6379 -> 6379
Handling connection for 6379
```

在本地新开一个终端使用redis-cli工具连接

```shell
# 如果提示未安装，则需要先进行安装
$ sudo apt install redis-tools -y

$ redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> ping
PONG

```

也可以使用 `NodePort` 的方式对外暴露服务。

```shell
$ kubectl expose deploy/redis --port=6379 --protocol=TCP --target-port=6379 --name=redis-server-nodeport --type=NodePort
service/redis-server-nodeport exposed
$ kubectl get service/redis-server-nodeport -o wide
NAME                    TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE   SELECTOR
redis-server-nodeport   NodePort   10.0.161.158   <none>        6379:31621/TCP   88s   run=redis

# 启动Redis让外部访问
$ kubectl port-forward svc/redis-server 6379:6379
Forwarding from 127.0.0.1:6379 -> 6379
Forwarding from [::1]:6379 -> 6379
Handling connection for 6379

```

可以通过任意 `Node` 上的 31913 端口便可连接我们的 redis 服务。当然，这里需要注意的是这个端口范围其实是可以通过 `kube-apiserver` 的 `service-node-port-range` 进行配置的，默认是 `30000-32767`。

在本地新开一个终端使用redis-cli工具连接

```shell
# 如果提示未安装，则需要先进行安装
$ sudo apt install redis-tools -y

# 先查看docker 容器对应的ip，然后进行访问
$ docker ps
$ docker inspect 容器id/容器名称
# 找到IPAddress就是容器对应的ip了
 "IPAddress": "172.17.0.9",

# 连接Redis
$ redis-cli -h 172.17.0.9 -p 31621
172.17.0.9:31621> ping
PONG
```

### 6、Pod

`Pod` 是 K8S 中的最小化部署单元。看下当前集群中 `Pod` 的状态。

```shell
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
redis-658d78cf9c-gb9qc   1/1     Running   0          67m

```

进行一次简单扩容

```shell
$ kubectl scale deploy/redis --replicas=2
deployment.extensions/redis scaled
$ kubectl get pods
NAME                     READY   STATUS              RESTARTS   AGE
redis-658d78cf9c-gb9qc   1/1     Running             0          68m
redis-658d78cf9c-t8ll6   0/1     ContainerCreating   0          3s

```

### 7、角色（Role）

先认识下 K8S 中的角色。K8S 中的角色从类别上主要有两类，`Role` 和 `ClusterRole`。

> - `Role`：可以当作是一组权限的集合，但被限制在某个 `Namespace` 内（K8S 的 `Namespace`）。
> - `ClusterRole`：对于集群级别的资源是不被 `Namespace` 所限制的，并且还有一些非资源类的请求，所以便产生了它。

当已经了解到角色后，剩下给用户授权也就只是需要做一次绑定即可。在 K8S 中将这一过程称之为 binding，即 `rolebinding` 和 `clusterrolebinding`。看下集群刚初始化后的情况：

```shell
$ kubectl get roles --all-namespaces=true
NAMESPACE     NAME                                             AGE
kube-public   kubeadm:bootstrap-signer-clusterinfo             17m
kube-public   system:controller:bootstrap-signer               17m
kube-system   extension-apiserver-authentication-reader        17m
kube-system   kube-proxy                                       17m
kube-system   kubeadm:kubelet-config-1.13                      17m
kube-system   kubeadm:nodes-kubeadm-config                     17m
kube-system   system::leader-locking-kube-controller-manager   17m
kube-system   system::leader-locking-kube-scheduler            17m
kube-system   system:controller:bootstrap-signer               17m
kube-system   system:controller:cloud-provider                 17m
kube-system   system:controller:token-cleaner                  17m
kube-system   weave-net                                        17m

$ kubectl get rolebindings --all-namespaces=true
NAMESPACE     NAME                                             AGE
kube-public   kubeadm:bootstrap-signer-clusterinfo             17m
kube-public   system:controller:bootstrap-signer               17m
kube-system   kube-proxy                                       17m
kube-system   kubeadm:kubelet-config-1.13                      17m
kube-system   kubeadm:nodes-kubeadm-config                     17m
kube-system   system::leader-locking-kube-controller-manager   17m
kube-system   system::leader-locking-kube-scheduler            17m
kube-system   system:controller:bootstrap-signer               17m
kube-system   system:controller:cloud-provider                 17m
kube-system   system:controller:token-cleaner                  17m
kube-system   weave-net                                        17m
darry@DR07:~/config$ 

```

可以看到默认已经存在了一些 `role` 和 `rolebindings`。 对于这部分暂且不做过多说明，来看下对于集群全局有效的 `ClusterRole` 。

```shell
$ kubectl get clusterroles
NAME                                                                   AGE
admin                                                                  21m
cluster-admin                                                          21m
edit                                                                   21m
system:aggregate-to-admin                                              21m
system:aggregate-to-edit                                               21m
system:aggregate-to-view                                               21m
system:auth-delegator                                                  21m
system:aws-cloud-provider                                              21m
system:basic-user                                                      21m
system:certificates.k8s.io:certificatesigningrequests:nodeclient       21m
system:certificates.k8s.io:certificatesigningrequests:selfnodeclient   21m
system:controller:attachdetach-controller                              21m
system:controller:certificate-controller                               21m
system:controller:clusterrole-aggregation-controller                   21m
system:controller:cronjob-controller                                   21m
system:controller:daemon-set-controller                                21m
system:controller:deployment-controller                                21m
system:controller:disruption-controller                                21m
system:controller:endpoint-controller                                  21m
system:controller:expand-controller                                    21m
system:controller:generic-garbage-collector                            21m
system:controller:horizontal-pod-autoscaler                            21m
system:controller:job-controller                                       21m
system:controller:namespace-controller                                 21m
system:controller:node-controller                                      21m
system:controller:persistent-volume-binder                             21m
system:controller:pod-garbage-collector                                21m
system:controller:pv-protection-controller                             21m
system:controller:pvc-protection-controller                            21m
system:controller:replicaset-controller                                21m
system:controller:replication-controller                               21m
system:controller:resourcequota-controller                             21m
system:controller:route-controller                                     21m
system:controller:service-account-controller                           21m
system:controller:service-controller                                   21m
system:controller:statefulset-controller                               21m
system:controller:ttl-controller                                       21m
system:coredns                                                         21m
system:csi-external-attacher                                           21m
system:csi-external-provisioner                                        21m
system:discovery                                                       21m
system:heapster                                                        21m
system:kube-aggregator                                                 21m
system:kube-controller-manager                                         21m
system:kube-dns                                                        21m
system:kube-scheduler                                                  21m
system:kubelet-api-admin                                               21m
system:node                                                            21m
system:node-bootstrapper                                               21m
system:node-problem-detector                                           21m
system:node-proxier                                                    21m
system:persistent-volume-provisioner                                   21m
system:volume-scheduler                                                21m
view                                                                   21m
weave-net                                                              21m

$ kubectl get clusterrolebindings
NAME                                                   AGE
cluster-admin                                          21m
kubeadm:kubelet-bootstrap                              21m
kubeadm:node-autoapprove-bootstrap                     21m
kubeadm:node-autoapprove-certificate-rotation          21m
kubeadm:node-proxier                                   21m
system:aws-cloud-provider                              21m
system:basic-user                                      21m
system:controller:attachdetach-controller              21m
system:controller:certificate-controller               21m
system:controller:clusterrole-aggregation-controller   21m
system:controller:cronjob-controller                   21m
system:controller:daemon-set-controller                21m
system:controller:deployment-controller                21m
system:controller:disruption-controller                21m
system:controller:endpoint-controller                  21m
system:controller:expand-controller                    21m
system:controller:generic-garbage-collector            21m
system:controller:horizontal-pod-autoscaler            21m
system:controller:job-controller                       21m
system:controller:namespace-controller                 21m
system:controller:node-controller                      21m
system:controller:persistent-volume-binder             21m
system:controller:pod-garbage-collector                21m
system:controller:pv-protection-controller             21m
system:controller:pvc-protection-controller            21m
system:controller:replicaset-controller                21m
system:controller:replication-controller               21m
system:controller:resourcequota-controller             21m
system:controller:route-controller                     21m
system:controller:service-account-controller           21m
system:controller:service-controller                   21m
system:controller:statefulset-controller               21m
system:controller:ttl-controller                       21m
system:coredns                                         21m
system:discovery                                       21m
system:kube-controller-manager                         21m
system:kube-dns                                        21m
system:kube-scheduler                                  21m
system:node                                            21m
system:node-proxier                                    21m
system:volume-scheduler                                21m
weave-net                                              21m

```

可以看到 K8S 中默认已经有很多的 `ClusterRole` 和 `clusterrolebindings` 了

### 8、查看用户权限

```shell
# 获取当前上下文
$ kubectl config current-context
 # 名为 kubernetes-admin 的用户，在名为 moelove-ha 的 cluster 上
kubernetes-admin@moelove-ha

# 新版本把users移除掉了，旧版本可以用，使用以下命令报错
$ kubectl config view users -o yaml
error: unexpected arguments: [users]
See 'kubectl config view -h' for help and examples
# 这是新版本的命令
$ kubectl config view -o yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://localhost:44549
  name: moelove-ha
contexts:
- context:
    cluster: moelove-ha
    user: kubernetes-admin
  name: kubernetes-admin@moelove-ha
current-context: kubernetes-admin@moelove-ha
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

`client-certificate-data` 的部分默认是不显示的，而它的**内容实际是通过 base64 加密后的证书内容**。可以通过通过以下方式进行查看

```shell
# 新版本把users移除掉了，旧版本可以用，使用以下命令报错
$ kubectl config view users --raw -o jsonpath='{ .users[?(@.name == "kubernetes-admin")].user.client-certificate-data}' |base64 -d
error: unexpected arguments: [users]
See 'kubectl config view -h' for help and examples
# 新版本命令
$ kubectl config view --raw -o jsonpath='{ .users[?(@.name == "kubernetes-admin")].user.client-certificate-data}' |base64 -d
-----BEGIN CERTIFICATE-----
MIIC8jCCAdqgAwIBAgIIbGrtzJpsNWkwDQYJKoZIhvcNAQELBQAwFTETMBEGA1UE
AxMKa3ViZXJuZXRlczAeFw0xOTA0MTQwMzM1MDRaFw0yMDA0MTMwMzM1MDRaMDQx
FzAVBgNVBAoTDnN5c3RlbTptYXN0ZXJzMRkwFwYDVQQDExBrdWJlcm5ldGVzLWFk
bWluMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAu+OptMxNwIycDFx2
tFnsC5MDogRrByrCQRI7JLS+25h+9tN7GExx5OCTI+9RqiG7qe+28s5AzoAezdjy
R0au/sWt5YF9lhOnYu6FZzh7w+n0K/wwrfWwyHfD60jKXvoodzdDi2/xWMdihaoh
0Q5amdtSNExMhJCUw9xG6ae/pTLrYGomGujueN8CroNgvzFZ9jN9p4ZhFbdnYeJZ
KvoIU7ioUuWG3R/aJnZKKT2eNBvyoOMxHyK3GJJgOIUqbWsiHMrTDSFlNJF/y0L0
vFjoimtmtKtnTCD87A1zZbDCcYB/Y2OAuz7NA8wY9J/nh1RqC52Tyb2SQG8MS8a9
W7kXJQIDAQABoycwJTAOBgNVHQ8BAf8EBAMCBaAwEwYDVR0lBAwwCgYIKwYBBQUH
AwIwDQYJKoZIhvcNAQELBQADggEBACHl9NTl1wYw6oQd4kSrEK5yH/Wy+7WDcHRj
179eH+w34SOPdyQlkzY93/n3YTXO0fCgKXYgImP+fqbdD+WCmhidzDakGBEU/hVk
RyOP5yRv5HF3Ys2VMT/yTy/deGmn695sOQracg/PlNzveY+5OykX0Yzi5gH3cBsp
yH/3tVLnd4NbPsjNtfBqwTbvaZoEOJoZncpb+dR83T7VHGXumtAJGRMRH7ojtnob
krBpzZ+P3QRAJFCR5y2QLExsjpE2Fg7ubEFx9CrxsIKgaPZFN9B9BhczXdoqHZFt
UUMlHCn/fVbrUPQZT0pjG7G153rqYJutU21aw3FrnWJuACOMKy0=
-----END CERTIFICATE-----

# 新版本把users移除掉了，旧版本可以用，使用以下命令报错
$ kubectl config view users --raw -o jsonpath='{ .users[?(@.name == "kubernetes-admin")].user.client-certificate-data}' |base64 -d |openssl x509 -text -noout
error: unexpected arguments: [users]
See 'kubectl config view -h' for help and examples
unable to load certificate
140125659509184:error:0906D06C:PEM routines:PEM_read_bio:no start line:../crypto/pem/pem_lib.c:691:Expecting: TRUSTED CERTIFICATE
# 新版本命令
$ kubectl config view --raw -o jsonpath='{ .users[?(@.name == "kubernetes-admin")].user.client-certificate-data}' |base64 -d |openssl x509 -text -noout
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 7812317966647440745 (0x6c6aedcc9a6c3569)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = kubernetes
        Validity
            Not Before: Apr 14 03:35:04 2019 GMT
            Not After : Apr 13 03:35:04 2020 GMT
        Subject: O = system:masters, CN = kubernetes-admin
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:bb:e3:a9:b4:cc:4d:c0:8c:9c:0c:5c:76:b4:59:
                    ........
                    6a:0b:9d:93:c9:bd:92:40:6f:0c:4b:c6:bd:5b:b9:
                    17:25
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Client Authentication
    Signature Algorithm: sha256WithRSAEncryption
         21:e5:f4:d4:e5:d7:06:30:ea:84:1d:e2:44:ab:10:ae:72:1f:
         ........
         b1:b5:e7:7a:ea:60:9b:ad:53:6d:5a:c3:71:6b:9d:62:6e:00:
         23:8c:2b:2d
```

根据前面认证部分的内容，知道当前的用户是 `kubernetes-admin` （CN 域），所属组是 `system:masters` （O 域） 。



看下 `clusterrolebindings` 中的 `cluster-admin` 。

```shell
$ kubectl get clusterrolebindings  cluster-admin  -o yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  creationTimestamp: "2019-04-14T03:35:24Z"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: cluster-admin
  resourceVersion: "94"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterrolebindings/cluster-admin
  uid: 56edcf92-5e66-11e9-85e8-0242f17142d4
# 重点内容 roleRef
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
# 重点内容 subjects
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:masters
```

重点内容在 `roleRef` 和 `subjects` 中，名为 `cluster-admin` 的 `ClusterRole` 与名为 `system:masters` 的 `Group` 相绑定。继续探究下它们所代表的含义。

先看看这个 `ClusterRole` 的实际内容：

```shell
$ kubectl get clusterrole cluster-admin -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  creationTimestamp: "2019-04-14T03:35:24Z"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: cluster-admin
  resourceVersion: "41"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterroles/cluster-admin
  uid: 569fbe54-5e66-11e9-85e8-0242f17142d4
rules:
- apiGroups:
  - '*'
  resources:
  - '*'
  verbs:
  - '*'
- nonResourceURLs:
  - '*'
  verbs:
  - '*'
```

`rules` 中定义了它所能操作的资源及对应动作，`*` 是通配符。

到这里，我们就可以得出结论了，当前用户 `kubernetes-admin` 属于 `system:masters` 组，而这个组与 `cluster-admin` 这个 `ClusterRole` 所绑定，所以用户也就继承了其权限。具备了对多种资源和 API 的相关操作权限。

### 9、实践：创建权限可控的用户

我们要创建的用户名为 `darry` 所属组为 `dev`。

#### 9.1、创建 NameSpace

为了演示，这里创建一个新的 `NameSpace` ，名为 `work`。

```shell
$ kubectl create namespace work
namespace/work created
$ kubectl get ns work
NAME   STATUS   AGE
work   Active   10s
```

#### 9.2、创建用户

##### 9.2.1、创建私钥

```shell
$ mkdir work
$ cd work
# 生成私钥
$ openssl genrsa -out darry.key 2048
Generating RSA private key, 2048 bit long modulus
...............................+++
...........+++
e is 65537 (0x010001)
$ ls
darry.key
# 查看私钥信息
$ cat darry.key 
-----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEArZhSlePPhnK+iBlWdNspYyLzkWJbUBgmGoe7ty4Jk/2B3FQK
........
dj3CXe59674VcFoEtgKDAYW1DGY8SK6C4ylduzJ4Gllvb662n1U/Fb7TIOS3DDjb
Y8RqafYyiNtXkNGSLV+kE5ps0CCdefclbjSfbmePiaaOBYZVajX8cIY=
-----END RSA PRIVATE KEY-----

```

##### 9.2.2、使用私钥生成证书请求

使用私钥生成证书请求，在这里需要指定 `subject` 信息，传递用户名和组名

```shell
# -subj 后面的就是用户名和组名 "/CN=darry/O=dev"
# 使用私钥生成证书请求
$ openssl req -new -key darry.key -out darry.csr -subj "/CN=darry/O=dev"
$ ls
darry.csr  darry.key
$ cat darry.csr 
-----BEGIN CERTIFICATE REQUEST-----
MIICYzCCAUsCAQAwHjEOMAwGA1UEAwwFZGFycnkxDDAKBgNVBAoMA2RldjCCASIw
.........
ZpF6kbmJp2eC974VXn5XwHfldFkxgJoH4INaCzl/8VXFg0DWlAiWysbblzpC6Jfh
fZAfL35cfQ/6Dvqk2VDtVuL5dGTimXd6i8jaXS3vCJkKbtGy0HxK
-----END CERTIFICATE REQUEST-----

```

##### 9.2.3、使用 CA 进行签名

使用 CA 进行签名，K8S 默认的证书目录为 `/etc/kubernetes/pki`。

> <font color=green>因为是使用的Kind方式搭建的环境，所以要进入到容器里去操作，如果 不是Kind方式搭建的环境可以不用那么麻烦</font>

```shell
# 利用之前已经启动的容器创建一个镜像，注意不是worker的容器是plane容器
$ docker commit -a "ybd" moelove-ha-control-plane kindest:v1.0.0
# 新建一个容器，挂载本地目录，这里挂载的是work ，注意后面的镜像需要跟上版本号
$ docker run -itd --name k8s-node -v /home/darry/work/:/work/ kindest:v1.0.0
# 进入容器
$ docker exec -it k8s-node /bin/bash

# 这条命令操作是进入到容器里操作的了，如果不是Kind方式搭建的环境，直接在本机执行
# 记得进入到私钥文件所在目录操作，或者指定私钥所在目录的路径
$ openssl x509 -req -in darry.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out darry.crt -days 365
# 上面这条命令复制到容器里去执行

root@1754b6c9fac9:/# cd work/
root@1754b6c9fac9:/work# openssl x509 -req -in darry.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out darry.crt -days 365
Signature ok
subject=CN = darry, O = dev
Getting CA Private Key
root@1754b6c9fac9:/work# ls
darry.crt  darry.csr  darry.key

```

##### 9.2.4、查看生成的证书

> 这命令可以在容器里操作，也可以在宿主机操作

```shell
$ openssl x509 -in darry.crt -text -noout
# 注意这两行信息
# Issuer: CN = kubernetes
# Subject: CN = darry, O = dev

Certificate:
    Data:
        Version: 1 (0x0)
        Serial Number:
            d9:e8:4d:37:e1:96:25:cb
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = kubernetes
        Validity
            Not Before: Apr 20 02:06:13 2019 GMT
            Not After : Apr 19 02:06:13 2020 GMT
        Subject: CN = darry, O = dev
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:dd:c3:99:4e:63:09:98:58:fd:48:4c:d9:0a:57:
                    ..........
                    40:64:51:01:83:84:48:7d:30:08:50:2a:f0:5b:42:
                    bb:1b
                Exponent: 65537 (0x10001)
    Signature Algorithm: sha256WithRSAEncryption
         0b:89:19:85:16:dc:68:5d:50:51:d8:78:14:da:9e:2f:d4:5b:
         .........
         b5:aa:0a:95:35:c9:b4:24:a5:90:df:92:0e:7c:96:57:e1:a9:
         46:88:82:f7
```

#### 9.3、添加context

```shell
$ kubectl config set-credentials darry --client-certificate=/home/darry/work/darry.crt  --client-key=/home/darry/work/darry.key
User "darry" set.
$ kubectl config set-context darry-context --cluster=kubernetes --namespace=work --user=darry
Context "darry-context" created.
```

#### 9.4、使用新用户测试访问

```shell
$ kubectl --context=darry-context get pods
Error from server (Forbidden): pods is forbidden: User "backend" cannot list resource "pods" in API group "" in the namespace "work"
# 可能看得不够清楚，我们添加 `-v` 参数来显示详情
$ kubectl --context=darry-context get pods -n work -v 5
```





















