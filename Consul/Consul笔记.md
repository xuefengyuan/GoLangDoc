[TOC]

## 一、Consul安装

系统环境为Ubuntu18.04

[Consul](https://www.consul.io/downloads.html)

```shell
# 下载Consul文件
$ wget https://releases.hashicorp.com/consul/1.4.4/consul_1.4.4_linux_amd64.zip
# 解压
$ unzip consul_1.4.4_linux_amd64.zip
# 移动到/usr相关目录下，全局可执行
$ sudo mv consul /usr/local/bin/consul

# 验证Consul是否安装成功
$ consul
# 查看Consul版本
$ consul --version
# 启动Consl
$ consul agent -dev  -client 0.0.0.0 -ui
	# 加上 -client 0.0.0.0 表示可以利用ip加8500可以访问
# 浏览器访问 http://192.168.1.4:8500/ui
```

