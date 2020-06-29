---
title:
weight:
date: 2020-06-29
draft: true
---

# 标签和标签选择器
标签是附加在对象上的键值对，应该与所在的对象有关且具有意思
标签可以用于组织和筛选一组对象
标签可以在对象创建时就定义，也可以在对象创建后任何时间添加或修改
一个对象可以有多个标签，但同一个对象所有标签的名称必须唯一
标签不适用于非标识性的内容，非标识性内容应该使用注解(annotations)

## 动机
标签让用户可以以松耦合的形式把组织架构映射到系统对象上，而不需要客户端存在这些映射
常用标签示例：
- "release" : "stable", "release" : "canary"
- "environment" : "dev", "environment" : "qa", "environment" : "production"
- "tier" : "frontend", "tier" : "backend", "tier" : "cache"
- "partition" : "customerA", "partition" : "customerB"
- "track" : "daily", "track" : "weekly"

## 语法和字符集
    - 键 由前缀和名称组成，由斜线(/)分隔
        - 前缀 可选，必须是一个 DNS 字域名 总长度不超过 253 个字符
        - 名称 不多于63个字符，必须以字母或数字 ([a-z0-9A-Z])开头和结束，包含中划线(-)，下划线 `(_)`，点 (.),字母,数字

    系统自动化组件(e.g. kube-scheduler, kube-controller-manager, kube-apiserver, kubectl, or other third-party automation), 给最终用户对象加标签时必须加前缀， `kubernetes.io/` 为 k8s 核心组件保留使用

    - 值 可以为空，不多于63个字符，必须以字母或数字 ([a-z0-9A-Z])开头和结束，包含中划线(-)，下划线 `(_)`，点 (.),字母,数字

## 标签选择器
    在对象之间，标签不需要唯一，并且一般来说，会有多个对象有相同的标签
    通过标签选择器，用户/客户端可以选择器，选取一个对象集
    标签选择器是k8s 核心分组基础
    目前支持两种选择方式： 等值选择，集合选择
    一个标签选择器，可以有多个选择条件，用逗号分隔，条件之间为逻辑与关系
    一个空选择器，选择所有对象
    一个null选择器，不选择任何对象
    两个controller 的选择器不能有交集，不然他们会打架

### 等值选择
    操作符
    - =,==, 表示等于; 对象标签的键和值都要一致
    - != 表示不等于; 对象标签有该键，但不是该值
    例：
    ```
    environment = production # 包含标签键为 environment， 标签值为 production
    tier != frontend # 包含标签键为 tier，标签值不为 frontend
    ```
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: cuda-test
    spec:
      containers:
        - name: cuda-test
          image: "k8s.gcr.io/cuda-vector-add:v0.1"
          resources:
            limits:
              nvidia.com/gpu: 1
      nodeSelector:
        accelerator: nvidia-tesla-p100
    ```

### 集合选择
    操作符
    - in `environment in (production, qa)` 对象标签有该键， 且值在这个集合内
    - notin `tier notin (frontend, backend)` 对象标签有该键， 且值不在这个集合内
    - exists
        `partition` 对象标签有该键
        `!partition` 对象标签没有该键

两种选择方式可以组合使用 例： `partition in (customerA, customerB),environment!=qa`

## 选择器在 API 上的使用
    LIST/ WATCH 过虑
    - equality-based requirements: ?labelSelector=environment%3Dproduction,tier%3Dfrontend
    - set-based requirements: ?labelSelector=environment+in+%28production%2Cqa%29%2Ctier+in+%28frontend%29

    ```sh
    $ kubectl get pods -l environment=production,tier=frontend

    $ kubectl get pods -l 'environment in (production),tier in (frontend)'

    # 达到 逻辑与的效果
    $ kubectl get pods -l 'environment in (production, qa)'
    #
    $ kubectl get pods -l 'environment,environment notin (frontend)'
    ```

### Service and ReplicationController 对选择器的使用
    ```yaml
    selector:
        component: redis # 相当于 component=redis 或 component in (redis)
    ```

### 支持集合选择的资源
    `Job`, `Deployment`, `Replica Set`, `Daemon Set`
    ```yaml
    selector:
      matchLabels:
        component: redis
      matchExpressions:
        - {key: tier, operator: In, values: [cache]}
        - {key: environment, operator: NotIn, values: [dev]}
    ```

    operator: In, NotIn, Exists, DoesNotExist
