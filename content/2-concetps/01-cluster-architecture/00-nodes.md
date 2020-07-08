---
title: 节点
weight: 20100
date: 2020-06-30
publishdate: 2020-07-08
---
k8s 将用户的工作负载窗口塞进 Pod 里然后运行在节点上，根据集群节点可以是虚拟机或物理机，每个节点必须要有运行 Pod 所需要的服务，并由 k8s 控制中心管理。
一般情况下一个集群会有多个节点; 在资源受限或学习的环境，可能只有一个节点。
每个节点包含的组件有 kubelet, 容器运行环境, kube-proxy

## 节点管理

向 api-server 添加节点的方式有以下两种:

1. 节点上的 kubelet 服务自动注册到控制中心
2. 管理员用户手动添加节点对象

当管理员用户创建节点对象或 kubelet 将节点自动注册，控制中心会检查新创建的新节点是否有效。例如，可以通过以下 JSON 创建一个新节点
```json
{
  "kind": "Node",
  "apiVersion": "v1",
  "metadata": {
    "name": "10.240.79.157",
    "labels": {
      "name": "my-first-k8s-node"
    }
  }
}
```

k8s 节点是内部创建的. k8s 检查通过 kubelet 注册到 api-server 的节点. 如果节点是健康的 (当所有必要的服务都正常运行), 这个节点就可以用来运行 Pod, 否则 这个节点状态在变为健康之前，这个节点会被所有集群活动忽略.

注意: k8s 会一直保留无效的节点并持续检查这个节点是否变更为健康. 用户或控制器必须要显示的删除这个节点对象，这种检测才会停止。

节点的名称必须要是一个有效的[DNS 子域名](../../00-overview/03-working-with-objects/01-names/#DNS%20子域名)

### 节点的自注册

当 kubelet 参数 `--register-node` 设置为 `true` 时(默认值)， kubelet 会自动把所在节点注册到 api-server. 这是首选的配置方式，被多数集群搭建工具使用。

对于自注册的节点， kubelet 需要以下配置参数:

- `--kubeconfig`  节点在 api-server 认证凭据所在目录
- `--cloud-provider` 怎么从云提供商获取节点元数据
- `--register-node` 自动注册到 api-server
- `--register-with-taints` 为注册节点添加 taints (格式为 <key>=<value>:<effect>，多个由逗号分隔)，如果 register-node 值设置为 false 则，该配置无操作
- `--node-ip` 节点的 IP 地址
- `-node-labels` 当节点注册是，添加到节点上的标签
- `--node-status-update-frequency` kubectl 向控制中心报告状态的频率

当 [Node authorization mode](https://kubernetes.io/docs/reference/access-authn-authz/node/) 和 [NodeRestriction admission](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#noderestriction) 插件打开进, kubelet 只被授权创建/修改自己的节点资源

### 节点手动管理

用户可以通过 kubectl 创建和修改节点对象
当用户需要手动创建一个节点对象时，需要 kubelet 设置参数  `--register-node=false`
也可修改节点配置忽略 `--register-node` 配置。 例如可以在存在的节点上设置标签或将节点标记为不可调度
可以通过节点的标签和Pod的节点标签选择器来控制调度。 例如， 限制某个 Pod 只能在某些节点上运行
标记一个节点为不可调度后，就会阻止调度器再向这个节点调度新的 Pod ， 但不会影响到节点上已经在运行的 Pod。 以下命令将标记指定节点为不可调度:
```sh
kubectl cordon $NODENAME
```

注意: 属于 DaemonSet 的 Pod 会运行在不可调度的节点上， 因为 `DaemonSets` 通常提供节点本地服务，所以即使节点被清空应用工作负载也应该运行在节点

## 节点状态

节点状态包含以下信息:

- 地址
- 条件
- 容量和可分配状态
- 其它信息

可以通过以命令查看节点状态与其它更多信息:
```sh
kubectl describe node <insert-node-name-here>
```
每个部分详细信息

### 地址

这些字段会根据节点是云提供或裸金属和等的不同而不同

- 主机名: 由节点的内核提供，可以通过 kubelet 的 --hostname-override 覆盖
- 外部IP: 在外部网络(集群之外)路由可达的IP地址
- 内部IP: 只在集群内部路由可达的IP地址

### 条件

conditions 字段描述的是 所有状态为 Running 的节点， 示例如下:

- Ready  
  - True 则表示节点健康，可以接收调度 Pod
  - False 则表示节点状不健康，不可接收 Pod
  - Unknown node 控制器在最近一个 `node-monitor-grace-period` 内(默认40s)没有收到节点的心跳

- DiskPressure
  - True 磁盘存储剩余空间紧张
  - False 磁盘存储剩余空间充足

- MemoryPressure
  - True 节点内存紧张
  - False 节点内存充足

- PIDPressure
  - True 节点上的进程太多了
  - False 节点上进程数量适度

- NetworkUnavailable
  - True 节点网络配置错误
  - False 节点网络配置正常

注意: 通过命令行工具打印清楚节点的详细信息中， 条件信息中包含 `SchedulingDisabled`， 但 SchedulingDisabled 不属于 k8s API, 而是节点被标记为不可调度

节点的条件可以表现为JSON对象，如下是一个健康的节点的示例:
```json
"conditions": [
  {
    "type": "Ready",
    "status": "True",
    "reason": "KubeletReady",
    "message": "kubelet is posting ready status",
    "lastHeartbeatTime": "2019-06-05T18:38:35Z",
    "lastTransitionTime": "2019-06-05T11:41:27Z"
  }
]

```

如果 `Ready` 条件的状态为 `Unknown` 或 `False` 持续时间超过 `pod-eviction-timeout` (通过  kube-controller-manager 参数配置)， 这个节点上所以的 Pod 都会被节点控制器调度为删除。 默认的踢除的超时时间为5分钟。 在某些情况下，当一个节点不可达时，api-server 就不能与节点上的 kubelet 进行通信。 在 kubelet 与 api-server 重新建立连接之前对 Pod 的删除指令传达不到 kubelet. 在这段时间内这些被调度为删除的节点可以继续在分区节点上运行。

节点控制器在确认 Pod 在集群中已经停止运行前是不会强制删除的。 所以可以会出现 Pod 运行在状态为  `Terminating` 或 `Unknown` 的不可达节点上。 有时候 k8s 不能在节点被永久移出集群后不能从基础设施自动的移除，需要管理员手动地删除对应的节点对象。从 k8s 中删除节点对象时，会同时从 api-server 中对应删除节点上运行的所有 Pod 对象，并释放其名称

节点的生命周期管理器会自动创建代理节点状态的 `Taint`. 调度器在分配 Pod 到节点时会顾及节点上的 Taint. Pod 也可以配置容忍节点的某些 `Taint`

了解更多 Taint 相关信息见[这里](../../09-scheduling-eviction/01-taint-and-toleration.md)

### 容量和可分配状态

表示节点上的可用资源: CPU, 内存， 节点可接受 Pod 数量的最大值
容量(capacity)块下的字段表示节点资源总数
可分配状态(allocatable)块下的字段表示可用于普通 Pod 的资源数

了解更多关于 容量和可分配状态 的信息见[这里](../../../3-tasks/01-administer-cluster/28-reserve-compute-resources.md)

### 其它信息

表示邛的通用信息，如 内核版本， k8s 版本(kubelet 和 kube-proxy 的版本)， Docker 版本， OS 名称。 这些信息都是节点上的 kubelet 生成的

### 节点控制器

节点控制器是 k8s 控制中心的组成部分，用于管理节点的各方面功能
节点控制器在节点的生命周期类扮演多个角色。 第一个就是为节点分配 CIDR 段(当 CIDR 分配开启时)
第二个是保持节点控制器内部的节点列表与云提供商提供的可用机器列表一致， 在运行在云环境时，当一个节点状态变为不健康时， 节点控制器会向云提供商查询该节点的虚拟机是否可用。 如果不可用就会从列表中删除该节点
第三个是监控节点状态， 节点控制器负责在节点变得不可达时(如， 因为某些原因收不到心跳，比如节点宕机)，更新节点就绪状态为 `ConditionUnknown`， 如果节点持续不可达则踢出节点上所有的Pod(使用优雅终结方式)。设置就绪状态的不可达时间为 40s, 踢除 Pod 的时间为 5 分钟。节点控制器检查节点状态的时间由 `--node-monitor-period` 配置


#### 心跳

心跳由k8s 节点发送，用于帮助判定节点是否可用
心跳的形式有两种， 一个更新 `NodeStatus` 另一个为租约对象。每个节点在 kube-node-lease 名字空间中有一个关联的租约对象。 租约是一个轻量级资源， 用户在集群范围内改善心跳的性能
由 kubelet 负责创建更新 `NodeStatus` 和 租约对象。

- kubelet  在状态发生改变或在配置的时间间隔内没有更新时更新 `NodeStatus`， `NodeStatus` 更新的默认的更新间隔为 5 分钟(比不可达节点默认超时的 40 秒长很多)
- kubelet 创建随后每隔10秒(默认时间间隔)更新租约对象。 租约对象的更新独立与 `NodeStatus` 更新。 如果租约更新失败， kubelet 使用从 200ms 到 7s 间的指数组补尝

#### 可靠性

大多数情况下， 节点控制器限制踢除速率由 `--node-eviction-rate`(每秒) 配置， 默认 `0.1`，即每10秒最多能踢除一个 Pod。
节点的踢除行为会在节点所有的可用区变为不健康时发生改变。 节点控制器会检查区域内同一时间不健康(`NodeReady` 条件为 `ConditionUnknown` 或 `ConditionFalse`)节点的百分比。 如果不健康的节点比达达到 `--unhealthy-zone-threshold` (默认 0.55)时踢除速率会降低: 如果集群较小(不多于 `--large-cluster-size-threshold` 节点, 默认 50)，踢除行为停止。否则踢除速率降低至 `--secondary-node-eviction-rate` (默认 0.01)每秒。 这个策略会在每个分区实现因为这个可用区可能与集群分隔， 但其它部分仍正常连接。 如果集群不是分散在多个云提供商的可用区，则只有一个可用区(即整个集群)。

将节点分散在不同可用区的主要原因就当一个可用区整体不可用时，可以将工作负载转移到另一个可用区。 如果一个可用区的节点都不可用时节点控制器使用正常踢除速率(`--node-eviction-rate`). 极限情况是当所有的可用区全部变得不可用时(整个集群每有一个健康节点)， 在这种情况下节点控制器认为主节点有连接问题并停止踢除行为直到连接恢复。

节点控制器负责踢除包含 NoExecute Taint 节点运行的 Pod, 除了包含容忍该 Taint 的 Pod。 节点控制器会为有问题(如不可达或未就绪的节点)添加相应的 Taint, 也就是说调度器不会向不健康的节点放置 Pod.

警告: `kubectl cordon` 命令标记一个节点为不可调度，而其作为是 服务控制器会将节点从所有负载列表中移除， 并再向该节点调度流量。

#### 节点容量

节点对象记录了节点的资源容量(如: 可用内存，CPU核心数)。 自动注册的节点在注册时会报告其容量，如果节点是手动添加，则需要在添加时提供对应容量信息。
k8s 调度器保证节点有足够的资源运行分配到其上的 Pod， 调度器检测节点不所有容器请求的资源不大于节点的容量。 请求资源总和包括由kubelet 管理的所有容器， 但不包括直接由容器运行环境启动的容器。也不包括其它所有不被kubelet 控制的进程。
注意: 如果需要为非 Pod 进程保留资源，见[为系统进程保留资源](../../../3-tasks/01-administer-cluster/28-reserve-compute-resources/),系统保留部分。

## 节点拓扑

功能特性状态: Kubernetes v1.16 [alpha]
如果通过[功能特性开关](../../../5-reference/06-command-line-tools-reference/00-feature-gates/)，开启了 `TopologyManager`， kubelet 在作资源分配决策时会参考拓扑信息。更多信息， 见[节点拓扑控制管理策略](../../../3-tasks/01-administer-cluster/14-topology-manager/)
