---
title: Ingress
weight: 60
date: 2020-09-21
publishdate: 2020-09-23
---
<!--
---
reviewers:
- bprashanth
title: Ingress
content_type: concept
weight: 40
--- -->

<!-- overview -->
{{< feature-state for_k8s_version="v1.19" state="stable" >}}
{{< glossary_definition term_id="ingress" length="all" >}}


<!-- body -->
<!--
## Terminology

For clarity, this guide defines the following terms:

* Node: A worker machine in Kubernetes, part of a cluster.
* Cluster: A set of Nodes that run containerized applications managed by Kubernetes. For this example, and in most common Kubernetes deployments, nodes in the cluster are not part of the public internet.
* Edge router: A router that enforces the firewall policy for your cluster. This could be a gateway managed by a cloud provider or a physical piece of hardware.
* Cluster network: A set of links, logical or physical, that facilitate communication within a cluster according to the Kubernetes [networking model](/docs/concepts/cluster-administration/networking/).
* Service: A Kubernetes {{< glossary_tooltip term_id="service" >}} that identifies a set of Pods using {{< glossary_tooltip text="label" term_id="label" >}} selectors. Unless mentioned otherwise, Services are assumed to have virtual IPs only routable within the cluster network.
 -->
## 术语

为了明确，本文定义了如下术语:
- 节点(Node): k8s 中的一个工作机，集群的一部分。
- 集群(Cluster): 一组由运行由 k8s 管理的容器化应用的节点集合。在本例和大多数常用的 k8s 部署
  中，集群中的节点是不在公网上的。
- 边缘路由: 一个为集群执行防火墙策略的路由。 可以是由云提供商管理的网关或一个物理硬件。
- 集群网络: 一组逻辑的或物理的连接，这些连接根据 k8s
  [网络模型](/k8sDocs/concepts/cluster-administration/networking/)简化集群内的通信
- Service: 一个 k8s {{< glossary_tooltip term_id="service" >}} 就是使用
  {{< glossary_tooltip term_id="label" >}} 选择器区分一组 Pod 的抽象概念。 如果没有特别
  说明， Service 假定是有一个只能在集群内路由的虚拟 IP 的。
<!--
## What is Ingress?

[Ingress](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#ingress-v1-networking-k8s-io) exposes HTTP and HTTPS routes from outside the cluster to
{{< link text="services" url="/docs/concepts/services-networking/service/" >}} within the cluster.
Traffic routing is controlled by rules defined on the Ingress resource.

```none
    internet
        |
   [ Ingress ]
   --|-----|--
   [ Services ]
```

An Ingress may be configured to give Services externally-reachable URLs, load balance traffic, terminate SSL / TLS, and offer name-based virtual hosting. An [Ingress controller](/docs/concepts/services-networking/ingress-controllers) is responsible for fulfilling the Ingress, usually with a load balancer, though it may also configure your edge router or additional frontends to help handle the traffic.

An Ingress does not expose arbitrary ports or protocols. Exposing services other than HTTP and HTTPS to the internet typically
uses a service of type [Service.Type=NodePort](/docs/concepts/services-networking/service/#nodeport) or
[Service.Type=LoadBalancer](/docs/concepts/services-networking/service/#loadbalancer).
 -->
## Ingress 是什么?

[Ingress](https://kubernetes.io/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#ingress-v1-networking-k8s-io)
将集群内部的
{{< link text="Service" url="/k8sDocs/concepts/services-networking/service/" >}}
通过 HTTP 和 HTTPS 路由暴露到集群外部。
流量路由由 Ingress 资源内部定义的规则控制。

```none
    internet
        |
   [ Ingress ]
   --|-----|--
   [ Services ]
```

可以配置一个 Ingress 为 Service 提供外部可以访问的 URL, 负载均衡，SSL / TLS, 提供基于名称的
虚拟主机名。
[Ingress 控制器](/k8sDocs/concepts/services-networking/ingress-controllers)将负责
实现 Ingress， 通常是通过负载均衡器，也可能是通过配置边缘路由或额外的前端组件来帮助处理流量。

Ingress 不是暴露任意的端口或协议。 要暴露非 HTTP / HTTPS 的 Service 通常使用
[Service.Type=NodePort](/k8sDocs/concepts/services-networking/service/#nodeport)
或
[Service.Type=LoadBalancer](/k8sDocs/concepts/services-networking/service/#loadbalancer).
<!--
## Prerequisites

You must have an [Ingress controller](/docs/concepts/services-networking/ingress-controllers) to satisfy an Ingress. Only creating an Ingress resource has no effect.

You may need to deploy an Ingress controller such as [ingress-nginx](https://kubernetes.github.io/ingress-nginx/deploy/). You can choose from a number of
[Ingress controllers](/docs/concepts/services-networking/ingress-controllers).

Ideally, all Ingress controllers should fit the reference specification. In reality, the various Ingress
controllers operate slightly differently.

{{< note >}}
Make sure you review your Ingress controller's documentation to understand the caveats of choosing it.
{{< /note >}}
 -->

## 前置条件

要使用 Ingress 首先得有一个
[Ingress 控制器](/k8sDocs/concepts/services-networking/ingress-controllers)。
不然只创建一个 Ingress 资源是没有效果的。

需要部署一个 Ingress 控制器，例如
[ingress-nginx](https://kubernetes.github.io/ingress-nginx/deploy/).
也可以从
[Ingress 控制器](/k8sDocs/concepts/services-networking/ingress-controllers)
中选几个.

理想情况下，所有的 Ingress 控制器都应该是符合参考规格说明的。 实际上，不同的 Ingress 运转方式
各自都有点不同。
{{< note >}}
在选择 Ingress 控制器前请仔细阅读相关文档，确定理解了相关注意事项。
{{< /note >}}
<!--
## The Ingress resource

A minimal Ingress resource example:

{{< codenew file="service/networking/minimal-ingress.yaml" >}}

As with all other Kubernetes resources, an Ingress needs `apiVersion`, `kind`, and `metadata` fields.
The name of an Ingress object must be a valid
[DNS subdomain name](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).
For general information about working with config files, see [deploying applications](/docs/tasks/run-application/run-stateless-application-deployment/), [configuring containers](/docs/tasks/configure-pod-container/configure-pod-configmap/), [managing resources](/docs/concepts/cluster-administration/manage-deployment/).
 Ingress frequently uses annotations to configure some options depending on the Ingress controller, an example of which
 is the [rewrite-target annotation](https://github.com/kubernetes/ingress-nginx/blob/master/docs/examples/rewrite/README.md).
Different [Ingress controller](/docs/concepts/services-networking/ingress-controllers) support different annotations. Review the documentation for
 your choice of Ingress controller to learn which annotations are supported.

The Ingress [spec](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status)
has all the information needed to configure a load balancer or proxy server. Most importantly, it
contains a list of rules matched against all incoming requests. Ingress resource only supports rules
for directing HTTP(S) traffic.
 -->

## Ingress 资源

一个最小化 Ingress 资源的示例:
{{< codenew file="service/networking/minimal-ingress.yaml" >}}

与所有其它的 k8s 资源一样， 一个 Ingress 的必要字段有 `apiVersion`, `kind`, `metadata`。
Ingress 对象的名称必须是一个有效的
[DNS 子域名](/k8sDocs/concepts/overview/working-with-objects/names#dns-subdomain-names).
关于配置文件的通用信息见
[部署应用](/k8sDocs/tasks/run-application/run-stateless-application-deployment/),
[配置容器](/k8sDocs/tasks/configure-pod-container/configure-pod-configmap/),
[管理资源](/k8sDocs/concepts/cluster-administration/manage-deployment/).
Ingress 经常使用注解(annotation) 来配置一些基于 Ingress 控制器的可选配置，这里有一个例子
[rewrite-target 注解](https://github.com/kubernetes/ingress-nginx/blob/master/docs/examples/rewrite/README.md).
不同的
[Ingress 控制器](/docs/concepts/services-networking/ingress-controllers)
会支持不同的注解。 查看对应的控制文档了解其支持的注解(annotation)。

Ingress [spec](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status)
中包含了配置负载均衡器或代理服务器的所有信息。 最重要的是它包含了一个匹配进入请求的规则列表。
Ingress 资源只支持对 HTTP(S) 流量的转发。

<!--
### Ingress rules

Each HTTP rule contains the following information:

* An optional host. In this example, no host is specified, so the rule applies to all inbound
  HTTP traffic through the IP address specified. If a host is provided (for example,
  foo.bar.com), the rules apply to that host.
* A list of paths (for example, `/testpath`), each of which has an associated
  backend defined with a `service.name` and a `service.port.name` or
  `service.port.number`. Both the host and path must match the content of an
  incoming request before the load balancer directs traffic to the referenced
  Service.
* A backend is a combination of Service and port names as described in the
  [Service doc](/docs/concepts/services-networking/service/) or a [custom resource backend](#resource-backend) by way of a {{< glossary_tooltip term_id="CustomResourceDefinition" text="CRD" >}}. HTTP (and HTTPS) requests to the
  Ingress that matches the host and path of the rule are sent to the listed backend.

A `defaultBackend` is often configured in an Ingress controller to service any requests that do not
match a path in the spec.
 -->

### Ingress 规则 {#ingress-rules}

每个 HTTP 规则包含以下信息:
- 一个可选的主机名。 在这个例子中就没有配，所以这个规则应用于所有通过指定 IP 的流量。如果主机名
  有设置(例如，foo.bar.com), 这个规则就应用于这个主机名。
- 一个路径列表(如例子中的 `/testpath`)， 每个路径都与一个后台通过一个 `service.name` 和
  一个 `service.port.name` 或 `service.port.number` 相关联。 必须要主机名和路径都匹配的
  进入请求才会重定向到相应的 Service
- 一个后端就是一个 Service 和 端口名称的组合，就如
  [Service](/docs/concepts/services-networking/service/)
  或
  [自定义资源后端](#resource-backend)通过
  {{< glossary_tooltip term_id="CustomResourceDefinition" text="CRD" >}}
  方式中所描述的一样。
  到达 Ingress 的 HTTP (和 HTTPS) 请求匹配到对应规则的主机名和路径就会发送到列举的后端。

一个 `defaultBackend` 通常是配置在一个 Ingress 控制器中，用于接收所有没有匹配到任何配置中的
路径的请求。
<!--
### DefaultBackend {#default-backend}

An Ingress with no rules sends all traffic to a single default backend. The `defaultBackend` is conventionally a configuration option
of the [Ingress controller](/docs/concepts/services-networking/ingress-controllers) and is not specified in your Ingress resources.

If none of the hosts or paths match the HTTP request in the Ingress objects, the traffic is
routed to your default backend.
 -->
### 默认后端(DefaultBackend) {#default-backend}

一个没有任何规则的 Ingress 会将所有的流量发送到一个默认的后端。 `defaultBackend` 是一个很方便
的配置在  [Ingress 控制器](/k8sDocs/concepts/services-networking/ingress-controllers)
并不需要在 Ingress 中配置。

如果没有一个 Ingress 中的主机名或路径匹配到的 HTTP 请求，这些请求就会路由到默认后端
<!--
### Resource backends {#resource-backend}

A `Resource` backend is an ObjectRef to another Kubernetes resource within the
same namespace as the Ingress object. A `Resource` is a mutually exclusive
setting with Service, and will fail validation if both are specified. A common
usage for a `Resource` backend is to ingress data to an object storage backend
with static assets.

{{< codenew file="service/networking/ingress-resource-backend.yaml" >}}

After creating the Ingress above, you can view it with the following command:

```bash
kubectl describe ingress ingress-resource-backend
```

```
Name:             ingress-resource-backend
Namespace:        default
Address:
Default backend:  APIGroup: k8s.example.com, Kind: StorageBucket, Name: static-assets
Rules:
  Host        Path  Backends
  ----        ----  --------
  *
              /icons   APIGroup: k8s.example.com, Kind: StorageBucket, Name: icon-assets
Annotations:  <none>
Events:       <none>
```
 -->

### 资源(Resource) 后端 {#resource-backend}

一个 `Resource` 后端是一个与 Ingress 对象在同一个名字空间的另一个 k8s 资源的引用。
`Resource` 与 Service 是互斥的，如果同时配置了两者，则不会通过验证。 `Resource` 的一个常见
应用场景是进入一个存放静态资源的对象存储后端。

{{< codenew file="service/networking/ingress-resource-backend.yaml" >}}

在创建以上定义的 Ingress 后，可以通过以下命令查看:

```bash
kubectl describe ingress ingress-resource-backend
```

```
Name:             ingress-resource-backend
Namespace:        default
Address:
Default backend:  APIGroup: k8s.example.com, Kind: StorageBucket, Name: static-assets
Rules:
  Host        Path  Backends
  ----        ----  --------
  *
              /icons   APIGroup: k8s.example.com, Kind: StorageBucket, Name: icon-assets
Annotations:  <none>
Events:       <none>
```
<!--
### Path types

Each path in an Ingress is required to have a corresponding path type. Paths
that do not include an explicit `pathType` will fail validation. There are three
supported path types:

* `ImplementationSpecific`: With this path type, matching is up to the
  IngressClass. Implementations can treat this as a separate `pathType` or treat
  it identically to `Prefix` or `Exact` path types.

* `Exact`: Matches the URL path exactly and with case sensitivity.

* `Prefix`: Matches based on a URL path prefix split by `/`. Matching is case
  sensitive and done on a path element by element basis. A path element refers
  to the list of labels in the path split by the `/` separator. A request is a
  match for path _p_ if every _p_ is an element-wise prefix of _p_ of the
  request path.

  {{< note >}} If the last element of the path is a substring of the last
  element in request path, it is not a match (for example: `/foo/bar`
  matches`/foo/bar/baz`, but does not match `/foo/barbaz`). {{< /note >}}
 -->

### 路径(Path)类型

Ingress 中的每一个路径都需要有一个相应的路径类型。 没有显示设置 `pathType` 是没办法通过验证的。
有以下三种支持的路径类型:

- `ImplementationSpecific`: 这个路径类型的匹配是由 IngressClass 决定。 具体实现可以将其认为是
  独立的 `pathType` 或认为是 `Prefix` 或 `Exact` 路径类型。

- `Exact`:  完全匹配路径，区分大小写。

- `Prefix`: 基于由 `/` 分割的 URL 路径前缀匹配。匹配区分大小写，并且逐个元素匹配。
  一个元素就是通过 `/` 分割后列表中的每一个元素。 一个请求与路径 _p_ 相匹配则请求中的元素可以
  依次匹配完路径 _p_ 的每一个元素。
  {{< note >}}
  如果路径的最后一个元素是请求路径的最后一个元素的子串，则不能匹配(例如：`/foo/bar` 可以匹配
  请求路径 `/foo/bar/baz`，但是不能匹配请求路径 `/foo/barbaz` )。
  {{< /note >}}
  <!--
### Examples

| Kind   | Path(s)                         | Request path(s)               | Matches?                           |
|--------|---------------------------------|-------------------------------|------------------------------------|
| Prefix | `/`                             | (all paths)                   | Yes                                |
| Exact  | `/foo`                          | `/foo`                        | Yes                                |
| Exact  | `/foo`                          | `/bar`                        | No                                 |
| Exact  | `/foo`                          | `/foo/`                       | No                                 |
| Exact  | `/foo/`                         | `/foo`                        | No                                 |
| Prefix | `/foo`                          | `/foo`, `/foo/`               | Yes                                |
| Prefix | `/foo/`                         | `/foo`, `/foo/`               | Yes                                |
| Prefix | `/aaa/bb`                       | `/aaa/bbb`                    | No                                 |
| Prefix | `/aaa/bbb`                      | `/aaa/bbb`                    | Yes                                |
| Prefix | `/aaa/bbb/`                     | `/aaa/bbb`                    | Yes, ignores trailing slash        |
| Prefix | `/aaa/bbb`                      | `/aaa/bbb/`                   | Yes,  matches trailing slash       |
| Prefix | `/aaa/bbb`                      | `/aaa/bbb/ccc`                | Yes, matches subpath               |
| Prefix | `/aaa/bbb`                      | `/aaa/bbbxyz`                 | No, does not match string prefix   |
| Prefix | `/`, `/aaa`                     | `/aaa/ccc`                    | Yes, matches `/aaa` prefix         |
| Prefix | `/`, `/aaa`, `/aaa/bbb`         | `/aaa/bbb`                    | Yes, matches `/aaa/bbb` prefix     |
| Prefix | `/`, `/aaa`, `/aaa/bbb`         | `/ccc`                        | Yes, matches `/` prefix            |
| Prefix | `/aaa`                          | `/ccc`                        | No, uses default backend           |
| Mixed  | `/foo` (Prefix), `/foo` (Exact) | `/foo`                        | Yes, prefers Exact                 |
 -->

### 示例

| 类型    | 路径                             | 请求路径                      | 匹配结果                             |
|--------|---------------------------------|-------------------------------|------------------------------------|
| Prefix | `/`                             | (all paths)                   | 匹配                                |
| Exact  | `/foo`                          | `/foo`                        | 匹配                                |
| Exact  | `/foo`                          | `/bar`                        | 不匹配                              |
| Exact  | `/foo`                          | `/foo/`                       | 不匹配                              |
| Exact  | `/foo/`                         | `/foo`                        | 不匹配                              |
| Prefix | `/foo`                          | `/foo`, `/foo/`               | 匹配                                |
| Prefix | `/foo/`                         | `/foo`, `/foo/`               | 匹配                                |
| Prefix | `/aaa/bb`                       | `/aaa/bbb`                    | 不匹配                              |
| Prefix | `/aaa/bbb`                      | `/aaa/bbb`                    | 匹配                                |
| Prefix | `/aaa/bbb/`                     | `/aaa/bbb`                    | 匹配, 忽略尾部的斜线                  |
| Prefix | `/aaa/bbb`                      | `/aaa/bbb/`                   | 匹配, 匹配尾部的斜线                  |
| Prefix | `/aaa/bbb`                      | `/aaa/bbb/ccc`                | 匹配, 匹配前缀                       |
| Prefix | `/aaa/bbb`                      | `/aaa/bbbxyz`                 | 不匹配, 不是匹配字符串前缀             |
| Prefix | `/`, `/aaa`                     | `/aaa/ccc`                    | 匹配, 匹配 `/aaa` 前缀               |
| Prefix | `/`, `/aaa`, `/aaa/bbb`         | `/aaa/bbb`                    | 匹配, 匹配 `/aaa/bbb` 前缀           |
| Prefix | `/`, `/aaa`, `/aaa/bbb`         | `/ccc`                        | 匹配, 匹配 `/` 前缀                  |
| Prefix | `/aaa`                          | `/ccc`                        | 不匹配, 使用默认后端                  |
| Mixed  | `/foo` (Prefix), `/foo` (Exact) | `/foo`                        | 匹配, 优先完全匹配                    |
<!--
#### Multiple matches
In some cases, multiple paths within an Ingress will match a request. In those
cases precedence will be given first to the longest matching path. If two paths
are still equally matched, precedence will be given to paths with an exact path
type over prefix path type.
 -->

#### 多匹配

在某些情况下，一个 Ingress 中的多个路径都能与请求路径相匹配。在这种情况下优先级最高的是最长匹配
路径。 如果还有两个路径匹配长度匹配相同，则完全匹配优先级高于前端匹配。
<!--
## Hostname wildcards
Hosts can be precise matches (for example “`foo.bar.com`”) or a wildcard (for
example “`*.foo.com`”). Precise matches require that the HTTP `host` header
matches the `host` field. Wildcard matches require the HTTP `host` header is
equal to the suffix of the wildcard rule.

| Host        | Host header       | Match?                                            |
| ----------- |-------------------| --------------------------------------------------|
| `*.foo.com` | `bar.foo.com`     | Matches based on shared suffix                    |
| `*.foo.com` | `baz.bar.foo.com` | No match, wildcard only covers a single DNS label |
| `*.foo.com` | `foo.com`         | No match, wildcard only covers a single DNS label |

{{< codenew file="service/networking/ingress-wildcard-host.yaml" >}}
 -->
## 主机名通配符

主机名可以是精确匹配(例如 `foo.bar.com`)，也可以有通配符(例如 `*.foo.com`)。
精确匹配必须要 HTTP 请求头中的 `host` 与规则的 `host` 字段完全匹配。
通配符则要 HTTP 请求头中的 `host` 与通配符规则的后缀相同。

例如:

| 规则         | 请求头`host`       | 匹配结果                     |
| ----------- |-------------------| ----------------------------|
| `*.foo.com` | `bar.foo.com`     | 匹配，因为后缀相同             |
| `*.foo.com` | `baz.bar.foo.com` | 不匹配, 通配符只能包含一级子域名 |
| `*.foo.com` | `foo.com`         | 不匹配, 通配符只能包含一级子域名 |

{{< codenew file="service/networking/ingress-wildcard-host.yaml" >}}
<!--
## Ingress class

Ingresses can be implemented by different controllers, often with different
configuration. Each Ingress should specify a class, a reference to an
IngressClass resource that contains additional configuration including the name
of the controller that should implement the class.

{{< codenew file="service/networking/external-lb.yaml" >}}

IngressClass resources contain an optional parameters field. This can be used to
reference additional configuration for this class.
 -->

## IngressClass

Ingress 可以由不同的控制器实现，通常也会有不同的配置。 每个 Ingress 都需要指定一个类型，
它是一个 IngressClass 资源的引用， 其中包含如实现该类型的控制器的名称等额外配置。

{{< codenew file="service/networking/external-lb.yaml" >}}

IngressClass 资源中包含一个可选参数字段。 可以用设置该类型额外配置的引用
<!--

### Deprecated annotation

Before the IngressClass resource and `ingressClassName` field were added in
Kubernetes 1.18, Ingress classes were specified with a
`kubernetes.io/ingress.class` annotation on the Ingress. This annotation was
never formally defined, but was widely supported by Ingress controllers.

The newer `ingressClassName` field on Ingresses is a replacement for that
annotation, but is not a direct equivalent. While the annotation was generally
used to reference the name of the Ingress controller that should implement the
Ingress, the field is a reference to an IngressClass resource that contains
additional Ingress configuration, including the name of the Ingress controller.
 -->

### 废弃的注解

在 k8s 1.18 版本中添加 IngressClass 资源和 `ingressClassName` 资源前，Ingress 的类型是
通过其上的 `kubernetes.io/ingress.class` 注解指定的。 这个注解从来没有正式的定义过，但它
在 Ingress 控制器中被广泛支持。

Ingress 中的新字段 `ingressClassName` 就是对这个注解的替代，但它们也不是直接等价的。
其中注解通常是用来指定实现该 Ingress 的控制器的名称， 新字段则是指定一个 IngressClass 资源的引用。
这个 IngressClass 资源中包含一额外的 Ingress 配置参数和 Ingress 控制器的名称。
<!--
### Default IngressClass {#default-ingress-class}

You can mark a particular IngressClass as default for your cluster. Setting the
`ingressclass.kubernetes.io/is-default-class` annotation to `true` on an
IngressClass resource will ensure that new Ingresses without an
`ingressClassName` field specified will be assigned this default IngressClass.

{{< caution >}}
If you have more than one IngressClass marked as the default for your cluster,
the admission controller prevents creating new Ingress objects that don't have
an `ingressClassName` specified. You can resolve this by ensuring that at most 1
IngressClass is marked as default in your cluster.
{{< /caution >}}
 -->

### 默认 IngressClass {#default-ingress-class}

用户可以将某个 IngressClass 设置为集群的默认 IngressClass。
在 IngressClass 资源上设置 `ingressclass.kubernetes.io/is-default-class` 注解值为 `true`。
这样新创建 Ingress 如果没有设置 `ingressClassName` 字段则会分配这个默认的 IngressClass。

{{< caution >}}
如果集群中有不止一个 IngressClass 被标记为默认，准入控制器(admission controller) 就会禁止
创建没有设置 `ingressClassName` 的 Ingress。 需要用户手动解决，确保集群中只有一个默认
IngressClass。
{{< /caution >}}
<!--
## Types of Ingress

### Ingress backed by a single Service {#single-service-ingress}

There are existing Kubernetes concepts that allow you to expose a single Service
(see [alternatives](#alternatives)). You can also do this with an Ingress by specifying a
*default backend* with no rules.

{{< codenew file="service/networking/test-ingress.yaml" >}}

If you create it using `kubectl apply -f` you should be able to view the state
of the Ingress you just added:

```bash
kubectl get ingress test-ingress
```

```
NAME           CLASS         HOSTS   ADDRESS         PORTS   AGE
test-ingress   external-lb   *       203.0.113.123   80      59s
```

Where `203.0.113.123` is the IP allocated by the Ingress controller to satisfy
this Ingress.

{{< note >}}
Ingress controllers and load balancers may take a minute or two to allocate an IP address.
Until that time, you often see the address listed as `<pending>`.
{{< /note >}}
 -->

## Ingress 的类型

### 后端只有一个 Service 的 Ingress  {#single-service-ingress}

k8s 存在概念可以让用户暴露单个 Service (见 [替代方案](#alternatives)).
也可以通过一个指定 *默认后端* 但没有配置规则的 Ingress 达到同样的效果。

{{< codenew file="service/networking/test-ingress.yaml" >}}

通过 `kubectl apply -f` 创建 Ingress，并通过以下命令查看:

```bash
kubectl get ingress test-ingress
```

```
NAME           CLASS         HOSTS   ADDRESS         PORTS   AGE
test-ingress   external-lb   *       203.0.113.123   80      59s
```

其中 `203.0.113.123` 是由 Ingress 控制器分配来满足这个 Ingress 的 IP 地址。

{{< note >}}
Ingress 控制器和负载均衡器可能需要一两分钟为分配一个 IP 地址。在这之前，通常看到地址列的值为 `<pending>`
{{< /note >}}
<!--
### Simple fanout

A fanout configuration routes traffic from a single IP address to more than one Service,
based on the HTTP URI being requested. An Ingress allows you to keep the number of load balancers
down to a minimum. For example, a setup like:

```
foo.bar.com -> 178.91.123.132 -> / foo    service1:4200
                                 / bar    service2:8080
```

would require an Ingress such as:

{{< codenew file="service/networking/simple-fanout-example.yaml" >}}

When you create the Ingress with `kubectl apply -f`:

```shell
kubectl describe ingress simple-fanout-example
```

```
Name:             simple-fanout-example
Namespace:        default
Address:          178.91.123.132
Default backend:  default-http-backend:80 (10.8.2.3:8080)
Rules:
  Host         Path  Backends
  ----         ----  --------
  foo.bar.com
               /foo   service1:4200 (10.8.0.90:4200)
               /bar   service2:8080 (10.8.0.91:8080)
Events:
  Type     Reason  Age                From                     Message
  ----     ------  ----               ----                     -------
  Normal   ADD     22s                loadbalancer-controller  default/test
```

The Ingress controller provisions an implementation-specific load balancer
that satisfies the Ingress, as long as the Services (`service1`, `service2`) exist.
When it has done so, you can see the address of the load balancer at the
Address field.

{{< note >}}
Depending on the [Ingress controller](/docs/concepts/services-networking/ingress-controllers/)
you are using, you may need to create a default-http-backend
[Service](/docs/concepts/services-networking/service/).
{{< /note >}}
 -->

### 简单分散(fanout)

一个分散就是基于请求的 HTTP URI 配置路由流量从一单个 IP 地址到多个 Service。
一个 Ingress 可以使负载均衡器的数量减少到最低。 例如如下配置:
```
foo.bar.com -> 178.91.123.132 -> / foo    service1:4200
                                 / bar    service2:8080
```

需要要一个如下 Ingress:

{{< codenew file="service/networking/simple-fanout-example.yaml" >}}

在使用 `kubectl apply -f` 命令创建 Ingress 后，通过以下命令查看详情:

```shell
kubectl describe ingress simple-fanout-example
```

```
Name:             simple-fanout-example
Namespace:        default
Address:          178.91.123.132
Default backend:  default-http-backend:80 (10.8.2.3:8080)
Rules:
  Host         Path  Backends
  ----         ----  --------
  foo.bar.com
               /foo   service1:4200 (10.8.0.90:4200)
               /bar   service2:8080 (10.8.0.91:8080)
Events:
  Type     Reason  Age                From                     Message
  ----     ------  ----               ----                     -------
  Normal   ADD     22s                loadbalancer-controller  default/test
```

Ingress 控制器会在目标 Service (`service1`, `service2`) 还存在时，
根据本身实现提供一个满足 Ingress 的负载均衡器直到。
当创建好后，可以在地址字段看到负载均衡器的地址。

{{< note >}}
基于使用的
[Ingress 控制器](/docs/concepts/services-networking/ingress-controllers/)
可能需要创建一个 default-http-backend
[Service](/k8sDocs/concepts/services-networking/service/).
{{< /note >}}
<!--
### Name based virtual hosting

Name-based virtual hosts support routing HTTP traffic to multiple host names at the same IP address.

```none
foo.bar.com --|                 |-> foo.bar.com service1:80
              | 178.91.123.132  |
bar.foo.com --|                 |-> bar.foo.com service2:80
```

The following Ingress tells the backing load balancer to route requests based on
the [Host header](https://tools.ietf.org/html/rfc7230#section-5.4).

{{< codenew file="service/networking/name-virtual-host-ingress.yaml" >}}

If you create an Ingress resource without any hosts defined in the rules, then any
web traffic to the IP address of your Ingress controller can be matched without a name based
virtual host being required.

For example, the following Ingress routes traffic
requested for `first.bar.com` to `service1`, `second.foo.com` to `service2`, and any traffic
to the IP address without a hostname defined in request (that is, without a request header being
presented) to `service3`.

{{< codenew file="service/networking/name-virtual-host-ingress-no-third-host.yaml" >}}
 -->

### 基于名称的虚拟主机

基于名称的虚拟主机支持在同一个IP上将 HTTP 流量路由到多个主机名上。

```none
foo.bar.com --|                 |-> foo.bar.com service1:80
              | 178.91.123.132  |
bar.foo.com --|                 |-> bar.foo.com service2:80
```

The following Ingress tells the backing load balancer to route requests based on
the [Host header](https://tools.ietf.org/html/rfc7230#section-5.4).
以下 Ingress 告诉后端的负载均衡器基于
[Host 头字段](https://tools.ietf.org/html/rfc7230#section-5.4)
路由请求。

{{< codenew file="service/networking/name-virtual-host-ingress.yaml" >}}

If you create an Ingress resource without any hosts defined in the rules, then any
web traffic to the IP address of your Ingress controller can be matched without a name based
virtual host being required.
如果创建的 Ingress 资源中没有定义任何主机规则，则所有到达这个 Ingress 控制 IP 的的 web 流量
则不需要基于名称的虚拟主机就能匹配。

例如， 以下 Ingress 路由 `first.bar.com` 的请求到 `service1`，
`second.foo.com` 的请求到 `service2`，其它任意到达这个 IP 地址，但在请求中没有包含主机名
(也就是没的请求头(或请求头中没有 Host 字段))的请求到 `service3`

{{< codenew file="service/networking/name-virtual-host-ingress-no-third-host.yaml" >}}
<!--
### TLS

You can secure an Ingress by specifying a {{< glossary_tooltip term_id="secret" >}}
that contains a TLS private key and certificate. The Ingress resource only
supports a single TLS port, 443, and assumes TLS termination at the ingress point
(traffic to the Service and its Pods is in plaintext).
If the TLS configuration section in an Ingress specifies different hosts, they are
multiplexed on the same port according to the hostname specified through the
SNI TLS extension (provided the Ingress controller supports SNI). The TLS secret
must contain keys named `tls.crt` and `tls.key` that contain the certificate
and private key to use for TLS. For example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: testsecret-tls
  namespace: default
data:
  tls.crt: base64 encoded cert
  tls.key: base64 encoded key
type: kubernetes.io/tls
```

Referencing this secret in an Ingress tells the Ingress controller to
secure the channel from the client to the load balancer using TLS. You need to make
sure the TLS secret you created came from a certificate that contains a Common
Name (CN), also known as a Fully Qualified Domain Name (FQDN) for `sslexample.foo.com`.

{{< codenew file="service/networking/tls-example-ingress.yaml" >}}

{{< note >}}
There is a gap between TLS features supported by various Ingress
controllers. Please refer to documentation on
[nginx](https://kubernetes.github.io/ingress-nginx/user-guide/tls/),
[GCE](https://git.k8s.io/ingress-gce/README.md#frontend-https), or any other
platform specific Ingress controller to understand how TLS works in your environment.
{{< /note >}}
 -->

### TLS

用户可以能过一个包含 TLS 私钥和证书的 {{< glossary_tooltip term_id="secret" >}} 为 Ingress
添加安全层。 Ingress 只支持一个 TLS 端口，`443`， 并且假定 TLS 在 Ingress 终结(也就是
到达 Service 及其 Pod 的流量是明文的)。
如果 Ingress TLS 配置区中包含了多个主机名，那么它们根据 SNI TLS 扩展(需要 Ingress 控制器
支持 SNI)中指定的主机名来区分请求。 TLS {{< glossary_tooltip term_id="secret" >}}
中必须要包含名叫 `tls.crt` 和 `tls.key` 的键，其它分别存放 TLS 的证书和私钥。 例如:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: testsecret-tls
  namespace: default
data:
  tls.crt: base64 encoded cert
  tls.key: base64 encoded key
type: kubernetes.io/tls
```

在一个 Ingress 中引用该 {{< glossary_tooltip term_id="secret" >}} 保证客户端到负载均衡器之间的
连接是使用 TLS。必须要保证创建 TLS Secret 的证书中包含公用名(CN), 也就是 `sslexample.foo.com`
的全限定名(FQDN).

{{< codenew file="service/networking/tls-example-ingress.yaml" >}}

{{< note >}}
不同的 Ingress 控制器对 TLS 特性的支持有较大差异。 请查阅
[nginx](https://kubernetes.github.io/ingress-nginx/user-guide/tls/),
[GCE](https://git.k8s.io/ingress-gce/README.md#frontend-https),
或其它平台相应的 Ingress 控制的说明文档，以理解在你的环境中 TLS 是怎么工作的。
{{< /note >}}
<!--
### Load balancing {#load-balancing}

An Ingress controller is bootstrapped with some load balancing policy settings
that it applies to all Ingress, such as the load balancing algorithm, backend
weight scheme, and others. More advanced load balancing concepts
(e.g. persistent sessions, dynamic weights) are not yet exposed through the
Ingress. You can instead get these features through the load balancer used for
a Service.

It's also worth noting that even though health checks are not exposed directly
through the Ingress, there exist parallel concepts in Kubernetes such as
[readiness probes](/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
that allow you to achieve the same end result. Please review the controller
specific documentation to see how they handle health checks (for example:
[nginx](https://git.k8s.io/ingress-nginx/README.md), or
[GCE](https://git.k8s.io/ingress-gce/README.md#health-checks)).
 -->

### 负载均衡 {#load-balancing}

Ingress 控制器天生带有一些会应用到所有 Ingress 负载均衡策略设置，如 负载均衡算法，后端权重，等。
更高级的负载均衡概念(如，持久会化，动态权重)目前还没能过 Ingress 实现。 要实现这些特性可以通过
Service 使用的负载均衡器。

还有值得注意的是健康检查不是直接通过 Ingress 暴露的。 k8s 中存在如
[存活探针](/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
的平行概念，允许用户达成同样的结果。 请查阅控制的说明文档，看看是怎么处理健康检查的(例如
  [nginx](https://git.k8s.io/ingress-nginx/README.md), or
  [GCE](https://git.k8s.io/ingress-gce/README.md#health-checks)).
)
<!--
## Updating an Ingress

To update an existing Ingress to add a new Host, you can update it by editing the resource:

```shell
kubectl describe ingress test
```

```
Name:             test
Namespace:        default
Address:          178.91.123.132
Default backend:  default-http-backend:80 (10.8.2.3:8080)
Rules:
  Host         Path  Backends
  ----         ----  --------
  foo.bar.com
               /foo   service1:80 (10.8.0.90:80)
Annotations:
  nginx.ingress.kubernetes.io/rewrite-target:  /
Events:
  Type     Reason  Age                From                     Message
  ----     ------  ----               ----                     -------
  Normal   ADD     35s                loadbalancer-controller  default/test
```

```shell
kubectl edit ingress test
```

This pops up an editor with the existing configuration in YAML format.
Modify it to include the new Host:

```yaml
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - backend:
          service:
            name: service1
            port:
              number: 80
        path: /foo
        pathType: Prefix
  - host: bar.baz.com
    http:
      paths:
      - backend:
          service:
            name: service2
            port:
              number: 80
        path: /foo
        pathType: Prefix
..
```

After you save your changes, kubectl updates the resource in the API server, which tells the
Ingress controller to reconfigure the load balancer.

Verify this:

```shell
kubectl describe ingress test
```

```
Name:             test
Namespace:        default
Address:          178.91.123.132
Default backend:  default-http-backend:80 (10.8.2.3:8080)
Rules:
  Host         Path  Backends
  ----         ----  --------
  foo.bar.com
               /foo   service1:80 (10.8.0.90:80)
  bar.baz.com
               /foo   service2:80 (10.8.0.91:80)
Annotations:
  nginx.ingress.kubernetes.io/rewrite-target:  /
Events:
  Type     Reason  Age                From                     Message
  ----     ------  ----               ----                     -------
  Normal   ADD     45s                loadbalancer-controller  default/test
```

You can achieve the same outcome by invoking `kubectl replace -f` on a modified Ingress YAML file.
 -->

## 更新 Ingress

要在已经存在的 Ingress 中添加一个新的主机规则，可以通过编辑 Ingress 资源实现:

```shell
kubectl describe ingress test
```

```
Name:             test
Namespace:        default
Address:          178.91.123.132
Default backend:  default-http-backend:80 (10.8.2.3:8080)
Rules:
  Host         Path  Backends
  ----         ----  --------
  foo.bar.com
               /foo   service1:80 (10.8.0.90:80)
Annotations:
  nginx.ingress.kubernetes.io/rewrite-target:  /
Events:
  Type     Reason  Age                From                     Message
  ----     ------  ----               ----                     -------
  Normal   ADD     35s                loadbalancer-controller  default/test
```

```shell
kubectl edit ingress test
```

这条命令会用文本编辑器打开已经存在的 Ingress  YAML 格式的配置文件，在其中添加新的主机规则配置:

```yaml
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - backend:
          service:
            name: service1
            port:
              number: 80
        path: /foo
        pathType: Prefix
  - host: bar.baz.com
    http:
      paths:
      - backend:
          service:
            name: service2
            port:
              number: 80
        path: /foo
        pathType: Prefix
..
```

在保存修改后， kubectl 就会在 API-server 中更新该资源，这会使得 Ingress 控制器重新配置
负载均衡器

通过以下命令验证:

```shell
kubectl describe ingress test
```

```
Name:             test
Namespace:        default
Address:          178.91.123.132
Default backend:  default-http-backend:80 (10.8.2.3:8080)
Rules:
  Host         Path  Backends
  ----         ----  --------
  foo.bar.com
               /foo   service1:80 (10.8.0.90:80)
  bar.baz.com
               /foo   service2:80 (10.8.0.91:80)
Annotations:
  nginx.ingress.kubernetes.io/rewrite-target:  /
Events:
  Type     Reason  Age                From                     Message
  ----     ------  ----               ----                     -------
  Normal   ADD     45s                loadbalancer-controller  default/test
```

也可以通过在外部修改 Ingress YAML 配置后，使用 `kubectl replace -f` 命令可以达成相同的效果。
<!--
## Failing across availability zones

Techniques for spreading traffic across failure domains differ between cloud providers.
Please check the documentation of the relevant [Ingress controller](/docs/concepts/services-networking/ingress-controllers) for details.
 -->
## 跨可用区失效

在故障域之间传播流量的方法在各家云提供商上的方式都不一样。
请查看 [Ingress 控制器](/k8sDocs/concepts/services-networking/ingress-controllers)文档了解详细信息
<!--
## Alternatives

You can expose a Service in multiple ways that don't directly involve the Ingress resource:

* Use [Service.Type=LoadBalancer](/docs/concepts/services-networking/service/#loadbalancer)
* Use [Service.Type=NodePort](/docs/concepts/services-networking/service/#nodeport)
 -->
## 替代方案 {#alternatives}

在不直接使用 Ingress 的情况下还有几种暴露 Service 的方法:

* 使用 [Service.Type=LoadBalancer](/k8sDocs/concepts/services-networking/service/#loadbalancer)
* 使用 [Service.Type=NodePort](/k8sDocs/concepts/services-networking/service/#nodeport)



## {{% heading "whatsnext" %}}

* 查阅 [Ingress API](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#ingress-v1beta1-networking-k8s-io)
* 概念 [Ingress 控制器](/docs/concepts/services-networking/ingress-controllers/)
* 实践 [使用 NGINX 控制器在 Minikube 上设置 Ingress](/docs/tasks/access-application-cluster/ingress-minikube/)
