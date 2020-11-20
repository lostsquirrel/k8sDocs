---
title: Service
weight: 10
date: 2020-06-30
publishdate: 2020-09-09
---
<!--
---
reviewers:
- bprashanth
title: Service
feature:
  title: Service discovery and load balancing
  description: >
    No need to modify your application to use an unfamiliar service discovery mechanism. Kubernetes gives Pods their own IP addresses and a single DNS name for a set of Pods, and can load-balance across them.

content_type: concept
weight: 10
---
 -->

<!-- overview -->
<!--
{{< glossary_definition term_id="service" length="short" >}}

With Kubernetes you don't need to modify your application to use an unfamiliar service discovery mechanism.
Kubernetes gives Pods their own IP addresses and a single DNS name for a set of Pods,
and can load-balance across them.
 -->

{{< glossary_definition term_id="service" length="short" >}}

在使用 k8s 时并不需要修改应用来使用不熟悉的服务发现机制。 k8s 为 Pod 提供了自己的 IP 地址和
也为 Pod 集合提供单个 DNS 名称，并为其提供负载均衡。

<!-- body -->
<!--
## Motivation

Kubernetes {{< glossary_tooltip term_id="pod" text="Pods" >}} are mortal.
They are born and when they die, they are not resurrected.
If you use a {{< glossary_tooltip term_id="deployment" >}} to run your app,
it can create and destroy Pods dynamically.

Each Pod gets its own IP address, however in a Deployment, the set of Pods
running in one moment in time could be different from
the set of Pods running that application a moment later.

This leads to a problem: if some set of Pods (call them "backends") provides
functionality to other Pods (call them "frontends") inside your cluster,
how do the frontends find out and keep track of which IP address to connect
to, so that the frontend can use the backend part of the workload?

Enter _Services_.
 -->
## 动机

k8s 的 {{< glossary_tooltip term_id="pod" text="Pod" >}} 是会挂掉的。它们出生然后挂掉，
它们挂了以后就不能再重生了。 如果使用
{{< glossary_tooltip term_id="deployment" >}}  
来运行应用，则它会动态地创建和销毁 Pod。

每个 Pod 都会有一个自己的 IP 地址， 但是在 Deployment 中，它所管理的 Pod 在这一个时间点和
另一个时间点可能是不一样的。

这就会导致一个问题: 如果在集群中有一组 Pod (称作 "后端")为另一组 Pod (称作 "前端")提供功能，
那么前端的 Pod 怎么能一直找到后端的连接 IP 地址，然后使用后端作为工作负载呢。

_Services_ 就闪亮登场了.
<!--
## Service resources {#service-resource}

In Kubernetes, a Service is an abstraction which defines a logical set of Pods
and a policy by which to access them (sometimes this pattern is called
a micro-service). The set of Pods targeted by a Service is usually determined
by a {{< glossary_tooltip text="selector" term_id="selector" >}}
(see [below](#services-without-selectors) for why you might want a Service
_without_ a selector).

For example, consider a stateless image-processing backend which is running with
3 replicas.  Those replicas are fungible&mdash;frontends do not care which backend
they use.  While the actual Pods that compose the backend set may change, the
frontend clients should not need to be aware of that, nor should they need to keep
track of the set of backends themselves.

The Service abstraction enables this decoupling.

### Cloud-native service discovery

If you're able to use Kubernetes APIs for service discovery in your application,
you can query the {{< glossary_tooltip text="API server" term_id="kube-apiserver" >}}
for Endpoints, that get updated whenever the set of Pods in a Service changes.

For non-native applications, Kubernetes offers ways to place a network port or load
balancer in between your application and the backend Pods.

 -->
## Service 资源 {#service-resource}

在 k8s 中， Service 是一个抽象概念，它定义的是逻辑组上的一组 Pod 与访问它们的策略(有时候这种模式也被称为 微服务)。
Service 所指向的是哪些 Pod 通常是由  
{{< glossary_tooltip term_id="selector" >}}
决定的([下面](#services-without-selectors)还介绍了可能 _不需要_ 选择器的 Service).

例如，假如有一个无状的图片处理后端，有3个副本在运行。 这些副本是可替代的 &mdash; 前端不关心它们
用的是哪个后台。 当组成后端的 Pod 可能发生变化， 但前端的客户端应该不能感知到，它们也不需要自己
来跟踪后端的具体成员。

Service 的抽象实现了这样的解耦。

### 云原生服务发现

如果能够在应用中使用 k8s API 来实现服务发现， 可以通过
{{< glossary_tooltip text="API server" term_id="kube-apiserver" >}}
查询 Endpoint, 通过这种方式可以实时更新到 Service 的 Pod 变更。

对于非原生应用， k8s 为应用与后端 Pod 之间通信提供了网络端口或负载均衡等方式。
<!--
## Defining a Service

A Service in Kubernetes is a REST object, similar to a Pod.  Like all of the
REST objects, you can `POST` a Service definition to the API server to create
a new instance.
The name of a Service object must be a valid
[DNS label name](/docs/concepts/overview/working-with-objects/names#dns-label-names).

For example, suppose you have a set of Pods that each listen on TCP port 9376
and carry a label `app=MyApp`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

This specification creates a new Service object named "my-service", which
targets TCP port 9376 on any Pod with the `app=MyApp` label.

Kubernetes assigns this Service an IP address (sometimes called the "cluster IP"),
which is used by the Service proxies
(see [Virtual IPs and service proxies](#virtual-ips-and-service-proxies) below).

The controller for the Service selector continuously scans for Pods that
match its selector, and then POSTs any updates to an Endpoint object
also named "my-service".

{{< note >}}
A Service can map _any_ incoming `port` to a `targetPort`. By default and
for convenience, the `targetPort` is set to the same value as the `port`
field.
{{< /note >}}

Port definitions in Pods have names, and you can reference these names in the
`targetPort` attribute of a Service. This works even if there is a mixture
of Pods in the Service using a single configured name, with the same network
protocol available via different port numbers.
This offers a lot of flexibility for deploying and evolving your Services.
For example, you can change the port numbers that Pods expose in the next
version of your backend software, without breaking clients.

The default protocol for Services is TCP; you can also use any other
[supported protocol](#protocol-support).

As many Services need to expose more than one port, Kubernetes supports multiple
port definitions on a Service object.
Each port definition can have the same `protocol`, or a different one.
 -->
## Service 定义

Service 在 k8s 中是一个 `REST` 对象， 与 Pod 类似。 与其它所有 `REST` 对象一样，
可以通过 `POST` 请求将 Service 定义发送到 api-server 来创建一个新的实例。
Service 的名称必须是一个有效的
[DNS 标签名称](/k8sDocs/docs/concepts/overview/working-with-objects/names#dns-label-names).

例如， 假如有一组 Pod， 每个 Pod 监听的端口都是 `9376`， 都打着一个标签为 `app=MyApp`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

上面的配置定义了一个 Service 对象，名字叫 "my-service"， 指向所有 TCP 端口为 `9376`， 带有
`app=MyApp` 标签的 Pod。

k8s 会为 Service 分配一个 IP 地址(有时称为 "集群IP (cluster IP)"), 这个 IP 地址会被
Service 代理使用。
(见下面的 [虚拟IP 和 service 代理](#virtual-ips-and-service-proxies)).

Service 选择器的控制器会持续扫描匹配其选择器的 Pod，然后把这些变更以 POST 请求方式发送到一个
叫 "my-service" 的 Endpoint 对象。

{{< note >}}
Service 可以映射 _任意_ 输入 `port` 到 `targetPort`。 默认情况和为了方便， `targetPort`
会设置与 `port` 字段相同的值。
{{< /note >}}

Pod 中的 Port 定义是有名字的， 这个名字可以在 Service `targetPort` 属性上引用。
这种方式甚至可以用在当 Service 中使用同一个配置名称的不同 Pod，使用相同的网络协议，不同的端口
这个特性为 Service 的部署和演进提供了很高的灵活性。
例如， 用户可以修改用于下一版后端软件 Pod 暴露的端口，而不影响客户端。

Services 默认协议为 TCP; 也可以使用其它[支持的协议](#protocol-support)

当许多的服务需要显露不止一个端口， k8s 支持在一个 Service 对象上定义多个端口。 每个端口定义
可以使用同样的 协议(`protocol`), 也可以使用不同的.

### 无标签选择器 Service

Service 最常见的用户就是作为 k8s Pod 的入口， 但它也可以作为其它类型的后端的抽象入口。
例如:

- 在生产环境使用的集群外的数库，但是在测试环境用的是内部的数据。
- 想要将 Service 集群中另一个命名空间中的 Service 或另一个集群的服务。
- 在迁移工作负载到 k8s 时，为了评估是否可行，先使用一部分后端服务在 k8s 中。

In any of these scenarios you can define a Service _without_ a Pod selector.
For example:
在以上的任意一种场景中都需要定义 _没有_ Pod 选择器的 Service。
例如:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

因为这些 Service 没有选择器，所以对象的 Endpoint 也 *不会* 自动创建。 可以通过手动创建 Endpoint
对象的方式将 Service 映射到实际运行的网络地址和端口。

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service
subsets:
  - addresses:
      - ip: 192.0.2.42
    ports:
      - port: 9376
```

Endpoint 对象的名称必以是一个有效的
[DNS 子域名](/k8sDocs/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).
{{< note >}}
Endpoint 对象所用的 IP _必须不能_ 是: 回环地址 (127.0.0.0/8 IPv4, ::1/128 IPv6)，或
链路本地(link-local) (169.254.0.0/16 和 224.0.0.0/24  IPv4, fe80::/64 IPv6).

Endpoint IP 地址也不能是其它 k8s Service 的集群 IP， 因为
{{< glossary_tooltip term_id="kube-proxy" >}}
不支持将虚拟IP作为目的地址。
{{< /note >}}

访问无选择器的 Service 与有选择器的 Service 是一样的。在上面的例子中， 流量会路由到YAML定义中
唯一的 Endpoint `192.0.2.42:9376` (TCP)

一个 ExternalName Service 是 Service 中的一种特殊情景， 它没有选择器而是 DNS 名称。
更多信息见本文下面的 [ExternalName](#externalname)。
<!--
### EndpointSlices
{{< feature-state for_k8s_version="v1.17" state="beta" >}}

EndpointSlices are an API resource that can provide a more scalable alternative
to Endpoints. Although conceptually quite similar to Endpoints, EndpointSlices
allow for distributing network endpoints across multiple resources. By default,
an EndpointSlice is considered "full" once it reaches 100 endpoints, at which
point additional EndpointSlices will be created to store any additional
endpoints.

EndpointSlices provide additional attributes and functionality which is
described in detail in [EndpointSlices](/docs/concepts/services-networking/endpoint-slices/).
 -->
### EndpointSlices

{{< feature-state for_k8s_version="v1.17" state="beta" >}}

EndpointSlice 是一种可比 Endpoint 提供更新好伸缩性替代方案的 API 资源。尽管在概念与 Endpoint
很相近， EndpointSlice 允许对铆中资源的网络末端进行分发. 默认情况下当一个 EndpointSlice
的网络末端数量达到 100 时就认为是 "满了", 这时候就会创建新的 EndpointSlice 来存储更多的网络末端。

更多 EndpointSlice 提供的属性和功能请见
[EndpointSlices](/k8sDocs/docs/concepts/services-networking/endpoint-slices/).
<!--
### Application protocol

{{< feature-state for_k8s_version="v1.19" state="beta" >}}

The AppProtocol field provides a way to specify an application protocol to be
used for each Service port. The value of this field is mirrored by corresponding
Endpoints and EndpointSlice resources.
-->

### 应用协议

{{< feature-state for_k8s_version="v1.19" state="beta" >}}

`AppProtocol` 字段提供了指定每个 Service 端口对应的应用协议的一种方式。
这个字段是对 Endpoint 和 EndpointSlice 对应字段的镜像。
<!--
## Virtual IPs and service proxies

Every node in a Kubernetes cluster runs a `kube-proxy`. `kube-proxy` is
responsible for implementing a form of virtual IP for `Services` of type other
than [`ExternalName`](#externalname).
-->

## 虚拟 IP 和 Service 代理

Every node in a Kubernetes cluster runs a `kube-proxy`. `kube-proxy` is
responsible for implementing a form of virtual IP for `Services` of type other
than [`ExternalName`](#externalname).

k8s 集群中的每一个节点上都运行了 `kube-proxy`， `kube-proxy` 负责实现除了
[`ExternalName`](#externalname)
外其它类型的 `Services` 虚拟 IP 的实现方式
<!--
### Why not use round-robin DNS?

A question that pops up every now and then is why Kubernetes relies on
proxying to forward inbound traffic to backends. What about other
approaches? For example, would it be possible to configure DNS records that
have multiple A values (or AAAA for IPv6), and rely on round-robin name
resolution?

There are a few reasons for using proxying for Services:

 * There is a long history of DNS implementations not respecting record TTLs,
   and caching the results of name lookups after they should have expired.
 * Some apps do DNS lookups only once and cache the results indefinitely.
 * Even if apps and libraries did proper re-resolution, the low or zero TTLs
   on the DNS records could impose a high load on DNS that then becomes
   difficult to manage.
 -->
### 为嘛不用 round-robin DNS ?

一个时不时被提起的问题就是为啥 k8s 信赖于代理来转发入站流量到后端。 为啥不用其它的方式？
例如，有没有可能通过配置包含多个 A 值的 DNS 记录(IPv6 用 AAAA)， 然后通过轮询域名解析结果。
以下为 Service 使用代理的几个原因:
- DNS 实现不遵循记录的 TTL有长久的历史， 并且在结果过期后继续使用缓存查询结果
- 有些应用一次查询 DNS 后永远使用缓存的查询结果
- 即便每个应用规范地来查 DNS， 但是 DNS 记录上的 TTL 的值很低或为0 会导致 DNS 的负载很高，并且变得难于管理
<!--
### User space proxy mode {#proxy-mode-userspace}

In this mode, kube-proxy watches the Kubernetes master for the addition and
removal of Service and Endpoint objects. For each Service it opens a
port (randomly chosen) on the local node.  Any connections to this "proxy port"
are
proxied to one of the Service's backend Pods (as reported via
Endpoints). kube-proxy takes the `SessionAffinity` setting of the Service into
account when deciding which backend Pod to use.

Lastly, the user-space proxy installs iptables rules which capture traffic to
the Service's `clusterIP` (which is virtual) and `port`. The rules
redirect that traffic to the proxy port which proxies the backend Pod.

By default, kube-proxy in userspace mode chooses a backend via a round-robin algorithm.

![Services overview diagram for userspace proxy](/images/docs/services-userspace-overview.svg)
 -->
### user-space 代理模式 {#proxy-mode-userspace}

在这种模式下， kube-proxy 监听 k8s 主控节点上 Service 和 Endpoint 对象的添加和删除。
对每一个 Service 它会在本地节点打开一个端口(随机选择)。任意一个连接到该 "代理端口"的流量都会代理
到 Service 后端 Pod(由 Endpoint 报告) 中的一个上。 kube-proxy 使用 Service 的 `SessionAffinity`
设置为决定使用哪个后端 Pod。

最后 user-space 代理会添加相应的 iptables 规则将捕获到 Service `clusterIP`(是一个虚拟IP) 和 `port` 的流量
然后这些规则将这些流量重定向到刚提供的会代理到后端 Pod 的代理端口上。

默认情况下， kube-proxy 在使用 user-space 代理模式是使用轮询算法选择后端的 Pod。
![Services overview diagram for userspace proxy](/k8sDocs/images/docs/services-userspace-overview.svg)
<!--
### `iptables` proxy mode {#proxy-mode-iptables}

In this mode, kube-proxy watches the Kubernetes control plane for the addition and
removal of Service and Endpoint objects. For each Service, it installs
iptables rules, which capture traffic to the Service's `clusterIP` and `port`,
and redirect that traffic to one of the Service's
backend sets.  For each Endpoint object, it installs iptables rules which
select a backend Pod.

By default, kube-proxy in iptables mode chooses a backend at random.

Using iptables to handle traffic has a lower system overhead, because traffic
is handled by Linux netfilter without the need to switch between userspace and the
kernel space. This approach is also likely to be more reliable.

If kube-proxy is running in iptables mode and the first Pod that's selected
does not respond, the connection fails. This is different from userspace
mode: in that scenario, kube-proxy would detect that the connection to the first
Pod had failed and would automatically retry with a different backend Pod.

You can use Pod [readiness probes](/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)
to verify that backend Pods are working OK, so that kube-proxy in iptables mode
only sees backends that test out as healthy. Doing this means you avoid
having traffic sent via kube-proxy to a Pod that's known to have failed.

![Services overview diagram for iptables proxy](/k8sDocs/images/docs/services-iptables-overview.svg)
-->

### `iptables` 代理模式 {#proxy-mode-iptables}

在这种模式下， kube-proxy 监听 k8s 主控节点上 Service 和 Endpoint 对象的添加和删除。
对于每个 Service， 会添加一个 iptables 规则， 这个规则会捕获所有目标是 Service 的 `clusterIP` 和 `port`
的流量并将其重定向到 Service 后端 Pod 中的一个上。 对于每个 Endpoint 也会添加一个 iptables
规则，用来选择后端的 Pod。

默认情况下，kube-proxy 在 `iptables` 代理模式下，随机选择后端 Pod。

使用 iptables 来处理流量系统开销更低， 因为流量由 Linux netfilter处理，而不需要在 userspace 和
kernelspace 之间来回切换。 这种方式也可能更加可靠。

如果 kube-proxy 使用的 iptables 模式， 如果选择的第一个 Pod 没有响应，这个连接就失败了。
这与 userspace 模式不同： 在这种场景下， kube-proxy 会检测到第一个 Pod 的连接失败了，会自动
地尝试其它的后端 Pod。

可以使用
[就绪探针](/k8sDocs/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)
来验证后端的 Pod 是在正常工作的， 因此 kube-proxy 在 iptables 模式下只会看到检测结果为健康的
后端 Pod。 这么做的意义在于避免了通过 kube-proxy 将流量发送到已经知道挂了的 Pod 上。
![Services overview diagram for iptables proxy](/k8sDocs/images/docs/services-iptables-overview.svg)
<!--
### IPVS proxy mode {#proxy-mode-ipvs}

{{< feature-state for_k8s_version="v1.11" state="stable" >}}

In `ipvs` mode, kube-proxy watches Kubernetes Services and Endpoints,
calls `netlink` interface to create IPVS rules accordingly and synchronizes
IPVS rules with Kubernetes Services and Endpoints periodically.
This control loop ensures that IPVS status matches the desired
state.
When accessing a Service, IPVS directs traffic to one of the backend Pods.

The IPVS proxy mode is based on netfilter hook function that is similar to
iptables mode, but uses a hash table as the underlying data structure and works
in the kernel space.
That means kube-proxy in IPVS mode redirects traffic with lower latency than
kube-proxy in iptables mode, with much better performance when synchronising
proxy rules. Compared to the other proxy modes, IPVS mode also supports a
higher throughput of network traffic.

IPVS provides more options for balancing traffic to backend Pods;
these are:

- `rr`: round-robin
- `lc`: least connection (smallest number of open connections)
- `dh`: destination hashing
- `sh`: source hashing
- `sed`: shortest expected delay
- `nq`: never queue

{{< note >}}
To run kube-proxy in IPVS mode, you must make IPVS available on
the node before starting kube-proxy.

When kube-proxy starts in IPVS proxy mode, it verifies whether IPVS
kernel modules are available. If the IPVS kernel modules are not detected, then kube-proxy
falls back to running in iptables proxy mode.
{{< /note >}}

![Services overview diagram for IPVS proxy](/k8sDocs/images/docs/services-ipvs-overview.svg)

In these proxy models, the traffic bound for the Service's IP:Port is
proxied to an appropriate backend without the clients knowing anything
about Kubernetes or Services or Pods.

If you want to make sure that connections from a particular client
are passed to the same Pod each time, you can select the session affinity based
on the client's IP addresses by setting `service.spec.sessionAffinity` to "ClientIP"
(the default is "None").
You can also set the maximum session sticky time by setting
`service.spec.sessionAffinityConfig.clientIP.timeoutSeconds` appropriately.
(the default value is 10800, which works out to be 3 hours).
 -->
### IPVS 代理 {#proxy-mode-ipvs}

{{< feature-state for_k8s_version="v1.11" state="stable" >}}

在 `ipvs` 模式下， kube-proxy 监听 k8s Services 和 Endpoint,
调用 `netlink` 接口来创建 IPVS 规则， 并定时根据 k8s Services 和 Endpoint 更新 IPVS 规则。
这个控制回环确保 IPVS 的状态与期望状态一至。 当访问一个 Service 时， IPVS 重定向流量到
后端 Pod 中的一个上。

IPVS 代理模式基于 netfilter 钩子功能，与 iptables 类似， 但底层使用的数据结构是一个哈希表
并且是在内核空间中工作的。
也就是 kube-proxy 在 IPVS 模式下， 重定向流量会比 iptables 模式有更低的延迟，在同步代理
规则时也会有更好的恨不能。 与其它的代理模式相比， IPVS 模式也支持更高吞吐量的网络流量。

IPVS 还提供了更多到后端 Pod 的负载均衡选择；
具体如下:

- `rr`: 轮询
- `lc`: 最少连接 (打开连接数最小的)
- `dh`: 目标哈希
- `sh`: 源哈希
- `sed`: 最短期望延迟 (shortest expected delay)
- `nq`: 无须队列等待 (never queue)

{{< note >}}
要让 kube-proxy 以 IPVS 运行，必须要在 kube-proxy 启动之前让 IPVS 在节点上是可用的。

当 kube-proxy 以 IPVS 代理模式启动时， 会检测 IPVS 内核模块是否可用。 如 IPVS 内核模块没有检测到，
则 kube-proxy 会回退以 iptables 模式运行。
{{< /note >}}

![Services overview diagram for IPVS proxy](/k8sDocs/images/docs/services-ipvs-overview.svg)

In these proxy models, the traffic bound for the Service's IP:Port is
proxied to an appropriate backend without the clients knowing anything
about Kubernetes or Services or Pods.
在使用这种代理模式时， 访问 Service IP:Port 的流量被代理到适当的后端时，客户端不会感知到
k8s 或 Service 或 Pod 这些的存在。

如果需要保证一个特定客户端的连接每次都要转发到同一个 Pod 上面， 可能设置
`service.spec.sessionAffinity` 为 "ClientIP" (默认为 "None") 来选择基于客户端IP的会话亲和性(session affinity).
也可以设置基于时间的会话黏性，为 `service.spec.sessionAffinityConfig.clientIP.timeoutSeconds`
设置一个合适的值。 (默认值为 10800， 也就是 3 个小时)
<!--
## Multi-Port Services

For some Services, you need to expose more than one port.
Kubernetes lets you configure multiple port definitions on a Service object.
When using multiple ports for a Service, you must give all of your ports names
so that these are unambiguous.
For example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
    - name: https
      protocol: TCP
      port: 443
      targetPort: 9377
```

{{< note >}}
As with Kubernetes {{< glossary_tooltip term_id="name" text="names">}} in general, names for ports
must only contain lowercase alphanumeric characters and `-`. Port names must
also start and end with an alphanumeric character.

For example, the names `123-abc` and `web` are valid, but `123_abc` and `-web` are not.
{{< /note >}}
 -->

## 多端口的 Service

对于有些 Service 需要显露多于一个端口， k8s 允许用户在 Service 对象上定义多个端口。
当在 Service 上使用多个端口时，必须要给所有的端口配置名字，这样才便于区分。
例如:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
    - name: https
      protocol: TCP
      port: 443
      targetPort: 9377
```

{{< note >}}
依照 k8s 通用 {{< glossary_tooltip term_id="name">}}， 端口的名称只能包含小写字母，数字，和 中划线(`-`)
且端口名只能以字母数字开始和结尾。

例如， `123-abc` 和 `web` 是有效的，`123_abc` 和 `-web` 是无效的
{{< /note >}}
<!--
## Choosing your own IP address

You can specify your own cluster IP address as part of a `Service` creation
request.  To do this, set the `.spec.clusterIP` field. For example, if you
already have an existing DNS entry that you wish to reuse, or legacy systems
that are configured for a specific IP address and difficult to re-configure.

The IP address that you choose must be a valid IPv4 or IPv6 address from within the
`service-cluster-ip-range` CIDR range that is configured for the API server.
If you try to create a Service with an invalid clusterIP address value, the API
server will return a 422 HTTP status code to indicate that there's a problem.

-->


## 选择自己的 IP 地址

在创建 `Service` 的时候可以通过设置 `.spec.clusterIP` 字段指定自己的集群IP地址。
例如， 希望复用已经存在的 DNS 记录，或者一个已经设置了IP 然后很难重新配置的旧系统。

选择设置的IP 必须要是 api-server 上配置的 `service-cluster-ip-range` CIDR 范围内有效的
IPv4 或 IPv6 地址。 如果尝试使用一个无效的集群IP地址，api-server 会返回一个 422 的 HTTP
状态码，表示配置有问题

<!--
## Discovering services

Kubernetes supports 2 primary modes of finding a Service - environment
variables and DNS.
 -->
## Service 查找

k8s 主要支持 2 种查找一个 Service的方式: 环境变量 和 DNS
<!--
### Environment variables

When a Pod is run on a Node, the kubelet adds a set of environment variables
for each active Service.  It supports both [Docker links
compatible](https://docs.docker.com/userguide/dockerlinks/) variables (see
[makeLinkVariables](https://releases.k8s.io/{{< param "githubbranch" >}}/pkg/kubelet/envvars/envvars.go#L49))
and simpler `{SVCNAME}_SERVICE_HOST` and `{SVCNAME}_SERVICE_PORT` variables,
where the Service name is upper-cased and dashes are converted to underscores.

For example, the Service `"redis-master"` which exposes TCP port 6379 and has been
allocated cluster IP address 10.0.0.11, produces the following environment
variables:

```shell
REDIS_MASTER_SERVICE_HOST=10.0.0.11
REDIS_MASTER_SERVICE_PORT=6379
REDIS_MASTER_PORT=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP_PROTO=tcp
REDIS_MASTER_PORT_6379_TCP_PORT=6379
REDIS_MASTER_PORT_6379_TCP_ADDR=10.0.0.11
```

{{< note >}}
When you have a Pod that needs to access a Service, and you are using
the environment variable method to publish the port and cluster IP to the client
Pods, you must create the Service *before* the client Pods come into existence.
Otherwise, those client Pods won't have their environment variables populated.

If you only use DNS to discover the cluster IP for a Service, you don't need to
worry about this ordering issue.
{{< /note >}}
-->

### 环境变量

当一个 Pod 运行到一个节点时， kubelet 会把每个活跃的 Service  作为环境变量添加到 Pod 中。
它支持
[Docker 连接兼容](https://docs.docker.com/userguide/dockerlinks/)
的变量
(见 [makeLinkVariables](https://releases.k8s.io/{{< param "githubbranch" >}}/pkg/kubelet/envvars/envvars.go#L49))
和简单些的 `{SVCNAME}_SERVICE_HOST` 和 `{SVCNAME}_SERVICE_PORT` 变量，其中 Service
的名称为大写，中划线会被转化为下划线。

例如， 一个叫 `"redis-master"` 的 Service， 暴露的端口是 TCP `6379`， 分配的集群IP地址为
`10.0.0.11`， 就会产生如下环境变量:
```shell
REDIS_MASTER_SERVICE_HOST=10.0.0.11
REDIS_MASTER_SERVICE_PORT=6379
REDIS_MASTER_PORT=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP_PROTO=tcp
REDIS_MASTER_PORT_6379_TCP_PORT=6379
REDIS_MASTER_PORT_6379_TCP_ADDR=10.0.0.11
```

{{< note >}}
当有一个 Pod 需要要访问一个 Service， 并且是使用环境变量的方式将端口和集群IP传递给客户端 Pod 的，
那么 Service 必须要在客户端 Pod 创建 *之前* 就要存在。 否则客户端 Pod 中就不会加入它对应的环境变量。

如果只使用 DNS 为查找 Service 的集群IP，则不需要担心这个顺序问题
{{< /note >}}
<!--
### DNS

You can (and almost always should) set up a DNS service for your Kubernetes
cluster using an [add-on](/docs/concepts/cluster-administration/addons/).

A cluster-aware DNS server, such as CoreDNS, watches the Kubernetes API for new
Services and creates a set of DNS records for each one.  If DNS has been enabled
throughout your cluster then all Pods should automatically be able to resolve
Services by their DNS name.

For example, if you have a Service called `"my-service"` in a Kubernetes
Namespace `"my-ns"`, the control plane and the DNS Service acting together
create a DNS record for `"my-service.my-ns"`. Pods in the `"my-ns"` Namespace
should be able to find it by simply doing a name lookup for `my-service`
(`"my-service.my-ns"` would also work).

Pods in other Namespaces must qualify the name as `my-service.my-ns`. These names
will resolve to the cluster IP assigned for the Service.

Kubernetes also supports DNS SRV (Service) records for named ports.  If the
`"my-service.my-ns"` Service has a port named `"http"` with the protocol set to
`TCP`, you can do a DNS SRV query for `_http._tcp.my-service.my-ns` to discover
the port number for `"http"`, as well as the IP address.

The Kubernetes DNS server is the only way to access `ExternalName` Services.
You can find more information about `ExternalName` resolution in
[DNS Pods and Services](/docs/concepts/services-networking/dns-pod-service/).
-->

### DNS

用户可以(并且几乎绝大多时候应该)通过使用
[插件](/k8sDocs/docs/concepts/cluster-administration/addons/).
为你的集群设置 DNS 服务。

一个可感知集群的 DNS 服务， 例如 CoreDNS， 会监听 k8s API 创建的 Service 并创建对应的 DNS 记录。
如果集群启用的 DNS 服务，则所以的 Pod 都应该会自动地通过 DNS 名称解析 Service。

例如，如果有一个名叫 `"my-service"` Service 于 `"my-ns"` 命名空间，控制中心和 DNS 服务会
协作创建一条 DNS 记录为 `"my-service.my-ns"`。 在 `"my-ns"` 命名空间的 Pod 可以只需要简单地
使用 `my-service` 就能查到(`"my-service.my-ns"` 也是可以的)

在其它命名空间的 Pod 必须使用 `my-service.my-ns` 这样的限定名。 这些名称会解析为 Service
分配的集群IP。

k8s 还支持命名端口的 DNS SRV (Service) 记录。 如果叫 `"my-service.my-ns"` 的 Service
有一个叫 `"http"` 的 `TCP` 端口， 就可以使用 `DNS SRV` 查询 `_http._tcp.my-service.my-ns`
得到 `"http"` 对应的端口号和 IP 地址。

k8s DNS 服务是访问 `ExternalName` Service 的唯一方式。更多关于 `ExternalName` 的信息见
[DNS Pod 和 Service](/k8sDocs/docs/concepts/services-networking/dns-pod-service/).
<!--
## Headless Services

Sometimes you don't need load-balancing and a single Service IP.  In
this case, you can create what are termed "headless" Services, by explicitly
specifying `"None"` for the cluster IP (`.spec.clusterIP`).

You can use a headless Service to interface with other service discovery mechanisms,
without being tied to Kubernetes' implementation.

For headless `Services`, a cluster IP is not allocated, kube-proxy does not handle
these Services, and there is no load balancing or proxying done by the platform
for them. How DNS is automatically configured depends on whether the Service has
selectors defined:
 -->
## Headless Services {#headless-services}

有时候并不需要负载均衡和一个 Service 的 IP。 在这种情况下就可以创建一个被称为 "无头" 的 Service，
更确切的说就是将集群 IP (`.spec.clusterIP`) 设置为 `"None"`

用户可以使用 无头 Service 作为其它服务发现机制的接口，而不需要与 k8s 实现耦合在一起。

对于 无头的 `Services` 是不会分配集群IP的， `kube-proxy` 不会处理这些 Service，
平台也不会对它们提供负载均衡或代理。 DNS 是如何自动配置的基于 Service 是否定义了选择器:
<!--
### With selectors

For headless Services that define selectors, the endpoints controller creates
`Endpoints` records in the API, and modifies the DNS configuration to return
records (addresses) that point directly to the `Pods` backing the `Service`.

### Without selectors

For headless Services that do not define selectors, the endpoints controller does
not create `Endpoints` records. However, the DNS system looks for and configures
either:

  * CNAME records for [`ExternalName`](#externalname)-type Services.
  * A records for any `Endpoints` that share a name with the Service, for all
    other types.
    -->

### 有选择器的

对于有选择器的无头 Service， Endpoint 选择器会创建 `Endpoints` 记录， 并修改 DNS 配置
直接返回记录为 `Service` 后端 `Pod` 的地址。

### 没有选择器的

For headless Services that do not define selectors, the endpoints controller does
not create `Endpoints` records. However, the DNS system looks for and configures
either:

  * CNAME records for [`ExternalName`](#externalname)-type Services.
  * A records for any `Endpoints` that share a name with the Service, for all
    other types.

对于没有选择器的无头 Service， Endpoint 选择器不会创建 `Endpoints` 记录，但是 DNS 系统会根据以
下情况来配置:

- [`ExternalName`](#externalname) 类型的 Service 创建 CNAME
- 所有其它类型，为其它任意与该 Service 同名的 `Endpoints` 创建 A 记录
<!--
## Publishing Services (ServiceTypes) {#publishing-services-service-types}

For some parts of your application (for example, frontends) you may want to expose a
Service onto an external IP address, that's outside of your cluster.

Kubernetes `ServiceTypes` allow you to specify what kind of Service you want.
The default is `ClusterIP`.

`Type` values and their behaviors are:

   * `ClusterIP`: Exposes the Service on a cluster-internal IP. Choosing this value
     makes the Service only reachable from within the cluster. This is the
     default `ServiceType`.
   * [`NodePort`](#nodeport): Exposes the Service on each Node's IP at a static port
     (the `NodePort`). A `ClusterIP` Service, to which the `NodePort` Service
     routes, is automatically created.  You'll be able to contact the `NodePort` Service,
     from outside the cluster,
     by requesting `<NodeIP>:<NodePort>`.
   * [`LoadBalancer`](#loadbalancer): Exposes the Service externally using a cloud
     provider's load balancer. `NodePort` and `ClusterIP` Services, to which the external
     load balancer routes, are automatically created.
   * [`ExternalName`](#externalname): Maps the Service to the contents of the
     `externalName` field (e.g. `foo.bar.example.com`), by returning a `CNAME` record

     with its value. No proxying of any kind is set up.
     {{< note >}}
     You need either kube-dns version 1.7 or CoreDNS version 0.0.8 or higher to use the `ExternalName` type.
     {{< /note >}}

You can also use [Ingress](/docs/concepts/services-networking/ingress/) to expose your Service. Ingress is not a Service type, but it acts as the entry point for your cluster. It lets you consolidate your routing rules into a single resource as it can expose multiple services under the same IP address.
-->

## 发布 Service (ServiceTypes) {#publishing-services-service-types}


对于应用的一些部分(例如，前端)，需要将 Service 暴露到集群外部 IP 地址。

k8s 可以通过 `ServiceTypes` 来指定想要创建的 Service 类型， 默认为 `ClusterIP`

`Type` 的值各行为如下:

- `ClusterIP`: 以集群内部 IP 的形式暴露 Service， 使用这种方式 Service 只能在集群内部访问。
  这是默认的 `ServiceType`

- [`NodePort`](#nodeport): 将 Service 暴露到每个节点 IP和一个静态端口上(可以通过 `NodePort`指定)
  (`ClusterIP` Service 到 `NodePort` Service 的路由会自动创建)。
  用户可以通过在集群外请求 `<NodeIP>:<NodePort>` 方式访问 `NodePort` Service。

- [`LoadBalancer`](#loadbalancer): 使用云提供商的负载均衡器对外暴露 Service。
  由 `NodePort` 和 `ClusterIP` Service 到外部负载均衡的路由会自动创建.

- [`ExternalName`](#externalname):
  通过返回 `CNAME` 的方式 将 Service 映射到 `externalName` 字段 (e.g. `foo.bar.example.com`)的服务
  没有设置任何类型的代理
  {{< note >}}
  如果使用 `ExternalName` 类型，需要 kube-dns `v1.7+` 或 CoreDNS `v0.0.8+`
  {{< /note >}}

用户也可以使用 [Ingress](/k8sDocs/docs/concepts/services-networking/ingress/) 来暴露 Service。
Ingress 不是 Service 的一个类型， 但它扮演的是集群切入点的角色。 它让路由规则可以统一为一个资源。
并可以在同一个IP地址上暴露多个 Service
<!--
### Type NodePort {#nodeport}

If you set the `type` field to `NodePort`, the Kubernetes control plane
allocates a port from a range specified by `--service-node-port-range` flag (default: 30000-32767).
Each node proxies that port (the same port number on every Node) into your Service.
Your Service reports the allocated port in its `.spec.ports[*].nodePort` field.


If you want to specify particular IP(s) to proxy the port, you can set the `--nodeport-addresses` flag in kube-proxy to particular IP block(s); this is supported since Kubernetes v1.10.
This flag takes a comma-delimited list of IP blocks (e.g. 10.0.0.0/8, 192.0.2.0/25) to specify IP address ranges that kube-proxy should consider as local to this node.

For example, if you start kube-proxy with the `--nodeport-addresses=127.0.0.0/8` flag, kube-proxy only selects the loopback interface for NodePort Services. The default for `--nodeport-addresses` is an empty list. This means that kube-proxy should consider all available network interfaces for NodePort. (That's also compatible with earlier Kubernetes releases).

If you want a specific port number, you can specify a value in the `nodePort`
field. The control plane will either allocate you that port or report that
the API transaction failed.
This means that you need to take care of possible port collisions yourself.
You also have to use a valid port number, one that's inside the range configured
for NodePort use.

Using a NodePort gives you the freedom to set up your own load balancing solution,
to configure environments that are not fully supported by Kubernetes, or even
to just expose one or more nodes' IPs directly.

Note that this Service is visible as `<NodeIP>:spec.ports[*].nodePort`
and `.spec.clusterIP:spec.ports[*].port`. (If the `--nodeport-addresses` flag in kube-proxy is set, <NodeIP> would be filtered NodeIP(s).)

For example:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: MyApp
  ports:
      # By default and for convenience, the `targetPort` is set to the same value as the `port` field.
    - port: 80
      targetPort: 80
      # Optional field
      # By default and for convenience, the Kubernetes control plane will allocate a port from a range (default: 30000-32767)
      nodePort: 30007
```
 -->

### NodePort {#nodeport}

如果将 `type` 字段设置为 `NodePort`， k8s 控制中心会从
`--service-node-port-range` 选择配置的范围(默认 30000-32767)中分配一个端口。
集群中的每个节点都会将那个端口(每个节点使用相同的端口)代理到 Service 上。
Service 会将分配的的端口存放在它的 `.spec.ports[*].nodePort` 字段。

如果想要指定某些IP来代理这个端口，可以通过 kube-proxy 中的 `--nodeport-addresses` 选择来
指定 IP 或 IP 段；这个特性自 k8s `v1.10` 开始支持。
这个选择支持逗号分隔的 IP 段列表(e.g. 10.0.0.0/8, 192.0.2.0/25) 来指定 kube-proxy 是不是应该认为是应该代理的 IP 地址范围。

例如，如果将 kube-proxy 设置 `--nodeport-addresses=127.0.0.0/8`， 则 kube-proxy 只会
选择本地回环接口用作 Service 的 NodePort。 默认的  `--nodeport-addresses` 是一个空列表。
其含义是 kube-proxy 会将所以可用的网络接口应用到 NodePort (这也与早期的 k8s 版本兼容)

如果用户想要指定端口，可以通过设置 `nodePort` 的值实现。 控制中心会分配那个端口或报告业务失败。
也就是说在设置该字段时需要用户自己解决端口冲突的问题。
并且首先设置的端口是一个有效的端口，其次端口是在之前提到所配置的 NodePort 的可用范围内。

使用 NodePort 时就将负载均衡的解决方案选择交到用户手上, 可以用于配置那些 k8s 不完全支持的环境，
甚至直接暴露一个或多个节点的IP

要注意 Service 可以通过 `<NodeIP>:spec.ports[*].nodePort`:`.spec.clusterIP:spec.ports[*].port` 访问。
(如果 kube-proxy 设置了 `--nodeport-addresses`， 则 <NodeIP> 只能是配置的范围内的IP)

例:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: MyApp
  ports:
      # 默认情况下和为了方便， `targetPort` 会与 `port` 字段使用相同的值
    - port: 80
      targetPort: 80
      # 可选字段
      # 默认情况下和为了方便，k8s 控制中心会在配置的端口范围(默认30000-32767) 内分配一个端口
      nodePort: 30007
```
<!--
### Type LoadBalancer {#loadbalancer}

On cloud providers which support external load balancers, setting the `type`
field to `LoadBalancer` provisions a load balancer for your Service.
The actual creation of the load balancer happens asynchronously, and
information about the provisioned balancer is published in the Service's
`.status.loadBalancer` field.
For example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
  clusterIP: 10.0.171.239
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
    - ip: 192.0.2.127
```

Traffic from the external load balancer is directed at the backend Pods. The cloud provider decides how it is load balanced.

For LoadBalancer type of Services, when there is more than one port defined, all
ports must have the same protocol and the protocol must be one of `TCP`, `UDP`,
and `SCTP`.

Some cloud providers allow you to specify the `loadBalancerIP`. In those cases, the load-balancer is created
with the user-specified `loadBalancerIP`. If the `loadBalancerIP` field is not specified,
the loadBalancer is set up with an ephemeral IP address. If you specify a `loadBalancerIP`
but your cloud provider does not support the feature, the `loadbalancerIP` field that you
set is ignored.

{{< note >}}
If you're using SCTP, see the [caveat](#caveat-sctp-loadbalancer-service-type) below about the
`LoadBalancer` Service type.
{{< /note >}}

{{< note >}}

On **Azure**, if you want to use a user-specified public type `loadBalancerIP`, you first need
to create a static type public IP address resource. This public IP address resource should
be in the same resource group of the other automatically created resources of the cluster.
For example, `MC_myResourceGroup_myAKSCluster_eastus`.

Specify the assigned IP address as loadBalancerIP. Ensure that you have updated the securityGroupName in the cloud provider configuration file. For information about troubleshooting `CreatingLoadBalancerFailed` permission issues see, [Use a static IP address with the Azure Kubernetes Service (AKS) load balancer](https://docs.microsoft.com/en-us/azure/aks/static-ip) or [CreatingLoadBalancerFailed on AKS cluster with advanced networking](https://github.com/Azure/AKS/issues/357).

{{< /note >}}
 -->
### LoadBalancer {#loadbalancer}

云提供商还支持外部的负载均衡器， 通过设置 `type` 的值为 `LoadBalancer` 为 Service 提供一个负载均衡器
实际上对负载均衡器的创建是异步的，关于添加的负载均衡器的信息会添加到 Service 的 `.status.loadBalancer` 字段。

例:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
  clusterIP: 10.0.171.239
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
    - ip: 192.0.2.127
```

从外部负载均衡器进来的流量会直接转发到后端的 Pod上。 云提供商会决定负载均衡方式。

对于 LoadBalancer 类型的 Service， 当一个 Service 上有不止一个端口时， 所以的端口必须要使用
相同的协议，并且协议只能是 `TCP`, `UDP`, `SCTP` 中的一种.

有些云提供商支持指定 `loadBalancerIP`， 在这个情况下， 负载均衡是使用指定的 `loadBalancerIP` 创建。
如果 `loadBalancerIP` 没有指定则会使用一个临时 IP 地址。 如果指定了 `loadBalancerIP`
但云提供商不支持该特性，则指定的 `loadbalancerIP` 字段会被忽略。
{{< note >}}
如果使用的是 SCTP 协议，见下面的 [caveat](#caveat-sctp-loadbalancer-service-type) 关于 `LoadBalancer`
Service 类型的相关信息
{{< /note >}}

{{< note >}}

在 **Azure** 上， 如果想要使用用户指定的 `loadBalancerIP`， 首先需要创建一个静态类型的
公网 IP 地址资源。 这个公网 IP 地址资源应该与集群其它自动创建的资源在同一个资源组。
例如，`MC_myResourceGroup_myAKSCluster_eastus`.

通过 loadBalancerIP 指定分配 IP， 需要确保已经更新云提供商配置文件中的 securityGroupName。
更多 `CreatingLoadBalancerFailed` 权限问题的调度信息见
[Use a static IP address with the Azure Kubernetes Service (AKS) load balancer](https://docs.microsoft.com/en-us/azure/aks/static-ip)
或
[CreatingLoadBalancerFailed on AKS cluster with advanced networking](https://github.com/Azure/AKS/issues/357)
{{< /note >}}
<!--
#### Internal load balancer
In a mixed environment it is sometimes necessary to route traffic from Services inside the same
(virtual) network address block.

In a split-horizon DNS environment you would need two Services to be able to route both external and internal traffic to your endpoints.

You can achieve this by adding one the following annotations to a Service.
The annotation to add depends on the cloud Service provider you're using.

{{< tabs name="service_tabs" >}}
{{% tab name="Default" %}}
Select one of the tabs.
{{% /tab %}}
{{% tab name="GCP" %}}
```yaml
[...]
metadata:
    name: my-service
    annotations:
        cloud.google.com/load-balancer-type: "Internal"
[...]
```
{{% /tab %}}
{{% tab name="AWS" %}}
```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/aws-load-balancer-internal: "true"
[...]
```
{{% /tab %}}
{{% tab name="Azure" %}}
```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/azure-load-balancer-internal: "true"
[...]
```
{{% /tab %}}
{{% tab name="IBM Cloud" %}}
```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.kubernetes.io/ibm-load-balancer-cloud-provider-ip-type: "private"
[...]
```
{{% /tab %}}
{{% tab name="OpenStack" %}}
```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/openstack-internal-load-balancer: "true"
[...]
```
{{% /tab %}}
{{% tab name="Baidu Cloud" %}}
```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/cce-load-balancer-internal-vpc: "true"
[...]
```
{{% /tab %}}
{{% tab name="Tencent Cloud" %}}
```yaml
[...]
metadata:
  annotations:
    service.kubernetes.io/qcloud-loadbalancer-internal-subnetid: subnet-xxxxx
[...]
```
{{% /tab %}}
{{% tab name="Alibaba Cloud" %}}
```yaml
[...]
metadata:
  annotations:
    service.beta.kubernetes.io/alibaba-cloud-loadbalancer-address-type: "intranet"
[...]
```
{{% /tab %}}
{{< /tabs >}}
 -->

#### 内部负载均衡器

在一个混搭的环境中，有时候需要在同一个(虚拟)网络地址段从 Service 间路由流量。

在一个水平分割的 DNS 环境，需要有两个 Service 才能够分别路由外部和内部的流量到 Endpoint.

可以通过在 Service 上添加以下注解中的一个来达成这个目的。 添加哪个注解基于你用的云提供商。

{{< tabs name="service_tabs" >}}
{{% tab name="Default" %}}
选择其中一个标签
{{% /tab %}}
{{% tab name="GCP" %}}
```yaml
[...]
metadata:
    name: my-service
    annotations:
        cloud.google.com/load-balancer-type: "Internal"
[...]
```
{{% /tab %}}
{{% tab name="AWS" %}}
```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/aws-load-balancer-internal: "true"
[...]
```
{{% /tab %}}
{{% tab name="Azure" %}}
```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/azure-load-balancer-internal: "true"
[...]
```
{{% /tab %}}
{{% tab name="IBM Cloud" %}}
```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.kubernetes.io/ibm-load-balancer-cloud-provider-ip-type: "private"
[...]
```
{{% /tab %}}
{{% tab name="OpenStack" %}}
```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/openstack-internal-load-balancer: "true"
[...]
```
{{% /tab %}}
{{% tab name="Baidu Cloud" %}}
```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/cce-load-balancer-internal-vpc: "true"
[...]
```
{{% /tab %}}
{{% tab name="Tencent Cloud" %}}
```yaml
[...]
metadata:
  annotations:
    service.kubernetes.io/qcloud-loadbalancer-internal-subnetid: subnet-xxxxx
[...]
```
{{% /tab %}}
{{% tab name="Alibaba Cloud" %}}
```yaml
[...]
metadata:
  annotations:
    service.beta.kubernetes.io/alibaba-cloud-loadbalancer-address-type: "intranet"
[...]
```
{{% /tab %}}
{{< /tabs >}}

<!--
#### TLS support on AWS {#ssl-support-on-aws}

For partial TLS / SSL support on clusters running on AWS, you can add three
annotations to a `LoadBalancer` service:

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: arn:aws:acm:us-east-1:123456789012:certificate/12345678-1234-1234-1234-123456789012
```

The first specifies the ARN of the certificate to use. It can be either a
certificate from a third party issuer that was uploaded to IAM or one created
within AWS Certificate Manager.

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: (https|http|ssl|tcp)
```

The second annotation specifies which protocol a Pod speaks. For HTTPS and
SSL, the ELB expects the Pod to authenticate itself over the encrypted
connection, using a certificate.

HTTP and HTTPS selects layer 7 proxying: the ELB terminates
the connection with the user, parses headers, and injects the `X-Forwarded-For`
header with the user's IP address (Pods only see the IP address of the
ELB at the other end of its connection) when forwarding requests.

TCP and SSL selects layer 4 proxying: the ELB forwards traffic without
modifying the headers.

In a mixed-use environment where some ports are secured and others are left unencrypted,
you can use the following annotations:

```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-backend-protocol: http
        service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "443,8443"
```

In the above example, if the Service contained three ports, `80`, `443`, and
`8443`, then `443` and `8443` would use the SSL certificate, but `80` would just
be proxied HTTP.

From Kubernetes v1.9 onwards you can use [predefined AWS SSL policies](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-security-policy-table.html) with HTTPS or SSL listeners for your Services.
To see which policies are available for use, you can use the `aws` command line tool:

```bash
aws elb describe-load-balancer-policies --query 'PolicyDescriptions[].PolicyName'
```

You can then specify any one of those policies using the
"`service.beta.kubernetes.io/aws-load-balancer-ssl-negotiation-policy`"
annotation; for example:

```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-ssl-negotiation-policy: "ELBSecurityPolicy-TLS-1-2-2017-01"
```
 -->

#### AWS 对 TLS 的支持 {#ssl-support-on-aws}

运行在 AWS 上的集群部分支持 TLS / SSL，可以添加以下三个注解到一个 `LoadBalancer` 的 Service:

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: arn:aws:acm:us-east-1:123456789012:certificate/12345678-1234-1234-1234-123456789012
```

第一个指定 证书使用的 ARN。 可以是上传到  IAM 的第三方发行者的证书 或者是在 AWS 证书管理器创建的证书。

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: (https|http|ssl|tcp)
```

connection, using a certificate.
第二个注解指定 Pod 使用的是哪个协议。 对于 HTTPS 和 SSL，负载均衡期望的是 Pod 使用证书的安全连接来认证它自己。

HTTP 和 HTTPS 使用第 7 层代理: 转发请求是由负载均衡器来终止用户的连接，解析头，插入包含用户 IP 地址
的 `X-Forwarded-For` 头(Pod 只能看到连接另一头的负载均衡的IP地址)

TCP 和 SSL 使用 4 层代理: 负载均衡器转发流量是不会修改头信息

在混搭环境中，有些端口是安全的有是又是未加密的，可以使用如下注解:
```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-backend-protocol: http
        service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "443,8443"
```

在上面的例子中， 如果 Service 包含三个端口， `80`, `443`, `8443`, 其中 `443` 和 `8443`
是使用 SSL 证书的，但 `80` 只通过 HTTP 代理

从 k8s v1.9 开始，可以使用
[predefined AWS SSL policies](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-security-policy-table.html)
来配置 Service 的 HTTPS 或 SSL 监控器。
可以通过以下 aws 命令行工具来查看哪些可用的策略:
```bash
aws elb describe-load-balancer-policies --query 'PolicyDescriptions[].PolicyName'
```



然后可以使
"`service.beta.kubernetes.io/aws-load-balancer-ssl-negotiation-policy`"
注解来使用其中的某个策略，例:
```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-ssl-negotiation-policy: "ELBSecurityPolicy-TLS-1-2-2017-01"
```
<!--
#### PROXY protocol support on AWS

To enable [PROXY protocol](https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt)
support for clusters running on AWS, you can use the following service
annotation:

```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-proxy-protocol: "*"
```

Since version 1.3.0, the use of this annotation applies to all ports proxied by the ELB
and cannot be configured otherwise. -->

####  AWS 支持的 PROXY 协议

要在 AWS 运行的集群中启用 [PROXY protocol](https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt)
可以在 Service 上添加以下注解:

```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-proxy-protocol: "*"
```

从 v1.3.0 开始，就只能通过这个注解来让所以的端口通过 ELB 代理，不能通过其它方式配置了。
<!--
#### ELB Access Logs on AWS

There are several annotations to manage access logs for ELB Services on AWS.

The annotation `service.beta.kubernetes.io/aws-load-balancer-access-log-enabled`
controls whether access logs are enabled.

The annotation `service.beta.kubernetes.io/aws-load-balancer-access-log-emit-interval`
controls the interval in minutes for publishing the access logs. You can specify
an interval of either 5 or 60 minutes.

The annotation `service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-name`
controls the name of the Amazon S3 bucket where load balancer access logs are
stored.

The annotation `service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-prefix`
specifies the logical hierarchy you created for your Amazon S3 bucket.

```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-access-log-enabled: "true"
        # Specifies whether access logs are enabled for the load balancer
        service.beta.kubernetes.io/aws-load-balancer-access-log-emit-interval: "60"
        # The interval for publishing the access logs. You can specify an interval of either 5 or 60 (minutes).
        service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-name: "my-bucket"
        # The name of the Amazon S3 bucket where the access logs are stored
        service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-prefix: "my-bucket-prefix/prod"
        # The logical hierarchy you created for your Amazon S3 bucket, for example `my-bucket-prefix/prod`
```
 -->
####  AWS 上 ELB 的访问日志

AWS 上 ELB 的访问日志 可以通过几个注解来管理。

`service.beta.kubernetes.io/aws-load-balancer-access-log-enabled` 注解控制是否开启访问日志

`service.beta.kubernetes.io/aws-load-balancer-access-log-emit-interval` 注解控制发布
日志的时间间隔(单位为分钟)。 时间间隔可以是 5 分钟 或 60 分钟

`service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-name` 注解控制访问
日志存储的 Amazon S3 bucket 名称
`service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-prefix` 注解
它指定创建的 Amazon S3 bucket 的逻辑层次

```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-access-log-enabled: "true"
        # 否开启访问日志
        service.beta.kubernetes.io/aws-load-balancer-access-log-emit-interval: "60"
        # 发布 日志的时间间隔(单位为分钟)。 时间间隔可以是 5 分钟 或 60 分钟
        service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-name: "my-bucket"
        # 日志存储的 Amazon S3 bucket 名称
        service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-prefix: "my-bucket-prefix/prod"
        # 创建的 Amazon S3 bucket 的逻辑层次, 例如 `my-bucket-prefix/prod`
```
<!--
#### Connection Draining on AWS

Connection draining for Classic ELBs can be managed with the annotation
`service.beta.kubernetes.io/aws-load-balancer-connection-draining-enabled` set
to the value of `"true"`. The annotation
`service.beta.kubernetes.io/aws-load-balancer-connection-draining-timeout` can
also be used to set maximum time, in seconds, to keep the existing connections open before deregistering the instances.


```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-connection-draining-enabled: "true"
        service.beta.kubernetes.io/aws-load-balancer-connection-draining-timeout: "60"
``` -->


#### AWS 连接控制

经典ELB 的连接使用可以通过将注解
`service.beta.kubernetes.io/aws-load-balancer-connection-draining-enabled`
的值设置为 `"true"` 来管理。
还可以通过注解
`service.beta.kubernetes.io/aws-load-balancer-connection-draining-timeout`
也可以用来设置保证已有连接的最大时间(单位秒)，

```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-connection-draining-enabled: "true"
        service.beta.kubernetes.io/aws-load-balancer-connection-draining-timeout: "60"
```
<!--
#### Other ELB annotations

There are other annotations to manage Classic Elastic Load Balancers that are described below.

```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: "60"
        # The time, in seconds, that the connection is allowed to be idle (no data has been sent over the connection) before it is closed by the load balancer

        service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
        # Specifies whether cross-zone load balancing is enabled for the load balancer

        service.beta.kubernetes.io/aws-load-balancer-additional-resource-tags: "environment=prod,owner=devops"
        # A comma-separated list of key-value pairs which will be recorded as
        # additional tags in the ELB.

        service.beta.kubernetes.io/aws-load-balancer-healthcheck-healthy-threshold: ""
        # The number of successive successful health checks required for a backend to
        # be considered healthy for traffic. Defaults to 2, must be between 2 and 10

        service.beta.kubernetes.io/aws-load-balancer-healthcheck-unhealthy-threshold: "3"
        # The number of unsuccessful health checks required for a backend to be
        # considered unhealthy for traffic. Defaults to 6, must be between 2 and 10

        service.beta.kubernetes.io/aws-load-balancer-healthcheck-interval: "20"
        # The approximate interval, in seconds, between health checks of an
        # individual instance. Defaults to 10, must be between 5 and 300

        service.beta.kubernetes.io/aws-load-balancer-healthcheck-timeout: "5"
        # The amount of time, in seconds, during which no response means a failed
        # health check. This value must be less than the service.beta.kubernetes.io/aws-load-balancer-healthcheck-interval
        # value. Defaults to 5, must be between 2 and 60

        service.beta.kubernetes.io/aws-load-balancer-extra-security-groups: "sg-53fae93f,sg-42efd82e"
        # A list of additional security groups to be added to the ELB

        service.beta.kubernetes.io/aws-load-balancer-target-node-labels: "ingress-gw,gw-name=public-api"
        # A comma separated list of key-value pairs which are used
        # to select the target nodes for the load balancer
```
 -->

#### ELB 其它注解

以下介绍管理经典弹性负载均衡器的其它注解.

```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: "60"
        # 连接在被负载均衡关闭前允许空闲(没有从连接发送数据)的时间，单位秒

        service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
        # 指定负载均衡器是否开启跨区负载均衡

        service.beta.kubernetes.io/aws-load-balancer-additional-resource-tags: "environment=prod,owner=devops"
        # 一个以逗号分隔的键值对列表，用于记在 ELB 中的额外标签

        service.beta.kubernetes.io/aws-load-balancer-healthcheck-healthy-threshold: ""
        # 一个后端服务实例被认为是健康可以处理流量所需要的成功健康检查次数，默认为 2， 只能在 2 到 10 之间

        service.beta.kubernetes.io/aws-load-balancer-healthcheck-unhealthy-threshold: "3"
        # 一个后端服务实例被认为是不健康不能处理流量需要的失败的健康检查次数， 默认为 6， 必须在 2 到 10 之间

        service.beta.kubernetes.io/aws-load-balancer-healthcheck-interval: "20"
        # 针对每个独立实例进行健康检查的时间间隔，单位秒，默认为 10， 必须在 5 到 300 之间

        service.beta.kubernetes.io/aws-load-balancer-healthcheck-timeout: "5"
        # 如果在这个时间(单位秒)内健康检查没有响应，则认为检测结果为失败。这个值必须小于
        # service.beta.kubernetes.io/aws-load-balancer-healthcheck-interval 的值
        # 默认为 5， 必须在 2 到 10 之间。

        service.beta.kubernetes.io/aws-load-balancer-extra-security-groups: "sg-53fae93f,sg-42efd82e"
        # 被添加到 ELB 的额外的安全组列表

        service.beta.kubernetes.io/aws-load-balancer-target-node-labels: "ingress-gw,gw-name=public-api"
        # 一个以逗号分隔的键值对列表，用于选择负载均衡的节点
```
<!--
#### Network Load Balancer support on AWS {#aws-nlb-support}

{{< feature-state for_k8s_version="v1.15" state="beta" >}}

To use a Network Load Balancer on AWS, use the annotation `service.beta.kubernetes.io/aws-load-balancer-type` with the value set to `nlb`.

```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
```

{{< note >}}
NLB only works with certain instance classes; see the [AWS documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/target-group-register-targets.html#register-deregister-targets)
on Elastic Load Balancing for a list of supported instance types.
{{< /note >}}

Unlike Classic Elastic Load Balancers, Network Load Balancers (NLBs) forward the
client's IP address through to the node. If a Service's `.spec.externalTrafficPolicy`
is set to `Cluster`, the client's IP address is not propagated to the end
Pods.

By setting `.spec.externalTrafficPolicy` to `Local`, the client IP addresses is
propagated to the end Pods, but this could result in uneven distribution of
traffic. Nodes without any Pods for a particular LoadBalancer Service will fail
the NLB Target Group's health check on the auto-assigned
`.spec.healthCheckNodePort` and not receive any traffic.

In order to achieve even traffic, either use a DaemonSet or specify a
[pod anti-affinity](/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity)
to not locate on the same node.

You can also use NLB Services with the [internal load balancer](/docs/concepts/services-networking/service/#internal-load-balancer)
annotation.

In order for client traffic to reach instances behind an NLB, the Node security
groups are modified with the following IP rules:

| Rule | Protocol | Port(s) | IpRange(s) | IpRange Description |
|------|----------|---------|------------|---------------------|
| Health Check | TCP | NodePort(s) (`.spec.healthCheckNodePort` for `.spec.externalTrafficPolicy = Local`) | VPC CIDR | kubernetes.io/rule/nlb/health=\<loadBalancerName\> |
| Client Traffic | TCP | NodePort(s) | `.spec.loadBalancerSourceRanges` (defaults to `0.0.0.0/0`) | kubernetes.io/rule/nlb/client=\<loadBalancerName\> |
| MTU Discovery | ICMP | 3,4 | `.spec.loadBalancerSourceRanges` (defaults to `0.0.0.0/0`) | kubernetes.io/rule/nlb/mtu=\<loadBalancerName\> |

In order to limit which client IP's can access the Network Load Balancer,
specify `loadBalancerSourceRanges`.

```yaml
spec:
  loadBalancerSourceRanges:
    - "143.231.0.0/16"
```

{{< note >}}
If `.spec.loadBalancerSourceRanges` is not set, Kubernetes
allows traffic from `0.0.0.0/0` to the Node Security Group(s). If nodes have
public IP addresses, be aware that non-NLB traffic can also reach all instances
in those modified security groups.

{{< /note >}}
 -->

#### AWS 上支持的网络负载均衡(NLB) {#aws-nlb-support}

{{< feature-state for_k8s_version="v1.15" state="beta" >}}

要使用 AWS 上的网络负载均衡器，使用注解 `service.beta.kubernetes.io/aws-load-balancer-type`
设置值为 `nlb`
```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
```

{{< note >}}
NLB 只适用的特定类型的实例，ELB 支持的实例类型见
[AWS 文档](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/target-group-register-targets.html#register-deregister-targets)
{{< /note >}}

与经典 ELB 不同， NLB 转发的客户端 IP 地址穿透节点。 如果 Service 的
`.spec.externalTrafficPolicy` 设置为 `Cluster`， 客户端 IP 地址不会传递到最终的 Pod。


`.spec.healthCheckNodePort` and not receive any traffic.
通过设置 `.spec.externalTrafficPolicy` 为 `Local`， 客户端 IP 地址会传递到最终的 Pod。
但这会导向流量分发不均衡。 没有包含该负载均衡器对应 Service 的 Pod 的节点，会在 NLB 目标组
健康检查时分配的 `.spec.healthCheckNodePort` 检查失败。 并不会收到任何流量


为了达成平衡的负载，要么使用 DaemonSet，要么设置
[pod anti-affinity](/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity)
让 Pod 不要调度到同一个节点上


也可以在使用 NLB Service 时在其中包含
[内部负载均衡器](/docs/concepts/services-networking/service/#internal-load-balancer)
注解

为了能让客户端流量到达 NLB 后面的实例。 节点安全组需要以以下 IP 规则进行修改:

| Rule | Protocol | Port(s) | IpRange(s) | IpRange Description |
|------|----------|---------|------------|---------------------|
| Health Check | TCP | NodePort(s) (`.spec.healthCheckNodePort` for `.spec.externalTrafficPolicy = Local`) | VPC CIDR | kubernetes.io/rule/nlb/health=\<loadBalancerName\> |
| Client Traffic | TCP | NodePort(s) | `.spec.loadBalancerSourceRanges` (defaults to `0.0.0.0/0`) | kubernetes.io/rule/nlb/client=\<loadBalancerName\> |
| MTU Discovery | ICMP | 3,4 | `.spec.loadBalancerSourceRanges` (defaults to `0.0.0.0/0`) | kubernetes.io/rule/nlb/mtu=\<loadBalancerName\> |

为限所哪个客户端 IP 可以访问 NLB， 指定 `loadBalancerSourceRanges`

```yaml
spec:
  loadBalancerSourceRanges:
    - "143.231.0.0/16"
```

{{< note >}}
如果 `.spec.loadBalancerSourceRanges` 没有设置，k8s 允许来自 `0.0.0.0/0` 流量到节点安全组。
如果节点有公网 IP 地址，要注意那些非 NLB 流量也会到达所有这些修改过的安全组。
{{< /note >}}

{{< todo-optimize text="对 AWS 使用不熟悉">}}
<!--
#### Other CLB annotations on Tencent Kubernetes Engine (TKE)

There are other annotations for managing Cloud Load Balancers on TKE as shown below.

```yaml
    metadata:
      name: my-service
      annotations:
        # Bind Loadbalancers with specified nodes
        service.kubernetes.io/qcloud-loadbalancer-backends-label: key in (value1, value2)

        # ID of an existing load balancer
        service.kubernetes.io/tke-existed-lbid：lb-6swtxxxx

        # Custom parameters for the load balancer (LB), does not support modification of LB type yet
        service.kubernetes.io/service.extensiveParameters: ""

        # Custom parameters for the LB listener
        service.kubernetes.io/service.listenerParameters: ""

        # Specifies the type of Load balancer;
        # valid values: classic (Classic Cloud Load Balancer) or application (Application Cloud Load Balancer)
        service.kubernetes.io/loadbalance-type: xxxxx

        # Specifies the public network bandwidth billing method;
        # valid values: TRAFFIC_POSTPAID_BY_HOUR(bill-by-traffic) and BANDWIDTH_POSTPAID_BY_HOUR (bill-by-bandwidth).
        service.kubernetes.io/qcloud-loadbalancer-internet-charge-type: xxxxxx

        # Specifies the bandwidth value (value range: [1,2000] Mbps).
        service.kubernetes.io/qcloud-loadbalancer-internet-max-bandwidth-out: "10"

        # When this annotation is set，the loadbalancers will only register nodes
        # with pod running on it, otherwise all nodes will be registered.
        service.kubernetes.io/local-svc-only-bind-node-with-pod: true
```
 -->

#### Tencent Kubernetes Engine (TKE) 上其它 CLB 注解

以下为管理 TKE 上 CLB 的其它注解说明。

```yaml
    metadata:
      name: my-service
      annotations:
        # 将 负载均衡器与指定节点绑定
        service.kubernetes.io/qcloud-loadbalancer-backends-label: key in (value1, value2)

        # 已经存在的负载均衡器的 ID
        service.kubernetes.io/tke-existed-lbid：lb-6swtxxxx

        # Custom parameters for the load balancer (LB), does not support modification of LB type yet
        # 负载均衡器的自定义参数， 还不支持对负载均衡类型的修改
        service.kubernetes.io/service.extensiveParameters: ""

        # Custom parameters for the LB listener
        # 负载均衡监听器的自定义参数
        service.kubernetes.io/service.listenerParameters: ""

        # 设置负载均衡器的类型
        # 有效值为: classic (Classic Cloud Load Balancer) 或  application (Application Cloud Load Balancer)
        service.kubernetes.io/loadbalance-type: xxxxx

        # 设置公网带宽收费方式
        # 有效值为: TRAFFIC_POSTPAID_BY_HOUR(bill-by-traffic) 或 BANDWIDTH_POSTPAID_BY_HOUR (bill-by-bandwidth).
        service.kubernetes.io/qcloud-loadbalancer-internet-charge-type: xxxxxx

        # 设置带宽的值(范围: [1,2000] Mbps)
        service.kubernetes.io/qcloud-loadbalancer-internet-max-bandwidth-out: "10"

        # 当这个注解被设置，负载均衡器只会注册有 Pod 在上面运行的节点，否则所有节点都会注册
        service.kubernetes.io/local-svc-only-bind-node-with-pod: true
```
<!--
### Type ExternalName {#externalname}

Services of type ExternalName map a Service to a DNS name, not to a typical selector such as
`my-service` or `cassandra`. You specify these Services with the `spec.externalName` parameter.

This Service definition, for example, maps
the `my-service` Service in the `prod` namespace to `my.database.example.com`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```
{{< note >}}
ExternalName accepts an IPv4 address string, but as a DNS names comprised of digits, not as an IP address. ExternalNames that resemble IPv4 addresses are not resolved by CoreDNS or ingress-nginx because ExternalName
is intended to specify a canonical DNS name. To hardcode an IP address, consider using
[headless Services](#headless-services).
{{< /note >}}

When looking up the host `my-service.prod.svc.cluster.local`, the cluster DNS Service
returns a `CNAME` record with the value `my.database.example.com`. Accessing
`my-service` works in the same way as other Services but with the crucial
difference that redirection happens at the DNS level rather than via proxying or
forwarding. Should you later decide to move your database into your cluster, you
can start its Pods, add appropriate selectors or endpoints, and change the
Service's `type`.

{{< warning >}}
You may have trouble using ExternalName for some common protocols, including HTTP and HTTPS. If you use ExternalName then the hostname used by clients inside your cluster is different from the name that the ExternalName references.

For protocols that use hostnames this difference may lead to errors or unexpected responses. HTTP requests will have a `Host:` header that the origin server does not recognize; TLS servers will not be able to provide a certificate matching the hostname that the client connected to.
{{< /warning >}}

{{< note >}}
This section is indebted to the [Kubernetes Tips - Part
1](https://akomljen.com/kubernetes-tips-part-1/) blog post from [Alen Komljen](https://akomljen.com/).
{{< /note >}}
 -->

### ExternalName {#externalname}

ExternalName 类型的 Service 将 Service 映射到一个 DNS 名称， 而不是一个常见的选择器，如
`my-service` 或 `cassandra`. 通过 Service `spec.externalName` 字段设置 DNS 名称。

以面示例配置中将在 `prod` 命名空间中一个名叫 `my-service` 的 Service 映射到 `my.database.example.com`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```
{{< note >}}
ExternalName 可以使用一个 IPv4 地址的字符串， 但是会被当作一个由数字组成的 DNS 名称，而不是
一个 IP 地址。 ExternalName 值与 IPv4 地址相似的不会被 CoreDNS 或 ingress-nginx 解析，
因为 ExternalName 在设计上就是用作一个标准的 DNS 名称的。 如果要硬编码一个 IP 地址，
考虑使用 [无头 Services](#headless-services)

{{< /note >}}

当查询主机名 `my-service.prod.svc.cluster.local` 时， 集群 DNS 服务会返回一个 `CNAME`
记录，值为 `my.database.example.com`. 访问 `my-service` 的效果与访问其它的 Service 一样，
关键不同点在于重定义发生在 DNS 层， 而不是通过代理或转发。 如果用户决定以后会将数据库移到集群内，
就可以先启动 Pod， 再添加恰当 的 选择器 或 Endpoint, 再修改 Service 的类型就完成迁移了，而客户端不会有感知。

{{< warning >}}
那么集群内客户端使用的主机名与 ExternalName 所引用的名称必然是不一样的。
对于使用主机名的协议，这个不同可能会导致错误或意外的响应。 HTTP 请求会有一个 `Host:` 头，原始
的服务会认不得。 TLS 服务也不能提供客户端连接主机名匹配的证书。
{{< /warning >}}

{{< note >}}
这节感谢
[Alen Komljen](https://akomljen.com/)
的博客
[Kubernetes Tips - Part 1](https://akomljen.com/kubernetes-tips-part-1/)
{{< /note >}}
<!--
### External IPs

If there are external IPs that route to one or more cluster nodes, Kubernetes Services can be exposed on those
`externalIPs`. Traffic that ingresses into the cluster with the external IP (as destination IP), on the Service port,
will be routed to one of the Service endpoints. `externalIPs` are not managed by Kubernetes and are the responsibility
of the cluster administrator.

In the Service spec, `externalIPs` can be specified along with any of the `ServiceTypes`.
In the example below, "`my-service`" can be accessed by clients on "`80.11.12.10:80`" (`externalIP:port`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
  externalIPs:
    - 80.11.12.10
```
 -->

### 外部 IP

如果有外部IP 能够路由到集群的一个或多个节点， k8s Service 可以通过这些 `externalIPs` 来对外暴露。
通过外部IP(作为目标IP)的流量进入集群，到 Service 的端口，会被路由到一个 Service 的 Endpoint
上。 `externalIPs` 不是由 k8s 管理的，这是集群管理员负责的。

在 Service 中，`externalIPs` 可以用在任意  `ServiceTypes` 的 Service 上。 在下面的示例中
客户端可以通过 `80.11.12.10:80` (`externalIP:port`) 访问 `my-service`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
  externalIPs:
    - 80.11.12.10
```
<!--
## Shortcomings

Using the userspace proxy for VIPs, work at small to medium scale, but will
not scale to very large clusters with thousands of Services.  The
[original design proposal for portals](https://github.com/kubernetes/kubernetes/issues/1107)
has more details on this.

Using the userspace proxy obscures the source IP address of a packet accessing
a Service.
This makes some kinds of network filtering (firewalling) impossible.  The iptables
proxy mode does not
obscure in-cluster source IPs, but it does still impact clients coming through
a load balancer or node-port.

The `Type` field is designed as nested functionality - each level adds to the
previous.  This is not strictly required on all cloud providers (e.g. Google Compute Engine does
not need to allocate a `NodePort` to make `LoadBalancer` work, but AWS does)
but the current API requires it.
 -->

## 缺陷

使用 userspace 代理的 VIP，只适用于中小规模，但不适用于有上千 Service 的大规模集群。
更多细节见
[original design proposal for portals](https://github.com/kubernetes/kubernetes/issues/1107)

使用 userspace 代理会吃掉进入 Service 的包的源 IP 地址。 这会使得某些类型的网络过虑(防火墙)
变得不可能， iptables 代理模式不会吃掉进入集群的源 IP 地址，但依然与来自负载均衡或 NodePort
的客户端有冲突。

`Type` 在设计上是可以嵌套的，每一级添加到前一级上。 但这不是所有的云提供商都严格必须要的(例如
  GCE 就不会分配一个 `NodePort` 来让 `LoadBalancer` 工作，但 AWS 又是需要的)但目前的
  API请求必须得有它

<!--
## Virtual IP implementation {#the-gory-details-of-virtual-ips}

The previous information should be sufficient for many people who just want to
use Services.  However, there is a lot going on behind the scenes that may be
worth understanding.
 -->

## 虚拟 IP 的实现 {#the-gory-details-of-virtual-ips}

The previous information should be sufficient for many people who just want to
use Services.  However, there is a lot going on behind the scenes that may be
worth understanding.
之间的信息对于许多只想用 Service 的用户来说已经足够了。但是这些信息之下还有许多值得理解的东西。
<!--
### Avoiding collisions

One of the primary philosophies of Kubernetes is that you should not be
exposed to situations that could cause your actions to fail through no fault
of your own. For the design of the Service resource, this means not making
you choose your own port number if that choice might collide with
someone else's choice.  That is an isolation failure.

In order to allow you to choose a port number for your Services, we must
ensure that no two Services can collide. Kubernetes does that by allocating each
Service its own IP address.

To ensure each Service receives a unique IP, an internal allocator atomically
updates a global allocation map in {{< glossary_tooltip term_id="etcd" >}}
prior to creating each Service. The map object must exist in the registry for
Services to get IP address assignments, otherwise creations will
fail with a message indicating an IP address could not be allocated.

In the control plane, a background controller is responsible for creating that
map (needed to support migrating from older versions of Kubernetes that used
in-memory locking). Kubernetes also uses controllers to check for invalid
assignments (eg due to administrator intervention) and for cleaning up allocated
IP addresses that are no longer used by any Services.
 -->

### 避免冲突

k8s 一个主要的宗旨就是让用户不是因为自己的错误而引起出错， 就拿 Service 资源的设计来说，就是如果
你选择的端口可能与别人选择的端口可能冲突，则不你来选择这个端口。 这是一种故障隔离。

为了允许用户为 Service  选择一个端口，我们必须要确保不能有两个 Service 端口冲突。k8s 通过为每
个 Service 分配 IP 地址来避免这个问题。

为了保证每个 Service 接收的 IP 地址都是唯一的，在创建每个 Service 之间一个内部分配器原子地
在 {{< glossary_tooltip term_id="etcd" >}} 中更新一个全局的分配字典。 必须要能在这个映射中
为 Service 分配指定 IP， 否则就会失败，错误信息为不能分配该 IP 地址。

在控制中心中，一个后台控制器负载创建这个映射(需要支持使用内存锁的老版本k8s迁移)。k8s 还使用
控制器检查每个分配(例如，因为管理员介入)并清除那些不被任何 Service 使用的 IP 地址。
<!--
### Service IP addresses {#ips-and-vips}

Unlike Pod IP addresses, which actually route to a fixed destination,
Service IPs are not actually answered by a single host.  Instead, kube-proxy
uses iptables (packet processing logic in Linux) to define _virtual_ IP addresses
which are transparently redirected as needed.  When clients connect to the
VIP, their traffic is automatically transported to an appropriate endpoint.
The environment variables and DNS for Services are actually populated in
terms of the Service's virtual IP address (and port).

kube-proxy supports three proxy modes&mdash;userspace, iptables and IPVS&mdash;which
each operate slightly differently.
 -->

### Service IP 地址 {#ips-and-vips}

与 Pod 的 IP 地址不同， 当 Pod 实际是路由到一个固定的地址， Service  IP 通常不是由单个主机响应的。
而是 kube-proxy 使用 iptables (Linux 中的包处理逻辑) 来定义一个 _虚拟的_ IP 地址，根据需要透明地
转发流量。 当客户端连接到 VIP 时， 它们的流量自动传输到恰当和端点上。 Pod 中被注入的环境变量
和 Service DNS 记录指向的也是 Service 的虚拟 IP 地址(和端口)。

kube-proxy 支持三种代理模式 &mdash;userspace, iptables 和 IPVS， 它们之间的运转方式各有不同。
<!--
#### Userspace

As an example, consider the image processing application described above.
When the backend Service is created, the Kubernetes master assigns a virtual
IP address, for example 10.0.0.1.  Assuming the Service port is 1234, the
Service is observed by all of the kube-proxy instances in the cluster.
When a proxy sees a new Service, it opens a new random port, establishes an
iptables redirect from the virtual IP address to this new port, and starts accepting
connections on it.

When a client connects to the Service's virtual IP address, the iptables
rule kicks in, and redirects the packets to the proxy's own port.
The "Service proxy" chooses a backend, and starts proxying traffic from the client to the backend.

This means that Service owners can choose any port they want without risk of
collision.  Clients can simply connect to an IP and port, without being aware
of which Pods they are actually accessing.
 -->

#### userspace

例如， 上面提到过那个处理图片的应用。 当后端的 Service 创建时， k8s 控制中心为它分配一个虚拟
的 IP 地址，假如是 `10.0.0.1`。 假定 Service 的端口为 `1234`， 该 Service 会被集群中所有
的 kube-proxy 实例监控。 当 一个代理发现一个新的 Service， 它会打开一个随机端口， 创建一个
iptables 规则将虚拟 IP 地址重定向到这个新创建的端口，并开始接收连接。

当有一个客户端连接到这个 Service 的虚拟 IP 时， iptables 规则工作将包转发到代理自己的端口。
然后 Service 的代理选择后端，然后开始将代理从客户端到后端的流量。

这么做的意义在于 Service 拥有者可以选择任意端口这就避免的端口冲突的风险。 客户端只是简单地连接
到一个 IP 地址和端口， 并不会感知到它实际上是连接到后端的哪个 Pod。
<!--
#### iptables

Again, consider the image processing application described above.
When the backend Service is created, the Kubernetes control plane assigns a virtual
IP address, for example 10.0.0.1.  Assuming the Service port is 1234, the
Service is observed by all of the kube-proxy instances in the cluster.
When a proxy sees a new Service, it installs a series of iptables rules which
redirect from the virtual IP address  to per-Service rules.  The per-Service
rules link to per-Endpoint rules which redirect traffic (using destination NAT)
to the backends.

When a client connects to the Service's virtual IP address the iptables rule kicks in.
A backend is chosen (either based on session affinity or randomly) and packets are
redirected to the backend.  Unlike the userspace proxy, packets are never
copied to userspace, the kube-proxy does not have to be running for the virtual
IP address to work, and Nodes see traffic arriving from the unaltered client IP
address.

This same basic flow executes when traffic comes in through a node-port or
through a load-balancer, though in those cases the client IP does get altered.
 -->

#### iptables

再来，还是上面那个处理图片的应用。 当后端的 Service 创建后， k8s 控制中心分配了一个虚拟IP地址
，假如是 `10.0.0.1`.假定 Service 的端口为 `1234`， 该 Service 会被集群中所有 的 kube-proxy 实例监控。
当代理发现一个新的 Service, 它会插入一系统 iptables 规则， 这些规则将重定向到虚拟IP地址的流量
到每个 Service 的规则。 每个 Service 的规则又与每个 Endpoint 的规则相连，将流量重定向(通过目标 NAT)
到后端。

当一个客户端连接到 Service 的虚拟 IP 地址时， iptables 开始插一脚。 选择一个后端(要么基于会话粘性，要么随机)
并将数据包转发到该后端。 与 userspace 代理模式不同， 网络包不会拷贝到用户空间，kube-proxy 也不
需要为为虚拟 IP 运行工作任务， 节点可以看到接收到流量中未修改的客户端 IP 地址

来自 NodePort 或 负载均衡器的流量也是使用这样的基本执行流程， 在这种情况下客户端 IP 不会被修改。
<!--
#### IPVS

iptables operations slow down dramatically in large scale cluster e.g 10,000 Services.
IPVS is designed for load balancing and based on in-kernel hash tables. So you can achieve performance consistency in large number of Services from IPVS-based kube-proxy. Meanwhile, IPVS-based kube-proxy has more sophisticated load balancing algorithms (least conns, locality, weighted, persistence).
 -->

#### IPVS

iptables 在大规模集群中(如，上万个 Service)是急剧下降。IPVS 设计上是基于内核内部的哈希表来
实现负载均衡的。 所以基于 IPVS 的 kube-proxy 可以实现在大量 Service 的情况下性能稳定。
同时基于 IPVS kube-proxy 也包含更丰富的负载均衡算法(最少连接，位置，权重，维持)
<!--
## API Object

Service is a top-level resource in the Kubernetes REST API. You can find more details
about the API object at: [Service API object](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#service-v1-core).
 -->
## API 对象

Service is a top-level resource in the Kubernetes REST API. You can find more details
about the API object at: [Service API object](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#service-v1-core).
Service 是 k8s REST API 中的顶级资源， 更多相关信息见
[Service API 对象](https://kubernetes.io/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#service-v1-core).
<!--
## Supported protocols {#protocol-support}

### TCP

You can use TCP for any kind of Service, and it's the default network protocol.

### UDP

You can use UDP for most Services. For type=LoadBalancer Services, UDP support
depends on the cloud provider offering this facility.

### HTTP

If your cloud provider supports it, you can use a Service in LoadBalancer mode
to set up external HTTP / HTTPS reverse proxying, forwarded to the Endpoints
of the Service.

{{< note >}}
You can also use {{< glossary_tooltip term_id="ingress" >}} in place of Service
to expose HTTP / HTTPS Services.
{{< /note >}}
 -->

## 支持的协议 {#protocol-support}

### TCP

TCP 可用于任意类型的 Service， 也是 Service  的默认网络协议

### UDP

UDP 可用于大多数 Service, 对于 type=LoadBalancer 的 Service, 是否支持 UDP 基于云提供商
是否提供该功能。

### HTTP

如果云提供商支持，则可以使用 type=LoadBalancer 的 Service 通过外部的 HTTP / HTTPS 反向代理
转发到 Service 的 Endpoint

{{< note >}}
在 Service 暴露的是 HTTP / HTTPS 时也可以使用 {{< glossary_tooltip term_id="ingress" >}}
{{< /note >}}
<!--
### PROXY protocol

If your cloud provider supports it (eg, [AWS](/docs/concepts/cluster-administration/cloud-providers/#aws)),
you can use a Service in LoadBalancer mode to configure a load balancer outside
of Kubernetes itself, that will forward connections prefixed with
[PROXY protocol](https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt).

The load balancer will send an initial series of octets describing the
incoming connection, similar to this example

```
PROXY TCP4 192.0.2.202 10.0.42.7 12345 7\r\n
```
followed by the data from the client.
 -->

### PROXY 协议

如果云提供商支持， (比如 , [AWS](/docs/concepts/cluster-administration/cloud-providers/#aws))，
可以在使用 LoadBalancer 类型的 Service 时在 k8s 外部配置负载均衡器， 它会转发带前缀
[PROXY 协议](https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt).
的连接。

负载均衡器会发送一个初始化八进制序列描述进入的连接，类似如下

```
PROXY TCP4 192.0.2.202 10.0.42.7 12345 7\r\n
```
紧哪关就是客户端发送的数据。
<!--
### SCTP

{{< feature-state for_k8s_version="v1.19" state="beta" >}}

Kubernetes supports SCTP as a `protocol` value in Service, Endpoints, EndpointSlice, NetworkPolicy and Pod definitions. As a beta feature, this is enabled by default. To disable SCTP at a cluster level, you (or your cluster administrator) will need to disable the `SCTPSupport` [feature gate](/docs/reference/command-line-tools-reference/feature-gates/) for the API server with `--feature-gates=SCTPSupport=false,…`.

When the feature gate is enabled, you can set the `protocol` field of a Service, Endpoints, EndpointSlice, NetworkPolicy or Pod to `SCTP`. Kubernetes sets up the network accordingly for the SCTP associations, just like it does for TCP connections.
 -->

### SCTP

{{< feature-state for_k8s_version="v1.19" state="beta" >}}

k8s 支持在 Service, Endpoints, EndpointSlice, NetworkPolicy 和 Pod 定义中 `protocol`
的值为 `SCTP`。 因为这是一个 beta 版本的特性，所以默认是开启的。要在集群级别禁用 `SCTP`，
需要在 api-server 上通过设置 `--feature-gates=SCTPSupport=false,…` 禁用 `SCTPSupport`
[功能阀](/docs/reference/command-line-tools-reference/feature-gates/)

当该特性被启用时，可以设置 Service, Endpoints, EndpointSlice, NetworkPolicy, Pod 的
`protocol` 为 `SCTP`。 k8s 会根据 SCTP 设置网络，就像设置 TCP 连接一样。
<!--
#### Warnings {#caveat-sctp-overview}

##### Support for multihomed SCTP associations {#caveat-sctp-multihomed}

{{< warning >}}
The support of multihomed SCTP associations requires that the CNI plugin can support the assignment of multiple interfaces and IP addresses to a Pod.

NAT for multihomed SCTP associations requires special logic in the corresponding kernel modules.
{{< /warning >}}

##### Service with type=LoadBalancer {#caveat-sctp-loadbalancer-service-type}

{{< warning >}}
You can only create a Service with `type` LoadBalancer plus `protocol` SCTP if the cloud provider's load balancer implementation supports SCTP as a protocol. Otherwise, the Service creation request is rejected. The current set of cloud load balancer providers (Azure, AWS, CloudStack, GCE, OpenStack) all lack support for SCTP.
{{< /warning >}}

##### Windows {#caveat-sctp-windows-os}

{{< warning >}}
SCTP is not supported on Windows based nodes.
{{< /warning >}}

##### Userspace kube-proxy {#caveat-sctp-kube-proxy-userspace}

{{< warning >}}
The kube-proxy does not support the management of SCTP associations when it is in userspace mode.
{{< /warning >}}
 -->

#### 警告 {#caveat-sctp-overview}

##### SCTP 多重连接支持 {#caveat-sctp-multihomed}

{{< warning >}}
SCTP 多重连接支持的前提是 CNI 插件支持为一个 Pod 分配多个网上和IP地址
SCTP 多重连接的 NAT 需要在对应的逻辑模块中有特殊逻辑
{{< /warning >}}

##### type=LoadBalancer 的 Service {#caveat-sctp-loadbalancer-service-type}

{{< warning >}}
如有在云提供商的负载均衡器实现了对 `SCTP` 协议支持时才能够创建一个类型为 LoadBalancer 并且
`protocol` 为 SCTP 的 Service. 否则 Service 的创建请求会被拒绝。 目前的云负载均衡提供者
(Azure, AWS, CloudStack, GCE, OpenStack) 都缺乏对 SCTP 的支持。
{{< /warning >}}

##### Windows {#caveat-sctp-windows-os}

{{< warning >}}
基于 Windows 的节点不支持 SCTP
{{< /warning >}}

##### userspace kube-proxy {#caveat-sctp-kube-proxy-userspace}

{{< warning >}}
使用 userspace 模式的 kube-proxy 不支持对 SCTP 连接的管理
{{< /warning >}}

{{< todo-optimize text="对SCTP不了解">}}

## {{% heading "whatsnext" %}}


* 概念 [通过 Service 连接应用](/k8sDocs/docs/concepts/services-networking/connect-applications-service/)
* 概念 [Ingress](/k8sDocs/docs/concepts/services-networking/ingress/)
* 概念 [EndpointSlices](/k8sDocs/docs/concepts/services-networking/endpoint-slices/)
