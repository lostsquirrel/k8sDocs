---
title: 容器
weight: 202
description: Technology for packaging an application along with its runtime dependencies.
publishdate: 2020-07-19
---

<!-- overview -->
用户运行的每一个容器都是可重复的，标准中说明容器中包含的所需要的依赖，所以每次运行的行为都应该是一致的

容器技术让应用与底层主机的基础设施解耦。这使在不去云环境或操作系统上的部署变得更容易。



<!-- body -->

## 容器镜像
[容器镜像](../images/) 是一个直接可以运行的软件包，其中含了这个应用运行所需要的一切: 应用程序和对应的运行环境，应用和系统的依赖库，必要配置的默认值

在设计上一个容器是不可变更的：不能修改一个已经运行的窗口的代码(应用)。如果用户想对一个容器化的应用进行变更，需要构建一个包含这些变更的新镜像，并用这个新的镜像再创建一个新的容器代替原来的容器继续运行

## 容器运行时

容器运行时就是负责运行容器的软件， k8s 支持以下几种运行时:
- Docker
- containerd
- CRI-O
- 任意实现 k8s CRI(Container Runtime Interface)的应用

## {{% heading "whatsnext" %}}

* 看看 [容器镜像](/docs/concepts/containers/images/)
* 看看 [Pod](/k8sDoc/concepts/workloads/pods/)
