---
title: 控制中心与节点的通信
date: 2020-07-08
publishdate: 2020-07-08
weight: 20101
---
本文主要介结控制中心(实际就是 api-server)与 k8s 集群通信的几种途径, 目的是使用户可以选择健壮的网络配置使集群可以运行在不受信的网络环境中(或在云提供商的公网IP上)

## 从节点到控制中心的通信

k8s 使用轴辐式(hub-and-spoke) API 模式，所有节点(包括节点上运行的Pod)使用到的API都指向 api-server(其它所有的组件在设计上就不提供远程服务)。api-server 配置成安全的HTTPS端口(通常是 443)来对外提供服务，并且开启一种或多种[认证](../../../5-reference/03-access-authn-authz/01-authentication/)方式。还需要开启一种或多种[授权](../../../5-reference/03-access-authn-authz/07-authorization/)方式， 特别是 匿名请求 或 service account tokens 可用(具体见认证相关部分)。
每个节点上都需要有集群的公开根证书，这样节点才能通过有效的客户凭据安全的连接到 api-server. 例如， 在默认的 GKE 部署中， 客户端为 kubelet 提供的凭据格式为客户端证书， 自动化提供客户端证书的方式见[这里](../../../5-reference/06-command-line-tools-reference/08-kubelet-tls-bootstrapping/)
如果集群中的Pod想要连接到 api-server 可以借助`service account`实现安全连接， k8s 会在 Pod 启动时自动注入公开根谈不上和令牌.
在每个名字空间下有一个叫 `kubernetes` 的 Service， 指向一个虚拟IP地址，并重写向(通过 kube-proxy)向 api-server 的HTTPS端口上。
控制中心组件也是通过安全端口与集群的 api-server 通信。
在默认的操作模式下，节点和节点上的Pod 与控制中心的连接默认就是安全的可以信赖于不受信的网络环境中

## 从控制中心向节点的通信

由控制中心(api-server)向节点的通信路径注要有两条， 第一条从 api-server 直接连接到集群中每个节点上的 kubelet 进程。 第二条是通过 api-server 的代理功能来实现 api-server 到任意节点， Pod， Service的连接

### 从 api-server 连接到 kubelet

从 api-server 到 kubelet 通信主要有以下用途:
- 摘取 Pod 日志
- 通过 kubelet 终端连接到运行的Pod
- 为 kubelet 提供端口转发功能

连接指向的是 kubelet 的 HTTPS 端口， 默认 api-server 不验证 kubelet 提供的证书，这样会存在中间人攻击的风险，在不受网络环境中是不安全的。

要验证这个连接，需要在 api-server 配置启动参数 `--kubelet-certificate-authority` 指向 根证书用于验证 kubelet 的服务证书。
如果做不到， 则在非受信网络或公网中使用 SSH 遂道连接 api-server 和 kubelet.
最后，需要开启 [kubelet 认证和授权](../../../5-reference/06-command-line-tools-reference/07-kubelet-authentication-authorization/)，以保护 kubelet API 安全。

### 从 api-server 到 kubelet, 节点, Pod,  Service 的连接

默认情况下 从 api-server 到 kubelet, 节点, Pod,  Service 的连接是通过 HTTP 明文，因此没有认证也没加密。在连接到 节点， Pod, Service 名称对应的 API URL时可以添加 https 前缀来使用 HTTPS 来建立安全连接，但不会验证HTTPS证书也不验证客户端提供的凭据。 因此也不能保证任何完整性。 所以目前这能连接都不能用于非受信网络或公网。

### SSH 遂道

k8s 支持通过 SSH 遂道的方式来实现从 控制中心到节点的连接路径的安全。 在这个配置中， 由 api-server 来初始化 SSH 遂道连接到集群中每个节点(连接到ssh服务监听端口22)，并通过这个连接来传输 kubelet, 节点, Pod,  Service 所有流量， 这样可以使流量不会显露给节点运行的外部网络

SSH 遂道连接方式当前已经被废弃，用户在开启该功能需要清楚为什么为开启。  Konnectivity 服务是替代方案

### Konnectivity 服务

{{< feature-state for_k8s_version="v1.18" state="beta" >}}

作为 SSH 遂道 的替代方案， Konnectivity 服务 通过提供 TCP 层的代理来实现 控制中心与集群之间的通信。 Konnectivity 服务主要由两部分组成， Konnectivity 服务端和Konnectivity 代理程序，分别运行在 控制中心网络和节点网络上。 Konnectivity 代理程序初始化并维护到服务端的连接。 在开启 Konnectivity 服务后，控制中心到节的所有连接都通过这些连接。
在集群中开启 Konnectivity 服务见 [这里](../../../3-tasks/09-extend-kubernetes/01-setup-konnectivity/)
