---
title: 标签和标签选择器
weight: 2000303
date: 2020-06-29
publishdate: 2020-07-02
---

标签是附加在对象上的键值对，应该与所在的对象有关且具有意义
标签可以用于组织和筛选一组对象
标签可以在对象创建时就定义，也可以在对象创建后任何时间添加或修改
一个对象可以有多个标签，但同一个对象所有标签的名称必须唯一
标签可以让UI或命令行工具快速地查询和监听对象，所以标签不适用于非标识性的内容，非标识性内容应该使用注解([annotations](../04-annotation/))

## 动机

标签让用户可以以松耦合的形式把组织架构映射到系统对象上，而不需要客户端存在这些映射
服务部署和批处理流水线通常胡是多维的实体(比如: 多个分区或部署， 多条发布线， 多个层级， 每个次级又有多个微服务)。 要管理这些对象经常需要多维度分割操作， 这就需要打破严格的层级结构的表现形式， 特别是由基础设施而不是由用户决定的死板的层级结构

常用标签示例：

- `"release" : "stable"`, `"release" : "canary"`
- `"environment" : "dev"`, `"environment" : "qa"`, `"environment" : "production"`
- `"tier" : "frontend"`, `"tier" : "backend"`, `"tier" : "cache"`
- `"partition" : "customerA"`, `"partition" : "customerB"`
- `"track" : "daily"`, `"track" : "weekly"`

## 语法和字符集 {#syntax-and-character-set}

标签由 键值对组成。 合法的键可以由两上部分组成， 一个可选的前级加上本身的名称中间用斜线(`/`)分隔。
名称部分 不得多于63个字符，必须以字母或数字 ([a-z0-9A-Z])开头和结束，中间部分可以包含中划线(`-`)，下划线 `(_)`，点 (`.`),字母,数字。
如果要使用前缀， 前端必须是一个合法的 DNS 字域名，由多个 DNS 标签组成，中间由点(`.`)分隔， 总长度不超过 253 个字符

如果一个标签键没有前缀则假定它是属于用于私有的，
由系统自动化组件(e.g. `kube-scheduler`, `kube-controller-manager`, `kube-apiserver`, `kubectl`, 或其它第三方自动化工具), 在给用户对象加标签时必须加前缀， `kubernetes.io/`和 `k8s.io/` 为 k8s 核心组件保留前缀

值 可以为空，不多于63个字符，必须以字母或数字 ([`a-z0-9A-Z`])开头和结束，中间可以包含中划线(`-`)，下划线 `(_)`，点 (.),字母,数字

以下示例中的 Pod 包含 `environment: production` 和 `app: nginx` 两个标签:
```yaml

apiVersion: v1
kind: Pod
metadata:
  name: label-demo
  labels:
    environment: production
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

## 标签选择器

与对象名称和对象UID不同，对象标签在对象之间不需要唯一，并且一般来说，会有多个对象有相同的标签
用户或客户端程序可以选择器可以通过标签选择器选取一个对象集. 标签选择器是 k8s 核心分组r 基础
目前标签选择器支持两种选择方式： 等值选择，集合选择。
在待值选择时， 一个标签选择器可以由多个选择条件组成，每个条件用逗号分隔，表示匹配同时满足这些条件的对象，所以这里逗号相当与逻辑与(`&&`)关系
空选择器或不使用选择在不同的情况下，有不同的表现。用户标签的 API 需要要正确和有意义的说明文档
注意: 对于有些 API 类型，比如 `ReplicaSets` 同一个名字空间两个不同实例的标签选择器不能有交叉， 否些控制器就会将些认为是冲突，导致不能别副本数是否正确(TODO 这里需要有一个示例，描述不太好理解)

警告: 在编写选择器条件时需要注意，对于 等值选择，集合选择 都没有逻辑与(`||`)操作符

### 等值选择

等值选择可以分别对于标签的 键和值的相等与不相等，只要对象的标签含有选择器所有的条件相匹配的标签就会选中，不管该对象是否还有其它标签
操作符
- =,==, 表示等于; 对象标签的键和值都要一致
- != 表示不等于; 对象标签有该键，但不是该值
例：
```
environment = production # 筛选包含标签键为 environment， 且对应值为 production 的对象
tier != frontend # 筛选包含标签键为 tier，且对应值不为 frontend 的对象
```
筛选`production`环境中层级除 `frontend` 外的对象可以写成 `environment=production,tier!=frontend`

等值选择标签选择器的一个应用场景为为 Pod 指定 节点选择的条件。以下 Pod 节点选择的条件为 `accelerator=nvidia-tesla-p100`

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

### 集合选择器

标签的集合选择器可以且于多值过虑。 支持三种操作符 `in`,`notin` 和 `exists`(仅用在键上)
示例:
```
environment in (production, qa) # 筛选 有键为 environment 且对应值为 production 或 qa 的对象
tier notin (frontend, backend) # 筛选 有键为 tier 且值 不是frontend 也不是 backend 的对象和所有不包含键为 tier 的对象
partition # 筛选 存在键为 tier 不管值是啥的对象
!partition # 筛选 不存在键为 tier 不管值是啥的对象
partition,environment notin (qa) # 筛选 存在键为 partition 且 environment 的值不是 qa
```
多个条件之间的逗号赞同一逻辑与(`&&`)
`environment=production` 与 `environment in (production)` 等同
 `!=` 与 `notin` 单值时等同
两种标签选择方式可以组合使用 例： `partition in (customerA, customerB),environment!=qa`

## 选择器在 API 上的使用

### LIST/ WATCH 操作时过虑对象

在时行LIST/ WATCH 操作时可以通过标签选择筛选需要的对象。 上节提及的两种方式都可以使用， 在URL中请求参数类似如下:
- 等值选择: `?labelSelector=environment%3Dproduction,tier%3Dfrontend`
- 集合选择: `?labelSelector=environment+in+%28production%2Cqa%29%2Ctier+in+%28frontend%29`

也可以用于 REST 客户端 或 `kubectl`
例如 `kubectl` 中使用

等值选择
```sh
kubectl get pods -l environment=production,tier=frontend
```
集合选择
```sh
kubectl get pods -l 'environment in (production),tier in (frontend)'
```
集合选择的表达更宽泛，比如这个可以达到或的效果
```sh
kubectl get pods -l 'environment in (production, qa)'
```
逻辑否的效果
```sh
kubectl get pods -l 'environment,environment notin (frontend)'
```
### Set references in API objects 啥意思

有些 k8s 对象，比如 [Service](../../../04-services-networking/00-service/) [ReplicationController](../../../03-workloads/01-controllers/01-replicationcontroller/) 也是通过标签选择器来限定被其管理的资源对象，比如 [Pod](../../../03-workloads/00-pods/)

### `Service` 和 `ReplicationController` 对选择器的使用

Service 通过标签选择器来指定负载均衡的 Pod
ReplicationController 也是通过标签选择器来指定被其管理的 Pod

只支持等值选择，可以为 `yaml` 或 `JSON` 格式
示例:
```yaml
selector:
    component: redis # 相当于 component=redis 或 component in (redis)
```

### 支持集合选择的资源

较新的资源，如 `Job`, `Deployment`, `ReplicaSet`, `DaemonSet` 都支持集合选择
```yaml
selector:
  matchLabels:
    component: redis
  matchExpressions:
    - {key: tier, operator: In, values: [cache]}
    - {key: environment, operator: NotIn, values: [dev]}
```
- `matchLabels` 是一个键值对字典
```yaml
selector:
  matchLabels:
    component: redis
# 等同与
selector:
  matchExpressions:
    - {key: component, operator: In, values: [redis]}
```

- `matchExpressions` 是一个条件的集合
其条件中可用的操作符(operator)包括: `In`, `NotIn`, `Exists`, `DoesNotExist`
`NotIn` 的值必须非空
包括 `matchLabels` 和 `matchExpressions` 定义的条件,所有条件之间的关系为逻辑与.

### 选择节点集合

标签的另一个应用场景为筛选 Pod 可以调度的 节点。 具体见[这里](../../../09-scheduling-eviction/02-assign-pod-node/)
