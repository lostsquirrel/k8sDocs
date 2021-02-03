---
title: 入门
content_type: concept
weight: 20
---

<!-- overview -->
<!--
This section lists the different ways to set up and run Kubernetes.
When you install Kubernetes, choose an installation type based on: ease of maintenance, security,
control, available resources, and expertise required to operate and manage a cluster.

You can deploy a Kubernetes cluster on a local machine, cloud, on-prem datacenter, or choose a managed Kubernetes cluster. There are also custom solutions across a wide range of cloud providers, or bare metal environments.
 -->

本节列举了几种不同的搭建和运行 k8s 的方式。
在安全 k8s 时，选择安装方案基于: 易于维护， 安全，控制，可用资源，集群运维和管理的经验要求等

你可以将 k8s 部署在本地主机，云主机，本地数据中心, 或选择一个由托管的 k8s 集群。
也可以基于云供应商或 bare metal来搭建自定义解决方案

<!--
## Learning environment

If you're learning Kubernetes, use the tools supported by the Kubernetes community, or tools in the ecosystem to set up a Kubernetes cluster on a local machine.

 -->
## 学习环境

如果学习k8s 可以使用基于 Docker的解决方案(工具由k8s社区维护)或在本地机器上搭建 k8s 集群

|社区                                     |生态|
|----------------------------------------|------|
|Minikube                                |Docker Desktop|
|kind(Kubernetes IN Docker)              |Minishit|
|                                        |MicroK8s|

- Minikube
  [源地址](https://kubernetes.io/docs/setup/learning-environment/minikube/)
  [本站地址](/k8sDocs/docs/setup/00-minikube/)

- kind
  [源地址](https://kubernetes.io/docs/setup/learning-environment/kind/)
  [本站地址](/k8sDocs/docs/setup/01-kind/)

- [Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Minishift](https://docs.okd.io/latest/minishift/)
- [MicroK8s](https://microk8s.io/)

<!--
## Production environment

When evaluating a solution for a production environment, consider which aspects of operating a Kubernetes cluster (or _abstractions_) you want to manage yourself or offload to a provider.

[Kubernetes Partners](https://kubernetes.io/partners/#conformance) includes a list of [Certified Kubernetes](https://github.com/cncf/k8s-conformance/#certified-kubernetes) providers.
 -->
## 生产环境

在评估生产环境解决方案时，需要考虑是自己管理k8s集群，或直接使用其它服务商提供的

[k8s 合作伙伴](https://kubernetes.io/partners/#conformance)包含一批有[证](https://github.com/cncf/k8s-conformance/#certified-kubernetes)提供商的列表
