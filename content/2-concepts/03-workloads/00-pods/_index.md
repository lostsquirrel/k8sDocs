---
title: Pods
weight: 20300
date: 2020-07-02
publishdate: 2020-07-30
---
<!-- overview -->

<!--
_Pods_ are the smallest deployable units of computing that you can create and manage in Kubernetes.

A _Pod_ (as in a pod of whales or pea pod) is a group of one or more
{{< glossary_tooltip text="containers" term_id="container" >}}, with shared storage/network resources, and a specification
for how to run the containers. A Pod's contents are always co-located and
co-scheduled, and run in a shared context. A Pod models an
application-specific "logical host": it contains one or more application
containers which are relatively tightly coupled.
In non-cloud contexts, applications executed on the same physical or virtual machine are analogous to cloud applications executed on the same logical host.

As well as application containers, a Pod can contain
[init containers](/docs/concepts/workloads/pods/init-containers/) that run
during Pod startup. You can also inject
[ephemeral containers](/docs/concepts/workloads/pods/ephemeral-containers/)
for debugging if your cluster offers this.
-->
_Pods_ 是在 k8s 集群中可创建和管理的最小可部署计算单元。

一个 _Pods_ (可以类比一个豌豆荚或一个鲸落)就是由一组共享存储/网络，并带有运行指示的一个或多个{{< glossary_tooltip text="容器" term_id="container" >}}。 同一个 Pod 中的{{< glossary_tooltip text="容器" term_id="container" >}}共享同一个上下文并始终统一调度并最终落地在一个{{< glossary_tooltip text="节点"  term_id="node" >}}. 一个 Pod 的模型为应用专用的逻辑主机： 包含一个或多个紧密关联的应用容器。 应用运行在非云环境情景的同一个物理或虚拟机上，与云应用运行在同一个逻辑主机上。

Pod 在包含应用容器的同时也可以加入[初始化容器](../00-init-containers/),用于在 Pod 启动时执行初始化工作，和 [临时容器](../05-ephemeral-containers/)用于调试。

<!-- body -->

## Pod 是个啥东西?
<!--
{{< note >}}
While Kubernetes supports more
{{< glossary_tooltip text="container runtimes" term_id="container-runtime" >}}
than just Docker, [Docker](https://www.docker.com/) is the most commonly known
runtime, and it helps to describe Pods using some terminology from Docker.
{{< /note >}}

The shared context of a Pod is a set of Linux namespaces, cgroups, and
potentially other facets of isolation - the same things that isolate a Docker
container.  Within a Pod's context, the individual applications may have
further sub-isolations applied.

In terms of Docker concepts, a Pod is similar to a group of Docker containers
with shared namespaces and shared filesystem volumes.
-->
{{< note >}}
k8s 支持的 {{< glossary_tooltip term_id="container-runtime" >}}不止有 Docker, [Docker](https://www.docker.com/) 只是大家比较熟悉的容器运行环境，但是也可能通过 Docker 的一些术语来帮助解说 Pod
{{< /note >}}

Pod 中共享的上下文包括 Linux namespaces, cgroups, 和其它方面的隔离， 与 Docker 容器的隔离范围相同。 在一个 Pod 的上下文中，各个不同的应用可能还有更细分的隔离设置。

以 Docker 的概念上来说， 一个 Pod 与 一组共享 namespaces 和文件系统卷的容器相似。

## Pod 怎么用

<!--
Usually you don't need to create Pods directly, even singleton Pods. Instead, create them using workload resources such as {{< glossary_tooltip text="Deployment"
term_id="deployment" >}} or {{< glossary_tooltip text="Job" term_id="job" >}}.
If your Pods need to track state, consider the
{{< glossary_tooltip text="StatefulSet" term_id="statefulset" >}} resource.
-->
通常用户不需要直接创建 Pod， 即使是单例 Pod(?啥玩意儿). 而是通过创建类似 {{< glossary_tooltip text="Deployment"
term_id="deployment" >}} 或 {{< glossary_tooltip text="Job" term_id="job" >}} 的工作负载资源间接创建 Pod。如果 Pod 需要跟踪状态(目前理解是包含持久化数据)，可以考虑使用 {{< glossary_tooltip text="StatefulSet" term_id="statefulset" >}} 资源。

<!--
Pods in a Kubernetes cluster are used in two main ways:

* **Pods that run a single container**. The "one-container-per-Pod" model is the
  most common Kubernetes use case; in this case, you can think of a Pod as a
  wrapper around a single container; Kubernetes manages Pods rather than managing
  the containers directly.
* **Pods that run multiple containers that need to work together**. A Pod can
  encapsulate an application composed of multiple co-located containers that are
  tightly coupled and need to share resources. These co-located containers
  form a single cohesive unit of service—for example, one container serving data
  stored in a shared volume to the public, while a separate _sidecar_ container
  refreshes or updates those files.  
  The Pod wraps these containers, storage resources, and an ephemeral network
  identity together as a single unit.

  {{< note >}}
  Grouping multiple co-located and co-managed containers in a single Pod is a
  relatively advanced use case. You should use this pattern only in specific
  instances in which your containers are tightly coupled.
  {{< /note >}}
-->
在 k8s 集群中 Pod 主要有以下两种使用方式:

- **在 Pod 中运行一个容器**。 一个 Pod 装一个容器的的模式是 k8s 最常见的应用方式，就可以认为Pod就是给容器包了一个壳, 因为 k8s 管理的是 Pod 而不直接管理容器

- **在 Pod 中运行多个紧密相关的容器**。 一个 Pod 可以封装一个由多个紧密耦合在一起，需要共享资源的多个容器组成的应用。这些容器最终组合成一个服务， 例如一个容器对外提供存储在共享数据卷中的数据， 另一个 _边车_ 容器负责刷新或更新这些文件。
Pod 把这些容器，存储资源和一个临时的网络地址打包成一个整体。

  {{< note >}}
  在一个Pod 中包含多个相互关联共同管理的容器是一种相对高级的应用场景。用户应该在应用中的容器之间紧密耦合的时候使用这种方式。
  {{< /note >}}

<!--
Each Pod is meant to run a single instance of a given application. If you want to
scale your application horizontally (to provide more overall resources by running
more instances), you should use multiple Pods, one for each instance. In
Kubernetes, this is typically referred to as _replication_.
Replicated Pods are usually created and managed as a group by a workload resource
and its {{< glossary_tooltip text="controller" term_id="controller" >}}.

See [Pods and controllers](#pods-and-controllers) for more information on how
Kubernetes uses workload resources, and their controllers, to implement application
scaling and auto-healing.
-->
每一个Pod 表示运行该应用的一个实例。 如果想要水平扩展应用(通过运行更多的实例来提供更多的资源)，则需要运行多个 Pod， 一个Pod对应一个实例。在 k8s 中，这通常被称为 _副本(replication)_
这些副本 Pod 通常由一个工作负载资源也就是 {{< glossary_tooltip text="controller" term_id="controller" >}} 以组的形式创建和管理。

下面的 [Pods 与 controllers](#pods-and-controllers) 有更多关于 k8s 是怎么通过工作负载资源和相应的控制器实现应用的扩容和自我恢复的
### Pod 怎么管理多个容器

<!--
Pods are designed to support multiple cooperating processes (as containers) that form
a cohesive unit of service. The containers in a Pod are automatically co-located and
co-scheduled on the same physical or virtual machine in the cluster. The containers
can share resources and dependencies, communicate with one another, and coordinate
when and how they are terminated.

For example, you might have a container that
acts as a web server for files in a shared volume, and a separate "sidecar" container
that updates those files from a remote source, as in the following diagram:

{{< figure src="/images/docs/pod.svg" alt="example pod diagram" width="50%" >}}

Some Pods have {{< glossary_tooltip text="init containers" term_id="init-container" >}} as well as {{< glossary_tooltip text="app containers" term_id="app-container" >}}. Init containers run and complete before the app containers are started.

Pods natively provide two kinds of shared resources for their constituent containers:
[networking](#pod-networking) and [storage](#pod-storage).
-->
Pod 在设计上就支持多个协作进程(容器进程)组成一个服务。Pod 中的所有容器都会自动的同时调度到集群中的同一个物理机或虚拟机上。 这些容器可以共享资源和信赖，相互通信，并能协调什么时候以什么样的方式来终止容器。

例如，用户可能有一个容器作为 web 服务，为共享数据卷的文件提供访问服务，另一个独立的容器作为 边车，负责从远程的源更新这些文件， 如下图所示:

{{< figure src="https://d33wubrfki0l68.cloudfront.net/aecab1f649bc640ebef1f05581bfcc91a48038c4/728d6/images/docs/pod.svg" alt="example pod diagram" width="50%" >}}

有些 Pod 中会包含 {{< glossary_tooltip text="init containers" term_id="init-container" >}}， {{< glossary_tooltip text="app containers" term_id="app-container" >}}。 初始化化容器在应用容器开启之前运行并完成。

在同一个 Pod 中的容器天然共享两种资源: [网络](#pod-networking) 和 [存储](#pod-storage).

## Pod 操作

<!--
You'll rarely create individual Pods directly in Kubernetes—even singleton Pods. This
is because Pods are designed as relatively ephemeral, disposable entities. When
a Pod gets created (directly by you, or indirectly by a
{{< glossary_tooltip text="controller" term_id="controller" >}}), the new Pod is
scheduled to run on a {{< glossary_tooltip term_id="node" >}} in your cluster.
The Pod remains on that node until the Pod finishes execution, the Pod object is deleted,
the Pod is *evicted* for lack of resources, or the node fails.

{{< note >}}
Restarting a container in a Pod should not be confused with restarting a Pod. A Pod
is not a process, but an environment for running container(s). A Pod persists until
it is deleted.
{{< /note >}}

When you create the manifest for a Pod object, make sure the name specified is a valid
[DNS subdomain name](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).
-->
用户一般很少在 k8s 中直接创建独立的 Pod，即便是 singleton Pod。 这是因为在设计上，Pod 相对来是的临时的， 一次性的实体。
当一个 Pod 被创建之后(无论是用户直接创建，还是能过 {{< glossary_tooltip text="controller" term_id="controller" >}} 间接创建)，这个新建的 Pod 都会调度并运行在集群中的一个{{< glossary_tooltip term_id="node" >}}上。
这个 Pod 会存在于这个节点上直接执行完成，然后这个 Pod 对象就会被删除掉，Pod 也可能因为资源不足或节点挂掉而被 *驱逐(evicted)*

{{< note >}}
重启 Pod 中的容器和重启一个 Pod 可能会搞混。 一个 Pod 并不是一个进程，而是运行容器的一个环境。一个 Pod 在被删除前都是一直存在
{{< /note >}}

在创建 Pod 对象的定义文件时， 名称必须是一个有效的 [DNS 子域名](../../../00-overview/03-working-with-objects/01-names/#dns-subdomain-names)
### {{< glossary_tooltip term_id="pod" >}} 和 {{< glossary_tooltip term_id="controller" >}}
<!--
You can use workload resources to create and manage multiple Pods for you. A controller
for the resource handles replication and rollout and automatic healing in case of
Pod failure. For example, if a Node fails, a controller notices that Pods on that
Node have stopped working and creates a replacement Pod. The scheduler places the
replacement Pod onto a healthy Node.

Here are some examples of workload resources that manage one or more Pods:

* {{< glossary_tooltip text="Deployment" term_id="deployment" >}}
* {{< glossary_tooltip text="StatefulSet" term_id="statefulset" >}}
* {{< glossary_tooltip text="DaemonSet" term_id="daemonset" >}}
-->
用户可以使用工作负载资源来创建和管理多个 Pod。 资源的控制器处理 副本， 回滚，在 Pod 持掉时自动恢复。
例如， 如果一个节点挂了，一个控制器注意到这个节点上的 Pod 都不工作了，则会创建一个替代的 Pod。
然后调度器将这些替代 Pod 放到健康的节点上。
以下为常见的可以管理一个或多个 Pod 的工作负载示例:

* {{< glossary_tooltip text="Deployment" term_id="deployment" >}}
* {{< glossary_tooltip text="StatefulSet" term_id="statefulset" >}}
* {{< glossary_tooltip text="DaemonSet" term_id="daemonset" >}}
### Pod 模板

<!--
Controllers for {{< glossary_tooltip text="workload" term_id="workload" >}} resources create Pods
from a _pod template_ and manage those Pods on your behalf.

PodTemplates are specifications for creating Pods, and are included in workload resources such as
[Deployments](/docs/concepts/workloads/controllers/deployment/),
[Jobs](/docs/concepts/jobs/run-to-completion-finite-workloads/), and
[DaemonSets](/docs/concepts/workloads/controllers/daemonset/).

Each controller for a workload resource uses the `PodTemplate` inside the workload
object to make actual Pods. The `PodTemplate` is part of the desired state of whatever
workload resource you used to run your app.
 -->
{{< glossary_tooltip term_id="workload" >}} 资源对应的控制器通过用户提供的 _Pod 模板_ 来创建和管理这个 Pod。
诸如
[Deployments](/docs/concepts/workloads/controllers/deployment/),
[Jobs](/docs/concepts/jobs/run-to-completion-finite-workloads/),
[DaemonSets](/docs/concepts/workloads/controllers/daemonset/)的{{< glossary_tooltip term_id="workload" >}} 资源都会包含创建 Pod 的定义字段 `PodTemplates`。
一个{{< glossary_tooltip term_id="workload" >}} 资源对应的控制通过工作负载对象中的 `PodTemplate` 来创建实际的 Pod。 `PodTemplate` 是用户用于运行应用的工作负载的期望状态的一部分。

<!--
The sample below is a manifest for a simple Job with a `template` that starts one
container. The container in that Pod prints a message then pauses.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  template:
    # This is the pod template
    spec:
      containers:
      - name: hello
        image: busybox
        command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
      restartPolicy: OnFailure
    # The pod template ends here
```
 -->
以下为一个简单 {{< glossary_tooltip term_id="job" >}} 的定义文件，其中 `template` 定义会启动一个容器。
Pod 中的这个容器会打包一条信息，然后睡一个小时(3600秒)

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  template:
    # Pod 模板从这里开始
    spec:
      containers:
      - name: hello
        image: busybox
        command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
      restartPolicy: OnFailure
    # Pod 模板到这里结束
```

<!--
Modifying the pod template or switching to a new pod template has no effect on the
Pods that already exist. Pods do not receive template updates directly. Instead,
a new Pod is created to match the revised pod template.

For example, the deployment controller ensures that the running Pods match the current
pod template for each Deployment object. If the template is updated, the Deployment has
to remove the existing Pods and create new Pods based on the updated template. Each workload
resource implements its own rules for handling changes to the Pod template.

On Nodes, the {{< glossary_tooltip term_id="kubelet" text="kubelet" >}} does not
directly observe or manage any of the details around pod templates and updates; those
details are abstracted away. That abstraction and separation of concerns simplifies
system semantics, and makes it feasible to extend the cluster's behavior without
changing existing code.

 -->
修改 Pod 模板甚至替换为新的模板都不会对已经存在的 Pod 有任何影响。 Pod 不接收直接的模板更新，而是基于新的模板创建一个新的 Pod。
例如， {{< glossary_tooltip term_id="deployment" >}} 控制器会保证每个 {{< glossary_tooltip term_id="deployment" >}} 对象对应的 Pod 与其械反定义是一致的.
如果 {{< glossary_tooltip term_id="deployment" >}} Pod 模板有更新，就对移除旧模板对应的 Pod，并以新模板创建新的 Pod。 每个工作负载资源都实现了一个自己的模板更新处理方式。

在节点上， {{< glossary_tooltip term_id="kubelet" text="kubelet" >}} 并不会直接监测或管理 Pod 模板的详情或更新；这些详情都已经被抽象剥离。
通过抽象和分离系统关注点，使得可以在不动已存在代码的情况下轻松实现对集群行为的扩展。
{{< todo-optimize >}}

## 资源共享和网络通信
<!--
## Resources sharing and communications
Pods enable data sharing and communication among their constituent
containters.
-->
Pod 为内部容器提供了数据和网络的共享环境
<!--
### Storage in Pods {#pod-storage}

A Pod can specify a set of shared storage
{{< glossary_tooltip text="volumes" term_id="volume" >}}. All containers
in the Pod can access the shared volumes, allowing those containers to
share data. Volumes also allow persistent data in a Pod to survive
in case one of the containers within needs to be restarted. See
[Storage](/docs/concepts/storage/) for more information on how
Kubernetes implements shared storage and makes it available to Pods.

 -->
### Pod 的存储 {#pod-storage}

一个 Pod 可以指定一组共享的存储 {{< glossary_tooltip term_id="volume" >}}。
这个 Pod 内的所有容器都可以使用这个共享卷， 实现容器间的数据共享。卷也可以让其中的数据在， Pod
被销毁重建后依然存在。更多关于 k8s 怎么实现共享存储及其在 Pod 中的使用，见 [存储](../../05-storage/)

<!--
### Pod networking

Each Pod is assigned a unique IP address for each address family. Every
container in a Pod shares the network namespace, including the IP address and
network ports. Inside a Pod (and **only** then), the containers that belong to the Pod
can communicate with one another using `localhost`. When containers in a Pod communicate
with entities *outside the Pod*,
they must coordinate how they use the shared network resources (such as ports).
Within a Pod, containers share an IP address and port space, and
can find each other via `localhost`. The containers in a Pod can also communicate
with each other using standard inter-process communications like SystemV semaphores
or POSIX shared memory.  Containers in different Pods have distinct IP addresses
and can not communicate by IPC without
[special configuration](/docs/concepts/policy/pod-security-policy/).
Containers that want to interact with a container running in a different Pod can
use IP networking to comunicate.

Containers within the Pod see the system hostname as being the same as the configured
`name` for the Pod. There's more about this in the [networking](/docs/concepts/cluster-administration/networking/)
section.


 -->
### Pod 的网络

每个 Pod 在一个地址族内分配唯一IP地址，在Pod内的所有容器共享网络空间，包括IP地址和网络端口，
在一个Pod中(且仅在此时)， 属于该 Pod的容器相互之间可以通过 `localhost`通信。 当 Pod 中的容器
与 *Pod 外* 的实体通信时需要协调好，它们(Pod 中的容器)之间怎么使用共享的网络资源(如端口)。
在同一个Pod中的容器共享IP地址和端口池，并且相互之间可以通过 `localhost` 找到彼此。
在同一个Pod中的容器还可以使用如 SystemV 信号量 或 POSIX 共享内存之类的标准进程间通信。
不同Pod中的容器拥有不同的IP地址，并且在不进行[特殊配置](../../08-policy/02-pod-security-policy/)的情况下不能通过进程间通信(IPC)

Pod 中的容器看到的系统主机名就是定义文件中这个 Pod 的 `name` 字段的值。 更多相关信息请见 [网络](../../10-cluster-administration/03-networking/)

<!--
## Privileged mode for containers

Any container in a Pod can enable privileged mode, using the `privileged` flag on the [security context](/docs/tasks/configure-pod-container/security-context/) of the container spec. This is useful for containers that want to use operating system administrative capabilities such as manipulating the network stack or accessing hardware devices.
Processes within a privileged container get almost the same privileges that are available to processes outside a container.

{{< note >}}
Your {{< glossary_tooltip text="container runtime" term_id="container-runtime" >}} must support the concept of a privileged container for this setting to be relevant.
{{< /note >}}

 -->
## 容器的提权模式

Pod 中的任意容器都可以开启提权模式，通过容器定义中的 [安全上下文](../../../3-tasks/02-configure-pod-container/09-security-context/) 中有一个 `privileged` 标志 开启或关闭(默认是关闭的). 这个模式对需要使用操作系统管理员权限如操作网络栈或访问硬件设备相当有用。
在提权模式开启的容器中的进行基本与容器外的的进程拥有相同的权限。

{{< note >}}
提权模式生效需要 {{< glossary_tooltip term_id="container-runtime" >}} 支持提权容器这个概念。
{{< /note >}}
<!--
## Static Pods

_Static Pods_ are managed directly by the kubelet daemon on a specific node,
without the {{< glossary_tooltip text="API server" term_id="kube-apiserver" >}}
observing them.
Whereas most Pods are managed by the control plane (for example, a
{{< glossary_tooltip text="Deployment" term_id="deployment" >}}), for static
Pods, the kubelet directly supervises each static Pod (and restarts it if it fails).

Static Pods are always bound to one {{< glossary_tooltip term_id="kubelet" >}} on a specific node.
The main use for static Pods is to run a self-hosted control plane: in other words,
using the kubelet to supervise the individual [control plane components](/docs/concepts/overview/components/#control-plane-components).

The kubelet automatically tries to create a {{< glossary_tooltip text="mirror Pod" term_id="mirror-pod" >}}
on the Kubernetes API server for each static Pod.
This means that the Pods running on a node are visible on the API server,
but cannot be controlled from there.

 -->

## 静态 Pod

_静态 Pod_ 是由节点上的 kubelet 直接管理的 Pod， 并不被 {{< glossary_tooltip term_id="kube-apiserver" >}}
监控。 然而多数 Pod 都是受控制中心管理(例，{{< glossary_tooltip text="Deployment" term_id="deployment" >}})，
静态 Pod 则是直接由 kubelet 监控和管理(如果挂也是由 kubelet 来重启)。
静态 Pod 总是与节点上的 {{< glossary_tooltip term_id="kubelet" >}} 绑定在一起的。
静态 Pod 主要用于运行自建(相对于云提供商提供)控制中心，也就是用 kubelet 来监管各个[控制中心组件](../../00-overview/01-components/#control-plane-components)

kubelet 会自动的尝试在 {{< glossary_tooltip term_id="kube-apiserver" >}} 上为每个静态 Pod 去创建 {{< glossary_tooltip term_id="mirror-pod" >}}。
也就是说这些运行在节点上的Pod在 {{< glossary_tooltip term_id="kube-apiserver" >}} 上也是可以看到的，但不能通过 {{< glossary_tooltip term_id="kube-apiserver" >}} 对其进行修改。

## {{% heading "whatsnext" %}}
<!--

* Learn about the [lifecycle of a Pod](/docs/concepts/workloads/pods/pod-lifecycle/).
* Learn about [PodPresets](/docs/concepts/workloads/pods/podpreset/).
* Lean about [RuntimeClass](/docs/concepts/containers/runtime-class/) and how you can use it to
  configure different Pods with different container runtime configurations.
* Read about [Pod topology spread constraints](/docs/concepts/workloads/pods/pod-topology-spread-constraints/).
* Read about [PodDisruptionBudget](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) and how you can use it to manage application availability during disruptions.
* Pod is a top-level resource in the Kubernetes REST API.  
  The [Pod](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#pod-v1-core)
  object definition describes the object in detail.
* [The Distributed System Toolkit: Patterns for Composite Containers](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns) explains common layouts for Pods with more than one container.

To understand the context for why Kubernetes wraps a common Pod API in other resources (such as {{< glossary_tooltip text="StatefulSets" term_id="statefulset" >}} or {{< glossary_tooltip text="Deployments" term_id="deployment" >}}, you can read about the prior art, including:
  * [Aurora](http://aurora.apache.org/documentation/latest/reference/configuration/#job-schema)
  * [Borg](https://research.google.com/pubs/pub43438.html)
  * [Marathon](https://mesosphere.github.io/marathon/docs/rest-api.html)
  * [Omega](https://research.google/pubs/pub41684/)
  * [Tupperware](https://engineering.fb.com/data-center-engineering/tupperware/).
 -->
* 了解 [Pod生命周期](../00-pod-lifecycle/).
* 了解 [PodPresets](../03-podpreset/).
* 了解 [RuntimeClass](../../02-containers/02runtime-class/) 并如何使用它来在不同的运行时上运行不同的 Pod
* 了解 [Pod topology spread constraints](../02-pod-topology-spread-constraints/).
* 了解 [PodDisruptionBudget](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) 并用它来在干扰时管理应用的可用性
* Pod k8s REST API 中顶级资源.  
  这里是 [Pod](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#pod-v1-core)
  对象文档
* [The Distributed System Toolkit: Patterns for Composite Containers](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns) 介绍多容器 Pod 的常用布局

要理解为啥k8s 要把 Pod 包在其它(如 {{< glossary_tooltip text="StatefulSets" term_id="statefulset" >}} 或 {{< glossary_tooltip text="Deployments" term_id="deployment" >}})的资源中，请阅读以下先前技术(prior art)：
  * [Aurora](http://aurora.apache.org/documentation/latest/reference/configuration/#job-schema)
  * [Borg](https://research.google.com/pubs/pub43438.html)
  * [Marathon](https://mesosphere.github.io/marathon/docs/rest-api.html)
  * [Omega](https://research.google/pubs/pub41684/)
  * [Tupperware](https://engineering.fb.com/data-center-engineering/tupperware/).
