---
title: Service 和 Pod 的 DNS
weight: 30
date: 2020-06-30
publishdate: 2020-09-10
---
<!--
---
reviewers:
- davidopp
- thockin
title: DNS for Services and Pods
content_type: concept
weight: 20
---
 -->
<!-- overview -->
<!--
This page provides an overview of DNS support by Kubernetes.
 -->
本文简述 k8s 对 DNS 的支持
<!-- body -->
<!--
## Introduction

Kubernetes DNS schedules a DNS Pod and Service on the cluster, and configures
the kubelets to tell individual containers to use the DNS Service's IP to
resolve DNS names
 -->

## 介绍

k8s DNS 先在集群中部署了一个 DNS 的 Pod 和 Service， 并配置 kubelet 让每一个容器都使用
DNS Service 的 IP 来解析 DNS 名称。
<!--
### What things get DNS names?

Every Service defined in the cluster (including the DNS server itself) is
assigned a DNS name.  By default, a client Pod's DNS search list will
include the Pod's own namespace and the cluster's default domain.  This is best
illustrated by example:

Assume a Service named `foo` in the Kubernetes namespace `bar`.  A Pod running
in namespace `bar` can look up this service by simply doing a DNS query for
`foo`.  A Pod running in namespace `quux` can look up this service by doing a
DNS query for `foo.bar`.

The following sections detail the supported record types and layout that is
supported.  Any other layout or names or queries that happen to work are
considered implementation details and are subject to change without warning.
For more up-to-date specification, see
[Kubernetes DNS-Based Service Discovery](https://github.com/kubernetes/dns/blob/master/docs/specification.md).
 -->
### 啥东西会有 DNS 名称?

集群中定义的每一个 Service (包括 DNS 服务本身) 都会分配一个 DNS 名称。 默认情况下，一个客户端
Pod 的 DNS 检索列表会包含 Pod 自己所在的名字空间和集群的默认域。 一例胜千言:

假设在 k8s 的 `bar` 名字空间有一个叫 `foo` 的 Service. 一个运行在 `bar` 名字空间的 Pod
只需要简单地使用 `foo` 作为 DNS 查询条目就可以找到这个 Service。 另一个运行在 `quux` 名字空间的 Pod
同要为查询这个 Service。 DNS 的查询条目就需要是 `foo.bar` (带上名字空间的名称)

接下来的章节会详细介绍支持的 DNS 记录类型的规划。 (这一句没懂)
最新的规格说明书见
[Kubernetes DNS-Based Service Discovery](https://github.com/kubernetes/dns/blob/master/docs/specification.md).
<!--
## Services

### A/AAAA records

"Normal" (not headless) Services are assigned a DNS A or AAAA record,
depending on the IP family of the service, for a name of the form
`my-svc.my-namespace.svc.cluster-domain.example`.  This resolves to the cluster IP
of the Service.

"Headless" (without a cluster IP) Services are also assigned a DNS A or AAAA record,
depending on the IP family of the service, for a name of the form
`my-svc.my-namespace.svc.cluster-domain.example`.  Unlike normal
Services, this resolves to the set of IPs of the pods selected by the Service.
Clients are expected to consume the set or else use standard round-robin
selection from the set.

### SRV records

SRV Records are created for named ports that are part of normal or [Headless
Services](/docs/concepts/services-networking/service/#headless-services).
For each named port, the SRV record would have the form
`_my-port-name._my-port-protocol.my-svc.my-namespace.svc.cluster-domain.example`.
For a regular service, this resolves to the port number and the domain name:
`my-svc.my-namespace.svc.cluster-domain.example`.
For a headless service, this resolves to multiple answers, one for each pod
that is backing the service, and contains the port number and the domain name of the pod
of the form `auto-generated-name.my-svc.my-namespace.svc.cluster-domain.example`.
 -->
## Service

### A/AAAA 记录

"普通的"(不是无头(headless)的) Service 会基于使用的是 IPv4 还是 IPv6 分配一个 DNS A 或 AAAA 记录，
记录名为 `my-svc.my-namespace.svc.cluster-domain.example`。 记录的值是 Service 的
集群 IP (cluster IP)

无头(headless)(没有设置 `clusterIP`) Service 也会基于使用的是 IPv4 还是 IPv6
分配一个 DNS A 或 AAAA 记录，记录名为 `my-svc.my-namespace.svc.cluster-domain.example`，
与普通 Service 不同的是记录值是由 Service 选择的 Pod 的IP 的集合。 客户端在设计要需要能能够
接受 IP 集合或使用标准轮询 IP 集合。


### SRV 记录

SRV 记录是为命名端口创建的，是普通或
[Headless Services](/docs/concepts/services-networking/service/#headless-services)
的一部分。 对于每个命名端口的 SRV 记录格式如下:
`_my-port-name._my-port-protocol.my-svc.my-namespace.svc.cluster-domain.example`.
对于普通 Service , 解析的结果为 端口号和 域名:
`my-svc.my-namespace.svc.cluster-domain.example`.
对于 无头(headless) 的 Service, 解析结果有多个应答， 一个是 Service 后端的每个 Pod。
另一个则包含端口号和类似这种格式
`auto-generated-name.my-svc.my-namespace.svc.cluster-domain.example`
的 Pod 的域名

<!--
## Pods

### A/AAAA records

In general a pod has the following DNS resolution:

`pod-ip-address.my-namespace.pod.cluster-domain.example`.

For example, if a pod in the `default` namespace has the IP address 172.17.0.3,
and the domain name for your cluster is `cluster.local`, then the Pod has a DNS name:

`172-17-0-3.default.pod.cluster.local`.

Any pods created by a Deployment or DaemonSet exposed by a Service have the
following DNS resolution available:

`pod-ip-address.deployment-name.my-namespace.svc.cluster-domain.example`.
 -->
## Pod

### A/AAAA 记录

通常 Pod 的 DNS 记录名为如下格式:

`pod-ip-address.my-namespace.pod.cluster-domain.example`.

例如， 如果有一个在 `default` 名字空间的 Pod， 它的 IP 地址为 `172.17.0.3`， 集群配置的
域名叫 `cluster.local`， 这样这个 Pod 的 DNS 名称就是:
`172-17-0-3.default.pod.cluster.local`.

任意由 Deployment 或 DaemonSet 创建，由 Service 暴露的 Pod 会拥有一个如下的 DNS 名称:
`pod-ip-address.deployment-name.my-namespace.svc.cluster-domain.example`.

<!--
### Pod's hostname and subdomain fields

Currently when a pod is created, its hostname is the Pod's `metadata.name` value.

The Pod spec has an optional `hostname` field, which can be used to specify the
Pod's hostname. When specified, it takes precedence over the Pod's name to be
the hostname of the pod. For example, given a Pod with `hostname` set to
"`my-host`", the Pod will have its hostname set to "`my-host`".

The Pod spec also has an optional `subdomain` field which can be used to specify
its subdomain. For example, a Pod with `hostname` set to "`foo`", and `subdomain`
set to "`bar`", in namespace "`my-namespace`", will have the fully qualified
domain name (FQDN) "`foo.bar.my-namespace.svc.cluster-domain.example`".

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: default-subdomain
spec:
  selector:
    name: busybox
  clusterIP: None
  ports:
  - name: foo # Actually, no port is needed.
    port: 1234
    targetPort: 1234
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox1
  labels:
    name: busybox
spec:
  hostname: busybox-1
  subdomain: default-subdomain
  containers:
  - image: busybox:1.28
    command:
      - sleep
      - "3600"
    name: busybox
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox2
  labels:
    name: busybox
spec:
  hostname: busybox-2
  subdomain: default-subdomain
  containers:
  - image: busybox:1.28
    command:
      - sleep
      - "3600"
    name: busybox
```

If there exists a headless service in the same namespace as the pod and with
the same name as the subdomain, the cluster's DNS Server also returns an A or AAAA
record for the Pod's fully qualified hostname.
For example, given a Pod with the hostname set to "`busybox-1`" and the subdomain set to
"`default-subdomain`", and a headless Service named "`default-subdomain`" in
the same namespace, the pod will see its own FQDN as
"`busybox-1.default-subdomain.my-namespace.svc.cluster-domain.example`". DNS serves an
A or AAAA record at that name, pointing to the Pod's IP. Both pods "`busybox1`" and
"`busybox2`" can have their distinct A or AAAA records.

The Endpoints object can specify the `hostname` for any endpoint addresses,
along with its IP.

{{< note >}}
Because A or AAAA records are not created for Pod names, `hostname` is required for the Pod's A or AAAA
record to be created. A Pod with no `hostname` but with `subdomain` will only create the
A or AAAA record for the headless service (`default-subdomain.my-namespace.svc.cluster-domain.example`),
pointing to the Pod's IP address. Also, Pod needs to become ready in order to have a
record unless `publishNotReadyAddresses=True` is set on the Service.
{{< /note >}}
 -->

### Pod 的 `hostname` 和 `subdomain` 字段

目前，当一个 Pod 创建后， 它的主机名是 Pod 的 `metadata.name` 字段的值。

Pod 的定义中有一个可选字段 `hostname`， 可以用来指定 Pod 的主机名。 当设置了主机名时，它的优先
级是高于 Pod 名称作为 Pod 的主机名的。 例如， 设置一个 Pod 的 `hostname` 为 `my-host`，
这个 Pod 的主机名就会设置为 `my-host`

Pod 定义中还有一个可选择字段 `subdomain`， 可以用来指定它的子域名。 例如， 一个 Pod 的
`hostname` 设置为 `foo`， `subdomain` 设置为 `bar`， 处理 `my-namespace` 名字空间。
它的全限定名(FQDN) 就是 `foo.bar.my-namespace.svc.cluster-domain.example`

示例:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: default-subdomain
spec:
  selector:
    name: busybox
  clusterIP: None
  ports:
  - name: foo # 实际上是不需要端口的
    port: 1234
    targetPort: 1234
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox1
  labels:
    name: busybox
spec:
  hostname: busybox-1
  subdomain: default-subdomain
  containers:
  - image: busybox:1.28
    command:
      - sleep
      - "3600"
    name: busybox
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox2
  labels:
    name: busybox
spec:
  hostname: busybox-2
  subdomain: default-subdomain
  containers:
  - image: busybox:1.28
    command:
      - sleep
      - "3600"
    name: busybox
```

如果存在这样一个无头(headless) 的 Service, 它与另一个 Pod 在同一个名字空间，且 Pod 的
`subdomain` 与这个 Service 的名称是一样的。 集群 DNS 服务一样会返回 Pod 全限制名的 A/AAAA 记录。
例如，假定一个 Pod 的 `hostname` 设置为 `busybox-1`， `subdomain` 设置为 `default-subdomain`，
一个无头(headless) 的 Service 名字是 `default-subdomain`， 它们在同一个名字空间。
Pod 就可以看到它自己的全限定名(FQDN)为
`busybox-1.default-subdomain.my-namespace.svc.cluster-domain.example`。
DNS 服务为这个名称提供一个 A/AAAA 记录， 指向该 Pod 的 IP。 `busybox1` 和 `busybox2`
都能有它们各自的 A/AAAA 记录。
{{< todo text="这里需要配个完整的例子">}}

Endpoint 对象可以将任意端点地址和IP 设置为 `hostname`
{{< todo text="这里不理解，并且需要一个例子">}}

{{< note >}}
因为 A/AAAA 记录不是为 Pod 名称创建的， `hostname` 是 Pod A/AAAA 记录创建所必须的。
一个没有 `hostname` 字段，但是有 `subdomain` 字段的 Pod 只会为无头(headless)Service
创建 A/AAAA 记录(`default-subdomain.my-namespace.svc.cluster-domain.example`)
该记录指向的是该 Pod 的 IP 地址。还有，如果 Service 上没有设置 `publishNotReadyAddresses=True`
则 Pod 状态变为就绪后才的 DNS 记录
{{< /note >}}
<!--
### Pod's setHostnameAsFQDN field {#pod-sethostnameasfqdn-field}

{{< feature-state for_k8s_version="v1.19" state="alpha" >}}

**Prerequisites**: The `SetHostnameAsFQDN` [feature gate](/docs/reference/command-line-tools-reference/feature-gates/)
must be enabled for the
{{< glossary_tooltip text="API Server" term_id="kube-apiserver" >}}

When a Pod is configured to have fully qualified domain name (FQDN), its hostname is the short hostname. For example, if you have a Pod with the fully qualified domain name `busybox-1.default-subdomain.my-namespace.svc.cluster-domain.example`, then by default the `hostname` command inside that Pod returns `busybox-1` and  the `hostname --fqdn` command returns the FQDN.

When you set `setHostnameAsFQDN: true` in the Pod spec, the kubelet writes the Pod's FQDN into the hostname for that Pod's namespace. In this case, both `hostname` and `hostname --fqdn` return the Pod's FQDN.

{{< note >}}
In Linux, the hostname field of the kernel (the `nodename` field of `struct utsname`) is limited to 64 characters.

If a Pod enables this feature and its FQDN is longer than 64 character, it will fail to start. The Pod will remain in `Pending` status (`ContainerCreating` as seen by `kubectl`) generating error events, such as Failed to construct FQDN from pod hostname and cluster domain, FQDN `long-FDQN` is too long (64 characters is the max, 70 characters requested). One way of improving user experience for this scenario is to create an [admission webhook controller](/docs/reference/access-authn-authz/extensible-admission-controllers/#admission-webhooks) to control FQDN size when users create top level objects, for example, Deployment.
{{< /note >}}
 -->

### Pod 的 setHostnameAsFQDN 字段 {#pod-sethostnameasfqdn-field}

{{< feature-state for_k8s_version="v1.19" state="alpha" >}}

**前提条件**:
需要在 {{< glossary_tooltip text="API Server" term_id="kube-apiserver" >}} 启用
`SetHostnameAsFQDN` [功能阀](/docs/reference/command-line-tools-reference/feature-gates/)


当一个 Pod 配置了全限定名(FQDN)时，它的主机名是短主机名。 例如， 如果有一个全限定名为
`busybox-1.default-subdomain.my-namespace.svc.cluster-domain.example` 的 Pod，
在 Pod 内使用默认的 `hostname` 命令时返回的是 `busybox-1`， 而 `hostname --fqdn` 命令
返回的是全限定名(FQDN)。


当在 Pod 的定义中设置 `setHostnameAsFQDN: true` 时， kubelet 会将 Pod 的 全限定名(FQDN)
写到那个 Pod 名字空间的 hostname. 在这种情况下 `hostname` 和 `hostname --fqdn` 两个命令
返回的都是全限定名。

{{< note >}}
在 Linux 中， 内核中的 `hostname` 字段(`struct utsname` 的 `nodename` 字段)限制最多只能有 64 个字符。
{{< /note >}}
如果一个 Pod 开启了该特性，并且它的 全限定名(FQDN) 长度大于 64 个字符，就会启动失败。
Pod 会一停在 `Pending` 状态(通过 `kubectl` 看到的是 `ContainerCreating`)，最终会产生一个
错误事件，错误信息类似基于 Pod 主机名和集群域构建全限定名(FQDN)失败， `long-FDQN`  FQDN 太长了
(最长只能有 64 个字符，但实际有 70 个字符)。 在这种情况下改善用户体验的一种方式是创建一个
[admission webhook controller](/docs/reference/access-authn-authz/extensible-admission-controllers/#admission-webhooks)
来在创建顶级对象(如，Deployment)时控制全限定名(FQDN)的长度
<!--
### Pod's DNS Policy

DNS policies can be set on a per-pod basis. Currently Kubernetes supports the
following pod-specific DNS policies. These policies are specified in the
`dnsPolicy` field of a Pod Spec.

- "`Default`": The Pod inherits the name resolution configuration from the node
  that the pods run on.
  See [related discussion](/docs/tasks/administer-cluster/dns-custom-nameservers/#inheriting-dns-from-the-node)
  for more details.
- "`ClusterFirst`": Any DNS query that does not match the configured cluster
  domain suffix, such as "`www.kubernetes.io`", is forwarded to the upstream
  nameserver inherited from the node. Cluster administrators may have extra
  stub-domain and upstream DNS servers configured.
  See [related discussion](/docs/tasks/administer-cluster/dns-custom-nameservers/#effects-on-pods)
  for details on how DNS queries are handled in those cases.
- "`ClusterFirstWithHostNet`": For Pods running with hostNetwork, you should
  explicitly set its DNS policy "`ClusterFirstWithHostNet`".
- "`None`": It allows a Pod to ignore DNS settings from the Kubernetes
  environment. All DNS settings are supposed to be provided using the
  `dnsConfig` field in the Pod Spec.
  See [Pod's DNS config](#pod-dns-config) subsection below.

{{< note >}}
"Default" is not the default DNS policy. If `dnsPolicy` is not
explicitly specified, then "ClusterFirst" is used.
{{< /note >}}


The example below shows a Pod with its DNS policy set to
"`ClusterFirstWithHostNet`" because it has `hostNetwork` set to `true`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  containers:
  - image: busybox:1.28
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
    name: busybox
  restartPolicy: Always
  hostNetwork: true
  dnsPolicy: ClusterFirstWithHostNet
```
 -->
### Pod 的 DNS 策略

DNS policies can be set on a per-pod basis. Currently Kubernetes supports the
following pod-specific DNS policies. These policies are specified in the
`dnsPolicy` field of a Pod Spec.
DNS 策略可以在 Pod 级别设置， 目前 k8s 支持以下的 Pod 级别 DNS 策略。 这个策略通过 Pod
定义的 `dnsPolicy` 字段指定。

- "`Default`": Pod 从它自己运行的节点上继承域名解析配置。更多信息见
  [相关讨论](/docs/tasks/administer-cluster/dns-custom-nameservers/#inheriting-dns-from-the-node)

- "`ClusterFirst`": 任何与配置的集群域后缀匹配的 DNS 查询，如   "`www.kubernetes.io`"，
  会被转发到由节点继承的上游域名服务器。 集群管理员可以配置额外的 存根域(stub-domain) 和上游
  DNS 服务器。

- "`ClusterFirstWithHostNet`": 以 `hostNetwork` 运行的 Pod，需要显式的设置它的 DNS
  策略为 "`ClusterFirstWithHostNet`"

- "`None`": 这种策略允许 Pod 无视 k8s 环境的 DNS 配置。 所有的 DNS 配置都应该由 Pod 定义中
  的 `dnsConfig` 字段提供。
{{< note >}}
"Default" 并不是默认的 DNS 策略， 如果没有显式的设置 `dnsPolicy`，则使用 "ClusterFirst"
{{< /note >}}

以面的例子中的 Pod 的 DNS 策略被设置为 "`ClusterFirstWithHostNet`" 因为它的 `hostNetwork`
被设置为了 `true`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  containers:
  - image: busybox:1.28
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
    name: busybox
  restartPolicy: Always
  hostNetwork: true
  dnsPolicy: ClusterFirstWithHostNet
```
<!--
### Pod's DNS Config {#pod-dns-config}

Pod's DNS Config allows users more control on the DNS settings for a Pod.

The `dnsConfig` field is optional and it can work with any `dnsPolicy` settings.
However, when a Pod's `dnsPolicy` is set to "`None`", the `dnsConfig` field has
to be specified.

Below are the properties a user can specify in the `dnsConfig` field:

- `nameservers`: a list of IP addresses that will be used as DNS servers for the
  Pod. There can be at most 3 IP addresses specified. When the Pod's `dnsPolicy`
  is set to "`None`", the list must contain at least one IP address, otherwise
  this property is optional.
  The servers listed will be combined to the base nameservers generated from the
  specified DNS policy with duplicate addresses removed.
- `searches`: a list of DNS search domains for hostname lookup in the Pod.
  This property is optional. When specified, the provided list will be merged
  into the base search domain names generated from the chosen DNS policy.
  Duplicate domain names are removed.
  Kubernetes allows for at most 6 search domains.
- `options`: an optional list of objects where each object may have a `name`
  property (required) and a `value` property (optional). The contents in this
  property will be merged to the options generated from the specified DNS policy.
  Duplicate entries are removed.

The following is an example Pod with custom DNS settings:

{{< codenew file="service/networking/custom-dns.yaml" >}}

When the Pod above is created, the container `test` gets the following contents
in its `/etc/resolv.conf` file:

```
nameserver 1.2.3.4
search ns1.svc.cluster-domain.example my.dns.search.suffix
options ndots:2 edns0
```

For IPv6 setup, search path and name server should be setup like this:

```shell
kubectl exec -it dns-example -- cat /etc/resolv.conf
```
The output is similar to this:
```shell
nameserver fd00:79:30::a
search default.svc.cluster-domain.example svc.cluster-domain.example cluster-domain.example
options ndots:5
```
 -->
### Pod 的 DNS 配置 {#pod-dns-config}

Pod's DNS Config allows users more control on the DNS settings for a Pod.
Pod 的 DNS 配置让用户可以更多地控制 Pod 上的 DNS 配置。

`dnsConfig` 是一个可选字段， 它可以与 `dnsPolicy` 配置配合使用。 但是当一个 Pod 的 `dnsPolicy`
设置为 "`None`"时，就必须要设置 `dnsConfig` 字段。
以下是 `dnsConfig` 字段中用户可以配置的字段:

- `nameservers`: Pod 用作 DNS 服务的一个 IP 地址列表。 最多可以指定三个 IP 地址。
  当 Pod `dnsPolicy` 设置为 "`None`"， 这个列表中至少包含一个 IP 地址，其它情况下这个字段为可选。
  这个 DNS 列表会与 DNS 策略配置产生的基础 DNS 服务器合并，重复的会被删除。

- `searches`: 一个 DNS 检索列表，用于 Pod 中的主机名查找。 这个属性为可选。当设置这个字段时，
  这个列表会合并到 DNS 策略配置产生的DNS 检索域名中，重复的条目会被删除。 k8s 允许最多 6 个检索域名。

- `options`: 一个可选的对象列表， 其中的每个对象有一个必要的 `name` 属性和一个可选的 `value` 属性。
  这些属性会被合并到配置的 DNS 策略生成的选项中

以下来一个包含自定义 DNS 配置的 Pod 的示例:

{{< codenew file="service/networking/custom-dns.yaml" >}}

当上面这个 Pod 被创建后，这个叫  `test` 容器中的 `/etc/resolv.conf` 就会有下面的这些内容:

```
nameserver 1.2.3.4
search ns1.svc.cluster-domain.example my.dns.search.suffix
options ndots:2 edns0
```

For IPv6 setup, search path and name server should be setup like this:
如果设置了 IPv6， 检索路径和DNS服务应该这么配置:

```shell
kubectl exec -it dns-example -- cat /etc/resolv.conf
```
输出结果类似:
```
nameserver fd00:79:30::a
search default.svc.cluster-domain.example svc.cluster-domain.example cluster-domain.example
options ndots:5
```

<!--
### Feature availability

The availability of Pod DNS Config and DNS Policy "`None`" is shown as below.

| k8s version | Feature support |
| :---------: |:-----------:|
| 1.14 | Stable |
| 1.10 | Beta (on by default)|
| 1.9 | Alpha |
 -->

### Feature 可用性

Pod DNS 配置 和  DNS 策略的 "`None`" 的可用性如下:

| k8s 版本 | 特性可用性 |
| :---------: |:-----------:|
| 1.14 | Stable |
| 1.10 | Beta (默认启用)|
| 1.9 | Alpha |



## {{% heading "whatsnext" %}}

<!--
For guidance on administering DNS configurations, check
[Configure DNS Service](/docs/tasks/administer-cluster/dns-custom-nameservers/)
 -->
DNS 配置的管理指导见
[Configure DNS Service](/k8sDocs/tasks/administer-cluster/dns-custom-nameservers/)
