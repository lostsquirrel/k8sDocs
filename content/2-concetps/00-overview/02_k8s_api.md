---
title: k8s API 说明
weight: 20002
date: 2020-06-24
publishdate: 2020-06-29
---

k8s API 提供查询操作 k8s 对应状态的功能。 k8s 控制中心的核心是 `api-server` 及其提供的 HTTP API。包括用户，集群的其它组件，外部组件都是与 `api-server` 通信

全部的API约定在[这里](https://git.k8s.io/community/contributors/devel/api-conventions.md)
API endpoints,资源类型，示例在[这里](https://kubernetes.io/docs/reference)
远程访问 API 的说明在[这里](https://kubernetes.io/docs/admin/accessing-the-api)
k8s API 是系统声明式配置的基础， 命令行工具 `[kubectl](https://kubernetes.io/docs/user-guide/kubectl/)` 可以用来进行对 API 对象的增删改查
k8s 以API 资源的形式也存储了序列化状态(目前使用 [etcd](https://coreos.com/docs/distributed-configuration/getting-started-with-etcd/))
k8s 自身也解耦成多个模块，通过 API 进行交互

## API 的变化

任何成功的系统都应该快速响应新的应用场景或现有需求的变更。 因此 k8s 在设计 API 时就提供了持续改进与成长的方式。k8s 项目主旨是不打破对已有客户端的普空性。并将此兼容性保持一段时间，让其它项目有时间来进行适配.
通常新的API 资源和新的API资源项，可以经常频繁增加， 移除资源或资源项则需要遵循 [API废弃策略](https://kubernetes.io/docs/reference/using-api/deprecation-policy/) TODO

关于怎么做兼容和怎么修改API,依照这个(https://git.k8s.io/community/contributors/devel/sig-architecture/api_changes.md#readme)


## OpenAPI 规范

完整 API 规范明细见 [这里](https://www.openapis.org/)
k8s api-server 通过 `/openapi/v2` 提供 OpenAPI 规范， 用户可以通过以下请求头发送请求获取响应格式
请求头                      | 可选值                                                      | 说明
---------------------------|------------------------------------------------------------|-
Accept-Encoding            |gzip                                                        | 可选
<td rowspan=3>Accept</td>  |application/com.github.proto-openapi.spec.v2@v1.0+protobuf  |  mainly for intra-cluster use   
                           |application/json                                            |  默认
                           |*                                                           | 响应 `application/json`
也可以是任务符合OpenAPI 规范的请求头
k8s 还有一个基于基于 `Protobuf` 序列化的API，这套API 主要用于集群内通信。具体见对应模块 Go 源码文档，文档设计准则在[这里](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/api-machinery/protobuf.md)

## API 版本

为方便移除资源项或修改资源结构，k8s 同时存在多个版本的 API, 每个版本的 API 路径都不一样，比如 `/api/v1` `/apis/extensions/v1beta1`
这样版本就通过区分就在API这一层完成，而不用到资源或资源荐，可以做到清晰明了,并提供一致的系统资源的行为。
也可以实现对过期和实验性 API 的访问控制
JSON和Protobuf序列化的定义结构变更策略与API定义资源结构的规则一致，具体如下:

API 的版本和软件的版本只是间接相关的(和 Docker 类似)，详见[k8s 版本发布](https://git.k8s.io/community/contributors/design-proposals/release/versioning.md)提议

不同的 API 版本提供不同级别的稳定性和支持，关于每个级别详细描述的细则见[这个文档](https://git.k8s.io/community/contributors/devel/sig-architecture/api_changes.md#alpha-beta-and-stable-versions)
可以总结为如下几个:

- Alpha 级别
  - api URL名字中带有 alpha (e.g. v1alpha1)
  - 可能有不少bug,启用这个物引可能会引入bug. 默认关闭
  - 所支持的功能可能在没有任何通知的情况下被移除
  - 在未来的版本中可能会在没有通知的情况谱得与之前不兼容
  - 只推荐用于短期实验集群，因为有较高bug多，缺乏长期支持的风险

- Beta 级别
  - api URL名字中带有 beta (e.g. v2beta3)
  - 代码是充分测试的，启用这个功能基本上是安全的，默认打开
  - 整体功能不会被删除，但细节可能会修改
  - 未来版本中可能会因资源的结构与资源项的含义出现不兼容情况，如果出现这种情况,官方会提供升级与该版本的迁移指导，可能需要删除，编辑或重建 API对象，在修改过程可能需要用户仔细思考需要的改动，此过程信赖些功能的应用可能会不可用
  - 只推荐 非关键商业 用户使用，因为在未来版本中可能存在潜在的不兼容变更，如果用户有几个可独立更新集群，也可以不受此限制
  - 强烈建议用户试用beta特性并给予反馈，因为一旦从beta 版本确认为稳定版，则特性将不可更改

- Stable
  - api 版本名称模式为 `vX`， 其中`X`为整数
  - 稳定版功能会存在于稳定版的软件的很多个版本中

## API 分组

为了方便API的扩展， k8s 实现了 API 分组的方式。 API 组在 REST 接口的Path 上的 apiVersion 字段
[设计提案之API分组](https://git.k8s.io/community/contributors/design-proposals/api-machinery/api-group.md)
集群中一般有如下一些组:

1. 核心组,也被称为经典组，REST 路径中为 `/api/v1` 对象定义 `apiVersion: v1`
2. 其它组 REST 路径为 `/apis/$GROUP_NAME/$VERSION` 对象定义为 `apiVersion: $GROUP_NAME/$VERSION (e.g. apiVersion: batch/v1)` [API文档](https://kubernetes.io/docs/reference/kubernetes-api/)是有完整的分级列表

通过自定义资源可以通过以下两种方式扩展API:

1. [CustomResourceDefinition](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) 通过声明方式定义 api-server 怎么提供用户选择资源的API (TODO 这里描述不太清楚)

2. 用户可能通过[实现自己的扩展api-server](https://kubernetes.io/docs/tasks/extend-kubernetes/setup-extension-api-server/), 并通过[聚合](https://kubernetes.io/docs/tasks/extend-kubernetes/configure-aggregation-layer/)使客户端可以访问

## API 分组开关

一些资源和API默认是开启的，可能通过 api-server 命令行启动参数中使用 --runtime-config 进行开启或关闭
关闭 --runtime-config=batch/v1=false
开启 --runtime-config=batch/v2alpha1
如果有多个分组进制设置，可能key=value并用逗号分隔
注意： 开启或关闭分组或资源一需要重启 api-server 和 kube-controller-manager

## 开启 `extensions/v1beta1` 组的指定资源

`extensions/v1beta1` 组默认启用资源的有 `DaemonSets`, `Deployments`, `StatefulSet`, `NetworkPolicies`, `PodSecurityPolicies`， `ReplicaSets`

例如要启用资源 `deployments` 和 `daemonsets`
也是在 api-server 启动参数中使用 --runtime-config 进行开启或关闭
`--runtime-config=extensions/v1beta1/deployments=false,extensions/v1beta1/ingress=false`

## 持久化

k8s 将序列化的 API 资源对象保存在 etcd 中

## 引申阅读

[API访问控制](https://kubernetes.io/docs/reference/access-authn-authz/controlling-access/)
[API约定](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#api-conventions)
[API文档](https://kubernetes.io/docs/reference/kubernetes-api/)
