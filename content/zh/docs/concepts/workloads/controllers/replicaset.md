---
title: ReplicaSet
date: 2020-08-13
publishdate: 2020-08-14
weight: 2030101
---
<!--
---
reviewers:
- Kashomon
- bprashanth
- madhusudancs
title: ReplicaSet
content_type: concept
weight: 20
---
 -->
<!-- overview -->
<!--
A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time. As such, it is often
used to guarantee the availability of a specified number of identical Pods.
 -->
`ReplicaSet` 的作用是在任意时间内维持一个稳定的 Pod 副本集。因此它经常被用来保证特定 Pod 在指定的数量以提供可用性。

<!-- body -->
<!--
## How a ReplicaSet works

A ReplicaSet is defined with fields, including a selector that specifies how to identify Pods it can acquire, a number
of replicas indicating how many Pods it should be maintaining, and a pod template specifying the data of new Pods
it should create to meet the number of replicas criteria. A ReplicaSet then fulfills its purpose by creating
and deleting Pods as needed to reach the desired number. When a ReplicaSet needs to create new Pods, it uses its Pod
template.

A ReplicaSet is linked to its Pods via the Pods' [metadata.ownerReferences](/docs/concepts/workloads/controllers/garbage-collection/#owners-and-dependents)
field, which specifies what resource the current object is owned by. All Pods acquired by a ReplicaSet have their owning
ReplicaSet's identifying information within their ownerReferences field. It's through this link that the ReplicaSet
knows of the state of the Pods it is maintaining and plans accordingly.

A ReplicaSet identifies new Pods to acquire by using its selector. If there is a Pod that has no OwnerReference or the
OwnerReference is not a {{< glossary_tooltip term_id="controller" >}} and it matches a ReplicaSet's selector, it will be immediately acquired by said
ReplicaSet.
 -->
## `ReplicaSet` 是怎么工作的 {#how-a-replicaset-works}

`ReplicaSet` 通过以下字段定义，
`selector` 标签选择器，决定集合中的 Pod，
`replicas` 副本数量， 需要维持多少个副本，
`template` Pod 的定义模板，
`ReplicaSet` 会创建 Pod 或删除 Pod 让 Pod 的数量达到预期的数量。
`ReplicaSet` 在需要创建新的 Pod 时会依照 Pod 的定义模来创建 Pod。

`ReplicaSet` 通过 Pod 上的 [metadata.ownerReferences](/docs/concepts/workloads/controllers/garbage-collection/#owners-and-dependents)
字段实现它们的所以关系。 所以属于该 `ReplicaSet` 的每一个 Pod 上面都有自己的 ownerReferences 字段来存储它们自己的 `ReplicaSet` 信息

通过这个链接信息 `ReplicaSet` 可以知道对应的 Pod 的状态，并根据状态进行相应的维护与计划

`ReplicaSet` 通过选择器识别新的 Pod。 如果一个 Pod 上没有 `OwnerReference` 或 `OwnerReference`
不是一个 {{< glossary_tooltip term_id="controller" >}} 然后它又匹配该 `ReplicaSet` 的选择器，
那么它马上就会被该 `ReplicaSet` 捕获。

{{<todo-optimize>}}
<!--
## When to use a ReplicaSet

A ReplicaSet ensures that a specified number of pod replicas are running at any given
time. However, a Deployment is a higher-level concept that manages ReplicaSets and
provides declarative updates to Pods along with a lot of other useful features.
Therefore, we recommend using Deployments instead of directly using ReplicaSets, unless
you require custom update orchestration or don't require updates at all.

This actually means that you may never need to manipulate ReplicaSet objects:
use a Deployment instead, and define your application in the spec section.
 -->
## 什么时候应该用 `ReplicaSet`

`ReplicaSet` 保证在任意时间点上某个 Pod 的拥有指定的副本数。
但是 `Deployment` 是一个更高层级的抽象概念，实现对 `ReplicaSet` 的管理并提供了对 Pod 的声明式更新及其它一系列有用的特性。
所以，我们建议使用 `Deployment` 而不是直接使用 `ReplicaSet`
除非用户需要自定义的更新编排甚至干脆不需要更新。

也就是说一般用户永远都不需要操作 `ReplicaSet`： 使用 `Deployment` 更佳，只需要在 `spec` 区域定义应用就行


<!--
## Example

{{< codenew file="controllers/frontend.yaml" >}}

Saving this manifest into `frontend.yaml` and submitting it to a Kubernetes cluster will
create the defined ReplicaSet and the Pods that it manages.

```shell
kubectl apply -f https://kubernetes.io/examples/controllers/frontend.yaml
```

You can then get the current ReplicaSets deployed:

```shell
kubectl get rs
```

And see the frontend one you created:

```shell
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         3       6s
```

You can also check on the state of the ReplicaSet:

```shell
kubectl describe rs/frontend
```

And you will see output similar to:

```shell
Name:         frontend
Namespace:    default
Selector:     tier=frontend
Labels:       app=guestbook
              tier=frontend
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"apps/v1","kind":"ReplicaSet","metadata":{"annotations":{},"labels":{"app":"guestbook","tier":"frontend"},"name":"frontend",...
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  tier=frontend
  Containers:
   php-redis:
    Image:        gcr.io/google_samples/gb-frontend:v3
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  117s  replicaset-controller  Created pod: frontend-wtsmm
  Normal  SuccessfulCreate  116s  replicaset-controller  Created pod: frontend-b2zdv
  Normal  SuccessfulCreate  116s  replicaset-controller  Created pod: frontend-vcmts
```

And lastly you can check for the Pods brought up:

```shell
kubectl get pods
```

You should see Pod information similar to:

```shell
NAME             READY   STATUS    RESTARTS   AGE
frontend-b2zdv   1/1     Running   0          6m36s
frontend-vcmts   1/1     Running   0          6m36s
frontend-wtsmm   1/1     Running   0          6m36s
```

You can also verify that the owner reference of these pods is set to the frontend ReplicaSet.
To do this, get the yaml of one of the Pods running:

```shell
kubectl get pods frontend-b2zdv -o yaml
```

The output will look similar to this, with the frontend ReplicaSet's info set in the metadata's ownerReferences field:

```shell
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2020-02-12T07:06:16Z"
  generateName: frontend-
  labels:
    tier: frontend
  name: frontend-b2zdv
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: frontend
    uid: f391f6db-bb9b-4c09-ae74-6a1f77f3d5cf
...
```
 -->
## 示例

{{< codenew file="controllers/frontend.yaml" >}}

将以上配置保存到 `frontend.yaml`， 然后发布到 k8s 集群，就会在集群中创建一个 ReplicaSet 和对应 Pod。

```shell
kubectl apply -f frontend.yaml
```

可以通过以下命令查看当前部署的 `ReplicaSet`
```shell
kubectl get rs
```

输出结果中有类似如下如果:

```shell
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         3       6s
```

也可以通过以下命令查看 `ReplicaSet` 状态:

```shell
kubectl describe rs/frontend
```

输出结果类似如下:

```shell
Name:         frontend
Namespace:    default
Selector:     tier=frontend
Labels:       app=guestbook
              tier=frontend
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"apps/v1","kind":"ReplicaSet","metadata":{"annotations":{},"labels":{"app":"guestbook","tier":"frontend"},"name":"frontend",...
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  tier=frontend
  Containers:
   php-redis:
    Image:        gcr.io/google_samples/gb-frontend:v3
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  117s  replicaset-controller  Created pod: frontend-wtsmm
  Normal  SuccessfulCreate  116s  replicaset-controller  Created pod: frontend-b2zdv
  Normal  SuccessfulCreate  116s  replicaset-controller  Created pod: frontend-vcmts
```

最后再用以下命令查看创建的 Pod:

```shell
kubectl get pods
```

输出结果类似如下:

```shell
NAME             READY   STATUS    RESTARTS   AGE
frontend-b2zdv   1/1     Running   0          6m36s
frontend-vcmts   1/1     Running   0          6m36s
frontend-wtsmm   1/1     Running   0          6m36s
```

通过以正下命令输出的 yaml 可以验证这些 Pod 所属 `ReplicaSet`：

```shell
kubectl get pods frontend-b2zdv -o yaml
```

输出结果类似如下，可以看到 `metadata.ownerReferences` 字段中就是这个叫 `frontend` 的 `ReplicaSet`
```shell
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2020-02-12T07:06:16Z"
  generateName: frontend-
  labels:
    tier: frontend
  name: frontend-b2zdv
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: frontend
    uid: f391f6db-bb9b-4c09-ae74-6a1f77f3d5cf
...
```
<!--
## Non-Template Pod acquisitions

While you can create bare Pods with no problems, it is strongly recommended to make sure that the bare Pods do not have
labels which match the selector of one of your ReplicaSets. The reason for this is because a ReplicaSet is not limited
to owning Pods specified by its template-- it can acquire other Pods in the manner specified in the previous sections.

Take the previous frontend ReplicaSet example, and the Pods specified in the  following manifest:

{{< codenew file="pods/pod-rs.yaml" >}}

As those Pods do not have a Controller (or any object) as their owner reference and match the selector of the frontend
ReplicaSet, they will immediately be acquired by it.

Suppose you create the Pods after the frontend ReplicaSet has been deployed and has set up its initial Pod replicas to
fulfill its replica count requirement:

```shell
kubectl apply -f https://kubernetes.io/examples/pods/pod-rs.yaml
```

The new Pods will be acquired by the ReplicaSet, and then immediately terminated as the ReplicaSet would be over
its desired count.

Fetching the Pods:

```shell
kubectl get pods
```

The output shows that the new Pods are either already terminated, or in the process of being terminated:

```shell
NAME             READY   STATUS        RESTARTS   AGE
frontend-b2zdv   1/1     Running       0          10m
frontend-vcmts   1/1     Running       0          10m
frontend-wtsmm   1/1     Running       0          10m
pod1             0/1     Terminating   0          1s
pod2             0/1     Terminating   0          1s
```

If you create the Pods first:

```shell
kubectl apply -f https://kubernetes.io/examples/pods/pod-rs.yaml
```

And then create the ReplicaSet however:

```shell
kubectl apply -f https://kubernetes.io/examples/controllers/frontend.yaml
```

You shall see that the ReplicaSet has acquired the Pods and has only created new ones according to its spec until the
number of its new Pods and the original matches its desired count. As fetching the Pods:

```shell
kubectl get pods
```

Will reveal in its output:
```shell
NAME             READY   STATUS    RESTARTS   AGE
frontend-hmmj2   1/1     Running   0          9s
pod1             1/1     Running   0          36s
pod2             1/1     Running   0          36s
```

In this manner, a ReplicaSet can own a non-homogenous set of Pods
 -->
## 不是自己模板定义的 Pod 的捕获

用户可以自由的创建裸 Pod， 但是推荐在创建裸 Pod 时不要在上面打上已经存在的 `ReplicaSet` 选择器对应的标签。
不这么做的原因是 `ReplicaSet` 并不仅限于自身模板创建的 Pod， 也可以捕获其它拥有对应标签的 Pod(上一节介绍过)

继续上一个例子中的 `frontend` `ReplicaSet`， 再加下如下定义的 Pod

{{< codenew file="pods/pod-rs.yaml" >}}

这些 Pod 没并有在所属者引用上有控制器(或其它对象)， 但是它上面的标签又被 `frontend` `ReplicaSet`
的选择器所匹配，它们马上就会被这个 `ReplicaSet` 捕获。

假设 Pod 的创建时间是在 `frontend` `ReplicaSet` 部署之后，并且其创建的 Pod 已经达到了所期望的数量：

```shell
kubectl apply -f pod-rs.yaml
```

新创建的 Pod 会被 `ReplicaSet` 捕获，然后发现 `ReplicaSet` 副本数已经超过预期数量，所以就会终止这些多余的 Pod

查看 Pod 状态:

```shell
kubectl get pods
```

输出类似如下，会发现新创建的 Pod 正在被干掉或已经被干掉:

```shell
NAME             READY   STATUS        RESTARTS   AGE
frontend-b2zdv   1/1     Running       0          10m
frontend-vcmts   1/1     Running       0          10m
frontend-wtsmm   1/1     Running       0          10m
pod1             0/1     Terminating   0          1s
pod2             0/1     Terminating   0          1s
```

如果创建的顺序调换一下，先创建这些 Pod:

```shell
kubectl apply -f pod-rs.yaml
```

然后再创建 `ReplicaSet`:

```shell
kubectl apply -f frontend.yaml
```

这次就会发现 `ReplicaSet` 会捕获已经创建的两个 Pod，然后根据预期的状态中需要3个，所以就再创建一个新的，就完工了。
再次查看 Pod 状态

```shell
kubectl get pods
```

输出结果就会变得不一样，具体如下:
```shell
NAME             READY   STATUS    RESTARTS   AGE
frontend-hmmj2   1/1     Running   0          9s
pod1             1/1     Running   0          36s
pod2             1/1     Running   0          36s
```

也就是， `ReplicaSet` 下属的 Pod 可以是不是通过本身模板创建的 Pod
<!--  
## Writing a ReplicaSet manifest

As with all other Kubernetes API objects, a ReplicaSet needs the `apiVersion`, `kind`, and `metadata` fields.
For ReplicaSets, the kind is always just ReplicaSet.
In Kubernetes 1.9 the API version `apps/v1` on the ReplicaSet kind is the current version and is enabled by default. The API version `apps/v1beta2` is deprecated.
Refer to the first lines of the `frontend.yaml` example for guidance.

The name of a ReplicaSet object must be a valid
[DNS subdomain name](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

A ReplicaSet also needs a [`.spec` section](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status).
-->
## 编写 `ReplicaSet` 定义清单

与其它所以其它 k8s API 对象一样， `ReplicaSet` 需要有 `apiVersion`, `kind`, `metadata` 字段.
对于 `ReplicaSet`， `kind` 字段就一直是 `ReplicaSet`。
在 k8s v1.9+ `apiVersion` 字段的值为 `apps/v1`，默认开启。 API 版本 `apps/v1beta2` 被废弃。
具体可以看之前示例中 `frontend.yaml` 文件内容。

`ReplicaSet` 名称必须是一个有效的 [DNS 子域名](/k8sDocs/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).
`ReplicaSet` 还必须要有一个
[`.spec` 字段](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status).
<!--
### Pod Template

The `.spec.template` is a [pod template](/docs/concepts/workloads/Pods/pod-overview/#pod-templates) which is also
required to have labels in place. In our `frontend.yaml` example we had one label: `tier: frontend`.
Be careful not to overlap with the selectors of other controllers, lest they try to adopt this Pod.

For the template's [restart policy](/docs/concepts/workloads/Pods/pod-lifecycle/#restart-policy) field,
`.spec.template.spec.restartPolicy`, the only allowed value is `Always`, which is the default.
 -->
### Pod 模板

`.spec.template` 字段就是 [pod template](/k8sDocs/docs/concepts/workloads/Pods/pod-overview/#pod-templates) 而且其中还必须要定义标签.
在示例中的 `frontend.yaml` 打的标签是: `tier: frontend`.
注意不能与其它控制器的选择器相重叠，以免它们会争抢这个 Pod

模板中的 [重启策略](/docs/concepts/workloads/Pods/pod-lifecycle/#restart-policy) 字段,
`.spec.template.spec.restartPolicy`, 允许的值只能是 `Always`, 默认也是  `Always`.

<!--
### Pod Selector

The `.spec.selector` field is a [label selector](/docs/concepts/overview/working-with-objects/labels/). As discussed
[earlier](#how-a-replicaset-works) these are the labels used to identify potential Pods to acquire. In our
`frontend.yaml` example, the selector was:
```shell
matchLabels:
	tier: frontend
```

In the ReplicaSet, `.spec.template.metadata.labels` must match `spec.selector`, or it will
be rejected by the API.

{{< note >}}
For 2 ReplicaSets specifying the same `.spec.selector` but different `.spec.template.metadata.labels` and `.spec.template.spec` fields, each ReplicaSet ignores the Pods created by the other ReplicaSet.
{{< /note >}}
 -->
### Pod 选择器

`.spec.selector` 字段是一个 [标签选择器](/k8sDocs/docs/concepts/overview/working-with-objects/labels/)
[之前](#how-a-replicaset-works)讨论过，这些标签用于识别潜在的捕获对象。 在之前示例中 `frontend.yaml`
定义的选择器如下:

```yaml
matchLabels:
  tier: frontend
```

在 `ReplicaSet` 中， `.spec.template.metadata.labels` 必须要与 `spec.selector` 匹配，否则会被 API 拒绝。
{{< note >}}
如果两个 `ReplicaSet` 拥有相同的 `.spec.selector`， 但 `.spec.template.metadata.labels` 和 `.spec.template.spec` 字段不同
则相互之间会忽略对方创建的 Pod
{{< /note >}}

HOW？
<!--
### Replicas

You can specify how many Pods should run concurrently by setting `.spec.replicas`. The ReplicaSet will create/delete
its Pods to match this number.

If you do not specify `.spec.replicas`, then it defaults to 1.
 -->
### 副本数

用户可以通过 `.spec.replicas` 设置需要同时运行 Pod 的数量。 `ReplicaSet` 会创建/删除 它管理的 Pod 以达成该数量。
如果在配置中没有 `.spec.replicas`， 则默认为 1.
<!--
## Working with ReplicaSets

### Deleting a ReplicaSet and its Pods

To delete a ReplicaSet and all of its Pods, use [`kubectl delete`](/docs/reference/generated/kubectl/kubectl-commands#delete). The [Garbage collector](/docs/concepts/workloads/controllers/garbage-collection/) automatically deletes all of the dependent Pods by default.

When using the REST API or the `client-go` library, you must set `propagationPolicy` to `Background` or `Foreground` in
the -d option.
For example:
```shell
kubectl proxy --port=8080
curl -X DELETE  'localhost:8080/apis/apps/v1/namespaces/default/replicasets/frontend' \
> -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Foreground"}' \
> -H "Content-Type: application/json"
```
 -->
## ReplicaSet 的使用

### 删除一个 `ReplicaSet` 及其 Pod

要删除一个 `ReplicaSet` 及其所属的全部 Pod， 可以使用 [`kubectl delete`](/docs/reference/generated/kubectl/kubectl-commands#delete)
[垃圾回收](/docs/concepts/workloads/controllers/garbage-collection/) 默认会自动删除它所属的全部 Pod。

当使用  `REST API` 或 `client-go` 库， 必须将 `propagationPolicy` 设置为 `Background` 或 `Foreground`
例如:

```shell
kubectl proxy --port=8080
curl -X DELETE  'localhost:8080/apis/apps/v1/namespaces/default/replicasets/frontend' \
> -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Foreground"}' \
> -H "Content-Type: application/json"
```

<!--
### Deleting just a ReplicaSet

You can delete a ReplicaSet without affecting any of its Pods using [`kubectl delete`](/docs/reference/generated/kubectl/kubectl-commands#delete) with the `--cascade=false` option.
When using the REST API or the `client-go` library, you must set `propagationPolicy` to `Orphan`.
For example:
```shell
kubectl proxy --port=8080
curl -X DELETE  'localhost:8080/apis/apps/v1/namespaces/default/replicasets/frontend' \
> -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Orphan"}' \
> -H "Content-Type: application/json"
```

Once the original is deleted, you can create a new ReplicaSet to replace it.  As long
as the old and new `.spec.selector` are the same, then the new one will adopt the old Pods.
However, it will not make any effort to make existing Pods match a new, different pod template.
To update Pods to a new spec in a controlled way, use a
[Deployment](/docs/concepts/workloads/controllers/deployment/#creating-a-deployment), as ReplicaSets do not support a rolling update directly.
 -->
### 仅删除 `ReplicaSet`
用户可以通过 [`kubectl delete`](/docs/reference/generated/kubectl/kubectl-commands#delete)
加 `--cascade=false` 选项，实现仅删除 `ReplicaSet` 对象，而不删除其所属的 Pod。
当使用  `REST API` 或 `client-go` 库， 必须将 `propagationPolicy` 设置为 `Orphan`

例如:

```shell
kubectl proxy --port=8080
curl -X DELETE  'localhost:8080/apis/apps/v1/namespaces/default/replicasets/frontend' \
> -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Orphan"}' \
> -H "Content-Type: application/json"
```

当原来的 `ReplicaSet` 就可以创建一个新的来替代它。 新创建的 ReplicaSet 的 `.spec.selector` 需要与原来的一样， 这样它就能接管这些 Pod。
但是它不会让旧的 Pod 使用新的模板。
而想要以控制器方式更新 Pod 的模板，需要使用 [Deployment](/k8sDocs/docs/concepts/workloads/controllers/deployment/#creating-a-deployment)
因为 `ReplicaSet` 不支持直接的滚动更新。

<!--
### Isolating Pods from a ReplicaSet

You can remove Pods from a ReplicaSet by changing their labels. This technique may be used to remove Pods
from service for debugging, data recovery, etc. Pods that are removed in this way will be replaced automatically (
assuming that the number of replicas is not also changed).
 -->
### 将 Pod 从 `ReplicaSet` 剥离出来

用户可以通过修改标签的方式将 Pod 从 `ReplicaSet` 中移出， 也可以通过这种方式将 Pod 从其它的服务中移出后用作调度，数据恢复等。
用这种方式移出的 Pod 会自动被替代(如果副本数没有同步修改)
<!--
### Scaling a ReplicaSet

A ReplicaSet can be easily scaled up or down by simply updating the `.spec.replicas` field. The ReplicaSet controller
ensures that a desired number of Pods with a matching label selector are available and operational.
 -->
### `ReplicaSet` 容量伸缩

`ReplicaSet` 可以通过修改 `.spec.replicas` 字段来实现容量的扩大或缩小。 `ReplicaSet` 控制器会保证 所属的 Pod 的数量与预期的相同
<!--
### ReplicaSet as a Horizontal Pod Autoscaler Target

A ReplicaSet can also be a target for
[Horizontal Pod Autoscalers (HPA)](/docs/tasks/run-application/horizontal-pod-autoscale/). That is,
a ReplicaSet can be auto-scaled by an HPA. Here is an example HPA targeting
the ReplicaSet we created in the previous example.

{{< codenew file="controllers/hpa-rs.yaml" >}}

Saving this manifest into `hpa-rs.yaml` and submitting it to a Kubernetes cluster should
create the defined HPA that autoscales the target ReplicaSet depending on the CPU usage
of the replicated Pods.

```shell
kubectl apply -f https://k8s.io/examples/controllers/hpa-rs.yaml
```

Alternatively, you can use the `kubectl autoscale` command to accomplish the same
(and it's easier!)

```shell
kubectl autoscale rs frontend --max=10 --min=3 --cpu-percent=50
```
 -->
### `ReplicaSet` 作为 Pod 水平自动扩展目标

`ReplicaSet` 也可作为 [Horizontal Pod Autoscalers (HPA)](/k8sDocs/tasks/run-application/horizontal-pod-autoscale/)
的目标。 也就是 `ReplicaSet` 可以通过一个 HPA 来自动伸缩。 以下为一个 `ReplicaSet` 为目标的 HPA 的示例:

{{< codenew file="controllers/hpa-rs.yaml" >}}

将配置文件内容存入 `hpa-rs.yaml` 并提交到 k8s 集群， 会创建一个 HPA，它会根据所属 Pod CPU使用自动伸缩目标 `ReplicaSet` 的容量

```shell
kubectl apply -f hpa-rs.yaml
```

相应的，也可能通过 `kubectl autoscale` 达到相同的效果(更容易)

```shell
kubectl autoscale rs frontend --max=10 --min=3 --cpu-percent=50
```
<!--
## Alternatives to ReplicaSet

### Deployment (recommended)

[`Deployment`](/docs/concepts/workloads/controllers/deployment/) is an object which can own ReplicaSets and update
them and their Pods via declarative, server-side rolling updates.
While ReplicaSets can be used independently, today they're  mainly used by Deployments as a mechanism to orchestrate Pod
creation, deletion and updates. When you use Deployments you don’t have to worry about managing the ReplicaSets that
they create. Deployments own and manage their ReplicaSets.
As such, it is recommended to use Deployments when you want ReplicaSets.
 -->
## `ReplicaSet` 替代方案

### Deployment (推荐)

[`Deployment`](/k8sDocs/docs/concepts/workloads/controllers/deployment/) 对象可以包含 `ReplicaSet` 并且可以通过声明方式更新自身及所属的 Pod，
服务端滚动更新。 虽然 ReplicaSet 可以独立使用，但是现在主要是使用 Deployment 作为组织 Pod 创建，删除，更新的方式。
当用户使用 `Deployment` 不需要关心它所创建的 `ReplicaSet`， `Deployment` 会管理所属的 `ReplicaSet`
因此用户在想要使用 `ReplicaSet` 时推荐使用 `Deployment` 代替。

<!--
### Bare Pods

Unlike the case where a user directly created Pods, a ReplicaSet replaces Pods that are deleted or terminated for any reason, such as in the case of node failure or disruptive node maintenance, such as a kernel upgrade. For this reason, we recommend that you use a ReplicaSet even if your application requires only a single Pod. Think of it similarly to a process supervisor, only it supervises multiple Pods across multiple nodes instead of individual processes on a single node. A ReplicaSet delegates local container restarts to some agent on the node (for example, Kubelet or Docker).
 -->
### 让 Pod 裸奔

与用户直接创建 Pod 不同， `ReplicaSet` 在 Pod 因为某些原因被删除或终止时会创建替代的 Pod， 这些原因可能是 节点挂了，节点因维护如升级内格而引起的计划内故障。
对于这些原因，我们建议用户使用 `ReplicaSet` 即便这个应用只需要一个 Pod。 可以把它认为是一个进程监控， 只是它可以监控多个节点上的多个 Pod，而不是一个节点上的一个进程。
`ReplicaSet` 会委托节点上的代理(如 kubelet 或 docker)来实现对本地容器的重启。
<!--
### Job

Use a [`Job`](/docs/concepts/jobs/run-to-completion-finite-workloads/) instead of a ReplicaSet for Pods that are expected to terminate on their own
(that is, batch jobs).
 -->
### Job

在运行那会在有限时间内自己运行结束后自动终止的 Pod(批处理任务)，请使用 [`Job`](/k8sDocs/docs/concepts/jobs/run-to-completion-finite-workloads/),就不要用 `ReplicaSet`了
<!--
### DaemonSet

Use a [`DaemonSet`](/docs/concepts/workloads/controllers/daemonset/) instead of a ReplicaSet for Pods that provide a
machine-level function, such as machine monitoring or machine logging.  These Pods have a lifetime that is tied
to a machine lifetime: the Pod needs to be running on the machine before other Pods start, and are
safe to terminate when the machine is otherwise ready to be rebooted/shutdown.
 -->
### DaemonSet

当 Pod 需要使用到机器级别的功能，如机器监控或机器日志时，使用 [`DaemonSet`](/k8sDocs/docs/concepts/workloads/controllers/daemonset/)
因为这些 Pod 的生存期会与对应机器的生存期绑定在一起: 这些 Pod 需要在机器上其它 Pod 运行之前就在节点上运行， 只有在机器准备重启或关机时才能
安全的终止。
<!--
### ReplicationController

ReplicaSets are the successors to [_ReplicationControllers_](/docs/concepts/workloads/controllers/replicationcontroller/).
The two serve the same purpose, and behave similarly, except that a ReplicationController does not support set-based
selector requirements as described in the [labels user guide](/docs/concepts/overview/working-with-objects/labels/#label-selectors).
As such, ReplicaSets are preferred over ReplicationControllers
 -->
### ReplicationController

ReplicaSet 是  [_ReplicationControllers_](/k8sDocs/docs/concepts/workloads/controllers/replicationcontroller/) 的继任者。
它们两个可以达成相同的目的，而且行为方式也相似， 除了 `ReplicationController` 不支持像
[标签使用](/k8sDocs/docs/concepts/overview/working-with-objects/labels/#label-selectors) 中介绍的基于集合的选择器。
因此相对于 ReplicationControllers 推荐使用 `ReplicaSet`
