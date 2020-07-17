---
title: Cloud Controller Manager
date: 2020-07-15
draft: true
weight: 20103
---
{{< feature-state for_k8s_version="v1.11" state="beta" >}}

云基础设施技术让用户可以在公有云，私有云，混合云上运行 k8s. k8s 倡导自动化， API 驱动，组件之间松耦合的基础设施

`cloud-controller-manager` 是集成了云提供商控制逻辑的 k8s 控制中心组件。 云提供商控制管理器让集群与云提供商提供的 API 相关系并让与云平台交互的组件和与集群交互的组件分离。

`cloud-controller-manager` 组件通过让 k8s 与底层云基础设施的交互逻辑解耦，使得云提供上功能发布的节奏与 k8s 项目功能发的节奏分离
`cloud-controller-manager` 以插件结构的方便让不同的云提供与可以让其平台可以与 k8s 集成。

以下为 `cloud-controller-manager` 在 k8s 架构中的位置：
![kubernetes architecture](https://d33wubrfki0l68.cloudfront.net/7016517375d10c702489167e704dcb99e570df85/7bb53/images/docs/components-of-kubernetes.png)

The cloud controller manager runs in the control plane as a replicated set of processes (usually, these are containers in Pods). Each cloud-controller-manager implements multiple controllers in a single process.
云提供商控制管理器在控制中心中以副本集进程的形式运行(通常为 Pod 中的容器)。 每个 `cloud-controller-manager` 在一个进程中实现了多个控制器。

{{ <node> }}
云提供商控制管理器通常以插件的方式运行而不是以控制中心组件的方式运行
{{ </node> }}



### 路由控制器

路由控制器负责在云提供商配置适当的路由以实现不同节点之间的通信。
根据云提供商的不同， 路由控制器还可能要为 Pod 网络申请一个IP段

### Service 控制器

在云环境中 Service 会与一些云基础设施组件集群，比如负载均衡，IP地址， 网络包过滤，目标健康检测。 Service 控制器在创建 Service 时调用云提供商的API 设置 Service 需要的负载均衡和其它云基础设施组件

## 授权

本节将分别说明云提供商管理器执行对应操作所需要访问的各种 API 对象的

### 节点控制器

节点控制器只需要访问节点对象。需要提供节点对的的读写权限

`v1/Node`:

  - Get
  - List
  - Create
  - Update
  - Patch
  - Watch
  - Delete

### 路由控制器

路由控制器需要监听节点对象的创建来配置对应的路由。 所以需要节点对象的 `Get` 权限

`v1/Node`:
  - Get

### Service 控制器

Service 控制器监听 Service 对象的创建，更新，删除事件来配置对应的 Endpoint

访问 Service 需要 List， Watch 权限
更新 Service 需要 Patch， Update 权限
设置 Service 对的 Endpoint 需要 Create, List, Get, Watch, Update
最终需要如下：

`v1/Service`:
  - List
  - Get
  - Watch
  - Patch
  - Update

### 其它权限需求

云提供商控制管理器核心的实现中需要创建  Event 对象的权限。 为了设置安全操作，还需要创建 ServiceAccount 的权限

`v1/Event`:

  - Create
  - Patch
  - Update

`v1/ServiceAccount`:

  - Create

最终云提供商控制管理器基于 RBAC ClusterRole 配置如下:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cloud-controller-manager
rules:
- apiGroups:
  - ""
  resources:
  - events
  verbs:
  - create
  - patch
  - update
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - '*'
- apiGroups:
  - ""
  resources:
  - nodes/status
  verbs:
  - patch
- apiGroups:
  - ""
  resources:
  - services
  verbs:
  - list
  - patch
  - update
  - watch
- apiGroups:
  - ""
  resources:
  - serviceaccounts
  verbs:
  - create
- apiGroups:
  - ""
  resources:
  - persistentvolumes
  verbs:
  - get
  - list
  - update
  - watch
- apiGroups:
  - ""
  resources:
  - endpoints
  verbs:
  - create
  - get
  - list
  - watch
  - update
```

## 接下来应该看啥

这篇讲怎么运行管理 云提供商控制管理器 [云提供商控制管理器管理](../../../3-tasks/01-administer-cluster/09-running-cloud-controller/#cloud-controller-manager)

想要自己开发一个 云提供商控制管理器 [云提供商控制管理器开发](../../../3-tasks/01-administer-cluster/18-developing-cloud-controller-manager/)
