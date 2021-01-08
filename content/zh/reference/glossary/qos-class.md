---
title: QoS 类别
id: qos-class
date: 2019-04-15
full_link:
short_description: >
  QoS 类别 (服务质量类别)提供了对集群中的 Pod 分为几种不同类型的方式，并用于调度和驱逐决策。
aka:
tags:
- core-object
- fundamental
- architecture
related:
 - pod

---
 QoS 类别 (服务质量类别)提供了对集群中的 Pod 分为几种不同类型的方式，并用于调度和驱逐决策。

<!--more-->
QoS Class of a Pod is set at creation time  based on its compute resources requests and limits settings. QoS classes are used to make decisions about Pods scheduling and eviction.
Kubernetes can assign one of the following  QoS classes to a Pod: `Guaranteed`, `Burstable` or `BestEffort`.

Pod QoS 类别是在创建时基于它的计算资源请求和限制设置来生成的。 QoS 类别被用于调度和驱逐决策。
k8s 可以给一个 Pod 分配以下 QoS 类别中的一种: 保证(`Guaranteed`), `Burstable` 或 尽力(`BestEffort`).
