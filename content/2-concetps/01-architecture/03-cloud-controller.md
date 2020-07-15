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

{{<node>}}
云提供商控制管理器通常以插件的方式运行而不是以控制中心组件的方式运行
{{</node>}}
