---
title: Pod 开销
content_type: concept
weight: 50
---
<!--
---
reviewers:
- dchen1107
- egernst
- tallclair
title: Pod Overhead
content_type: concept
weight: 50
---
 -->
<!-- overview -->

{{< feature-state for_k8s_version="v1.18" state="beta" >}}

<!--
When you run a Pod on a Node, the Pod itself takes an amount of system resources. These
resources are additional to the resources needed to run the container(s) inside the Pod.
_Pod Overhead_ is a feature for accounting for the resources consumed by the Pod infrastructure
on top of the container requests & limits.
 -->

当在节点上运行一个 Pod 时， Pod 本身也会占用一定量的系统资源。这些资源与 Pod 中运行的容器所需要
的资源外额外的资源。 _Pod 开销_ 就是一个用来计量 Pod 容器保底要求和限制之外消耗的资源。


<!-- body -->
<!--
In Kubernetes, the Pod's overhead is set at
[admission](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#what-are-admission-webhooks)
time according to the overhead associated with the Pod's
[RuntimeClass](/docs/concepts/containers/runtime-class/).
 -->

在 k8s 中， Pod 的开销是在
[准入](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#what-are-admission-webhooks)
时根据与 Pod 关联的
[RuntimeClass](/k8sDocs/docs/conceptscontainers/runtime-class/)
上配置的开销来的。
<!--
When Pod Overhead is enabled, the overhead is considered in addition to the sum of container
resource requests when scheduling a Pod. Similarly, the kubelet will include the Pod overhead when sizing
the Pod cgroup, and when carrying out Pod eviction ranking.
 -->

在启用 Pod 开销后， 在 Pod 调度时就是考量这个开销加上 Pod 中所有容器的资源要求。 类似的，
kubelet 在在进行 Pod 驱逐排名时也会计算 Pod 的 cgroup 时也会包含 Pod 的开销。

<!--
## Enabling Pod Overhead {#set-up}

You need to make sure that the `PodOverhead`
[feature gate](/docs/reference/command-line-tools-reference/feature-gates/) is enabled (it is on by default as of 1.18)
across your cluster, and a `RuntimeClass` is utilized which defines the `overhead` field.
 -->

## 启用 Pod 开销 {#set-up}

需要确保集群中的 `PodOverhead`
[功能阀](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/) 是开启的
(v1.18 默认是开启的)
并使用了定义了 `overhead` 字段的 `RuntimeClass` 。

<!--
## Usage example

To use the PodOverhead feature, you need a RuntimeClass that defines the `overhead` field. As
an example, you could use the following RuntimeClass definition with a virtualizing container runtime
that uses around 120MiB per Pod for the virtual machine and the guest OS:

```yaml
---
kind: RuntimeClass
apiVersion: node.k8s.io/v1
metadata:
    name: kata-fc
handler: kata-fc
overhead:
    podFixed:
        memory: "120Mi"
        cpu: "250m"
```

Workloads which are created which specify the `kata-fc` RuntimeClass handler will take the memory and
cpu overheads into account for resource quota calculations, node scheduling, as well as Pod cgroup sizing.

Consider running the given example workload, test-pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  runtimeClassName: kata-fc
  containers:
  - name: busybox-ctr
    image: busybox
    stdin: true
    tty: true
    resources:
      limits:
        cpu: 500m
        memory: 100Mi
  - name: nginx-ctr
    image: nginx
    resources:
      limits:
        cpu: 1500m
        memory: 100Mi
```

At admission time the RuntimeClass [admission controller](/docs/reference/access-authn-authz/admission-controllers/)
updates the workload's PodSpec to include the `overhead` as described in the RuntimeClass. If the PodSpec already has this field defined,
the Pod will be rejected. In the given example, since only the RuntimeClass name is specified, the admission controller mutates the Pod
to include an `overhead`.

After the RuntimeClass admission controller, you can check the updated PodSpec:

```bash
kubectl get pod test-pod -o jsonpath='{.spec.overhead}'
```

The output is:
```
map[cpu:250m memory:120Mi]
```

If a ResourceQuota is defined, the sum of container requests as well as the
`overhead` field are counted.

When the kube-scheduler is deciding which node should run a new Pod, the scheduler considers that Pod's
`overhead` as well as the sum of container requests for that Pod. For this example, the scheduler adds the
requests and the overhead, then looks for a node that has 2.25 CPU and 320 MiB of memory available.

Once a Pod is scheduled to a node, the kubelet on that node creates a new {{< glossary_tooltip text="cgroup" term_id="cgroup" >}}
for the Pod. It is within this pod that the underlying container runtime will create containers.

If the resource has a limit defined for each container (Guaranteed QoS or Bustrable QoS with limits defined),
the kubelet will set an upper limit for the pod cgroup associated with that resource (cpu.cfs_quota_us for CPU
and memory.limit_in_bytes memory). This upper limit is based on the sum of the container limits plus the `overhead`
defined in the PodSpec.

For CPU, if the Pod is Guaranteed or Burstable QoS, the kubelet will set `cpu.shares` based on the sum of container
requests plus the `overhead` defined in the PodSpec.

Looking at our example, verify the container requests for the workload:
```bash
kubectl get pod test-pod -o jsonpath='{.spec.containers[*].resources.limits}'
```

The total container requests are 2000m CPU and 200MiB of memory:
```
map[cpu: 500m memory:100Mi] map[cpu:1500m memory:100Mi]
```

Check this against what is observed by the node:
```bash
kubectl describe node | grep test-pod -B2
```

The output shows 2250m CPU and 320MiB of memory are requested, which includes PodOverhead:
```
  Namespace                   Name                CPU Requests  CPU Limits   Memory Requests  Memory Limits  AGE
  ---------                   ----                ------------  ----------   ---------------  -------------  ---
  default                     test-pod            2250m (56%)   2250m (56%)  320Mi (1%)       320Mi (1%)     36m
```
 -->

## 使用示例 {#usage-example}
<!--
To use the PodOverhead feature, you need a RuntimeClass that defines the `overhead` field. As
an example, you could use the following RuntimeClass definition with a virtualizing container runtime
that uses around 120MiB per Pod for the virtual machine and the guest OS:
 -->

要使用 `PodOverhead` 特性， 需要要有一个定义了 `overhead` 字段的 `RuntimeClass`。 下面
这个示例，可以使用下面的这个 `RuntimeClass` 定义了一个包含虚拟容器运行时，其中每个 Pod 使用
大约 120MiB 来运行虚拟机和客户机操作系统:

```yaml
---
kind: RuntimeClass
apiVersion: node.k8s.io/v1
metadata:
    name: kata-fc
handler: kata-fc
overhead:
    podFixed:
        memory: "120Mi"
        cpu: "250m"
```

那些在配置中指定 `kata-fc` 作为 `RuntimeClass` 处理器的工作负载，在进行资源配额諸，节点调度，
还有 Pod cgroup 计算是都会计算这部分内存和CPU开销。

运行下面示例的工作负载, test-pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  runtimeClassName: kata-fc
  containers:
  - name: busybox-ctr
    image: busybox
    stdin: true
    tty: true
    resources:
      limits:
        cpu: 500m
        memory: 100Mi
  - name: nginx-ctr
    image: nginx
    resources:
      limits:
        cpu: 1500m
        memory: 100Mi
```

在准入时 `RuntimeClass`
[准入控制器](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)
会更新工作负载的 `PodSpec` 把 `RuntimeClass` 中定义的 `overhead` 加进去， 如果 `PodSpec`
中已经定义了这个字段，Pod 就会被拒绝。 在上面的这个例子中， 因为只指定了 `RuntimeClass` 名称，
准入控制器会修改 Pod 配置将 `overhead` 加进去。

After the RuntimeClass admission controller, you can check the updated PodSpec:

在 `RuntimeClass` 准入控制器之后， 可以查看被更新后的 Pod 定义

```bash
kubectl get pod test-pod -o jsonpath='{.spec.overhead}'
```

输出结果如下:
```
map[cpu:250m memory:120Mi]
```

如果定义了一个 `ResourceQuota`， 也会同时计算容器资源和 `overhead` 字段。

当 `kube-scheduler` 在决定由哪个节点来运行一个新的 Pod 时， 调度器也会同时计算这个 Pod 中
容器资源和 `overhead` 字段。 就上面的这个例子，调度器会将资源要求和 Pod 开销加起来，寻找一个
有 2.25 CPU 和 320 MiB 内存可用的节点。

当一个 Pod 被调度到一个节点后， 这个节点上的 kubelet 会为这个 Pod 创建一个新的
{{< glossary_tooltip text="cgroup" term_id="cgroup" >}}
，底层的容器运行时再在其中创建这个 Pod 的容器。

如果还为每个容器定义了资源限制(Guaranteed QoS or Bustrable QoS 中定义的限制)，kubelet 会为 Pod cgroup
对应的资源(`cpu.cfs_quota_us` 是 CPU 然后 `memory.limit_in_bytes` 是内存)设置一个上限。
这个上限是基于容器限制的上限加上 `PodSpec` 中定义的开销(`overhead`)

对于 CPU， 如果 Pod 是 Guaranteed 或 Burstable QoS, kubelet 会基于容器请求加上
`PodSpec` 中定义的开销(`overhead`) 来设置 `cpu.shares`

再看我们的示例， 检查工作负载的容器请求:
```bash
kubectl get pod test-pod -o jsonpath='{.spec.containers[*].resources.limits}'
```

容器请求的总和为 2000m CPU 和 200MiB 内存:
```
map[cpu: 500m memory:100Mi] map[cpu:1500m memory:100Mi]
```

再通过观察节点来验证这一点:
```bash
kubectl describe node | grep test-pod -B2
```

输出显示要求 2250m CPU 和 320MiB 内存，其中包含了 Pod 开销(`PodOverhead`):
```
  Namespace                   Name                CPU Requests  CPU Limits   Memory Requests  Memory Limits  AGE
  ---------                   ----                ------------  ----------   ---------------  -------------  ---
  default                     test-pod            2250m (56%)   2250m (56%)  320Mi (1%)       320Mi (1%)     36m
```

<!--
## Verify Pod cgroup limits

Check the Pod's memory cgroups on the node where the workload is running. In the following example, [`crictl`](https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md)
is used on the node, which provides a CLI for CRI-compatible container runtimes. This is an
advanced example to show PodOverhead behavior, and it is not expected that users should need to check
cgroups directly on the node.

First, on the particular node, determine the Pod identifier:

```bash
# Run this on the node where the Pod is scheduled
POD_ID="$(sudo crictl pods --name test-pod -q)"
```

From this, you can determine the cgroup path for the Pod:
```bash
# Run this on the node where the Pod is scheduled
sudo crictl inspectp -o=json $POD_ID | grep cgroupsPath
```

The resulting cgroup path includes the Pod's `pause` container. The Pod level cgroup is one directory above.
```
        "cgroupsPath": "/kubepods/podd7f4b509-cf94-4951-9417-d1087c92a5b2/7ccf55aee35dd16aca4189c952d83487297f3cd760f1bbf09620e206e7d0c27a"
```

In this specific case, the pod cgroup path is `kubepods/podd7f4b509-cf94-4951-9417-d1087c92a5b2`. Verify the Pod level cgroup setting for memory:
```bash
# Run this on the node where the Pod is scheduled.
# Also, change the name of the cgroup to match the cgroup allocated for your pod.
 cat /sys/fs/cgroup/memory/kubepods/podd7f4b509-cf94-4951-9417-d1087c92a5b2/memory.limit_in_bytes
```

This is 320 MiB, as expected:
```
335544320
```
 -->

## 验证 Pod cgroup 限制 {#verify-pod-cgroup-limits}


检查工作负载运行节点上 Pod 的内存 cgroup。 在下面的例子中，在节点中使用了
[`crictl`](https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md)
为 CRI 兼容的容器运行时提供 CLI。 这是一个显示 `PodOverhead` 行为的高级示例， 没有预期用户
有必须直接在节点上检查 cgroup

首先，在特定的节点上，定位 Pod 的标识符:

```bash
# 在 Pod 调度的目标节点上运行
POD_ID="$(sudo crictl pods --name test-pod -q)"
```

从这里，可以确定 Pod 的 cgroup 路径:
```bash
# 在 Pod 调度的目标节点上运行
sudo crictl inspectp -o=json $POD_ID | grep cgroupsPath
```

结果中的 cgroup 路径包含了 Pod 的 `pause` 容器。 Pod 级别的 cgroup 在这个路径向上一层
```
        "cgroupsPath": "/kubepods/podd7f4b509-cf94-4951-9417-d1087c92a5b2/7ccf55aee35dd16aca4189c952d83487297f3cd760f1bbf09620e206e7d0c27a"
```

In this specific case, the pod cgroup path is `kubepods/podd7f4b509-cf94-4951-9417-d1087c92a5b2`. Verify the Pod level cgroup setting for memory:
在这个例子中， Pod cgroup 路径就是 `kubepods/podd7f4b509-cf94-4951-9417-d1087c92a5b2`。
验证 Pod 级别 cgroup 配置的中的内存配置:
```bash
# 在 Pod 调度的目标节点上运行
# 同时，修改 cgroup 的名称，让它与为 Pod 分配的 cgroup 相同
 cat /sys/fs/cgroup/memory/kubepods/podd7f4b509-cf94-4951-9417-d1087c92a5b2/memory.limit_in_bytes
```

这个例子是 320 MiB, 所以就是:
```
335544320
```
<!--
### Observability

A `kube_pod_overhead` metric is available in [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics)
to help identify when PodOverhead is being utilized and to help observe stability of workloads
running with a defined Overhead. This functionality is not available in the 1.9 release of
kube-state-metrics, but is expected in a following release. Users will need to build kube-state-metrics
from source in the meantime.
 -->

### 可观察性 {#observability}

在
[kube-state-metrics](https://github.com/kubernetes/kube-state-metrics)
中有一个 `kube_pod_overhead` 指标，可以用来在使用 PodOverhead 帮助观测使用了 Pod 开销的
工作负载的稳定性。 这个功能在 v1.9 的 `kube-state-metrics` 中是没有的，但应该会在接下来的
版本中提供。同时需要用户从源码构建 `kube-state-metrics`


## {{% heading "whatsnext" %}}

<!--
* [RuntimeClass](/docs/concepts/containers/runtime-class/)
* [PodOverhead Design](https://github.com/kubernetes/enhancements/tree/master/keps/sig-node/688-pod-overhead)
 -->
* [RuntimeClass](/k8sDocs/docs/conceptscontainers/runtime-class/)
* [PodOverhead 设计文稿](https://github.com/kubernetes/enhancements/tree/master/keps/sig-node/688-pod-overhead)
