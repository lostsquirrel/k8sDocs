---
title: 节点
weight: 20100
date: 2020-06-30
draft: true
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
