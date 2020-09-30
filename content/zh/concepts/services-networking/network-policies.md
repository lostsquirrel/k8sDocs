---
title: 网络策略
content_type: concept
weight: 80
date: 2020-09-23
publishdate: 2020-09-23
draft: true
---
<!--
---
reviewers:
- thockin
- caseydavenport
- danwinship
title: Network Policies
content_type: concept
weight: 50
---
 -->
<!-- overview -->
<!--
If you want to control traffic flow at the IP address or port level (OSI layer 3 or 4), then you might consider using Kubernetes NetworkPolicies for particular applications in your cluster.  NetworkPolicies are an application-centric construct which allow you to specify how a {{< glossary_tooltip text="pod" term_id="pod">}} is allowed to communicate with various network "entities" (we use the word "entity" here to avoid overloading the more common terms such as "endpoints" and "services", which have specific Kubernetes connotations) over the network.

The entities that a Pod can communicate with are identified through a combination of the following 3 identifiers:

1. Other pods that are allowed (exception: a pod cannot block access to itself)
2. Namespaces that are allowed
3. IP blocks (exception: traffic to and from the node where a Pod is running is always allowed, regardless of the IP address of the Pod or the node)

When defining a pod- or namespace- based NetworkPolicy, you use a {{< glossary_tooltip text="selector" term_id="selector">}} to specify what traffic is allowed to and from the Pod(s) that match the selector.

Meanwhile, when IP based NetworkPolicies are created, we define policies based on IP blocks (CIDR ranges).
 -->

如果想要在 IP 地址或端口层(OSI 3 或 4 层)控制网络流量，可以考虑对集群中的特定应用使用 k8s 网络策略。
网络策略是一个应用为中心的构造，它允许用户指定怎么控制一个 {{< glossary_tooltip text="pod" term_id="pod">}} 以允许它与其它多种网络实体(这里使用 实体(entity)避免与 "Endpoint" "Service"
这此在 k8s 中有明确含义的词混淆)通信。

Pod 可以与之通信的实体可以由以下三个标识组合来识别:

1. 其它被允许的 Pod (例外: Pod 不可以禁止与它本身通信)
2. 被允许的命名空间
3. IP 段(例外: Pod 所在的节点进出流量都是允许的，Pod 和 节点的 IP 如果在禁用段则会被忽略)

在定义一个基于 Pod 或 命名空间的网络策略时，可以使用{{< glossary_tooltip term_id="selector">}}
来指定只与选择器相匹配的 Pod 在允许与之进行流量往来。

同时，当创建基于 IP 的网络策略时，定义的策略基于 IP 段(CIDR 范围)。
<!-- body -->
<!--
## Prerequisites

Network policies are implemented by the [network plugin](/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/). To use network policies, you must be using a networking solution which supports NetworkPolicy. Creating a NetworkPolicy resource without a controller that implements it will have no effect.
-->

## 前置条件

网络策略是由
[网络插件](/k8sDocs/concepts/extend-kubernetes/compute-storage-net/network-plugins/)
实现的。 要使用网络策略，就必要使用一个支持网络策略的网络解决方案。 创建一个没有实现的控制器的
`NetworkPolicy` 资源是没有效果的
<!--
## Isolated and Non-isolated Pods

By default, pods are non-isolated; they accept traffic from any source.

Pods become isolated by having a NetworkPolicy that selects them. Once there is any NetworkPolicy in a namespace selecting a particular pod, that pod will reject any connections that are not allowed by any NetworkPolicy. (Other pods in the namespace that are not selected by any NetworkPolicy will continue to accept all traffic.)

Network policies do not conflict; they are additive. If any policy or policies select a pod, the pod is restricted to what is allowed by the union of those policies' ingress/egress rules. Thus, order of evaluation does not affect the policy result.
 -->

## 隔离和非隔离的 Pod

默认情况下，所有的 Pod 都是非隔离的; 它们可以接收来自任意源的流量。

当 Pod 被一个 NetworkPolicy 选中时就会变成隔离的。 当在一个命名空间中有任意 NetworkPolicy
选中了某个 Pod， 这个 Pod 就会拒绝那些不被这些 NetworkPolicy 允许的连接就会被拒绝。(同一个
命名空间中其它没有被任何 NetworkPolicy 选择的 Pod 依然继续接收所有流量。)

网络策略不会冲突； 它们可叠加。 如果任意策略选中了一个 Pod， 该 Pod 就会被限制在这些策略的
`ingress/egress` 规则交集允许的范围。 因此，执行的顺序并不会影响策略的结果。
<!--
## The NetworkPolicy resource {#networkpolicy-resource}

See the [NetworkPolicy](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#networkpolicy-v1-networking-k8s-io) reference for a full definition of the resource.

An example NetworkPolicy might look like this:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

{{< note >}}
POSTing this to the API server for your cluster will have no effect unless your chosen networking solution supports network policy.
{{< /note >}}

__Mandatory Fields__: As with all other Kubernetes config, a NetworkPolicy
needs `apiVersion`, `kind`, and `metadata` fields.  For general information
about working with config files, see
[Configure Containers Using a ConfigMap](/docs/tasks/configure-pod-container/configure-pod-configmap/),
and [Object Management](/docs/concepts/overview/working-with-objects/object-management).

__spec__: NetworkPolicy [spec](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#spec-and-status) has all the information needed to define a particular network policy in the given namespace.

__podSelector__: Each NetworkPolicy includes a `podSelector` which selects the grouping of pods to which the policy applies. The example policy selects pods with the label "role=db". An empty `podSelector` selects all pods in the namespace.

__policyTypes__: Each NetworkPolicy includes a `policyTypes` list which may include either `Ingress`, `Egress`, or both. The `policyTypes` field indicates whether or not the given policy applies to ingress traffic to selected pod, egress traffic from selected pods, or both. If no `policyTypes` are specified on a NetworkPolicy then by default `Ingress` will always be set and `Egress` will be set if the NetworkPolicy has any egress rules.

__ingress__: Each NetworkPolicy may include a list of allowed `ingress` rules.  Each rule allows traffic which matches both the `from` and `ports` sections. The example policy contains a single rule, which matches traffic on a single port, from one of three sources, the first specified via an `ipBlock`, the second via a `namespaceSelector` and the third via a `podSelector`.

__egress__: Each NetworkPolicy may include a list of allowed `egress` rules.  Each rule allows traffic which matches both the `to` and `ports` sections. The example policy contains a single rule, which matches traffic on a single port to any destination in `10.0.0.0/24`.

So, the example NetworkPolicy:

1. isolates "role=db" pods in the "default" namespace for both ingress and egress traffic (if they weren't already isolated)
2. (Ingress rules) allows connections to all pods in the "default" namespace with the label "role=db" on TCP port 6379 from:

   * any pod in the "default" namespace with the label "role=frontend"
   * any pod in a namespace with the label "project=myproject"
   * IP addresses in the ranges 172.17.0.0–172.17.0.255 and 172.17.2.0–172.17.255.255 (ie, all of 172.17.0.0/16 except 172.17.1.0/24)
3. (Egress rules) allows connections from any pod in the "default" namespace with the label "role=db" to CIDR 10.0.0.0/24 on TCP port 5978

See the [Declare Network Policy](/docs/tasks/administer-cluster/declare-network-policy/) walkthrough for further examples.
-->

## 网络策略资源 {#networkpolicy-resource}

可以在 [NetworkPolicy](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#networkpolicy-v1-networking-k8s-io) 查阅详细定义文档.

一个 NetworkPolicy 示例可能长成这个样子:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

{{< note >}}
如果集群的网络解决方案(网络插件)不支持网络策略，POST 这个配置到集群 api-server 将是没有效果的
也就是说要使用网络策略，就必须要安装支持网络策略的网络插件。
{{< /note >}}

__必要字段__: 与其它所有的 k8s 配置一样， NetworkPolicy 必须有 `apiVersion`, `kind`, `metadata`
字段。 配置文件常用信息请见
[使用 ConfigMap 配置容器](/k8sDocs/tasks/configure-pod-container/configure-pod-configmap/),
和 [对象管理](/k8sDocs/concepts/overview/working-with-objects/object-management).

__spec__: NetworkPolicy [spec](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#spec-and-status) 包含了在指定命名空间创建特定网络策略的所有信息

__podSelector__: 每个 NetworkPolicy 都包含了一个 `podSelector` 字段，用户选择应用该策略的 Pod 组。
上面例子中的策略选择包含 "`role=db`" 标签的 Pod。 如果 `podSelector` 是空则选择该命名空间中所有的 Pod。

__policyTypes__: 每个 NetworkPolicy 包含一个 `policyTypes` 字段，该字段值为一个列表，
这个列表的值可以是 `Ingress`, `Egress` 中的任意一个或同时包含两个。 该字段值是否包含
`Ingress` 表示是否将该策略执行到输入流量到达选定的 Pod；
`Egress` 表示是否将该策略执行到选择 Pod 输出的流量。
如果没有指定 `policyTypes` 字段，则默认情况下 `Ingress` 问题要设置，而 `Egress` 则只在
NetworkPolicy 中包含 `egress` 规则是才设置。

__ingress__: 每个 NetworkPolicy 可能包含一个被允许的 `ingress` 规则列表。
每个规则所允许的流量会同时匹配 `from` 和 `ports` 两个部分。上面例子中的策略就包含一个规则
这个规则匹配来自一个端口加上三个源(第一个是通过 `ipBlock`指定一个 IP 段； 第三个是通过
`namespaceSelector`；第三个是通过 `podSelector`)中的一个的组合的流量

__egress__: 每个 NetworkPolicy 可能包含一个被允许的 `egress` 规则列表。 每个规则允许的流量
由 `to` 和 `ports` 组合匹配。 上面的例子中的策略包含一个规则，这个规则匹配一个端口加上
`10.0.0.0/24` 中的任意一个目标的组合的流量。

因此，上面例子中 NetworkPolicy 如下:

1. 在默认(default)命名空间中隔离 "role=db" 所选择 Pod 的 `ingress` 和 `egress` 流量(如果它们还没有被隔离)

2. (Ingress 规则)允许以下范围的所有连接到默认(default)命名空间中包含 "role=db" 标签的 Pod 的 TCP 端口 6379:
  - 默认(default)命名空间中包含 "role=frontend" 标签的任意 Pod
  - 所在命名空间包含 "project=myproject" 标签的任意 Pod
  - `172.17.0.0–172.17.0.255` 和 `172.17.2.0–172.17.255.255` IP 段中的地址(
    `172.17.0.0/16` 中除了 `172.17.1.0/24` 之外的地址
    )

3. (Egress 规则) 允许 默认(default)命名空间中包含 "role=db" 的任意 Pod 到
  CIDR 10.0.0.0/24 TCP 端口 5978 的所有连接

更多示例见 [声明网络策略](/k8sDocs/tasks/administer-cluster/declare-network-policy/)
<!--
## Behavior of `to` and `from` selectors

There are four kinds of selectors that can be specified in an `ingress` `from` section or `egress` `to` section:

__podSelector__: This selects particular Pods in the same namespace as the NetworkPolicy which should be allowed as ingress sources or egress destinations.

__namespaceSelector__: This selects particular namespaces for which all Pods should be allowed as ingress sources or egress destinations.

__namespaceSelector__ *and* __podSelector__: A single `to`/`from` entry that specifies both `namespaceSelector` and `podSelector` selects particular Pods within particular namespaces. Be careful to use correct YAML syntax; this policy:

```yaml
  ...
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          user: alice
      podSelector:
        matchLabels:
          role: client
  ...
```

contains a single `from` element allowing connections from Pods with the label `role=client` in namespaces with the label `user=alice`. But *this* policy:

```yaml
  ...
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          user: alice
    - podSelector:
        matchLabels:
          role: client
  ...
```

contains two elements in the `from` array, and allows connections from Pods in the local Namespace with the label `role=client`, *or* from any Pod in any namespace with the label `user=alice`.

When in doubt, use `kubectl describe` to see how Kubernetes has interpreted the policy.

__ipBlock__: This selects particular IP CIDR ranges to allow as ingress sources or egress destinations. These should be cluster-external IPs, since Pod IPs are ephemeral and unpredictable.

Cluster ingress and egress mechanisms often require rewriting the source or destination IP
of packets. In cases where this happens, it is not defined whether this happens before or
after NetworkPolicy processing, and the behavior may be different for different
combinations of network plugin, cloud provider, `Service` implementation, etc.

In the case of ingress, this means that in some cases you may be able to filter incoming
packets based on the actual original source IP, while in other cases, the "source IP" that
the NetworkPolicy acts on may be the IP of a `LoadBalancer` or of the Pod's node, etc.

For egress, this means that connections from pods to `Service` IPs that get rewritten to
cluster-external IPs may or may not be subject to `ipBlock`-based policies.
 -->

## `to` 和 `from` 选择器的行为方式

在 `ingress` 的 `from` 区 或 `egress` 的 `to` 区可以指定以下四种选择器：

__podSelector__:  选择与 NetworkPolicy 在同一个命名空间的指定 Pod 允许为 `ingress` 的源() 或 `egress` 的目的
__namespaceSelector__: 选择指定名称空间，其中所有的 Pod 允许为 `ingress` 的源或 `egress` 的目的
__namespaceSelector__ *加* __podSelector__: 单个 `to`/`from` 条目可以同时指定 `namespaceSelector` 和 `podSelector`
来选择指定命名空间的指定 Pod。 需要注意正确地使用 YAML 语法。
```yaml
  ...
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          user: alice
      podSelector:
        matchLabels:
          role: client
  ...
```

上面的这个策略包含一个 `from` 元素，它允许连接的 Pod 在包含 `user=alice` 标签的命名空间中
并且要包含 `role=client` 标签。

```yaml
  ...
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          user: alice
    - podSelector:
        matchLabels:
          role: client
  ...
```

这个策略包含两上 `from` 元素的数据， 它允许连接的 Pod 有: 同命名空间中有 `role=client` 标签的 Pod
*或* 任意命名空间中包含 `user=alice` 标签的任意 Pod

当搞不清楚的时候，就使用 `kubectl describe` 来看 k8s 是怎么来解释这个策略的。

__ipBlock__: 指定 IP CIDR 范围允许为 `ingress` 的源或 `egress` 的目的。这个 IP 范围
应该是集群外部 IP， 因为 Pod 的 IP 是临时的并且是不可预测的。

集群的 `ingress` 和 `egress` 经常需要重写包的源或目的 IP。 如果发生了这种情况， 它发生在
NetworkPolicy 之前还是之后不不确定的，由网络插件，云提供商，`Service` 实现的不同而有不同的行为方式

对于 `ingress`, 就意味着在某些情况下可能可以通过实际的原始源 IP 地址来过滤包，在另一情况下,
NetworkPolicy 处理的的 "源 IP" 可能是 `LoadBalancer` 的 IP 或 Pod 所在节点的 IP 等。

对于 `egress`， 这就意味着 Pod 连接的目标 `Service` IP 在重写到集群外部IP 就可能受 `ipBlock`
策略控制，也可能不受控制。
<!--
## Default policies

By default, if no policies exist in a namespace, then all ingress and egress traffic is allowed to and from pods in that namespace. The following examples let you change the default behavior
in that namespace.
 -->

## 默认策略

默认情况下，如果一个命名空间中没有策略存在，那这个命名空间中的 Pod 所有的 `ingress` 和 `egress`
流量都是允许的。 下面的例子让用户可以修改那个命名空间中的默认行为。
<!--
### Default deny all ingress traffic

You can create a "default" isolation policy for a namespace by creating a NetworkPolicy that selects all pods but does not allow any ingress traffic to those pods.

{{< codenew file="service/networking/network-policy-default-deny-ingress.yaml" >}}

This ensures that even pods that aren't selected by any other NetworkPolicy will still be isolated. This policy does not change the default egress isolation behavior.
 -->

### 默认拒绝所有 `ingress` 流量

可以能过创建一个 NetworkPolicy 为命名空间创建一个"默认"的隔离策略。这个 NetworkPolicy
会选择所有的 Pod 并且不允许任何 `ingress` 流量到这些 Pod。

{{< codenew file="service/networking/network-policy-default-deny-ingress.yaml" >}}

这样可以保证即便有些 Pod 没有被其它任意 NetworkPolicy 选择还是会被隔离。
这个策略不会影响默认的 `egress` 隔离行为。
<!--
### Default allow all ingress traffic

If you want to allow all traffic to all pods in a namespace (even if policies are added that cause some pods to be treated as "isolated"), you can create a policy that explicitly allows all traffic in that namespace.

{{< codenew file="service/networking/network-policy-allow-all-ingress.yaml" >}}
 -->

### 默认允许所有 `ingress` 流量

如果想要在一个命名空间中允许到达所有 Pod 的所有流量(即便已经添加了一些策略导致有些 Pod 被认为是隔离的)
可以通过创建一个策略显式地允许这个命名空间中的所有离题。

{{< codenew file="service/networking/network-policy-allow-all-ingress.yaml" >}}
<!--
### Default deny all egress traffic

You can create a "default" egress isolation policy for a namespace by creating a NetworkPolicy that selects all pods but does not allow any egress traffic from those pods.

{{< codenew file="service/networking/network-policy-default-deny-egress.yaml" >}}

This ensures that even pods that aren't selected by any other NetworkPolicy will not be allowed egress traffic. This policy does not
change the default ingress isolation behavior.
 -->

### 默认拒绝所有 `egress` 流量

可以通过创建一个选择所有 Pod 并不接受任何来自这些 Pod 的 egress 流量的 NetworkPolicy 来作为
这个命名空间的默认 `egress` 隔离策略。

{{< codenew file="service/networking/network-policy-default-deny-egress.yaml" >}}

这样可以保证即便有 Pod 没有被其它任意 NetworkPolicy 选中，也不会被允许其 egress 流量。
这个策略不影响默认 `ingress` 隔离行为。
<!--
### Default allow all egress traffic

If you want to allow all traffic from all pods in a namespace (even if policies are added that cause some pods to be treated as "isolated"), you can create a policy that explicitly allows all egress traffic in that namespace.

{{< codenew file="service/networking/network-policy-allow-all-egress.yaml" >}}
 -->

### 默认允许所有 `egress` 流量

如果想要允许一个命名空间中来自所有 Pod 的所有流量(即便已经添加了一些策略导致有些 Pod 被认为是隔离的)，
也可以通过创建一个策略显式地允许这个命名空间中所有的 `egress` 流量。

{{< codenew file="service/networking/network-policy-allow-all-egress.yaml" >}}
<!--
### Default deny all ingress and all egress traffic

You can create a "default" policy for a namespace which prevents all ingress AND egress traffic by creating the following NetworkPolicy in that namespace.

{{< codenew file="service/networking/network-policy-default-deny-all.yaml" >}}

This ensures that even pods that aren't selected by any other NetworkPolicy will not be allowed ingress or egress traffic.
 -->

### 拒绝所有 `ingress` 和所有 `egress` 流量

You can create a "default" policy for a namespace which prevents all ingress AND egress traffic by creating the following NetworkPolicy in that namespace.
可以在一个命名空间中创建一个如下默认的 NetworkPolicy，同时禁止所有的 `ingress` 和 `egress` 流量。

{{< codenew file="service/networking/network-policy-default-deny-all.yaml" >}}

这样确保即便有 Pod 没有被其它任意 NetworkPolicy 选中，也不会被允许其 `ingress` 或 `egress` 流量
<!--
## SCTP support

{{< feature-state for_k8s_version="v1.19" state="beta" >}}

As a beta feature, this is enabled by default. To disable SCTP at a cluster level, you (or your cluster administrator) will need to disable the `SCTPSupport` [feature gate](/docs/reference/command-line-tools-reference/feature-gates/) for the API server with `--feature-gates=SCTPSupport=false,…`.
When the feature gate is enabled, you can set the `protocol` field of a NetworkPolicy to `SCTP`.

{{< note >}}
You must be using a {{< glossary_tooltip text="CNI" term_id="cni" >}} plugin that supports SCTP protocol NetworkPolicies.
{{< /note >}}
 -->

## SCTP 支持

{{< feature-state for_k8s_version="v1.19" state="beta" >}}

作为一个 `beta` 版本的特性，默认是开启的。要在集群级别禁用 SCTP， 需要在 api-server 使用
`--feature-gates=SCTPSupport=false,…` 禁用
`SCTPSupport` [功能特性阀](/k8sDocs/reference/command-line-tools-reference/feature-gates/)
当这个特性被启用时，可能在 NetworkPolicy 的 `protocol` 字段使用 `SCTP` 协议。
{{< note >}}
必须要使用一个支持 NetworkPolicy SCTP 协议的 {{< glossary_tooltip text="CNI" term_id="cni" >}} 插件
{{< /note >}}
<!--
# What you CAN'T do with network policy's (at least, not yet)

As of Kubernetes 1.20, the following functionality does not exist in the NetworkPolicy API, but you might be able to implement workarounds using Operating System components (such as SELinux, OpenVSwitch, IPTables, and so on) or Layer 7 technologies (Ingress controllers, Service Mesh implementations) or admission controllers.  In case you are new to network security in Kubernetes, its worth noting that the following User Stories cannot (yet) be implemented using the NetworkPolicy API.  Some (but not all) of these user stories are actively being discussed for future releases of the NetworkPolicy API.

- Forcing internal cluster traffic to go through a common gateway (this might be best served with a service mesh or other proxy).
- Anything TLS related (use a service mesh or ingress controller for this).
- Node specific policies (you can use CIDR notation for these, but you cannot target nodes by their Kubernetes identities specifically).
- Targeting of namespaces or services by name (you can, however, target pods or namespaces by their{{< glossary_tooltip text="labels" term_id="label" >}}, which is often a viable workaround).
- Creation or management of "Policy requests" that are fulfilled by a third party.
- Default policies which are applied to all namespaces or pods (there are some third party Kubernetes distributions and projects which can do this).
- Advanced policy querying and reachability tooling.
- The ability to target ranges of Ports in a single policy declaration.
- The ability to log network security events (for example connections that are blocked or accepted).
- The ability to explicitly deny policies (currently the model for NetworkPolicies are deny by default, with only the ability to add allow rules).
- The ability to prevent loopback or incoming host traffic (Pods cannot currently block localhost access, nor do they have the ability to block access from their resident node).
 -->

# 不能通过网络策略做到的情况 (至少目前还不能)

到 k8s v1.20 为止，以下功能在 NetworkPolicy API 中是不存在的， 但可能可能通过操作系统组件
(如 SELinux， OpenVSwitch， IPTables 等) 或者通过 7 层技术(Ingress 控制器，`Service Mesh` 实现)
或者 准入控制器(admission controller) 来曲线救国。 假如用户对 k8s 网络安全了解比较少，
需要注意下面的用户故事(User Stories)(还)不能通过使用 NetworkPolicy API 实现。
有些(但不是所有)正在被讨论加入到未来版本中的 NetworkPolicy API 中。

- Forcing internal cluster traffic to go through a common gateway (this might be best served with a service mesh or other proxy).
- 强制集群内流量通过一个通用网关(这个最好使用服务网格或其它代理来提供)
- Anything TLS related (use a service mesh or ingress controller for this).
- 所有与 TLS 相关的事(使用服务网格或 Ingress 控制器来实现)
- Node specific policies (you can use CIDR notation for these, but you cannot target nodes by their Kubernetes identities specifically).
- 节点特有的策略(可以使用 CIDR 标注，但是不能能过 k8s 标识来特指一些节点)(说的啥？)
- Targeting of namespaces or services by name (you can, however, target pods or namespaces by their{{< glossary_tooltip text="labels" term_id="label" >}}, which is often a viable workaround).
- Creation or management of "Policy requests" that are fulfilled by a third party.
- Default policies which are applied to all namespaces or pods (there are some third party Kubernetes distributions and projects which can do this).
- Advanced policy querying and reachability tooling.
- The ability to target ranges of Ports in a single policy declaration.
- The ability to log network security events (for example connections that are blocked or accepted).
- The ability to explicitly deny policies (currently the model for NetworkPolicies are deny by default, with only the ability to add allow rules).
- The ability to prevent loopback or incoming host traffic (Pods cannot currently block localhost access, nor do they have the ability to block access from their resident node).

## {{% heading "whatsnext" %}}


- See the [Declare Network Policy](/docs/tasks/administer-cluster/declare-network-policy/)
  walkthrough for further examples.
- See more [recipes](https://github.com/ahmetb/kubernetes-network-policy-recipes) for common scenarios enabled by the NetworkPolicy resource.
