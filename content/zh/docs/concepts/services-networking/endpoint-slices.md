---
title: EndpointSlice
weight: 50
date: 2020-09-17
publishdate: 2020-09-17
---

<!--
---
reviewers:
- freehan
title: EndpointSlices
content_type: concept
weight: 35
---
 -->

<!-- overview -->
<!--
{{< feature-state for_k8s_version="v1.17" state="beta" >}}

_EndpointSlices_ provide a simple way to track network endpoints within a
Kubernetes cluster. They offer a more scalable and extensible alternative to
Endpoints.
 -->

{{< feature-state for_k8s_version="v1.17" state="beta" >}}

_EndpointSlice_ 提供了一种简单的方式来在 k8s 集群跟踪网络端点。 它提供了比 Endpoint 拥有更
好的伸缩性和扩展性的替代方案


<!-- body -->
<!--
## Motivation

The Endpoints API has provided a simple and straightforward way of
tracking network endpoints in Kubernetes. Unfortunately as Kubernetes clusters
and {{< glossary_tooltip text="Services" term_id="service" >}} have grown to handle and
send more traffic to more backend Pods, limitations of that original API became
more visible.
Most notably, those included challenges with scaling to larger numbers of
network endpoints.

Since all network endpoints for a Service were stored in a single Endpoints
resource, those resources could get quite large. That affected the performance
of Kubernetes components (notably the master control plane) and resulted in
significant amounts of network traffic and processing when Endpoints changed.
EndpointSlices help you mitigate those issues as well as provide an extensible
platform for additional features such as topological routing.
 -->

## 动机

Endpoint 的 API 提供了一个简单的直接的方式来跟踪 k8s 中的网络端点。不幸的是当 k8s 集群和
{{< glossary_tooltip term_id="service" >}} 处理和发送更多的流量到更多的后端 Pod 时，
这个 API 的限制就越来越明显了.  特别是当集群中扩充到很大数量的网络端点时，这些挑战就越发明显。

当一个 Service 所有的网络端点都被存储在一个 Endpoint 资源时，这个资源就会变得很大。这会影响
到 k8s 组件(特别是控制中心)的性能， 并且的 Endpoint 发生变化时导致大量的网络流量和相关处理业务。
EndpointSlice 缓解这些问题的同时提供了一个可扩展的平台，这个平台上还有包括拓扑路由等特性。
<!--
## EndpointSlice resources {#endpointslice-resource}

In Kubernetes, an EndpointSlice contains references to a set of network
endpoints. The control plane automatically creates EndpointSlices
for any Kubernetes Service that has a {{< glossary_tooltip text="selector"
term_id="selector" >}} specified. These EndpointSlices include
references to all the Pods that match the Service selector. EndpointSlices group
network endpoints together by unique combinations of protocol, port number, and
Service name.  
The name of a EndpointSlice object must be a valid
[DNS subdomain name](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

As an example, here's a sample EndpointSlice resource for the `example`
Kubernetes Service.

```yaml
apiVersion: discovery.k8s.io/v1beta1
kind: EndpointSlice
metadata:
  name: example-abc
  labels:
    kubernetes.io/service-name: example
addressType: IPv4
ports:
  - name: http
    protocol: TCP
    port: 80
endpoints:
  - addresses:
      - "10.1.2.3"
    conditions:
      ready: true
    hostname: pod-1
    topology:
      kubernetes.io/hostname: node-1
      topology.kubernetes.io/zone: us-west2-a
```

By default, the control plane creates and manages EndpointSlices to have no
more than 100 endpoints each. You can configure this with the
`--max-endpoints-per-slice`
{{< glossary_tooltip text="kube-controller-manager" term_id="kube-controller-manager" >}}
flag, up to a maximum of 1000.

EndpointSlices can act as the source of truth for
{{< glossary_tooltip term_id="kube-proxy" text="kube-proxy" >}} when it comes to
how to route internal traffic. When enabled, they should provide a performance
improvement for services with large numbers of endpoints.
 -->

## EndpointSlice 资源 {#endpointslice-resource}

在 k8s 中， 一个 EndpointSlice 包含一组网络端点集合的引用。 控制中心会自动为任意包含
{{< glossary_tooltip term_id="selector" >}}
的 Service 创建 EndpointSlice。 这些 EndpointSlice 包含所有匹配该 Service 的 Pod 的引用。
EndpointSlice 通过协议，端口号，Service 名称的唯一组合将这些网络端点组织在一起。
EndpointSlice 对象的名称必须是一个
[DNS 子域名](/k8sDocs/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

下面的例子中，是一个叫 `example` 的 Service 的 EndpointSlice 资源的简单示例:

```yaml
apiVersion: discovery.k8s.io/v1beta1
kind: EndpointSlice
metadata:
  name: example-abc
  labels:
    kubernetes.io/service-name: example
addressType: IPv4
ports:
  - name: http
    protocol: TCP
    port: 80
endpoints:
  - addresses:
      - "10.1.2.3"
    conditions:
      ready: true
    hostname: pod-1
    topology:
      kubernetes.io/hostname: node-1
      topology.kubernetes.io/zone: us-west2-a
```

默认情况下， 控制中心创建和管理的 EndpointSlice 每个不超过 100 个网络端点。用户可以通过
{{< glossary_tooltip text="kube-controller-manager" term_id="kube-controller-manager" >}}
的 `--max-endpoints-per-slice` 参数配置，最高可以到 1000。

EndpointSlices 可以在路由内部流量时被认作
{{< glossary_tooltip term_id="kube-proxy" text="kube-proxy" >}} 的真实的来源。
当启用后，可以改善拥有大量网络端点的 Service 的性能。
<!--
### Address types

EndpointSlices support three address types:

* IPv4
* IPv6
* FQDN (Fully Qualified Domain Name)
 -->
### 地址类型

EndpointSlice 支持以下三种类型:

* IPv4
* IPv6
* FQDN (全限定名)
<!--
### Topology information {#topology}

Each endpoint within an EndpointSlice can contain relevant topology information.
This is used to indicate where an endpoint is, containing information about the
corresponding Node, zone, and region. When the values are available, the
control plane sets the following Topology labels for EndpointSlices:

* `kubernetes.io/hostname` - The name of the Node this endpoint is on.
* `topology.kubernetes.io/zone` - The zone this endpoint is in.
* `topology.kubernetes.io/region` - The region this endpoint is in.

The values of these labels are derived from resources associated with each
endpoint in a slice. The hostname label represents the value of the NodeName
field on the corresponding Pod. The zone and region labels represent the value
of the labels with the same names on the corresponding Node.
 -->
### 拓扑信息 {#topology}

EndpointSlice 中的每一个网络端点都可以有一个有意义的拓扑信息。 用于指示这个网络端点在哪，包含对应的
节点信息，分区信息和地区信息。 当这些值存在时，控制中心会为 EndpointSlice 打上如下标签:
- `kubernetes.io/hostname` - 这个网络端点所在的节点名称.
- `topology.kubernetes.io/zone` - 这个网络端点所在分区.
- `topology.kubernetes.io/region` - 这个网络端点所在地区.

这些标签的值源于 EndpointSlice 中的每个网络端点对应的资源。  hostname 标签表示 对应 Pod
的 NodeName 字段的值。 `zone` 和 `region` 标签与对应节点同一标签一致。
<!--
### Management

Most often, the control plane (specifically, the endpoint slice
{{< glossary_tooltip text="controller" term_id="controller" >}}) creates and
manages EndpointSlice objects. There are a variety of other use cases for
EndpointSlices, such as service mesh implementations, that could result in other
entities or controllers managing additional sets of EndpointSlices.

To ensure that multiple entities can manage EndpointSlices without interfering
with each other, Kubernetes defines the
{{< glossary_tooltip term_id="label" text="label" >}}
`endpointslice.kubernetes.io/managed-by`, which indicates the entity managing
an EndpointSlice.
The endpoint slice controller sets `endpointslice-controller.k8s.io` as the value
for this label on all EndpointSlices it manages. Other entities managing
EndpointSlices should also set a unique value for this label.
 -->

### 管理

大多数时候，控制中心(确切的说，EndpointSlice {{< glossary_tooltip term_id="controller" >}})
创建和管理 EndpointSlice 对象。 除此之外 EndpointSlice 还有其它多种应用场景， 例如服务网格的实现，
由此可以会导致其它的实体或控制顺管理额外的 EndpointSlice 集合。

为了能保证多个实体在管理 EndpointSlice 时不会相互影响， k8s 定义了 `endpointslice.kubernetes.io/managed-by`
{{< glossary_tooltip term_id="label" text="label" >}}，
这个标签用于指示是哪个实体在管理这个 EndpointSlice。 EndpointSlice 控制器会为所有它管理的
EndpointSlice 设置 `endpointslice-controller.k8s.io` 标签。 其它实体管理 EndpointSlice
也应当为该标签设置唯一的值。
<!--
### Ownership

In most use cases, EndpointSlices are owned by the Service that the endpoint
slice object tracks endpoints for. This ownership is indicated by an owner
reference on each EndpointSlice as well as a `kubernetes.io/service-name`
label that enables simple lookups of all EndpointSlices belonging to a Service.
 -->

### 所有权

在大多数情况下， EndpointSlice 的所有者是它跟踪的网络端点对应的 Service。 这个所有权会通过
EndpointSlice 上的 `kubernetes.io/service-name` 标签显示，标签的值为其所属 Service 的名称
<!--
### EndpointSlice mirroring

In some cases, applications create custom Endpoints resources. To ensure that
these applications do not need to concurrently write to both Endpoints and
EndpointSlice resources, the cluster's control plane mirrors most Endpoints
resources to corresponding EndpointSlices.

The control plane mirrors Endpoints resources unless:

* the Endpoints resource has a `endpointslice.kubernetes.io/skip-mirror` label
  set to `true`.
* the Endpoints resource has a `control-plane.alpha.kubernetes.io/leader`
  annotation.
* the corresponding Service resource does not exist.
* the corresponding Service resource has a non-nil selector.

Individual Endpoints resources may translate into multiple EndpointSlices. This
will occur if an Endpoints resource has multiple subsets or includes endpoints
with multiple IP families (IPv4 and IPv6). A maximum of 1000 addresses per
subset will be mirrored to EndpointSlices.
 -->

### EndpointSlice 镜像


在某些情况下，应用会创建自定义的 Endpoint。 为了保证这些应用不会同时写入到 Endpoint 和 EndpointSlice
资源， 集群控制中心会为大多数 Endpoint 创建对应的 EndpointSlice。

以下情况控制中心不会镜像 Endpoint:

- Endpoint 资源有 `endpointslice.kubernetes.io/skip-mirror` 标签，且值为 `true`
- Endpoint 资源有 `control-plane.alpha.kubernetes.io/leader` 注解。
- 对应的 Service 资源不存在
- 对应的 Service 资源有非空选择器

一个 Endpoint 可能会被转化为多个 EndpointSlice。发生这种情况的原因有可能是 Endpoint 资源
包含多个子网或包含的网络端点同时包含 IPv4 和 IPv6。 每个子网最多可以把 1000 个地址镜像到一个 EndpointSlice
<!--
### Distribution of EndpointSlices

Each EndpointSlice has a set of ports that applies to all endpoints within the
resource. When named ports are used for a Service, Pods may end up with
different target port numbers for the same named port, requiring different
EndpointSlices. This is similar to the logic behind how subsets are grouped
with Endpoints.

The control plane tries to fill EndpointSlices as full as possible, but does not
actively rebalance them. The logic is fairly straightforward:

1. Iterate through existing EndpointSlices, remove endpoints that are no longer
   desired and update matching endpoints that have changed.
2. Iterate through EndpointSlices that have been modified in the first step and
   fill them up with any new endpoints needed.
3. If there's still new endpoints left to add, try to fit them into a previously
   unchanged slice and/or create new ones.

Importantly, the third step prioritizes limiting EndpointSlice updates over a
perfectly full distribution of EndpointSlices. As an example, if there are 10
new endpoints to add and 2 EndpointSlices with room for 5 more endpoints each,
this approach will create a new EndpointSlice instead of filling up the 2
existing EndpointSlices. In other words, a single EndpointSlice creation is
preferrable to multiple EndpointSlice updates.

With kube-proxy running on each Node and watching EndpointSlices, every change
to an EndpointSlice becomes relatively expensive since it will be transmitted to
every Node in the cluster. This approach is intended to limit the number of
changes that need to be sent to every Node, even if it may result with multiple
EndpointSlices that are not full.

In practice, this less than ideal distribution should be rare. Most changes
processed by the EndpointSlice controller will be small enough to fit in an
existing EndpointSlice, and if not, a new EndpointSlice is likely going to be
necessary soon anyway. Rolling updates of Deployments also provide a natural
repacking of EndpointSlices with all Pods and their corresponding endpoints
getting replaced.
 -->

### EndpointSlice 分布

每个 EndpointSlice 都有一系列端口，这些端口会被应用到该资源内的所有的网络端点上。当一个
Service 使用命名端口时， Pod 最终可以会有同一个名称的端口的目标端口不一样的情况，这样就需要不
同的 EndpointSlice。 这与子网是怎么对 Endpoint 分组的逻辑是类似的。

控制中心会尽量将 EndpointSlice 装满，但不会主动地重新平衡它们。 其中逻辑是相同简单的:

1. 迭代所有存在的 EndpointSlice，称除不需要的网络端点，更新发生变化的网络端点
2. 迭代每一步中被修改的 EndpointSlice ，添加所有需要的新的网络端点
3. 如果还有新的网络端点需要添加，尝试添加到之间没变更的 EndpointSlice 或者(同时)创建一个新的。

重要的是， 在第三步的优先级会限制 EndpointSlice 更新到一个完美分布。 例如，如果有 10 个新增的
网络端点要添加到两个 EndpointSlice，并且这两个 EndpointSlice 都还有 5 个空位，这种方式会
创建一个新的 EndpointSlice， 而不是装满这两个已经存在的 EndpointSlice。 换句话说，创建一个新的
EndpointSlice 优先于更新两个 EndpointSlice。

当 kube-proxy 运行在每个节点上并监听 EndpointSlice， 每一次对 EndpointSlice 修改都会变得
相对来多代价比较高的，因为每个更新都会传达到信念中的每一个节点上。这种方式旨在限制发送到每个节点
的变更次数，尽管它会导致产生多个没有被装满的 EndpointSlice

在实践中，这种不太理想分配应该是比较少见的。 大多数由 EndpointSlice 处理的变量应该都足够小到适应
已经存在的 EndpointSlice， 如果没，也就是创建一个很快就也必须要创建的新的 EndpointSlice。
Deployment 的滚动更新也提供了一个自然的  EndpointSlice 重新打包，因为所有的 Pod 和它们对
应的网络端点也是被替换。
<!--
### Duplicate endpoints

Due to the nature of EndpointSlice changes, endpoints may be represented in more
than one EndpointSlice at the same time. This naturally occurs as changes to
different EndpointSlice objects can arrive at the Kubernetes client watch/cache
at different times. Implementations using EndpointSlice must be able to have the
endpoint appear in more than one slice. A reference implementation of how to
perform endpoint deduplication can be found in the `EndpointSliceCache`
implementation in `kube-proxy`.
 -->

### 重复的网络端点

由于 EndpointSlice 的自然变更，网络端点可能同时存在于不同的 EndpointSlice 中。 这些自然地
发生在对不同 EndpointSlice 对象的变更，可能在不同的时间到到 k8s 客户端的监听或缓存。使用
EndpointSlice 的实现必须要能处理这种同一个网络端点出现在多个 EndpointSlice 的情况。
怎么处理重复网络端点的实现参考可以在 `kube-proxy` 中的 `EndpointSliceCache` 实现中找到。

## {{% heading "whatsnext" %}}

* 实践 [开启 EndpointSlices](/k8sDocs/tasks/administer-cluster/enabling-endpointslices)
* 概念 [通过 Service 连接应用](/k8sDocs/docs/concepts/services-networking/connect-applications-service/)
