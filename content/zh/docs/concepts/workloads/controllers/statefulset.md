---
title: StatefulSet
date: 2020-08-14
publishdate: 2020-08-24
weight: 2030102
---
<!--
---
reviewers:
- enisoc
- erictune
- foxish
- janetkuo
- kow3ns
- smarterclayton
title: StatefulSets
content_type: concept
weight: 30
---
 -->
<!-- overview -->
<!--
StatefulSet is the workload API object used to manage stateful applications.

{{< glossary_definition term_id="statefulset" length="all" >}}
 -->
StatefulSet 是一种用于运行有状态应用的工作负载 API 对象
{{< glossary_definition term_id="statefulset" length="all" >}}
<!-- body -->
<!--
## Using StatefulSets


StatefulSets are valuable for applications that require one or more of the
following.

* Stable, unique network identifiers.
* Stable, persistent storage.
* Ordered, graceful deployment and scaling.
* Ordered, automated rolling updates.

In the above, stable is synonymous with persistence across Pod (re)scheduling.
If an application doesn't require any stable identifiers or ordered deployment,
deletion, or scaling, you should deploy your application using a workload object
that provides a set of stateless replicas.
[Deployment](/docs/concepts/workloads/controllers/deployment/) or
[ReplicaSet](/docs/concepts/workloads/controllers/replicaset/) may be better suited to your stateless needs.
 -->
## 什么时候用 StatefulSet

StatefulSet 适用于那些满足以下一个或多个条件的应用.

- 需要拥有稳定且唯一的网络标识的
- 需要拥有稳定的持久化存储的
- 需要按顺序进行部署或进行容量伸缩的
- 需要按顺序进行自动滚动更新的

在上面的条件中，稳定是指在 Pod 调度(或重新调度)后依然不变(功能上).
如果一个应用不需要任何稳定的标识或顺序的部署，删除，伸缩容量，用户应该使用那些无状态的工作负载对象。
[Deployment](/k8sDocs/docs/concepts/workloads/controllers/deployment/) 或
[ReplicaSet](/k8sDocs/docs/concepts/workloads/controllers/replicaset/) 可能就更适合用于无状态的需求。
<!--
## Limitations

* The storage for a given Pod must either be provisioned by a [PersistentVolume Provisioner](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/persistent-volume-provisioning/README.md) based on the requested `storage class`, or pre-provisioned by an admin.
* Deleting and/or scaling a StatefulSet down will *not* delete the volumes associated with the StatefulSet. This is done to ensure data safety, which is generally more valuable than an automatic purge of all related StatefulSet resources.
* StatefulSets currently require a [Headless Service](/docs/concepts/services-networking/service/#headless-services) to be responsible for the network identity of the Pods. You are responsible for creating this Service.
* StatefulSets do not provide any guarantees on the termination of pods when a StatefulSet is deleted. To achieve ordered and graceful termination of the pods in the StatefulSet, it is possible to scale the StatefulSet down to 0 prior to deletion.
* When using [Rolling Updates](#rolling-updates) with the default
  [Pod Management Policy](#pod-management-policies) (`OrderedReady`),
  it's possible to get into a broken state that requires
  [manual intervention to repair](#forced-rollback).
-->
## 限制 {#limitations}

- 给予某个指定 Pod 的存储必须要由 [PersistentVolume Provisioner](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/persistent-volume-provisioning/README.md) 基于 `storage class` 的请求或由管理员预先创建好。
- 删除或收缩 StatefulSet 的副本数 *不会* 删除与这个 StatefulSet 相关联的卷。 这么做是为了保证数据这险，相对与自动删除所有 StatefulSet 相关资源，这样保留下来通常更有意义。
- StatefulSet 目前还需要一个 [Headless Service](/k8sDocs/docs/concepts/services-networking/service/#headless-services)  来作为这些 Pod 的网络标识。
  需要用户负责创建。
- 当删除一个 StatefulSet 时，并不能对 Pod 的终结提供任何保障。 而要达成有序和平滑的终止，将 StatefulSet 的副本设置为 0 比直接删除更有保障。
- 在使用基于 [Pod 管理策略](#pod-management-policies) (`OrderedReady`) 的 [Rolling Updates](#rolling-updates)时，
  可能会出错且需要 [人工介入进行修复](#forced-rollback).
<!--
## Components

The example below demonstrates the components of a StatefulSet.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```

In the above example:

* A Headless Service, named `nginx`, is used to control the network domain.
* The StatefulSet, named `web`, has a Spec that indicates that 3 replicas of the nginx container will be launched in unique Pods.
* The `volumeClaimTemplates` will provide stable storage using [PersistentVolumes](/docs/concepts/storage/persistent-volumes/) provisioned by a PersistentVolume Provisioner.

The name of a StatefulSet object must be a valid
[DNS subdomain name](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).
 -->
## 组件

以下示例演示了 StatefulSet 的组件。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # 与要匹配 .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # 默认为 1
  template:
    metadata:
      labels:
        app: nginx # 与要匹配 .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: registry.cn-hangzhou.aliyuncs.com/lisong/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class" # 需要替换成实际的
      resources:
        requests:
          storage: 1Gi
```

在上面的例子中:

- 一个 Headless Service， 名称为 `nginx`， 用来控制网络域
- StatefulSet， 名称为 `web`， 在定义中指定有 3 个副本的 nginx 的容器启动在不同的 Pod 中
- `volumeClaimTemplates` 通过 PersistentVolume 提供者提供的 [PersistentVolumes](/k8sDocs/docs/concepts/storage/persistent-volumes/)
  提供稳定存储

StatefulSet 的名称必须是一个有效的
[DNS 子域名](/k8sDocs/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).
<!--
## Pod Selector

You must set the `.spec.selector` field of a StatefulSet to match the labels of its `.spec.template.metadata.labels`. Prior to Kubernetes 1.8, the `.spec.selector` field was defaulted when omitted. In 1.8 and later versions, failing to specify a matching Pod Selector will result in a validation error during StatefulSet creation.
 -->
## Pod 选择器

用户必须让 StatefulSet 的 `.spec.selector` 字段与其 `.spec.template.metadata.labels` 配置的标签相匹配。
在 k8s `1.8` 之前  `.spec.selector` 不设置会被添加默认。 `1.8` 及之后的版本，如果标签与选择器不匹配，StatefulSet 在创建时就会验证不过。
<!--
## Pod Identity

StatefulSet Pods have a unique identity that is comprised of an ordinal, a
stable network identity, and stable storage. The identity sticks to the Pod,
regardless of which node it's (re)scheduled on.
 -->
## Pod 标识

StatefulSet 的 Pod 都有一个顺序的唯一的标识， 一个稳定的网络标识，和稳定的存储。
标识与 Pod 绑定，不受节点之间的调度影响
<!--
### Ordinal Index

For a StatefulSet with N replicas, each Pod in the StatefulSet will be
assigned an integer ordinal, from 0 up through N-1, that is unique over the Set.
 -->
### 有序索引

对于一个有 N 个副本的 StatefulSet， StatefulSet 中的每个 Pod 都会被分配一个从 0 到 N - 1 的整数序号，这些序号在这个集合中唯一

<!--
### Stable Network ID

Each Pod in a StatefulSet derives its hostname from the name of the StatefulSet
and the ordinal of the Pod. The pattern for the constructed hostname
is `$(statefulset name)-$(ordinal)`. The example above will create three Pods
named `web-0,web-1,web-2`.
A StatefulSet can use a [Headless Service](/docs/concepts/services-networking/service/#headless-services)
to control the domain of its Pods. The domain managed by this Service takes the form:
`$(service name).$(namespace).svc.cluster.local`, where "cluster.local" is the
cluster domain.
As each Pod is created, it gets a matching DNS subdomain, taking the form:
`$(podname).$(governing service domain)`, where the governing service is defined
by the `serviceName` field on the StatefulSet.

Depending on how DNS is configured in your cluster, you may not be able to look up the DNS
name for a newly-run Pod immediately. This behavior can occur when other clients in the
cluster have already sent queries for the hostname of the Pod before it was created.
Negative caching (normal in DNS) means that the results of previous failed lookups are
remembered and reused, even after the Pod is running, for at least a few seconds.

If you need to discover Pods promptly after they are created, you have a few options:

- Query the Kubernetes API directly (for example, using a watch) rather than relying on DNS lookups.
- Decrease the time of caching in your Kubernetes DNS provider (tpyically this means editing the config map for CoreDNS, which currently caches for 30 seconds).


As mentioned in the [limitations](#limitations) section, you are responsible for
creating the [Headless Service](/docs/concepts/services-networking/service/#headless-services)
responsible for the network identity of the pods.

Here are some examples of choices for Cluster Domain, Service name,
StatefulSet name, and how that affects the DNS names for the StatefulSet's Pods.

Cluster Domain | Service (ns/name) | StatefulSet (ns/name)  | StatefulSet Domain  | Pod DNS | Pod Hostname |
-------------- | ----------------- | ----------------- | -------------- | ------- | ------------ |
 cluster.local | default/nginx     | default/web       | nginx.default.svc.cluster.local | web-{0..N-1}.nginx.default.svc.cluster.local | web-{0..N-1} |
 cluster.local | foo/nginx         | foo/web           | nginx.foo.svc.cluster.local     | web-{0..N-1}.nginx.foo.svc.cluster.local     | web-{0..N-1} |
 kube.local    | foo/nginx         | foo/web           | nginx.foo.svc.kube.local        | web-{0..N-1}.nginx.foo.svc.kube.local        | web-{0..N-1} |

{{< note >}}
Cluster Domain will be set to `cluster.local` unless
[otherwise configured](/docs/concepts/services-networking/dns-pod-service/).
{{< /note >}}
 -->
### 稳定的网络 ID

StatefulSet 中 Pod 的主机名由 StatefulSet 名称和 Pod 的序号组成。 格式为 `$(statefulset name)-$(ordinal)`
如果 Pod 数量为 3 则名称依次为 `web-0,web-1,web-2`.
StatefulSet 可以使用一个 [Headless Service](/k8sDocs/docs/concepts/services-networking/service/#headless-services)
来控制其 Pod 的网络域。 Service 的域名格式为 `$(service name).$(namespace).svc.cluster.local`
`cluster.local` 为集群域。
当每个 Pod 被创建后，都会获得一个相应的 DNS 子域名，格式为 `$(podname).$(governing service domain)`
其中 `governing service` 就是 StatefulSet 定义中 `serviceName` 字段

基于集群中的 DNS 配置方式， 用户可能不能在 Pod 创建后，马上就能查询到对应的 DNS 名称。 当这个 Pod 在创建之前，
有其它的客户端对该主机名查询就可能出现这种情况。 因为 DNS 服务中会缓存之前失败的的查询记录，即便在 Pod 运行之后
至少几秒种内依然可能会有这种情况出现。

如果想要在 Pod 创建后就能查询有，有以下几个选择:
- 直接查询 k8s API(例如通过 watch), 而不是信赖 DNS 查询
- 减少 k8s DNS 提供者(通常是修改 CoreDNS 的配置， 当前配置的缓存时间是 30s)的缓存时间

在 [限制](#limitations) 一节种提到，需要用户在负责创建用于 Pod 网络标识的 [Headless Service](/k8sDocs/docs/concepts/services-networking/service/#headless-services)。

以下为怎么设置 集群域，Service 命名， StatefulSet 命名，以及这命名对 StatefulSet 的 Pod 的 DNS 记录的影响:

Cluster Domain | Service (ns/name) | StatefulSet (ns/name)  | StatefulSet Domain              | Pod DNS                                      | Pod Hostname |
-------------- | ----------------- | ---------------------- | ------------------------------- | -------------------------------------------- | ------------ |
 cluster.local | default/nginx     | default/web            | nginx.default.svc.cluster.local | web-{0..N-1}.nginx.default.svc.cluster.local | web-{0..N-1} |
 cluster.local | foo/nginx         | foo/web                | nginx.foo.svc.cluster.local     | web-{0..N-1}.nginx.foo.svc.cluster.local     | web-{0..N-1} |
 kube.local    | foo/nginx         | foo/web                | nginx.foo.svc.kube.local        | web-{0..N-1}.nginx.foo.svc.kube.local        | web-{0..N-1} |

{{< note >}}
如果没有 [其它的配置](/k8sDocs/docs/concepts/services-networking/dns-pod-service/) 集群域会是 `cluster.local`
{{< /note >}}
<!--
### Stable Storage

Kubernetes creates one [PersistentVolume](/docs/concepts/storage/persistent-volumes/) for each
VolumeClaimTemplate. In the nginx example above, each Pod will receive a single PersistentVolume
with a StorageClass of `my-storage-class` and 1 Gib of provisioned storage. If no StorageClass
is specified, then the default StorageClass will be used. When a Pod is (re)scheduled
onto a node, its `volumeMounts` mount the PersistentVolumes associated with its
PersistentVolume Claims. Note that, the PersistentVolumes associated with the
Pods' PersistentVolume Claims are not deleted when the Pods, or StatefulSet are deleted.
This must be done manually.
 -->
### 稳定的存储

k8s 会依照 VolumeClaimTemplate 为每个 Pod 创建一个 [PersistentVolume](/k8sDocs/docs/concepts/storage/persistent-volumes/)
在上面的示例中， 每个 Pod 都会收到一个 StorageClass 为 `my-storage-class` 容量为 1G 的 PersistentVolume。
如果没有配置 StorageClass， 则会使用默认的 StorageClass。 当一个 Pod 被(重新)调度到一个节点时，它的 `volumeMounts`
挂载 PersistentVolumeClaim 中对应的 PersistentVolume。 要注意与 PersistentVolumeClaim 关联的 PersistentVolume
在 Pod 或 StatefulSet 删除时不会被删除。想要删除必须要手动删除才行。
<!--
### Pod Name Label

When the StatefulSet {{< glossary_tooltip term_id="controller" >}} creates a Pod,
it adds a label, `statefulset.kubernetes.io/pod-name`, that is set to the name of
the Pod. This label allows you to attach a Service to a specific Pod in
the StatefulSet.
 -->
### Pod 名称的标签

当 StatefulSet {{< glossary_tooltip term_id="controller" >}} 创建一个 Pod 进会向它塞一个标签
标签的键为 `statefulset.kubernetes.io/pod-name` 值为 Pod 的名称。这个标签可以让用户配置
Service 指向 StatefulSet 的特定 Pod。
<!--
## Deployment and Scaling Guarantees

* For a StatefulSet with N replicas, when Pods are being deployed, they are created sequentially, in order from {0..N-1}.
* When Pods are being deleted, they are terminated in reverse order, from {N-1..0}.
* Before a scaling operation is applied to a Pod, all of its predecessors must be Running and Ready.
* Before a Pod is terminated, all of its successors must be completely shutdown.

The StatefulSet should not specify a `pod.Spec.TerminationGracePeriodSeconds` of 0. This practice is unsafe and strongly discouraged. For further explanation, please refer to [force deleting StatefulSet Pods](/docs/tasks/run-application/force-delete-stateful-set-pod/).

When the nginx example above is created, three Pods will be deployed in the order
web-0, web-1, web-2. web-1 will not be deployed before web-0 is
[Running and Ready](/docs/concepts/workloads/pods/pod-lifecycle/), and web-2 will not be deployed until
web-1 is Running and Ready. If web-0 should fail, after web-1 is Running and Ready, but before
web-2 is launched, web-2 will not be launched until web-0 is successfully relaunched and
becomes Running and Ready.

If a user were to scale the deployed example by patching the StatefulSet such that
`replicas=1`, web-2 would be terminated first. web-1 would not be terminated until web-2
is fully shutdown and deleted. If web-0 were to fail after web-2 has been terminated and
is completely shutdown, but prior to web-1's termination, web-1 would not be terminated
until web-0 is Running and Ready.
 -->
## 部署和容量伸缩保证 {#deployment-and-scaling-guarantees}

- 对于一个有 N 个副本的 StatefulSet， 当 Pod 被部署时，是以 {0..N-1} 的顺序依次创建的
- 当 Pod 被删除除时，则是以 {N-1..0} 与创建相反的顺序终止
- 在一个扩容操作之前，它的前辈 Pod 必须是运行和就绪的
- 在一个 Pod 终止之前，它的后辈必须全部完全关闭

StatefulSet 不应该配置 `pod.Spec.TerminationGracePeriodSeconds` 为 0. 这个操作是不安全的，强烈建议不要这样搞。
更多解决请见 [强制删除 StatefulSet 的 Pod](/k8sDocs/tasks/run-application/force-delete-stateful-set-pod/)

在上面的例子中， 三个 Pod 会以 web-0, web-1, web-2 的顺序创建， web-1 不会在 web-0 [运行并就绪](/k8sDocs/docs/concepts/workloads/pods/pod-lifecycle/)
之前部署。 如果 web-0 在 web-1 运行并就绪后挂了，则 web-2 在 web-0 重新成功启动然后运行并就绪之前不会启动。

在上面的例子中，三个副本成功运行后，用户设置 `replicas=1`。 web-2 会先终止。 web-1 会在 web-2 关闭并删除后
才会开始终止。 如果 web-0 刚好在 web-2 完全关闭，但 web-1 还没终止之前挂了， 则 web-1 会在 web-0 重新运行并就绪后才会终止。
<!--
### Pod Management Policies
In Kubernetes 1.7 and later, StatefulSet allows you to relax its ordering guarantees while
preserving its uniqueness and identity guarantees via its `.spec.podManagementPolicy` field.

#### OrderedReady Pod Management

`OrderedReady` pod management is the default for StatefulSets. It implements the behavior
described [above](#deployment-and-scaling-guarantees).

#### Parallel Pod Management

`Parallel` pod management tells the StatefulSet controller to launch or
terminate all Pods in parallel, and to not wait for Pods to become Running
and Ready or completely terminated prior to launching or terminating another
Pod. This option only affects the behavior for scaling operations. Updates are not
affected.
 -->
### Pod 管理策略 {#pod-management-policies}

在 k8s 1.7+, StatefulSet 允许使用宽松的序号保证，通过 `.spec.podManagementPolicy` 字段来保证唯一性和标识保证。

#### OrderedReady Pod 管理

`OrderedReady` Pod 管理是 StatefulSet 的默认方式。它实现的行为有 [上面](#deployment-and-scaling-guarantees)讲过.

#### Parallel Pod 管理

`Parallel` Pod 管理，这种策略让 StatefulSet 控制器在启动或终止 Pod 并行进行， 不需要在启动的时候等待前辈运行并就绪，
也不需要在终止的时候等待后台完成终止并删除。 这个选项只影响容量伸缩行为。更新行为不受影响

<!--
## Update Strategies

In Kubernetes 1.7 and later, StatefulSet's `.spec.updateStrategy` field allows you to configure
and disable automated rolling updates for containers, labels, resource request/limits, and
annotations for the Pods in a StatefulSet.
 -->
## 更新策略

在 k8s 1.7+, StatefulSet `.spec.updateStrategy` 允许用户配置和禁用在对容器，标签，资源需求/限制，和标签的更新触发的自动滚动更新
<!--
### On Delete

The `OnDelete` update strategy implements the legacy (1.6 and prior) behavior. When a StatefulSet's
`.spec.updateStrategy.type` is set to `OnDelete`, the StatefulSet controller will not automatically
update the Pods in a StatefulSet. Users must manually delete Pods to cause the controller to
create new Pods that reflect modifications made to a StatefulSet's `.spec.template`.
 -->
### `OnDelete`

`OnDelete` 更新策略实现的是经典(`v1.6-`)行为. 当 StatefulSet 的 `.spec.updateStrategy.type` 设置为 `OnDelete`，
StatefulSet 就不会自动的更新 StatefulSet 中的 Pod。只有用户手动删除 Pod 后 控制器才会以 StatefulSet 中 `.spec.template`
创建应用修改后的新 Pod。
<!--
### Rolling Updates

The `RollingUpdate` update strategy implements automated, rolling update for the Pods in a
StatefulSet. It is the default strategy when `.spec.updateStrategy` is left unspecified. When a StatefulSet's `.spec.updateStrategy.type` is set to `RollingUpdate`, the
StatefulSet controller will delete and recreate each Pod in the StatefulSet. It will proceed
in the same order as Pod termination (from the largest ordinal to the smallest), updating
each Pod one at a time. It will wait until an updated Pod is Running and Ready prior to
updating its predecessor.
 -->
### `RollingUpdate` {#rolling-updates}

`RollingUpdate` 更新策略实现了自动，滚动更新 StatefulSet 中的 Pod。如果 `.spec.updateStrategy` 没有指定这，默认使用该策略。
当 StatefulSet 的 `.spec.updateStrategy.type` 设置为 `RollingUpdate`， StatefulSet 控制器会删除并重建 StatefulSet 中的每一个 Pod。
处理顺序与终止顺序相同(序号从大到小)，每次更新一个 Pod。 会等待上一个更新的 Pod 运行并就绪后，才会进行下一个。
<!--
#### Partitions

The `RollingUpdate` update strategy can be partitioned, by specifying a
`.spec.updateStrategy.rollingUpdate.partition`. If a partition is specified, all Pods with an
ordinal that is greater than or equal to the partition will be updated when the StatefulSet's
`.spec.template` is updated. All Pods with an ordinal that is less than the partition will not
be updated, and, even if they are deleted, they will be recreated at the previous version. If a
StatefulSet's `.spec.updateStrategy.rollingUpdate.partition` is greater than its `.spec.replicas`,
updates to its `.spec.template` will not be propagated to its Pods.
In most cases you will not need to use a partition, but they are useful if you want to stage an
update, roll out a canary, or perform a phased roll out.
 -->
#### 分区

`RollingUpdate` 更新策略可以分区， 通过 `.spec.updateStrategy.rollingUpdate.partition`的方式。
如果分区指定， 所有序号大于等于分区的 Pod 会在 StatefulSet `.spec.template` 更新时更新。
所有序号小于分区的 Pod 都不更新， 即便被删除，也会以之前的版本重建。 如果一个 StatefulSet 的 `.spec.updateStrategy.rollingUpdate.partition`
的值比它的 `.spec.replicas` 还大，则所有对  `.spec.template` 的修改都不会应用到 Pod 上。
大多数情况下，用户不需要使用到分区。但当用户想要分步更新，或金丝雀发布，或分阶段发布(phased roll out)时都相当有用

<!--
#### Forced Rollback

When using [Rolling Updates](#rolling-updates) with the default
[Pod Management Policy](#pod-management-policies) (`OrderedReady`),
it's possible to get into a broken state that requires manual intervention to repair.

If you update the Pod template to a configuration that never becomes Running and
Ready (for example, due to a bad binary or application-level configuration error),
StatefulSet will stop the rollout and wait.

In this state, it's not enough to revert the Pod template to a good configuration.
Due to a [known issue](https://github.com/kubernetes/kubernetes/issues/67250),
StatefulSet will continue to wait for the broken Pod to become Ready
(which never happens) before it will attempt to revert it back to the working
configuration.

After reverting the template, you must also delete any Pods that StatefulSet had
already attempted to run with the bad configuration.
StatefulSet will then begin to recreate the Pods using the reverted template.
 -->
#### 强制回退 {#forced-rollback}

当使用 [滚动更新](#rolling-updates) 配合 [Pod 管理策略](#pod-management-policies) (`OrderedReady`) 可能会造成故障，需要人工介入修复。

如果用户更新的 Pod 模板永远都不会进入运行和就绪状态(例如，因为二进制文件损坏或应用级配置错误)。 StatefulSet 就会停止发布并等待。

在这种状态下。就不能回退 Pod 模板到正常的配置。 因为一个 [已知的问题单](https://github.com/kubernetes/kubernetes/issues/67250)
StatefulSet 在尝试回滚到正常的配置之前会一直等着那个故障的 Pod 变成就绪状态(但永远不会)

在回退模板后，用户还需要删除所以 StatefulSet 使用错误的配置已经启动的 Pod 。StatefulSet 这时候才会以回退后的模板重新创建 Pod。

## {{% heading "whatsnext" %}}


* 实践 [部署一个有状态的应用](/k8sDocs/tutorials/stateful-application/basic-stateful-set/).
* 实践 [用StatefulSet 部署  Cassandra ](/k8sDocs/tutorials/stateful-application/cassandra/).
* 实践 [部署一个多副本的 有状态的应用](/k8sDocs/tasks/run-application/run-replicated-stateful-application/).
