---
title: ReplicationController
weight: 2030108
date: 2020-07-02
publishdate: 2020-09-02
---
<!--
---
reviewers:
- bprashanth
- janetkuo
title: ReplicationController
feature:
  title: Self-healing
  anchor: How a ReplicationController Works
  description: >
    Restarts containers that fail, replaces and reschedules containers when nodes die, kills containers that don't respond to your user-defined health check, and doesn't advertise them to clients until they are ready to serve.

content_type: concept
weight: 90
---
 -->
<!-- overview -->
<!--  
{{< note >}}
A [`Deployment`](/docs/concepts/workloads/controllers/deployment/) that configures a [`ReplicaSet`](/docs/concepts/workloads/controllers/replicaset/) is now the recommended way to set up replication.
{{< /note >}}

A _ReplicationController_ ensures that a specified number of pod replicas are running at any one
time. In other words, a ReplicationController makes sure that a pod or a homogeneous set of pods is
always up and available.
-->
{{< note >}}
使用 [`Deployment`](/k8sDocs/docs/concepts/workloads/controllers/deployment/) 管理
[`ReplicaSet`](/k8sDocs/docs/concepts/workloads/controllers/replicaset/)
是目前推荐的运行多副本的方式。
{{< /note >}}
``
_ReplicationController_ 确保在任意时刻都有指定数量的 Pod 副本在运行， 换句话来说
就是 `ReplicationController` 保证一个 Pod 或 一组同质 Pod 的集群始终运行并可用
<!-- body -->
<!--
## How a ReplicationController Works

If there are too many pods, the ReplicationController terminates the extra pods. If there are too few, the
ReplicationController starts more pods. Unlike manually created pods, the pods maintained by a
ReplicationController are automatically replaced if they fail, are deleted, or are terminated.
For example, your pods are re-created on a node after disruptive maintenance such as a kernel upgrade.
For this reason, you should use a ReplicationController even if your application requires
only a single pod. A ReplicationController is similar to a process supervisor,
but instead of supervising individual processes on a single node, the ReplicationController supervises multiple pods
across multiple nodes.

ReplicationController is often abbreviated to "rc" in discussion, and as a shortcut in
kubectl commands.

A simple case is to create one ReplicationController object to reliably run one instance of
a Pod indefinitely.  A more complex use case is to run several identical replicas of a replicated
service, such as web servers.
 -->
## ReplicationController 是怎么工作的

如果同时运行的 Pod 太多了， ReplicationController 终止多余的 Pod。
如果同时运行的 Pod 太少了， ReplicationController 启动更多的 Pod。
与手动创建 Pod 不同， 由 ReplicationController 维护的 Pod 在挂掉，被删除，或被终止都会自动被替换。
例如， Pod 因为所在的节点升级内核引起维护故障而被重新创建。因为这些原因，即便应用只需要一单个 Pod 也应该使用 ReplicationController。
ReplicationController 与进程监督类似，但是与只监督一个节点上的单个线程不同，
ReplicationController 监督多个节点上的多个 Pod。

简单的一个场景为创建一个 ReplicationController 对象来运行一个不限期运行的单实例 Pod。
复杂的一个应用场景为运行一个多副本应用的多个副本，如 web 服务。
<!--
## Running an example ReplicationController

This example ReplicationController config runs three copies of the nginx web server.

{{< codenew file="controllers/replication.yaml" >}}

Run the example job by downloading the example file and then running this command:

```shell
kubectl apply -f https://k8s.io/examples/controllers/replication.yaml
```
```
replicationcontroller/nginx created
```

Check on the status of the ReplicationController using this command:

```shell
kubectl describe replicationcontrollers/nginx
```
```
Name:        nginx
Namespace:   default
Selector:    app=nginx
Labels:      app=nginx
Annotations:    <none>
Replicas:    3 current / 3 desired
Pods Status: 0 Running / 3 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:       app=nginx
  Containers:
   nginx:
    Image:              nginx
    Port:               80/TCP
    Environment:        <none>
    Mounts:             <none>
  Volumes:              <none>
Events:
  FirstSeen       LastSeen     Count    From                        SubobjectPath    Type      Reason              Message
  ---------       --------     -----    ----                        -------------    ----      ------              -------
  20s             20s          1        {replication-controller }                    Normal    SuccessfulCreate    Created pod: nginx-qrm3m
  20s             20s          1        {replication-controller }                    Normal    SuccessfulCreate    Created pod: nginx-3ntk0
  20s             20s          1        {replication-controller }                    Normal    SuccessfulCreate    Created pod: nginx-4ok8v
```

Here, three pods are created, but none is running yet, perhaps because the image is being pulled.
A little later, the same command may show:

```shell
Pods Status:    3 Running / 0 Waiting / 0 Succeeded / 0 Failed
```

To list all the pods that belong to the ReplicationController in a machine readable form, you can use a command like this:

```shell
pods=$(kubectl get pods --selector=app=nginx --output=jsonpath={.items..metadata.name})
echo $pods
```
```
nginx-3ntk0 nginx-4ok8v nginx-qrm3m
```

Here, the selector is the same as the selector for the ReplicationController (seen in the
`kubectl describe` output), and in a different form in `replication.yaml`.  The `--output=jsonpath` option
specifies an expression that just gets the name from each pod in the returned list.
 -->
## 一个 ReplicationController 的示例

这是一个运行三个 nginx web 服务副本的 ReplicationController 配置：

{{< codenew file="controllers/replication.yaml" >}}

下载这个示例配置文件，并运行以下命令:
```shell
kubectl apply -f replication.yaml
```
输出结果为:
```
replicationcontroller/nginx created
```

使用以下命令查看 ReplicationController 的状态:

```shell
kubectl describe replicationcontrollers/nginx
```
输出结果类似如下:
```
Name:        nginx
Namespace:   default
Selector:    app=nginx
Labels:      app=nginx
Annotations:    <none>
Replicas:    3 current / 3 desired
Pods Status: 0 Running / 3 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:       app=nginx
  Containers:
   nginx:
    Image:              nginx
    Port:               80/TCP
    Environment:        <none>
    Mounts:             <none>
  Volumes:              <none>
Events:
  FirstSeen       LastSeen     Count    From                        SubobjectPath    Type      Reason              Message
  ---------       --------     -----    ----                        -------------    ----      ------              -------
  20s             20s          1        {replication-controller }                    Normal    SuccessfulCreate    Created pod: nginx-qrm3m
  20s             20s          1        {replication-controller }                    Normal    SuccessfulCreate    Created pod: nginx-3ntk0
  20s             20s          1        {replication-controller }                    Normal    SuccessfulCreate    Created pod: nginx-4ok8v
```

这时，三个 Pod 都已经衩上创建， 但还没有一个在运行，可能是因为还在拉镜像。
再过一会，同一条命令的输出结果可能如下:

```
Pods Status:    3 Running / 0 Waiting / 0 Succeeded / 0 Failed
```

以机器可读的横笔列举所以属于 ReplicationController Pod，可运行以下命令：

```shell
pods=$(kubectl get pods --selector=app=nginx --output=jsonpath={.items..metadata.name})
echo $pods
```
输出结果类似如下：
```
nginx-3ntk0 nginx-4ok8v nginx-qrm3m
```

这里使用的选择器与 ReplicationController 的选择器相同(可以通过 `kubectl describe` 输出验证)，
与 `replication.yaml` 的格式不一样。`--output=jsonpath` 指定了一个表达式， 表示只返回每个 Pod 的名称为一个列表。
<!--
## Writing a ReplicationController Spec

As with all other Kubernetes config, a ReplicationController needs `apiVersion`, `kind`, and `metadata` fields.
The name of a ReplicationController object must be a valid
[DNS subdomain name](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).
For general information about working with config files, see [object management ](/docs/concepts/overview/working-with-objects/object-management/).

A ReplicationController also needs a [`.spec` section](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status).
 -->
## 编写 ReplicationController Spec

与所以其它的 k8s 配置一样， ReplicationController 必要字段有 `apiVersion`, `kind`, `metadata`。
ReplicationController 对象的名称必须是一个有效的
[DNS 子域名](/k8sDocs/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).
更多编写配置文件所需要的信息，见
[对象管理 ](/k8sDocs/docs/concepts/overview/working-with-objects/object-management/).
ReplicationController 还需要一个 [`.spec` 配置区](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status).
<!--
### Pod Template

The `.spec.template` is the only required field of the `.spec`.

The `.spec.template` is a [pod template](/docs/concepts/workloads/pods/#pod-templates). It has exactly the same schema as a {{< glossary_tooltip text="Pod" term_id="pod" >}}, except it is nested and does not have an `apiVersion` or `kind`.

In addition to required fields for a Pod, a pod template in a ReplicationController must specify appropriate
labels and an appropriate restart policy. For labels, make sure not to overlap with other controllers. See [pod selector](#pod-selector).

Only a [`.spec.template.spec.restartPolicy`](/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy) equal to `Always` is allowed, which is the default if not specified.

For local container restarts, ReplicationControllers delegate to an agent on the node,
for example the [Kubelet](/docs/reference/command-line-tools-reference/kubelet/) or Docker.
 -->
### Pod 模板

`.spec.template` 是 `.spec` 的唯一必要字段。

与 {{< glossary_tooltip text="Pod" term_id="pod" >}}， 除了因为嵌套没有 `apiVersion` 或 `kind`外完全相同的结构。

写 Pod 相比额外的必要字段是 ReplicationController 的 Pod 模板必须要指定恰当的标签和一个恰当的重启策略。
对于标签，需要确保不能与其他控制器重叠。 见 [pod 选择器](#pod-selector)

[`.spec.template.spec.restartPolicy`](/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy)
的值只能是 `Always`， 也是不指定时的默认值。

ReplicationControllers 委任节点上的代理，如，
[Kubelet](/docs/reference/command-line-tools-reference/kubelet/)
或 Docker 来实现对于本地容器的重启。
<!--
### Labels on the ReplicationController

The ReplicationController can itself have labels (`.metadata.labels`).  Typically, you
would set these the same as the `.spec.template.metadata.labels`; if `.metadata.labels` is not specified
then it defaults to  `.spec.template.metadata.labels`.  However, they are allowed to be
different, and the `.metadata.labels` do not affect the behavior of the ReplicationController.
 -->
### ReplicationController 的标签

ReplicationController 本身可以有标签(`.metadata.labels`)。 通常与 `.spec.template.metadata.labels` 设置相同的内容。
如果 `.metadata.labels` 没有设置，则默认会设置与 `.spec.template.metadata.labels` 相同。
但是，它们之间的内容可以不同， `.metadata.labels` 不会影响 ReplicationController 的行为。

<!--
### Pod Selector

The `.spec.selector` field is a [label selector](/docs/concepts/overview/working-with-objects/labels/#label-selectors). A ReplicationController
manages all the pods with labels that match the selector. It does not distinguish
between pods that it created or deleted and pods that another person or process created or
deleted. This allows the ReplicationController to be replaced without affecting the running pods.

If specified, the `.spec.template.metadata.labels` must be equal to the `.spec.selector`, or it will
be rejected by the API.  If `.spec.selector` is unspecified, it will be defaulted to
`.spec.template.metadata.labels`.

Also you should not normally create any pods whose labels match this selector, either directly, with
another ReplicationController, or with another controller such as Job. If you do so, the
ReplicationController thinks that it created the other pods.  Kubernetes does not stop you
from doing this.

If you do end up with multiple controllers that have overlapping selectors, you
will have to manage the deletion yourself (see [below](#working-with-replicationcontrollers)).
 -->
### Pod 选择器 {#pod-selector}

`.spec.selector` 字段是一个
[标签选择器](/k8sDocs/docs/concepts/overview/working-with-objects/labels/#label-selectors).
ReplicationController 会管理所有匹配它的选择器的 Pod。 不会区分这些 Pod 是由它自己创建或删除
还是由别的人或进行创建或删除的 Pod。 这让 ReplicationController 可以在不影响运行 Pod 的情况下被替换。

如果设置 `.spec.selector`，则它的内容必须要与 `.spec.template.metadata.labels` 相同，
否则会被 API 拒绝。 如果没有设置 `.spec.selector`， 则会被默认设置为 `.spec.template.metadata.labels` 的内容。

用户通常不应该再创建能被这个选择匹配标签的 Pod， 无论是直接创建还是通过另一个 ReplicationController，
或通过其它的诸如 Job 的控制器。 如果这样做， ReplicationController 也会把这些 Pod 认为是它的。
k8s 不会阻止用户这样做。

如果最终有多个控制器拥有重叠的选择器，用户需要自己来管理删除(见 [下面](#working-with-replicationcontrollers))
<!--
### Multiple Replicas

You can specify how many pods should run concurrently by setting `.spec.replicas` to the number
of pods you would like to have running concurrently.  The number running at any time may be higher
or lower, such as if the replicas were just increased or decreased, or if a pod is gracefully
shutdown, and a replacement starts early.

If you do not specify `.spec.replicas`, then it defaults to 1.
 -->
### 多副本

用户可以通过设置 `.spec.replicas` 来指定同时运行的 Pod 的数量。 在任意时间内运行的 Pod 的数量
可能比这个值大也可能小， 例如，刚好的增加或减少副本数量， 或一个 Pod 正在被平滑地关闭，而它的替代者先启动起来了。

如果没有指定 `.spec.replicas`， 则默认为 1
<!--
## Working with ReplicationControllers

### Deleting a ReplicationController and its Pods

To delete a ReplicationController and all its pods, use [`kubectl
delete`](/docs/reference/generated/kubectl/kubectl-commands#delete).  Kubectl will scale the ReplicationController to zero and wait
for it to delete each pod before deleting the ReplicationController itself.  If this kubectl
command is interrupted, it can be restarted.

When using the REST API or go client library, you need to do the steps explicitly (scale replicas to
0, wait for pod deletions, then delete the ReplicationController).

### Deleting just a ReplicationController

You can delete a ReplicationController without affecting any of its pods.

Using kubectl, specify the `--cascade=false` option to [`kubectl delete`](/docs/reference/generated/kubectl/kubectl-commands#delete).

When using the REST API or go client library, simply delete the ReplicationController object.

Once the original is deleted, you can create a new ReplicationController to replace it.  As long
as the old and new `.spec.selector` are the same, then the new one will adopt the old pods.
However, it will not make any effort to make existing pods match a new, different pod template.
To update pods to a new spec in a controlled way, use a [rolling update](#rolling-updates).

### Isolating pods from a ReplicationController

Pods may be removed from a ReplicationController's target set by changing their labels. This technique may be used to remove pods from service for debugging, data recovery, etc. Pods that are removed in this way will be replaced automatically (assuming that the number of replicas is not also changed).
 -->

## ReplicationController 的使用 {#working-with-replicationcontrollers}

### 删除 ReplicationController 及其 Pod

要删除一个 ReplicationController 及其所有的 Pod 可以通过命令
[`kubectl delete`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete).
kubectl 会将 ReplicationController 的副本数缩减为 0， 然后等待删除掉它的每一个 Pod，最后删除
ReplicationController 本身。 如果这个 kubectl 命令被打断，可以被重新启动。

当使用 REST API 或 go 客户端库时， 用户需要显示地进行这两个步骤(将副本数缩减为0，然后等待 Pod 被删除后，
  再删除 ReplicationController)。
### 仅删除 ReplicationController 对象


用户可以只删除 ReplicationController 而不影响任何它的 Pod。

在使用 kubectl 时，添加 `--cascade=false` 选项到  [`kubectl delete`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete).
在使用 REST API 或 go 客户端库， 直接删除 ReplicationController 对象。
当原来的对象被删除后，可以再创建一个新的 ReplicationController 来替代它。只要原来的对象和新创建的对象
拥有相同的 `.spec.selector`， 新的对象就会接管旧的 Pod。 但是，它不会尝试对旧的 Pod 做任何修改以适应新的不同的 Pod 模板。
要受控地更新 Pod 的配置，需要使用 [滚动更新](#rolling-updates)

### 从 ReplicationController 剥离 Pod

可以通过修改 Pod 标签的方式将 Pod 从 ReplicationController 中移出来。 通过这种方式可以将
Pod 从服务从移出来用作测试，数据恢复等。 通过这种方式移除的 Pod 会自动地被替代(假设副本数没有同步修改)
<!--
## Common usage patterns

### Rescheduling

As mentioned above, whether you have 1 pod you want to keep running, or 1000, a ReplicationController will ensure that the specified number of pods exists, even in the event of node failure or pod termination (for example, due to an action by another control agent).

### Scaling

The ReplicationController makes it easy to scale the number of replicas up or down, either manually or by an auto-scaling control agent, by simply updating the `replicas` field.
 -->
## 常见使用方式

### 重新调度

就是上面掉到的，无认是想运行 1 个 Pod 还是 1000 个， ReplicationController 就会确保指定数量
的 Pod 存在。 即便某个节点挂了或 Pod被终止(例如，由于其它控制代理的操作)
### 容量变更

ReplicationController 使得对副本数量的增加或减少变得很容易， 只需要手动或自动容量控制程序来更新一个 `replicas` 字段。
<!--
### Rolling updates

The ReplicationController is designed to facilitate rolling updates to a service by replacing pods one-by-one.

As explained in [#1353](https://issue.k8s.io/1353), the recommended approach is to create a new ReplicationController with 1 replica, scale the new (+1) and old (-1) controllers one by one, and then delete the old controller after it reaches 0 replicas. This predictably updates the set of pods regardless of unexpected failures.

Ideally, the rolling update controller would take application readiness into account, and would ensure that a sufficient number of pods were productively serving at any given time.

The two ReplicationControllers would need to create pods with at least one differentiating label, such as the image tag of the primary container of the pod, since it is typically image updates that motivate rolling updates.
 -->
### 滚动更新 {#rolling-updates}

ReplicationController 设计的初衷便是通过一个一个替换 服务的 Pod 的方式实现滚动更新。

就如 [#1353](https://issue.k8s.io/1353) 解释的那样。推荐的方是创建一个新的包含一个副本的 ReplicationController。
控制器一次一次地将 新的(+1) 然后 旧的 (-1)， 当旧的副本数降为 0 后再把它本身删除。 通过这种可预期的方式更新那认为是非预期的故障。

理想情况下，滚动更新控制会考虑应用的就绪探针， 会确保有足够数量的 Pod 在任何时间都能有效的提供服务。

这两个 ReplicationController 创建的 Pod 至少有一个标签是不同的， 例如 Pod 主要容器的镜像标签
因为通过滚动更新的动力就是更新镜像的版本。
<!--
### Multiple release tracks

In addition to running multiple releases of an application while a rolling update is in progress, it's common to run multiple releases for an extended period of time, or even continuously, using multiple release tracks. The tracks would be differentiated by labels.

For instance, a service might target all pods with `tier in (frontend), environment in (prod)`.  Now say you have 10 replicated pods that make up this tier.  But you want to be able to 'canary' a new version of this component.  You could set up a ReplicationController with `replicas` set to 9 for the bulk of the replicas, with labels `tier=frontend, environment=prod, track=stable`, and another ReplicationController with `replicas` set to 1 for the canary, with labels `tier=frontend, environment=prod, track=canary`.  Now the service is covering both the canary and non-canary pods.  But you can mess with the ReplicationControllers separately to test things out, monitor the results, etc.
 -->
### 多版本发布

在滚动更新过程中运行多个版本的基础上，通常也可能运行多个版本一段时间，或长久的运行多个版本。 这些版本之间通过标签来区分。

比如一个 Service 的标签选择器为 `tier in (frontend), environment in (prod)`。 这时候假设这一层由 10 个副本的 Pod 组成。
此时想要在该组件发布一个"金丝雀"的新版本. 这时可以配置一个 9 副本的 ReplicationController 打标签为
`tier=frontend, environment=prod, track=stable`， 再配置一个 1 副本的 ReplicationController
打标签为 `tier=frontend, environment=prod, track=canary`。 此时 Service 能匹配所以
金丝雀版本和稳定版本的 Pod。 但却可以分不同的 ReplicationController 进行测试和查看监控结果等。
<!--
### Using ReplicationControllers with Services

Multiple ReplicationControllers can sit behind a single service, so that, for example, some traffic
goes to the old version, and some goes to the new version.

A ReplicationController will never terminate on its own, but it isn't expected to be as long-lived as services. Services may be composed of pods controlled by multiple ReplicationControllers, and it is expected that many ReplicationControllers may be created and destroyed over the lifetime of a service (for instance, to perform an update of pods that run the service). Both services themselves and their clients should remain oblivious to the ReplicationControllers that maintain the pods of the services.
 -->
### ReplicationController 配合 Service 使用

Multiple ReplicationControllers can sit behind a single service, so that, for example, some traffic
goes to the old version, and some goes to the new version.
多个 ReplicationController 可以挂在一个 Service 下面， 这样可以让有些流量到新的版本，有的流量到旧的版本。

ReplicationController 永远都不会把自己干掉， 但也没有期望它可以和 Service 活得一样久。
Service 可能由多个 ReplicationController 控制的 Pod 组成。
在 Service 的生存期内或心预料有许多 ReplicationController 被创建另一些被删除
(例如， 更新 Service 运行的 Pod)。 Service 和它的客户端都不会注意到这些维护 Service Pod 的
ReplicationController。
<!--
## Writing programs for Replication

Pods created by a ReplicationController are intended to be fungible and semantically identical, though their configurations may become heterogeneous over time. This is an obvious fit for replicated stateless servers, but ReplicationControllers can also be used to maintain availability of master-elected, sharded, and worker-pool applications. Such applications should use dynamic work assignment mechanisms, such as the [RabbitMQ work queues](https://www.rabbitmq.com/tutorials/tutorial-two-python.html), as opposed to static/one-time customization of the configuration of each pod, which is considered an anti-pattern. Any pod customization performed, such as vertical auto-sizing of resources (for example, cpu or memory), should be performed by another online controller process, not unlike the ReplicationController itself.
 -->
<!--
## Writing programs for Replication

Pods created by a ReplicationController are intended to be fungible and semantically identical, though their configurations may become heterogeneous over time. This is an obvious fit for replicated stateless servers, but ReplicationControllers can also be used to maintain availability of master-elected, sharded, and worker-pool applications. Such applications should use dynamic work assignment mechanisms, such as the [RabbitMQ work queues](https://www.rabbitmq.com/tutorials/tutorial-two-python.html), as opposed to static/one-time customization of the configuration of each pod, which is considered an anti-pattern. Any pod customization performed, such as vertical auto-sizing of resources (for example, cpu or memory), should be performed by another online controller process, not unlike the ReplicationController itself.
  -->
## 编写多副本程序

ReplicationController 创建的 Pod 在设计上是能够替换的和语义相同的， 尽管经过时间的推移它们的配置可能变得不同。
这很明显适合于无状态多副本服务， 但 ReplicationControllers 也可以用于维护 主选举，分片，
工作池应用的可用性。 这些应用应该有动态的工作分配机制， 例如
[RabbitMQ 工作队列](https://www.rabbitmq.com/tutorials/tutorial-two-python.html),
这与静态的一次性对每个 Pod 的自定义配置相反， 被认为是反范式。
任意对 Pod 的自定义操作，如资源(如，CPU或RAM)资源的垂直自动容量调整， 应该通过另一个
在线控制器程序来执行， 而不是 ReplicationController 本身。
{{<todo-optimize>}}
<!--
## Responsibilities of the ReplicationController

The ReplicationController simply ensures that the desired number of pods matches its label selector and are operational. Currently, only terminated pods are excluded from its count. In the future, [readiness](https://issue.k8s.io/620) and other information available from the system may be taken into account, we may add more controls over the replacement policy, and we plan to emit events that could be used by external clients to implement arbitrarily sophisticated replacement and/or scale-down policies.

The ReplicationController is forever constrained to this narrow responsibility. It itself will not perform readiness nor liveness probes. Rather than performing auto-scaling, it is intended to be controlled by an external auto-scaler (as discussed in [#492](https://issue.k8s.io/492)), which would change its `replicas` field. We will not add scheduling policies (for example, [spreading](https://issue.k8s.io/367#issuecomment-48428019)) to the ReplicationController. Nor should it verify that the pods controlled match the currently specified template, as that would obstruct auto-sizing and other automated processes. Similarly, completion deadlines, ordering dependencies, configuration expansion, and other features belong elsewhere. We even plan to factor out the mechanism for bulk pod creation ([#170](https://issue.k8s.io/170)).

The ReplicationController is intended to be a composable building-block primitive. We expect higher-level APIs and/or tools to be built on top of it and other complementary primitives for user convenience in the future. The "macro" operations currently supported by kubectl (run, scale) are proof-of-concept examples of this. For instance, we could imagine something like [Asgard](https://techblog.netflix.com/2012/06/asgard-web-based-cloud-management-and.html) managing ReplicationControllers, auto-scalers, services, scheduling policies, canaries, etc.
 -->
##  ReplicationController 的职责

ReplicationController 只是确保匹配其标签选择器的 Pod 的数量与期望值相同，并且可用。
目前，只有被终止的 Pod 才不会被计算。 在将来，[就绪探针](https://issue.k8s.io/620)
和其它来自系统的可用性信息也会被计入， 我们还可能添加由通过替代策略的控制，
我们还计划发出可以用于外部客户端实现任意广泛代替和缩减容量策略的事件。

ReplicationController 永远被限制在这个小的责任范围内。 它本身也不会执行就绪探针或存活探针。
不执行自动容量控制， 在设计上就由外部自动容量控制器来控制(就如 [#492](https://issue.k8s.io/492) 中讨论的一样)
来修改它的 `replicas` 字段。 我们不会添加调度策略(例如，[spreading](https://issue.k8s.io/367#issuecomment-48428019))
到 ReplicationController， 也不会让它来验证它控制的 Pod 是否与当前的 Pod 模板一致，因为这会
妨碍自动容量这得和其它的自动化管理。 类似的 完成死线，顺序信赖，配置扩展和属于其它地方的其它鹅。
我们甚至计划把批量创建 Pod 的机制拆出来。 ([#170](https://issue.k8s.io/170)).

ReplicationController 设计上就是一个可组合的构建基础模块。我个预期的是在未来使用更高级的 API 或
工具基于它和其它的互补基础来为用户提供便利。 目前由 kubectl (run, scale) 的宏操作就是证明。
目前我们能想到的一些时，如 [Asgard](https://techblog.netflix.com/2012/06/asgard-web-based-cloud-management-and.html)
管理 ReplicationControllers，自动容量管理器， Service, 调度策略，金丝雀等。
<!--
## API Object

Replication controller is a top-level resource in the Kubernetes REST API. More details about the
API object can be found at:
[ReplicationController API object](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#replicationcontroller-v1-core).
 -->
## API 对象

ReplicationController 是一个在 k8s REST API 中的顶级资源。更多关于 API 对象的信息见:
[ReplicationController API 对象](https://kubernetes.io/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#replicationcontroller-v1-core).
<!--
## Alternatives to ReplicationController

### ReplicaSet

[`ReplicaSet`](/docs/concepts/workloads/controllers/replicaset/) is the next-generation ReplicationController that supports the new [set-based label selector](/docs/concepts/overview/working-with-objects/labels/#set-based-requirement).
It's mainly used by [Deployment](/docs/concepts/workloads/controllers/deployment/) as a mechanism to orchestrate pod creation, deletion and updates.
Note that we recommend using Deployments instead of directly using Replica Sets, unless you require custom update orchestration or don't require updates at all.


### Deployment (Recommended)

[`Deployment`](/docs/concepts/workloads/controllers/deployment/) is a higher-level API object that updates its underlying Replica Sets and their Pods. Deployments are recommended if you want this rolling update functionality because, they are declarative, server-side, and have additional features.

### Bare Pods

Unlike in the case where a user directly created pods, a ReplicationController replaces pods that are deleted or terminated for any reason, such as in the case of node failure or disruptive node maintenance, such as a kernel upgrade. For this reason, we recommend that you use a ReplicationController even if your application requires only a single pod. Think of it similarly to a process supervisor, only it supervises multiple pods across multiple nodes instead of individual processes on a single node.  A ReplicationController delegates local container restarts to some agent on the node (for example, Kubelet or Docker).

### Job

Use a [`Job`](/docs/concepts/workloads/controllers/job/) instead of a ReplicationController for pods that are expected to terminate on their own
(that is, batch jobs).

### DaemonSet

Use a [`DaemonSet`](/docs/concepts/workloads/controllers/daemonset/) instead of a ReplicationController for pods that provide a
machine-level function, such as machine monitoring or machine logging.  These pods have a lifetime that is tied
to a machine lifetime: the pod needs to be running on the machine before other pods start, and are
safe to terminate when the machine is otherwise ready to be rebooted/shutdown.
 -->
## ReplicationController 替代方案

### ReplicaSet

[`ReplicaSet`](/k8sDocs/docs/concepts/workloads/controllers/replicaset/)  
是下一代的 ReplicationController， 它支持新的
[基于集合的标签选择器](/k8sDocs/docs/concepts/overview/working-with-objects/labels/#set-based-requirement).
它主要被 [Deployment](/k8sDocs/docs/concepts/workloads/controllers/deployment/) 用于编排 Pod 创建，删除，更新的一个机制。
还要注意的是我们推荐使用 Deployment 而不是直接使用 ReplicaSet，除非你需要自定义的更新编排或者完全不需要更新。

### Deployment (推荐)

[`Deployment`](/k8sDocs/docs/concepts/workloads/controllers/deployment/)
是一个更高级别的 API 对象，用于更新它下层的 ReplicaSet 和它们的 Pod。
如果想要使用滚动更新推荐使用 Deployment， 因为它们是声明式的，服务端的，还有其它额外的特性。

### 裸 Pod

与直接创建 Pod 的情况不同， ReplicationController 会替换那些因任意原为被删除或终止的 Pod，
例如节点掉挂，或因为升级内格而维护。 因此我们推荐即便应用只需要一个 Pod 也使用 ReplicationController。
可以认为它是一个进程监督者，只是它监督的是多个节点上的多个 Pod 而不是一个节点上的一个进程。
ReplicationController 委托节点上的一些代理(如 kubelet 或 Docker)来重启本地容器。

### Job

如果 Pod 计划中会自己终止(如，批量任务)就应该使用  
[`Job`](/k8sDocs/docs/concepts/workloads/controllers/job/)
而不是 ReplicationController

### DaemonSet

Use a [`DaemonSet`](/docs/concepts/workloads/controllers/daemonset/) instead of a ReplicationController for pods that provide a
machine-level function, such as machine monitoring or machine logging.  These pods have a lifetime that is tied
to a machine lifetime: the pod needs to be running on the machine before other pods start, and are
safe to terminate when the machine is otherwise ready to be rebooted/shutdown.

如果 Pod 提供的是机器级别的功能，如机器监控，机器日志。就应该使用
[`DaemonSet`](/k8sDocs/docs/concepts/workloads/controllers/daemonset/)
而不是 ReplicationController。 这些 Pod 的生命期与机器绑定: 这些 Pod 需要先于其它的 Pod
在机器上运行， 且它们只有在机器准备重启或关机时才是终止的时候。

## 更多信息

实践 [使用 Deployment 运行无状态应用](/k8sDocs/tasks/run-application/run-stateless-application-deployment/).
