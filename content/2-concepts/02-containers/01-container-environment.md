---
title: 容器环境
date: 2020-07-27
publishdate: 2020-07-27
weight: 20201
---
<!-- overview -->
本文主要介绍以环境提供容器的资源信息
<!-- body -->
## 容器内的环境变量
k8s 环境为容器提供了一些重要的资源信息：
  - 一个包含[镜像](../00-images/)和一个或多个[卷](../../05-storage/00-volumes/)组合成的文件系统
  - 关于容器自身的相关信息
  - 关于集群中其它对象的相关信息

### 容器信息

容器的主机名，也就是容器运行的 Pod 的名称。 可以通过 `hostname` 命令或 `libc` 库中的 `gethostname` 函数获得
Pod 的名称和所在的名字空间可能 [downward API](../../../3-tasks/04-inject-data-application/04-downward-api-volume-expose-pod-information/#the-downward-api)以环境变量方式访问
在 Pod 定义中用户自定义的环境变量和Docker 镜像中的环境变量都会成为容器中的环境变量。

### 集群信息

在容器创建之前的所有 Service 列表会以环境变量方式加入容器。 这些环境变量与 Docker link 的语法一致。
例如 在一个叫 bar 的容器中加一个叫 foo 的 Service, 会加入以下环境变量:
```env
FOO_SERVICE_HOST=<the host the service is running on>
FOO_SERVICE_PORT=<the port the service is running on>
```
如果集群开启了 [DNS 插件](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/dns/)，容器也可能通过 DNS 方式获取 Service 的 IP 地址。

## {{% heading "whatsnext" %}}

- [容器生命周期钩子](../03-container-lifecycle-hooks/)
- [生命周期事件处理器](../../../3-tasks/02-configure-pod-container/16-attach-handler-lifecycle-event/)
