---
title: k8s 组件
weight: 20001
date: 2020-06-24
publishdate: 2020-06-28
---
当完成 k8s 部署后，用户就拥有一个 k8s 集群， 一个 k8s 一般包含多台工作机，称为节点，用于运行容器化应用。 每个集群至少包含一个节点。
在节点上运行应用工作负载的组件称为 Pod. k8s 控制面板 管理工作节点和集群中的 Pod. 在生产环境中，k8s 控制面板一般会运行在多个机器上，同样一个集群也会包含多个节点，用以提高集群的容错和高可用

本文主要介绍一个完整可工作的集群包含哪些组件
下图标示出了一个集群的所有组件及相互关系
![k8s components diagram](https://d33wubrfki0l68.cloudfront.net/7016517375d10c702489167e704dcb99e570df85/7bb53/images/docs/components-of-kubernetes.png) [来源 kubernetes.io](https://kubernetes.io/docs/concepts/overview/components/)


## 控制中心 组件 {#control-plane-components}

控制中心负责整个集群的全局决策(例如，调度)，也包含侦听和响应集群事件(例如，当应用的副本数增加时启动一个新的 Pod)
控制中心组件可以运行在集群的任意机器上。 但为了简单，创建脚本一般都会将组件放在同一个节点上，并不在这个节点上运行用户的容器。 更多示例参见
[高可用集群](TODO)


### kube-apiserver

提供 k8s API，是k8s控制中心的前端(此前端非)
k8s API 的大多数实现都在 kube-apiserver
提供了水平扩展的能力，扩容方式为部署多个实例，并提供负载均衡 [搭建高可能集群](TODO)

### etcd

强一致性和高可用键值存储，用于 k8s 存储所有集群数据
一定要对 etcd 数据做备份计划
需要深入了解 etcd 请移步 [官网](https://etcd.io/docs/)

### kube-scheduler

侦听新创建还没有指定 Node 的 Pod，为其选择一个 Node
调度因素：包含独立，集合资源需求，硬件，软件，策略，亲和性，反亲和性规范，数据位置，inter-workload interference and deadlines (TODO 没接触过，需要研究一下)

### kube-controller-manager

`kube-controller-manager` 包含控制器进程，逻辑上每个 控制器是独立的进程，但为了降低复杂度，将它们打入一个可执行文件并运行在一个进程里

有如下控制器：
- Node 控制器: 负责监视和响应 Node 是不是挂了
- Replication 控制器: 负责系统中的 Pod 维护在预期的副本数
- Endpoints 控制器: 负责管理 Endpoints对象(Services & Pods)
- Service Account & Token 控制器: 为新增的名字空间创建默认的账号和令牌


### cloud-controller-manager

k8s 控制中心集群了云环境的控制逻辑。 云控制管理器是集群与云提供商API交互的控制器。并与只与集群交互的控制器分离。
`cloud-controller-manager` 只运行与指定云提供商对应的控制器。 如果k8s集群没有运行在云上，则集群不包含 `cloud-controller-manager`
与 `kube-controller-manager` 一样，`cloud-controller-manager` 也是由多个独立的控制器放在一个二进制文件中，并运行在一个进程里。 可能运行多个运程实现水平扩展达到扩容和容灾的目的

以下控制器依赖于云提供商：
- Node 控制器 当机器不响应时向云提供商询问是否被删除
- Route 控制器 在云提供商的基础设施上创建路由
- Service 控制器: 创建，更新，删除云提供商的负载均衡


## Node 组件

  Node 组件运行在每个节点上，维护 Pod 运行，提供 k8s 运行环境

### kubelet

  保证容器在 Pod 运行
  读取`PodSpecs` 保证容器的健康运行
  kubelet 不管理非 k8s 创建的容器(例如由 manifests 创建的 Pod)

### kube-proxy

  kube-proxy 运行在集群中每个节点上的网络代理， 实现 Service 层抽象的组成部分
  kube-proxy 维护节点上的网络规则。 这些规则允计集群内外能够与 Pod 进行网络通信
  kube-proxy 优先使用系统层的包过虑，如果没有则自己转发

### Container Runtime

  容器的运行环境，
  k8s 支持容器运行环境有: Docker, containerd, CRI-O 和 任意实现了 k8s CRI([Container Runtime Interface](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md))的运行环境


## 插件

  插件基于 k8s 资源(DaemonSet, Deployment 等)来实现集群新功能。由于提供的是集群级的功能，所有插件所属的名字空间为 `kube-system`
  下面列举了一些插件，更多请见 [这里](https://kubernetes.io/docs/concepts/cluster-administration/addons/)
  TODO

### DNS

  其它插件不是十分必要，但这个必须要有，许多示例依赖这个插件
  Cluster DNS，是一个DNS服务，是对其它DNS服务的补充，为k8s 集群 Service 提供 DNS 记录
  由 k8s 启动的容器会自动包含 此 DNS 的配置

### Web UI (Dashboard)

  一个通用的 k8s 集群 web UI. 可用于集群和集群内应用的管理和调试
  [Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)

### 容器资源监控

  记录容器的时序监控数据到到一个中心数据库，并提供查看数据的UI
  [Container Resource Monitoring](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/)

### 集群级的日志

  提供集群级的容器日志采集并存储到中心日志存储的机制，并提供查询和阅览接口
  [Cluster-level Logging](https://kubernetes.io/docs/concepts/cluster-administration/logging/)

## 参考资料

https://kubernetes.io/docs/concepts/overview/components/

## 引申阅读

- [节点](https://kubernetes.io/docs/concepts/architecture/nodes/)
- [控制器](https://kubernetes.io/docs/concepts/architecture/controller/)
- [调度器](https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/)
- [etcd 官网](https://etcd.io/docs/)
- [k8s 集群高可用](https://kubernetes.io/docs/admin/high-availability/)
