---
title: Ingress 控制器
weight: 70
date: 2020-09-23
publishdate: 2020-09-23
---
<!--
---
title: Ingress Controllers
reviewers:
content_type: concept
weight: 40
--- -->

<!-- overview -->
<!--
In order for the Ingress resource to work, the cluster must have an ingress controller running.

Unlike other types of controllers which run as part of the `kube-controller-manager` binary, Ingress controllers
are not started automatically with a cluster. Use this page to choose the ingress controller implementation
that best fits your cluster.

Kubernetes as a project currently supports and maintains [GCE](https://git.k8s.io/ingress-gce/README.md) and
  [nginx](https://git.k8s.io/ingress-nginx/README.md) controllers.
 -->

要使一个 Ingress 工作的前提是集群中必须要有一个 Ingress 控制器在运行。

与其它类型的控制器作为 `kube-controller-manager` 的一部分运行不同， Ingress 不会自动在集群中
运行。 使用本文选择最适合你的集群的 Ingress 控制器实现。

k8s 项目目前支持和维护
[GCE](https://git.k8s.io/ingress-gce/README.md)
和
[nginx](https://git.k8s.io/ingress-nginx/README.md)
控制器


<!-- body -->
<!--
## Additional controllers

* [AKS Application Gateway Ingress Controller](https://github.com/Azure/application-gateway-kubernetes-ingress) is an ingress controller that enables ingress to [AKS clusters](https://docs.microsoft.com/azure/aks/kubernetes-walkthrough-portal) using the [Azure Application Gateway](https://docs.microsoft.com/azure/application-gateway/overview).
* [Ambassador](https://www.getambassador.io/) API Gateway is an [Envoy](https://www.envoyproxy.io) based ingress
  controller with [community](https://www.getambassador.io/docs) or
  [commercial](https://www.getambassador.io/pro/) support from [Datawire](https://www.datawire.io/).
* [AppsCode Inc.](https://appscode.com) offers support and maintenance for the most widely used [HAProxy](https://www.haproxy.org/) based ingress controller [Voyager](https://appscode.com/products/voyager).
* [AWS ALB Ingress Controller](https://github.com/kubernetes-sigs/aws-alb-ingress-controller) enables ingress using the [AWS Application Load Balancer](https://aws.amazon.com/elasticloadbalancing/).
* [Contour](https://projectcontour.io/) is an [Envoy](https://www.envoyproxy.io/) based ingress controller
  provided and supported by VMware.
* Citrix provides an [Ingress Controller](https://github.com/citrix/citrix-k8s-ingress-controller) for its hardware (MPX), virtualized (VPX) and [free containerized (CPX) ADC](https://www.citrix.com/products/citrix-adc/cpx-express.html) for [baremetal](https://github.com/citrix/citrix-k8s-ingress-controller/tree/master/deployment/baremetal) and [cloud](https://github.com/citrix/citrix-k8s-ingress-controller/tree/master/deployment) deployments.
* F5 Networks provides [support and maintenance](https://support.f5.com/csp/article/K86859508)
  for the [F5 BIG-IP Container Ingress Services for Kubernetes](https://clouddocs.f5.com/containers/latest/userguide/kubernetes/).
* [Gloo](https://gloo.solo.io) is an open-source ingress controller based on [Envoy](https://www.envoyproxy.io) which offers API Gateway functionality with enterprise support from [solo.io](https://www.solo.io).  
* [HAProxy Ingress](https://haproxy-ingress.github.io) is a highly customizable community-driven ingress controller for HAProxy.
* [HAProxy Technologies](https://www.haproxy.com/) offers support and maintenance for the [HAProxy Ingress Controller for Kubernetes](https://github.com/haproxytech/kubernetes-ingress). See the [official documentation](https://www.haproxy.com/documentation/hapee/1-9r1/traffic-management/kubernetes-ingress-controller/).
* [Istio](https://istio.io/) based ingress controller
  [Control Ingress Traffic](https://istio.io/docs/tasks/traffic-management/ingress/).
* [Kong](https://konghq.com/) offers [community](https://discuss.konghq.com/c/kubernetes) or
  [commercial](https://konghq.com/kong-enterprise/) support and maintenance for the
  [Kong Ingress Controller for Kubernetes](https://github.com/Kong/kubernetes-ingress-controller).
* [NGINX, Inc.](https://www.nginx.com/) offers support and maintenance for the
  [NGINX Ingress Controller for Kubernetes](https://www.nginx.com/products/nginx/kubernetes-ingress-controller).
* [Skipper](https://opensource.zalando.com/skipper/kubernetes/ingress-controller/) HTTP router and reverse proxy for service composition, including use cases like Kubernetes Ingress, designed as a library to build your custom proxy
* [Traefik](https://github.com/containous/traefik) is a fully featured ingress controller
  ([Let's Encrypt](https://letsencrypt.org), secrets, http2, websocket), and it also comes with commercial
  support by [Containous](https://containo.us/services).
 -->

## 其它的控制器

- [AKS Application Gateway Ingress Controller](https://github.com/Azure/application-gateway-kubernetes-ingress)
  是使用
  [Azure Application Gateway](https://docs.microsoft.com/azure/application-gateway/overview)
  为
  [AKS 集群](https://docs.microsoft.com/azure/aks/kubernetes-walkthrough-portal)
  提供入口的 Ingress 控制器
- [Ambassador](https://www.getambassador.io/) API 网关
  是一个基于
  [Envoy](https://www.envoyproxy.io)
  的 Ingress 控制器
  有 [社区](https://www.getambassador.io/docs) 支持和
  来自
  [Datawire](https://www.datawire.io/)
  的
  [商业](https://www.getambassador.io/pro/) 支持
- [AppsCode Inc.](https://appscode.com) 提供了最广泛使用的基于
  [HAProxy](https://www.haproxy.org/)
  的 Ingress 控制器
  [Voyager](https://appscode.com/products/voyager)
- [AWS ALB Ingress Controller](https://github.com/kubernetes-sigs/aws-alb-ingress-controller)
  使用
  [AWS Application Load Balancer](https://aws.amazon.com/elasticloadbalancing/)
  实现 Ingress
- [Contour](https://projectcontour.io/)
  由 VMware 提供和支持的基于 [Envoy](https://www.envoyproxy.io/) 的 Ingress 控制器
- Citrix 为它的
  [baremetal](https://github.com/citrix/citrix-k8s-ingress-controller/tree/master/deployment/baremetal)
  和
  [cloud](https://github.com/citrix/citrix-k8s-ingress-controller/tree/master/deployment)
  上部署的硬件(MPX),虚拟化(VPX)和
  [free containerized (CPX) ADC](https://www.citrix.com/products/citrix-adc/cpx-express.html)
  提供的 [Ingress 控制器](https://github.com/citrix/citrix-k8s-ingress-controller)
- `F5 Networks` 为
  [F5 BIG-IP Container Ingress Services for Kubernetes](https://clouddocs.f5.com/containers/latest/userguide/kubernetes/)
  提供 [支持和维护](https://support.f5.com/csp/article/K86859508)
- [Gloo](https://gloo.solo.io) 是基于
  [Envoy](https://www.envoyproxy.io) 的开源 Ingress 控制器
  提供了来自 [solo.io](https://www.solo.io) 企业支持的 API 网络功能
- [HAProxy Ingress](https://haproxy-ingress.github.io) 用于 HAProxy 调度可定制的社区驱动的 Ingress 控制器
- [HAProxy Technologies](https://www.haproxy.com/) 为
  [HAProxy Ingress Controller for Kubernetes](https://github.com/haproxytech/kubernetes-ingress)
  提供支持和维护。 详见
  [官方文档](https://www.haproxy.com/documentation/hapee/1-9r1/traffic-management/kubernetes-ingress-controller/)
- 基于 [Istio](https://istio.io/) 的
  [Control Ingress Traffic](https://istio.io/docs/tasks/traffic-management/ingress/)
- [Kong](https://konghq.com/) 为
  [Kong Ingress Controller for Kubernetes](https://github.com/Kong/kubernetes-ingress-controller)
  提供
  [社区](https://discuss.konghq.com/c/kubernetes)
  或
  [商业](https://konghq.com/kong-enterprise/)的
  支持和维护
  [NGINX Ingress Controller for Kubernetes](https://www.nginx.com/products/nginx/kubernetes-ingress-controller).
- [NGINX, Inc.](https://www.nginx.com/) 为
  [NGINX Ingress Controller for Kubernetes](https://www.nginx.com/products/nginx/kubernetes-ingress-controller)
  提供支持和维护
- [Skipper](https://opensource.zalando.com/skipper/kubernetes/ingress-controller/)
  为 Service 提供 HTTP 路由 和反向代理。还包含了类似 k8s Ingress 的使用场景，
  设计上可以作为自定义代理的库
- [Traefik](https://github.com/containous/traefik) 是个功能齐全的 Ingress 控制器
  ([Let's Encrypt](https://letsencrypt.org), secrets, http2, websocket)
  还有来自 [Containous](https://containo.us/services) 的商业支持
<!--
## Using multiple Ingress controllers

You may deploy [any number of ingress controllers](https://git.k8s.io/ingress-nginx/docs/user-guide/multiple-ingress.md#multiple-ingress-controllers)
within a cluster. When you create an ingress, you should annotate each ingress with the appropriate
[`ingress.class`](https://git.k8s.io/ingress-gce/docs/faq/README.md#how-do-i-run-multiple-ingress-controllers-in-the-same-cluster)
to indicate which ingress controller should be used if more than one exists within your cluster.

If you do not define a class, your cloud provider may use a default ingress controller.

Ideally, all ingress controllers should fulfill this specification, but the various ingress
controllers operate slightly differently.

{{< note >}}
Make sure you review your ingress controller's documentation to understand the caveats of choosing it.
{{< /note >}}
 -->

## Using multiple Ingress controllers

用户可以在集群中部署
[任意数量的 Ingress 控制器](https://git.k8s.io/ingress-nginx/docs/user-guide/multiple-ingress.md#multiple-ingress-controllers)
如果集群中有多个 Ingress 控制器，在创建 Ingress 时需要为每个 Ingress 标注恰当的
[`ingress.class`](https://git.k8s.io/ingress-gce/docs/faq/README.md#how-do-i-run-multiple-ingress-controllers-in-the-same-cluster)
来指示使用哪个 Ingress 控制器。

如果在 Ingress 中没有定义控制器类型，云提供商可能使用一个默认的 Ingress 控制器。

理想情况下，所有的 Ingress 控制器都应当满足这个规范，但是很多 Ingress 控制器动作方式都有点不同。

{{< note >}}
一定要仔细阅读你选择的 Ingress 控制器的文档，确保理解了相关注意事项。
{{< /note >}}



## {{% heading "whatsnext" %}}


* 概念 [Ingress](/k8sDocs/docs/concepts/services-networking/ingress/).
* 实践 [使用 NGINX 控制器在 Minikube 上设置 Ingress](/k8sDocs/tasks/access-application-cluster/ingress-minikube).
