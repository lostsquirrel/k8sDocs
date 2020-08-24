---
title: StatefulSet
id: statefulset
date: 2018-04-12
full_link: /k8sDocs/concepts/workloads/controllers/statefulset/
short_description: >
  管理一个 Pod 集合的部署与容量伸缩， 这些 Pod 所以使用的存储是持久的(Pod 被替代后，新的 Pod 继承老 Pod 的存储)， Pod 的标识也是持久化的(重建 Pod 后名字不会变)
aka:
tags:
- fundamental
- core-object
- workload
- storage
---
管理一个 {{< glossary_tooltip term_id="pod" >}} 集合的部署与容量伸缩，*并提供了顺序的一致性和唯一性*
<!--more-->

与 {{< glossary_tooltip term_id="deployment" >}} 类似， StatefulSet 也是基于相同的容器定义来管理 Pod 的。
与 Deployment 不同的是 StatefulSet 维护了其所管理的每个 Pod 的唯一身份。 这些 Pod 是使用现一份定义创建的，但它们之间是不可互换的:
每个 Pod 在所有的重新调度过程中都会被维护使其拥有唯一的持久化标识。

如果用户需要在工作负载中使用存储卷来提供持久化，可以使用 StatefulSet 作为解决方式的一部分。 尽管 StatefulSet 中每个独立的 Pod 是容易挂掉的，
但是拥有持久化身份标识的 Pod 能够很容易地将已经存在的数据卷与新创建用于替换挂掉的 Pod 重新绑定
