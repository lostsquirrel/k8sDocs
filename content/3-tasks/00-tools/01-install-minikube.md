---
title: 安装 Minikube
date: 2020-06-11
lastmod: 2020-06-17
---

本文介结 Minikube 安装， Minikube 是一个创建运行在虚拟机的的本地单节点 k8s 集群的工具

## 环境准备

- 检查本机虚拟化支持

`grep -E --color 'vmx|svm' /proc/cpuinfo`

- 安装 虚拟机管理程序(以下选择一个)

- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

- [KVM](https://www.linux-kvm.org/) 和 [QEMU](https://www.qemu.org/)

minikube 也支持配置 `--driver=none` 方式，此时 k8s 组件就不会运行在虚拟机中，而是直接运行在当前机器上(这种情况就不需要安装虚拟机)。 使用这种方式需要安装 Docker
如果确定使用 `--driver=none`，则k8s组件运行在主机上，只需要安装 Docker 环境，
如果在 Debain 系操作系统安装，需要以 `.deb` 方式安装，而不是 `snap` 安装， [Docker 下载地址](https://www.docker.com/products/docker-desktop)

注意: `--driver=none` 有安全问题和数据丢失的风险， 更多相关信自请参见[这一篇](https://minikube.sigs.k8s.io/docs/reference/drivers/none/)

minikube 还支持 `vm-driver=podman`， 与 Docker 驱动类似。 Podman 需要以超级管理员权限运行以保证容器有权限访问系统上的任意可用资源。

## Linux 安装 minikube


### 安装 kubectl

详见 [kubectl 安装配置]()

### 通过包管理器安装 minikube

Github库发行(https://github.com/kubernetes/minikube/releases) 有不同操作系统对应包提供下载。可前端下载并安装

### 直接下载二进制文件

```sh
$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
```

添加到一个 PATH 目录
```sh
sudo mkdir -p /usr/local/bin/
sudo install minikube /usr/local/bin/
```

## 确认正确安装

可以通过以下命令启动集群

```sh
minikube start --driver=<driver_name>
```

`driver_name` 具体请见[这里](https://kubernetes.io/docs/setup/learning-environment/minikube/#specifying-the-vm-driver)

注意: 当使用 KVM 时需要注意，在 Debian 等一些操作系统下 libvirt 默认 QEMU URI 是 `qemu:///session`， 而 minikube 默认的 QEMU URI 是 `qemu:///system`。 如果你的系统有此问题需要为 `minikube start` 添加参数 `--kvm-qemu-uri qemu:///session`

当 minikube 启动完成后，通过以下命令查看集群状态
```sh
minikube status
```
如果命令返回结果类似如下，则表示成功
```
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

如果需要停止集群则执行
```sh
minikube stop
```

如果在执行 `minikube start` 返回如下错误
```
machine does not exist
```
则需要要清除本地状态，再继续
```sh
minikube delete
```
