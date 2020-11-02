---
title: 命名空间(Namespaces)
weight: 2000302
date: 2020-06-29
publishdate: 2020-06-30
---
k8s 可以在一个物理集群上创建多个虚拟集群, 每个命名空间就是一个虚拟集群

## 啥时候用命名空间

命名空间是为用户分散于有多个组或项目下的场景设计的。 如果只有二三十个用户就不用想了，当需要用到命名空间的特性时才考虑用命名空间
命名空间是对象名称的一个作用域，对象命名只需要在一个命名空间唯一即可，命名空间不可以嵌套且一个资源对象只能属于一个命名空间
命名空间是集群中多个(组)用户分配资源的一个方式([通过资源配额](../../../08-policy/01-resource-quotas))
未来版本，一个命名空间下的对象可能有相同的默认访问控制策略
不必要用命名空间来区分差异较小的资源，比如同一个软件的不同版本，可以同一个命名空间下使用标签(labels)区分这些对象

## 管理命名空间

命名空间的创建和删除请见[管理指南命名空间部分](../../../../3-tasks/01-administer-cluster/34-namespaces)

注意: 在自定义命名空间是，避免使用 `kube-` 作为前缀， 这个前缀是 k8s 命名空间保留字

### 查看

可以通过以下命令查看当前集群所有命名空间:

```sh
kubectl get namespaces
```
输出类似如下:
```
NAME              STATUS   AGE
default           Active   1d
kube-node-lease   Active   1d
kube-public       Active   1d
kube-system       Active   1d
```

k8s 启动时会初始化创建以下4个命名空间:
- `default：` 当对象不指定命名空间时所属的命名空间
- `kube-system`： 由系统创建对象所在的命名空间
- `kube-public`：可以被所有人访问(包括未授权用户)，一般只能被系统使用，在这个命名空间的对象，整个集群都可访问，公开只是为了方便，不是强制要求
- `kube-node-lease`: 用于放置用于存放与每个节点关联的租约对象，在集群扩容时改善节点心跳性能

### 在请求中添加命名空间

为当前请求添加命名空间 使用 `--namespace` 参数
示例如下：
```sh
kubectl run nginx --image=nginx --namespace=<insert-namespace-name-here>
kubectl get pods --namespace=<insert-namespace-name-here>
```
### 设置默认命名空间

持久化配置 kubectl 默认操作的命名空间
```sh
kubectl config set-context --current --namespace=<insert-namespace-name-here>
# Validate it
kubectl config view --minify | grep namespace:
```

### 命名空间与DNS的关系

当用户创建 [Service](../../../services-networking/service/) 时会对应生成一条 [DNS 记录](../../../services-networking/03-dns-pod-service/), 而这条记录中的格式为 `<service-name>.<namespace-name>.svc.cluster.local` 也就是说 通过 `<service-name>` 只能解析到本命名空间的服务，这在不同命名空间使用同一套配置时相关有用，比如开发，演示，生产等不同环境。如果跨命名空间访问 Service 需要使用全限定名(FQDN)，一般来说只需要 `<service-name>.<namespace-name>` 也是可以的

## 那些不属于任何命名空间的对象

大多数 k8s 资源( pod, service, replication controller 等)都会属于某一个命名空间。 而 命名空间 资源则不属于任何命名空间。 还有一个底层资源 比如 [节点](../../../01-architecture/00-nodes/)， persistentVolumes 也不属于任何命名空间

以下命令可以查看资源是否属于命名空间:
```sh
# 在一个命名空间中
kubectl api-resources --namespaced=true

# 不属于任何命名空间
kubectl api-resources --namespaced=false

```
- Node
- persistentVolumes
- Events: 根据事件关联的对象，可能有属于命名空间也可能不属于命名空间
