---
title: Service, 负载均衡, 网络
weight: 500
date: 2020-06-30
---

<!--
Kubernetes networking addresses four concerns:
- Containers within a Pod use networking to communicate via loopback.
- Cluster networking provides communication between different Pods.
- The Service resource lets you expose an application running in Pods to be reachable from outside your cluster.
- You can also use Services to publish services only for consumption inside your cluster.
 -->

k8s 的网络有如下四个方面:

- 在同一个 Pod 内的容器使用圆环网络通信
- 集群网络提供不同 Pod 之间的通信
- Service 资源让运行在 Pod 中的应用可以在集群外访问
- Service 也可以用于集群内的服务发布
