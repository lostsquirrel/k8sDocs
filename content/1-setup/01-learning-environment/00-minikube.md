---
title: 使用 Minikube 搭建 K8s环境
weight: 11100
date: 2020-06-11
---

## 特点

一个用于创建易于部署，运行在虚拟机的的本地单节点 k8s 集群的工具

## 适用场景

1. 尝试与学习 k8s
2. 日常开发

## 功能特性

- DNS
- NodePorts
- ConfigMaps and Secrets
- Dashboards
- Container Runtime:
  [Docker](https://www.docker.com/),
  [CRI-O](https://cri-o.io/), and
  [containerd](https://github.com/containerd/containerd)
- Enabling CNI (Container Network Interface)
- Ingress

## 安装

[安装 Minikube](../../3-tasks/00-tools/install-minikube)

## 快速入门

以下简单介绍 minikube 的基本用法，如启动，使用，删除。

1. 启动 minikube 以创建集群

```sh
 minikube start
```
更多运行参数参见后续章节

2. 使用 kubectl 进行集群管理，更多见后续章节

以下命令使用 `echoserver` 镜像 创建一个 部署（`Deployment`）， 这个镜像是一个简单的 `http` 服务，服务端口 `8080`

```sh
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.10
kubectl create deployment hello-minikube --image=registry.cn-hangzhou.aliyuncs.com/google_containers/echoserver:1.10
```

根据网络选择一条命令执行，输出结果如下
```
deployment.apps/hello-minikube created
```

3. 为了能够访问这个 `Deployment，` 需要为其创建一个 `Service`

```sh
kubectl expose deployment hello-minikube --type=NodePort --port=8080
```

其中 `--type=NodePort` 参数指定服务的类型，输出结果如下
```
service/hello-minikube exposed
```

4. 需要等到 Deployment 对应的 Pod 启动后，才能在 `Service` 访问

检查 Pod 是否为 运行(running) 状态,执行如下命令
```sh
kubectl get pod
```

如果输出的 STATUS 列值为 ContainerCreating, 表示 Pod 还在创建中，需要再等一会儿，类似如下
```
NAME                              READY     STATUS              RESTARTS   AGE
hello-minikube-3383150820-vctvh   0/1       ContainerCreating   0          3s
```
如果输出的 STATUS 列值为 Running, 表示 Pod 启动完成，运行中，类似如下
```
NAME                              READY     STATUS    RESTARTS   AGE
hello-minikube-3383150820-vctvh   1/1       Running   0          13s
```

5. 获取 Service 的访问连接信息

```sh
minikube service hello-minikube --url
```

输出结果类型如下
```
http://172.17.0.3:31579
```

通过浏览器访问以下连接地址，输出类似如下
```

Hostname: hello-minikube-7df785b6bb-kjtnt

Pod Information:
	-no pod information available-

Server values:
	server_version=nginx: 1.13.3 - lua: 10008

Request Information:
	client_address=172.18.0.1
	method=GET
	real path=/
	query=
	request_version=1.1
	request_scheme=http
	request_uri=http://127.0.0.1:8080/

Request Headers:
	accept=*/*
	host=127.0.0.1:31579
	user-agent=curl/7.65.3

Request Body:
	-no body in request-
```

如果不需要这个服务，删除对应对象

7. 删除 `hello-minikube` `Service`

```sh
kubectl delete services hello-minikube
```
输出如下
```
service "hello-minikube" deleted
```

8. 删除 `hello-minikube` `Deployment`:

```sh
kubectl delete deployment hello-minikube
```
输出如下
```
deployment.extensions "hello-minikube" deleted
```

9. 停止本地集群

```sh
minikube stop
```
输出如下
```
Stopping "minikube"...
"minikube" stopped.
```
10. 删除本地集群

```sh
minikube delete
```
输出如下
```
Deleting "minikube" ...
The "minikube" cluster has been deleted.
```

后续章节还有更多关于 集群删除的内容

## 集群管理

### 集群启动

通过 `minikube start` 命令启动集群，包含创建和配置一个虚拟机(也可以是其它方式)来运行一个单节点的 k8s 集群。 同时添加 kubectl 与集群交互的配置

注意:
1. 如果需要使用代理命令如下,单独配置的环境变量不会生效
```sh
shell https_proxy=<my proxy> minikube start --docker-env http_proxy=<my proxy> --docker-env https_proxy=<my proxy> --docker-env no_proxy=192.168.99.0/24
```
2. kubectl 配置会生成一个叫  `minikube` 的上下文，并作为默认的上下文，如果需要切换为该上下文执行命令 `kubectl config use-context minikube`

### 指定 k8s 版本

可以通过为 `minikube start` 的 `--kubernetes-version` 选项指定集群的 k8s 版本，例如以下命令表示指定 k8s 版本为 `v1.18.0`

```sh
minikube start --driver=<driver_name>
```

### 指定虚拟机驱动

可以通过为 `minikube start` 的 `--driver=<enter_driver_name>` 选项指定集群组件宿主机的实现方式，命令类似如下
```sh
minikube start --driver=<driver_name>
```

minikube 支持驱动如下:
注意： 驱动详情及插件安装见 [这里](https://minikube.sigs.k8s.io/docs/reference/drivers/)

- docker (driver installation)
- virtualbox (driver installation)
- podman (driver installation) (EXPERIMENTAL)
- vmwarefusion
- kvm2 (driver installation)
- hyperkit (driver installation)
- hyperv (driver installation) Note that the IP below is dynamic and can change. It can be retrieved with - minikube ip.
- vmware (driver installation) (VMware unified driver)
- parallels (driver installation)
- none(直接将k8s组件运行在主机上，需要主机系统为 Linux 且安装了 Docker)

警告: 如果使用驱动为 none 时， 某些 k8s 组件容器以超级权限(privileged)运行，会对 minikube 之外的环境产生影响。因此不建议在个人工作站上使用

### 通过其它的容器运行环境启动集群

- containerd

```sh
minikube start \
    --network-plugin=cni \
    --enable-default-cni \
    --container-runtime=containerd \
    --bootstrapper=kubeadm
```
或者

```sh
minikube start \
    --network-plugin=cni \
    --enable-default-cni \
    --extra-config=kubelet.container-runtime=remote \
    --extra-config=kubelet.container-runtime-endpoint=unix:///run/containerd/containerd.sock \
    --extra-config=kubelet.image-service-endpoint=unix:///run/containerd/containerd.sock \
    --bootstrapper=kubeadm
```

- CRI-O

```sh
minikube start \
    --network-plugin=cni \
    --enable-default-cni \
    --container-runtime=cri-o \
    --bootstrapper=kubeadm
```
或者
```sh
minikube start \
    --network-plugin=cni \
    --enable-default-cni \
    --extra-config=kubelet.container-runtime=remote \
    --extra-config=kubelet.container-runtime-endpoint=/var/run/crio.sock \
    --extra-config=kubelet.image-service-endpoint=/var/run/crio.sock \
    --bootstrapper=kubeadm
```

### 对过重用 minkube 使用的 Docker daemon，访问集群内部镜像

在使用单个虚拟的k8s集群时，重启 minikube 内置的 Docker 后台来避免在你构建自己的镜像的时候需要使用的镜像仓库,这样可以简化操作，加快本地操作体验

注意: 不要使用 `latest` 作为镜像的标签，因为它是默认值。在镜像摘取策略为 `Always`的时候可能会出问题。

在 `Mac/Linux` 主机下可以通过 `minikube docker-env` 命令返回结果，切换 Docker 后台，切换后可以通过以下命令查看其中镜像
```
docker ps
```

注意:  Centos 7，可能会报如下错误
```
 Could not read CA certificate "/etc/docker/ca.pem": open /etc/docker/ca.pem: no such file or directory
```
可以通过修改 `/etc/sysconfig/docker`, 确保环境变量与 minikube 的一致，脚本类似如下

```sh
shell < DOCKER_CERT_PATH=/etc/docker --- > if [ -z "${DOCKER_CERT_PATH}" ]; then > DOCKER_CERT_PATH=/etc/docker > fi
```
