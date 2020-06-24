---
title: k8s 组件
weight: 20001
date: 2020-06-24
draft: true
---
当完成 k8s 部署后，用户就拥有一个 k8s 集群， 一个 k8s 一般包含多台工作机，称为节点，用于运行容器化应用。 每个集群至少包含一个节点。
在节点上运行应用工作负载的组件称为 Pod. k8s 控制面板 管理工作节点和集群中的 Pod. 在生产环境中，k8s 控制面板一般会运行在多个机器上，同样一个集群也会包含多个节点，用以提高集群的容错和高可用

本文主要介绍一个完整可工作的集群包含哪些组件
下图标示出了一个集群的所有组件及相互关系
![k8s components diagram](https://d33wubrfki0l68.cloudfront.net/7016517375d10c702489167e704dcb99e570df85/7bb53/images/docs/components-of-kubernetes.png) [来源 kubernetes.io](https://kubernetes.io/docs/concepts/overview/components/)


## 控制台 组件

控制台负责整个集群的全局决策(例如，调度)，也包含侦听和响应集群事件(例如，当应用的副本数增加时启动一个新的 Pod)
控制台组件可以运行在集群的任意机器上。 但为了简单，创建脚本一般都会将组件放在同一个节点上，并不在这个节点上运行用户的容器。 更多示例参见
[高可用集群](TODO)


### kube-apiserver

提供 k8s API，是k8s控制台的前端(此前端非)
k8s API 的大多数实现都在 kube-apiserver
提供了水平扩展的能力，扩容方式为部署多个实例，并提供负载均衡 [搭建高可能集群](TODO)

### etcd

强一致性和高可用键值存储，用于 k8s 存储所有集群数据
一定要对 etcd 数据做备份计划
需要深入了解 etcd 请移步 [官网](https://etcd.io/docs/)

### kube-scheduler

侦听新创建没有指定 Node 的 Pod，为其选择一个 Node
调度因素：包含独立，集合资源需求，硬件，软件，策略，亲和性，反亲和性规范，数据位置，inter-workload interference and deadlines (TODO 没接触过，需要研究一下)

### kube-controller-manager

`kube-controller-manager` 包含控制器进程，逻辑上每个 控制器是独立的进程，但为了降低复杂度，将它们打入一个可执行文件并运行在一个进程里

控制器如下：
- Node 控制器: 负责监视和响应 Node 是不是挂了
- Replication 控制器: 负责系统中的 Pod 维护在预期的副本数
- Endpoints 控制器: 负责管理 Endpoints对象(Services & Pods)
- Service Account & Token 控制器: 为新增的名字空间创建默认的账号和令牌



### cloud-controller-manager

    运行与底层云提供商交互的控制器，k8s v1.6的alpha功能，如果要使用，需要在 `kube-controller-manager` 中配置 `--cloud-provider` 为 `external`
    `cloud-controller-manager` 提供云提供商与k8s建立相互依赖，在之前的版本中，k8s 依赖于 `cloud-provider-specific` 功能，在未来的版本中，云提供商相关的将由云提供商自己维护，并在k8s运行时连接到`cloud-controller-manager`
    以下控制器依赖于云提供商：
    - Node Controller： 当机器不响应时向云提供商询问是否被删除
    - Route Controller：设置在云提供商底层基础设施上设置路由
    - Service Controller: 创建，更新，删除云提供商的负载均衡
    - Volume Controller： 创建，连接，挂载卷，与云提供商交互卷编排

## Node 组件
    Node 组件运行在每个节点上，维护 Pod 运行，提供 k8s 运行环境

### kubelet
    保证容器在 Pod 运行
    读取`PodSpecs` 保证容器的健康运行
    kubelet 不管理非 k8s 创建的容器(例如由 manifests 创建的 Pod)

### kube-proxy
    kube-proxy 支持 k8s 实现Service层抽象，维护主机上的网络规则和执行连接转发

### Container Runtime
    容器运行环境，k8s 支持 Docker, rkt

## 插件
    实现集群功能的 Pod 和 Service, Pod 可能由 Deployment，ReplicationControllers等管理，运行名字空间为 `kube-system`

### DNS
    其它插件不是十分必要，但这个必须要有，许多示例依赖这个插件
    Cluster DNS，是一个DNS服务，为k8s 集群 Service 提供 DNS 记录
    由 k8s 启动的容器会自动包含 此 DNS 的配置

### Web UI (Dashboard)
    k8s 集群和应用管理的 web 应用

### 容器资源监控 Container Resource Monitoring
    记录容器监控数据到到一个数据库，并提供查看数据的UI

### 集群级的日志 Cluster-level Logging
    提供集群级的容器日志收集机制，并提供查询和阅览功能

## 参考资料

https://kubernetes.io/docs/concepts/overview/components/
https://kubernetes.io/docs/concepts/cluster-administration/addons/
## 引申阅读
https://kubernetes.io/docs/admin/high-availability/
