---
title: 概念(Concepts)
weight: 30
date: 2020-06-23
---

本章主要介绍 k8s 系统的主要组成部分以及 k8s 是怎么对集群进行抽象的，以帮助读者更深入的理解 k8s 是怎么工作的

## 概览

在使用 k8s 时， 用户使用 k8s API 对象来描述集群的 期望状态: 比如需要运行什么应用或其它工作负载，容器要使用哪个镜像， 副本的数量， 需要提供多少的磁盘和网络资源等。用户通过 k8s API 创建对象设置期望状态。 一般是通过命令行工具 kubectl. 当然也可以能过直接调用 k8s API 来设置或修改期望的状态

当用户设置好所 期望的状态后， k8s 控制面板 将通过 Pod Lifecycle Event Generator (PLEG) 将集群状态由当前状态变更到期望的状态，具体是 k8s 通过自发的通过一系列任务实现， 比如启动或重启容器，增加指定应用的副本数等方式， k8s 控制器由集群中运行的一系统进程实现：
- Kubernetes Master 由三个进程组成，一般在集群运行在一个独立的主机上，被设计为主节点，其中三个进程为 `kube-apiserver`, kube-`controller-manager` `kube-scheduler` TODO.

- 而在其它非主节点上，每个都运行以下两个进程

  - kubelet TODO 用于与主节点通信
  - kube-proxy TODO 在每个节点上用于提供网络服务的代理服务


## k8s 对象

k8s 包含相当数量的抽象概念用于表示系统的状态: 部署应用容器和工作负载，以及其对应的网络和磁盘资源，和关于集群运行的其它信息。这些抽象概念在 k8s API 中表现为对象的形式。更相关信息见 TODO

k8s 基础对象如下(TODO)

- Pod
- Service
- Volume
- Namespace

k8s 还包含更高级的抽象概念，它们是通过控制器(TODO)基于基础对象构建而成，提供更多的功能和特性，包含如下(TODO)

- Deployment
- DaemonSet
- StatefulSet
- ReplicaSet
- Job

## k8s 控制面板

k8s 控制面板 由各种组件组成，用户k8s 掌控集群。
k8s 控制面板 维护系统中所有对应的记录并用一个持续循环管理这些对象的状态。 任何时候你设置了你期望的状态后， k8s 控制面板就会让系统中所有的对象状态变更至期望的状态

例如 用户创建一个新的 Deployment。 这时系统就有一个新的期望状态。首先 k8s 控制面板 会记录这个新创建的对象；然后开始启动这应用并将其调度到合适的节点上。这样系统就达成了这个新的期望状态

### k8s 主控节点

k8s 主控节点负责维护集群的期望状态。 当用户通过 kubectl 命令或 API 管理集群时，都是在与主控节点通信
主控节点是由一系列维护集群状态的进程组成。 通常这个进程都会运行在集群中的同一个节点上。因此这个节点被称为主控节点。 主控节点也可能通过增加副本的方式提供可用性和冗余度

### k8s 普通节点

集群就的普通节点就是运行用户应用和其它工作负载的机器(可以是虚拟机，物理机等)。 这些节点由主控节点控制，用户一般很少与其直接交互
