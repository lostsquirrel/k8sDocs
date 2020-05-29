---
title: Getting started
weight: 10100
---

## 入门
本文主要介绍 k8s 集群的选择与搭建，不同的解决方案对应不同需求: 易于维护， 安全，控制，可用资源，集群运维和管理的经验要求等
k8s 可以部署在本地主机，云主机，on-prem,
也可以基于云供应商或 bare metal来搭建自定义解决方案
往简单说，可以分为学习环境和生产环境集群


## 学习环境

如果学习k8s 可以使用基于 Docker的解决方案(工具由k8s社区维护)或在本地机器上搭建 k8s 集群

|社区                                     |生态|
|----------------------------------------|------|
|Minikube                                |Docker Desktop|
|kind(Kubernetes IN Docker)              |Minishit|
|                                        |MicroK8s|

- Minikube
  [源地址](https://kubernetes.io/docs/setup/learning-environment/minikube/)
  [本站地址](./01-learning-environment/00-minikube/)

- kind
  [源地址](https://kubernetes.io/docs/setup/learning-environment/kind/)
  [本站地址](./01-learning-environment/01-kind/)

- [Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Minishift](https://docs.okd.io/latest/minishift/)
- [MicroK8s](https://microk8s.io/)

## 生产环境

在评估生产环境解决方案时，需要考虑是自己管理k8s集群，或直接使用其它服务商提供的

[k8s 合作伙伴](https://kubernetes.io/partners/#conformance)包含一批有[证](https://github.com/cncf/k8s-conformance/#certified-kubernetes)提供商的列表
