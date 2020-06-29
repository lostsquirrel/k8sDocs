---
title: k8s 对象介绍
weight: 200031
date: 2020-06-29
draft: true
---

本文介绍 k8s 对象 是怎么在 k8s API 中表示的，怎么以 `.yaml` 格式输出 k8s 对象

## 简介

k8s 对象是 k8s 系统中持久化的实体， k8s 使用这些实体来表示集群的状态，它们具体可以表示如下:
- 哪些容器的应用在(如个节点上)运行
- 可用于运行应用的资源
- 应用的行为策略，比如重启策略，升级，容错性

k8s 对象是对用户一个意图的记录，当用户创建一个对象后，系统需要保证对象持续存在。用户通过创建一个来告知 k8s 需要一个什么样的 工作负载(workload), 也就是集群的期望状态(desired state)

要实现对 k8s 对象的管理，比如增删改查都需要调用 k8s API, 例如，当用户可以通过 kubectl 命令来实现对API的调用。也可以通过自己写程序实现对 k8s API 的调用，调用库详见(https://kubernetes.io/docs/reference/using-api/client-libraries/)

## 对象的 `Spec`，`Status` 属性

基本上所有的 k8s 都包含两个嵌套对象作为属性用于管理对象的配置，其中一个对象为 `spec`， 另一个为 `status`， 用户在创建对象进设置 `spec` 来定义所需资源的特性，也就是集群的期望状态。

而 `status` 字段，则是对象的当前的实际状态，由 k8s 系统及其组件进行提供和修改。 k8s 控制中心的任务就是始终让所有对象的实际状态与期望状态一致。

例如: 在 k8s 中， 一个  Deployment 对象表示运行有用户集群中的一个应用，当用户创建一个 Deployment 并在 spec 对象中设置应用副本数为 3时， k8s 会读取对象属性，启动用户所期望的三个实例并更新相应的状态以达成与 spec 配置的一致。 如果其它任意一个实例失效(某一状态发生变化)， k8s 将会对 spec 与 status 之间的差异采取行动，在当前描述的情况下就会再启动一个实例代替失效的实例。更多关于对象 `spec`, `status`, `metadata` 相关信息看[这里](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md)


## 表示 k8s 对象

    一般情况下使用 kubectl, kubectl 会将 .yaml 格式转化为 k8s API 需要的 JSON 模式
    示例：
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:1.7.9
            ports:
            - containerPort: 80
    ```

    通过 `kubectl create` 创建对象
    ```sh
    kubectl create -f https://k8s.io/docs/user-guide/nginx-deployment.yaml --record
    ```
## 必要字段

    使用 `.yaml` 创建对象时的必要字段
    - `apiVersion` 使用哪个版本的 k8s API 创建这个对象
    - `kind` 创建对象的类型
    - `metadata` 唯一标识对象的信息，
        - `name` 字符串
        - `UID` (系统会生成?)
        - `namespace`, 可选，默认 default
    - `spec` 对象实际定义， 每类对象不一样，详见(https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/)

## 源文件

https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/

## 引申阅读

https://git.k8s.io/community/contributors/devel/api-conventions.md
