---
title: IPv4/IPv6 双栈
weight: 100
date: 2020-10-09
publishdate: 2020-10-10
content_type: concept
---
<!--
---
reviewers:
- lachie83
- khenidak
- aramase
title: IPv4/IPv6 dual-stack
feature:
  title: IPv4/IPv6 dual-stack
  description: >
    Allocation of IPv4 and IPv6 addresses to Pods and Services

content_type: concept
weight: 70
---
-->

<!-- overview -->
<!--
{{< feature-state for_k8s_version="v1.16" state="alpha" >}}

 IPv4/IPv6 dual-stack enables the allocation of both IPv4 and IPv6 addresses to {{< glossary_tooltip text="Pods" term_id="pod" >}} and {{< glossary_tooltip text="Services" term_id="service" >}}.

If you enable IPv4/IPv6 dual-stack networking for your Kubernetes cluster, the cluster will support the simultaneous assignment of both IPv4 and IPv6 addresses.
 -->
{{< feature-state for_k8s_version="v1.16" state="alpha" >}}

IPv4/IPv6 双栈让 {{< glossary_tooltip term_id="pod" >}} 和
{{< glossary_tooltip term_id="service" >}} 能够同时分配到 IPv4 和 IPv6 地址。

如果在 k8s 集群中启用了 IPv4/IPv6 双栈网络，则集群支持同时分配 IPv4 和 IPv6 地址。

<!-- body -->
<!--
## Supported Features

Enabling IPv4/IPv6 dual-stack on your Kubernetes cluster provides the following features:

   * Dual-stack Pod networking (a single IPv4 and IPv6 address assignment per Pod)
   * IPv4 and IPv6 enabled Services (each Service must be for a single address family)
   * Pod off-cluster egress routing (eg. the Internet) via both IPv4 and IPv6 interfaces
 -->

## 支持的特性

在集群中开启 IPv4/IPv6 双栈可以提供以下特性:
   - Pod 双栈网络(每个 Pod 分配一个 IPv4 和 IPv6 地址)
   - Service 启用 IPv4 和 IPv6 (每个 Service 只能是 IPv4 或 IPv6)
   - Pod 的出站路由(如，到互联网)会同时通过 IPv4 和 IPv6 接口
<!--
## Prerequisites

The following prerequisites are needed in order to utilize IPv4/IPv6 dual-stack Kubernetes clusters:

   * Kubernetes 1.16 or later
   * Provider support for dual-stack networking (Cloud provider or otherwise must be able to provide Kubernetes nodes with routable IPv4/IPv6 network interfaces)
   * A network plugin that supports dual-stack (such as Kubenet or Calico)
 -->

## 前置条件

为集群启用 IPv4/IPv6 双栈需要做以下准备:
   - k8s `v1.16+`
   - 提供商支持双栈网络(云提供商或其它的提供者必须要为 k8s 节点提供IPv4/IPv6网络接口)
   - 一个支持双栈的网络插件(如 Kubenet 或 Calico)
<!--
## Enable IPv4/IPv6 dual-stack

To enable IPv4/IPv6 dual-stack, enable the `IPv6DualStack` [feature gate](/docs/reference/command-line-tools-reference/feature-gates/) for the relevant components of your cluster, and set dual-stack cluster network assignments:

   * kube-apiserver:
      * `--feature-gates="IPv6DualStack=true"`
      * `--service-cluster-ip-range=<IPv4 CIDR>,<IPv6 CIDR>`
   * kube-controller-manager:
      * `--feature-gates="IPv6DualStack=true"`
      * `--cluster-cidr=<IPv4 CIDR>,<IPv6 CIDR>`
      * `--service-cluster-ip-range=<IPv4 CIDR>,<IPv6 CIDR>`
      * `--node-cidr-mask-size-ipv4|--node-cidr-mask-size-ipv6` defaults to /24 for IPv4 and /64 for IPv6
   * kubelet:
      * `--feature-gates="IPv6DualStack=true"`
   * kube-proxy:
      * `--cluster-cidr=<IPv4 CIDR>,<IPv6 CIDR>`
      * `--feature-gates="IPv6DualStack=true"`

{{< note >}}
An example of an IPv4 CIDR: `10.244.0.0/16` (though you would supply your own address range)

An example of an IPv6 CIDR: `fdXY:IJKL:MNOP:15::/64` (this shows the format but is not a valid address - see [RFC 4193](https://tools.ietf.org/html/rfc4193))

{{< /note >}}
 -->

## 启用 IPv4/IPv6 双栈

要启用 IPv4/IPv6 双栈, 需要打开集群中的对应组件 `IPv6DualStack` [功能阀](/docs/reference/command-line-tools-reference/feature-gates/)
并设置集群双栈网络分配:

   * kube-apiserver:
      * `--feature-gates="IPv6DualStack=true"`
      * `--service-cluster-ip-range=<IPv4 CIDR>,<IPv6 CIDR>`
   * kube-controller-manager:
      * `--feature-gates="IPv6DualStack=true"`
      * `--cluster-cidr=<IPv4 CIDR>,<IPv6 CIDR>`
      * `--service-cluster-ip-range=<IPv4 CIDR>,<IPv6 CIDR>`
      * `--node-cidr-mask-size-ipv4|--node-cidr-mask-size-ipv6` IPv4 默认为 /24；IPv6 默认为 /64
   * kubelet:
      * `--feature-gates="IPv6DualStack=true"`
   * kube-proxy:
      * `--cluster-cidr=<IPv4 CIDR>,<IPv6 CIDR>`
      * `--feature-gates="IPv6DualStack=true"`

{{< note >}}
一个 IPv4 CIDR 示例: `10.244.0.0/16` (应该根据需要设置IP范围)

一个 IPv6 CIDR 示例: `fdXY:IJKL:MNOP:15::/64` (这里只显示格式，但不是一个有效的值 - 具体见 [RFC 4193](https://tools.ietf.org/html/rfc4193))

{{< /note >}}
<!--
## Services

If your cluster has IPv4/IPv6 dual-stack networking enabled, you can create {{< glossary_tooltip text="Services" term_id="service" >}} with either an IPv4 or an IPv6 address. You can choose the address family for the Service's cluster IP by setting a field, `.spec.ipFamily`, on that Service.
You can only set this field when creating a new Service. Setting the `.spec.ipFamily` field is optional and should only be used if you plan to enable IPv4 and IPv6 {{< glossary_tooltip text="Services" term_id="service" >}} and {{< glossary_tooltip text="Ingresses" term_id="ingress" >}} on your cluster. The configuration of this field not a requirement for [egress](#egress-traffic) traffic.

{{< note >}}
The default address family for your cluster is the address family of the first service cluster IP range configured via the `--service-cluster-ip-range` flag to the kube-controller-manager.
{{< /note >}}

You can set `.spec.ipFamily` to either:

   * `IPv4`: The API server will assign an IP from a `service-cluster-ip-range` that is `ipv4`
   * `IPv6`: The API server will assign an IP from a `service-cluster-ip-range` that is `ipv6`

The following Service specification does not include the `ipFamily` field. Kubernetes will assign an IP address (also known as a "cluster IP") from the first configured `service-cluster-ip-range` to this Service.

{{< codenew file="service/networking/dual-stack-default-svc.yaml" >}}

The following Service specification includes the `ipFamily` field. Kubernetes will assign an IPv6 address (also known as a "cluster IP") from the configured `service-cluster-ip-range` to this Service.

{{< codenew file="service/networking/dual-stack-ipv6-svc.yaml" >}}

For comparison, the following Service specification will be assigned an IPv4 address (also known as a "cluster IP") from the configured `service-cluster-ip-range` to this Service.

{{< codenew file="service/networking/dual-stack-ipv4-svc.yaml" >}}
 -->

## Service

如果集群启用的了 IPv4/IPv6 双栈网络，就可以创建一个带 IPv4 或 IPv6 地址的 {{< glossary_tooltip term_id="service" >}}。
可以通过 Service 的 `.spec.ipFamily` 来设置该 Service 是使用 IPv4 还是 IPv6 地址。
只有在创建一个新的 Service 才可以设置该字段。 `.spec.ipFamily` 是一个可选字段，只有在需要
在 {{< glossary_tooltip term_id="service" >}} 和 {{< glossary_tooltip term_id="ingress" >}}
上启用 IPv4 和 IPv6 时才需要配置该字段。 (这里还有一句不太明白在说啥)

{{< note >}}
集群默认使用的 IP 族是 kube-controller-manager 上设置 `--service-cluster-ip-range`的
值中，前面那一个 CIDR 对应的 IP 族。
{{< /note >}}

`.spec.ipFamily` 字段的值可以是:

   * `IPv4`: API server 会从 `service-cluster-ip-range` 中分配一个 `ipv4` 地址
   * `IPv6`: API server 会从 `service-cluster-ip-range` 中分配一个 `ipv6` 地址

下面的这个 Service 配置文件中没有包含 `ipFamily` 字段。 k8s 会为它分配一个 IP 地址
(也就是集群IP)，这个地址是 `service-cluster-ip-range` 配置值中第一个个值所定义的 IP 范围内的一个 IP 地址

{{< codenew file="service/networking/dual-stack-default-svc.yaml" >}}

下面的这个 Service 配置文件中包含 `ipFamily` 字段。 k8s 会为它分配一个 IPv6 地址(也是集群IP)
这个地址在 `service-cluster-ip-range` 配置的范围内

{{< codenew file="service/networking/dual-stack-ipv6-svc.yaml" >}}

为了对应，下面的这个 Service 的配置会被分配一个 IPv4 地址(作为集群IP)，这个地址在 `service-cluster-ip-range` 配置的范围内

{{< codenew file="service/networking/dual-stack-ipv4-svc.yaml" >}}
<!--
### Type LoadBalancer

On cloud providers which support IPv6 enabled external load balancers, setting the `type` field to `LoadBalancer` in additional to setting `ipFamily` field to `IPv6` provisions a cloud load balancer for your Service.
 -->

### Type LoadBalancer

在支持 IPv6 的云提供商上启用外部负载均衡器时，在 Service 上设置 `type` 字段值为 `LoadBalancer` 时，同
时还需要设置 `ipFamily` 字段的值为 `IPv6`。
<!--
## Egress Traffic

The use of publicly routable and non-publicly routable IPv6 address blocks is acceptable provided the underlying {{< glossary_tooltip text="CNI" term_id="cni" >}} provider is able to implement the transport. If you have a Pod that uses non-publicly routable IPv6 and want that Pod to reach off-cluster destinations (eg. the public Internet), you must set up IP masquerading for the egress traffic and any replies. The [ip-masq-agent](https://github.com/kubernetes-incubator/ip-masq-agent) is dual-stack aware, so you can use ip-masq-agent for IP masquerading on dual-stack clusters.
 -->
## Egress Traffic {#egress-traffic}

底层实现的 {{< glossary_tooltip text="CNI" term_id="cni" >}} 提供者可以提供对
公网可路由和公网不可路由的 IPv6 地址段。如果 Pod 使用的是公网不可路由的 IPv6 地址，但要求
这个 Pod 可以访问集群外的目标(如，互联网)， 需要为外出流量及其应答做 IP 地址转换。
[ip-masq-agent](https://github.com/kubernetes-incubator/ip-masq-agent) 支持双栈，
所以可以使用它在双栈集群中做地址转换。
<!--
## Known Issues

   * Kubenet forces IPv4,IPv6 positional reporting of IPs (--cluster-cidr)
 -->

## 已知问题

   * Kubenet forces IPv4,IPv6 positional reporting of IPs (--cluster-cidr)
   - kubenet 强制 IPv4,IPv6 位置汇报  (--cluster-cidr)

## {{% heading "whatsnext" %}}


* 实践 [验证 IPv4/IPv6 双栈网络](/k8sDocs/tasks/network/validate-dual-stack)
