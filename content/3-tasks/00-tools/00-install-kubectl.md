---
title: 下载安装配置 kubectl
weight: 30000
date: 2020-06-11
---

kubectl 是 k8s 的命令行工具，用于通过命令管理k8s集群。 kubectl 可以用于部署应用，查看和管理集群资源，查看日志，具体使用明细请见 [源地址](https://kubernetes.io/docs/reference/kubectl/overview/) [本站地址](../5-reference/07-kubectl/00-overview.md)

## 准备
  kubectl 版本与 k8s 集群必须只能有一个小版本的差异，比如 kubectl 是 v1.2, 只能用于 k8s 集群 master 节点版本为 v1.1 或 v1.3. 使用最新版的 kubectl 可以避免一些神奇的问题

## Linux(Ubuntu 20.04)下安装

1. 下载

- 下载最新版本
```sh
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
wget https://shangao.tech/kubectl
```

- 下载指定版本(例如: v1.18.0)

```sh
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl
```

2. 添加执行权限

```sh
chmod +x ./kubectl
```

3. 移动到PATH

```sh
sudo mv ./kubectl /usr/local/bin/kubectl
```

4. 查看安装的版本

```sh
kubectl version --client
```

## snap 安装

```sh
snap install kubectl --classic

kubectl version --client
```
## 验证配置

kubectl 与 k8s集群通信时需要一个配置文件以存放集群相关信息，这个文件一般在集群创建时会自动生成。默认情况下位于以下路径

```sh
 ~/.kube/config
```

通过以下命令查看集群状态以确定配置信息正确
```sh
kubectl cluster-info
```
如果出现类似如下信息，则表示成功
```
Kubernetes master is running at http://localhost:8080
```
如果出现如下类似信秘，则表示配置不正常, 执行 `kubectl cluster-info dump` 查看明细原因
```
The connection to the server <server-name:port> was refused - did you specify the right host or port?
```


## 可选配置

开启自动补全

### Bash 下开启

#### 说明

kubectl 自动被全脚本可以通过 `kubectl completion bash` 生成，此脚本信赖 [bash-completion](https://github.com/scop/bash-completion)，需要执行命令 `type _init_completion` 确定已经安装。

#### 安装 `bash-completion`

```sh
apt-get install bash-completion
```
加载 `bash-completion`

```sh
source /usr/share/bash-completion/bash_completion
```

#### 开启补全

1. 为当前用户开启

```sh
echo 'source <(kubectl completion bash)' >>~/.bashrc
```

2. 为所有用户开启

```sh
kubectl completion bash >/etc/bash_completion.d/kubectl
```

如果为 kubectl 添加了别名需要如下配置
```sh
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -F __start_kubectl k' >>~/.bashrc
```
