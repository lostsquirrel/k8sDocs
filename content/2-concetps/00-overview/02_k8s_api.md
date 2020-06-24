---
title: k8s API
weight: 20002
date: 2020-06-24
draft: true
---
# The Kubernetes API
全部的API约定在这(https://git.k8s.io/community/contributors/devel/api-conventions.md)
API endpoints,资源类型，示例在这(https://kubernetes.io/docs/reference)
远程访问 API 的说明在这(https://kubernetes.io/docs/admin/accessing-the-api)
k8s API 是系统声明式配置的基础， 命令行工具 `[kubectl](https://kubernetes.io/docs/user-guide/kubectl/)` 可以用来进行对 API 对象的增删改查
k8s 以API 资源的形式也存储了序列化状态(目前使用 [etcd](https://coreos.com/docs/distributed-configuration/getting-started-with-etcd/))
k8s 自身也解耦成多个模块，通过 API 进行交互

## API 的变化
因为满足更多的应用场景的需要，k8s API 会持续的变化和增长，并尽量保证兼容性
API中资源和字段的删除依照以这个策略(https://kubernetes.io/docs/reference/deprecation-policy/)
关于怎么做兼容和怎么修改API,依照这个(https://git.k8s.io/community/contributors/devel/api_changes.md)


## OpenAPI and Swagger definitions
k8s 使用 Swagger v1.2 和 OpenAPI 提供文档， swagger的地址为 /swaggerapi， swagger ui 的地址为 /swagger-ui，需要配置 apiserver 启动参数 --enable-swagger-ui=true
基于Go的序列化模式API 参见Go 源码文档

## API 版本
为做版本兼容，每个版本的 API 路径都不一样，比如 `/api/v1` `/apis/extensions/v1beta1`
通过 API 的变化来实现 对过期和实验性 API 的访问控制
API 的版本和软件的版本是间接相关的(和 Docker 类似)，详见(https://github.com/kubernetes/community/blob/master/contributors/design-proposals/release/versioning.md)
不同的 API 版本提供不同级别的稳定性和支持，详见(https://git.k8s.io/community/contributors/devel/api_changes.md#alpha-beta-and-stable-versions)

- Alpha
    - api 名字中带有 alpha (e.g. v1alpha1)
    - 可能有bug, 默认关闭
    - 所支持的功能可能在没有任何通知的情况下被移除
    - 在未来的版本中可能会与当前不兼容
    - 推荐用于短期实验集群，bug 风险高，缺乏长期支持

- Beta
    - api 名字中带有 beta (e.g. v2beta3)
    - 代码是充分测试的，打开这个功能基本上是安全的，默认打开
    - 整体功能不会被删除，但细节可能会修改
    - 未来版本可能会出现不兼容情况，官方提供迁移指导，可能需要删除，编辑或重建 API对象，此过程可能会引起相关应用不可用
    - 推荐 non-business-critical 用户使用，如果有几个集群，并可独立更新，也没限制

- Stable
    - api 版本名称模式为 vX X为整数
    - 稳定版功能会在以后很多个版本中存在

## API 分组
    详见(https://git.k8s.io/community/contributors/design-proposals/api-machinery/api-group.md)
    1. 核心组 路径为 `/api/v1` 使用 `apiVersion: v1`
    2. 其它组 路径为 `/apis/$GROUP_NAME/$VERSION` 使用 `apiVersion: $GROUP_NAME/$VERSION (e.g. apiVersion: batch/v1)`

    算定义扩展(暂时不作研究)

## API 分组开关

    在 apiserver 启动参数中使用 --runtime-config 进行打开或关闭
    关闭 --runtime-config=batch/v1=false
    打开 --runtime-config=batch/v2alpha1
    多个用逗号分隔
    需要重启 apiserver 和 controller-manager

## 组资源的开关
    默认打开的有 DaemonSets, Deployments, HorizontalPodAutoscalers, Ingress, Jobs, ReplicaSets
    也是在 apiserver 启动参数中使用 --runtime-config 进行打开或关闭
    --runtime-config=extensions/v1beta1/deployments=false,extensions/v1beta1/ingress=false
