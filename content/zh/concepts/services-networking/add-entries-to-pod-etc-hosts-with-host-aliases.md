---
title: 通过 HostAliases 向 Pod 的 /etc/hosts 文件中添加条目
content_type: concept
weight: 90
date: 2020-10-09
publishdate: 2020-10-09
min-kubernetes-server-version: 1.7
---
<!--
---
reviewers:
- rickypai
- thockin
title: Adding entries to Pod /etc/hosts with HostAliases
content_type: concept
weight: 60
min-kubernetes-server-version: 1.7
--- -->


<!-- overview -->
<!--
Adding entries to a Pod's `/etc/hosts` file provides Pod-level override of hostname resolution when DNS and other options are not applicable. You can add these custom entries with the HostAliases field in PodSpec.

Modification not using HostAliases is not suggested because the file is managed by the kubelet and can be overwritten on during Pod creation/restart.
-->

在 DNS 和其它方式都不可用时，想要向通过向 Pod 的 `/etc/hosts` 文件添加条目的方式来重写 Pod
级别的域名解析，可能通过 `PodSpec` 的 `HostAliases` 字段向该 Pod 的 `/etc/hosts` 添加自定义条目。

不建议直接修改文件而不使用 `HostAliases`， 因为这个文件是受 kubelet 管理并且在 Pod 的重新创建
或重启(能重启？)时会重写该文件，也就是直接修改文件在重建后会丢失。

<!-- body -->
<!--
## Default hosts file content

Start an Nginx Pod which is assigned a Pod IP:

```shell
kubectl run nginx --image nginx
```

```
pod/nginx created
```

Examine a Pod IP:

```shell
kubectl get pods --output=wide
```

```
NAME     READY     STATUS    RESTARTS   AGE    IP           NODE
nginx    1/1       Running   0          13s    10.200.0.4   worker0
```

The hosts file content would look like this:

```shell
kubectl exec nginx -- cat /etc/hosts
```

```
# Kubernetes-managed hosts file.
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
fe00::0	ip6-mcastprefix
fe00::1	ip6-allnodes
fe00::2	ip6-allrouters
10.200.0.4	nginx
```

By default, the `hosts` file only includes IPv4 and IPv6 boilerplates like
`localhost` and its own hostname.
 -->

## hosts 默认的内容

启动一个 Nginx 的 Pod，它会被分配一个 Pod IP:

```shell
kubectl run nginx --image nginx
```

```
pod/nginx created
```

检查 Pod IP:

```shell
kubectl get pods --output=wide
```

```
NAME     READY     STATUS    RESTARTS   AGE    IP           NODE
nginx    1/1       Running   0          13s    10.200.0.4   worker0
```

通过以下命令查看 hosts 文件内容

```shell
kubectl exec nginx -- cat /etc/hosts
```

输出结果类似如下:
```
# Kubernetes-managed hosts file.
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
fe00::0	ip6-mcastprefix
fe00::1	ip6-allnodes
fe00::2	ip6-allrouters
10.200.0.4	nginx
```

默认情况下， `hosts` 文件只会包含 IPv4/IPv6 如 `localhost` 这样的模板条目和主机名条目
<!--
## Adding additional entries with hostAliases

In addition to the default boilerplate, you can add additional entries to the
`hosts` file.
For example: to resolve `foo.local`, `bar.local` to `127.0.0.1` and `foo.remote`,
`bar.remote` to `10.1.2.3`, you can configure HostAliases for a Pod under
`.spec.hostAliases`:

{{< codenew file="service/networking/hostaliases-pod.yaml" >}}

You can start a Pod with that configuration by running:

```shell
kubectl apply -f https://k8s.io/examples/service/networking/hostaliases-pod.yaml
```

```
pod/hostaliases-pod created
```

Examine a Pod's details to see its IPv4 address and its status:

```shell
kubectl get pod --output=wide
```

```
NAME                           READY     STATUS      RESTARTS   AGE       IP              NODE
hostaliases-pod                0/1       Completed   0          6s        10.200.0.5      worker0
```

The `hosts` file content looks like this:

```shell
kubectl logs hostaliases-pod
```

```
# Kubernetes-managed hosts file.
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
fe00::0	ip6-mcastprefix
fe00::1	ip6-allnodes
fe00::2	ip6-allrouters
10.200.0.5	hostaliases-pod

# Entries added by HostAliases.
127.0.0.1	foo.local	bar.local
10.1.2.3	foo.remote	bar.remote
```

with the additional entries specified at the bottom. -->

## 通过 hostAliases 添加额外的条目

In addition to the default boilerplate, you can add additional entries to the
`hosts` file.
除了模板条目，可以手动向 `hosts` 文件添加条目

例如下面的示例中通过 Pod 的 `.spec.hostAliases` 字段实现配置：
将 `foo.local`, `bar.local` 解析到 `127.0.0.1`，
将 `foo.remote`, `bar.remote` 解析到 `10.1.2.3`

{{< codenew file="service/networking/hostaliases-pod.yaml" >}}

用户可以通过以下命令，使用该配置启动一个 Pod:

```shell
kubectl apply -f https://k8s.io/examples/service/networking/hostaliases-pod.yaml
```

```
pod/hostaliases-pod created
```

通过以下命令查看 Pod 的信息，查看其 IPv4 地址及其状态:

```shell
kubectl get pod --output=wide
```

```
NAME                           READY     STATUS      RESTARTS   AGE       IP              NODE
hostaliases-pod                0/1       Completed   0          6s        10.200.0.5      worker0
```

通过以下命令查看 `hosts` 文件的内存:

```shell
kubectl logs hostaliases-pod
```
输出结果类似:

```
# Kubernetes-managed hosts file.
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
fe00::0	ip6-mcastprefix
fe00::1	ip6-allnodes
fe00::2	ip6-allrouters
10.200.0.5	hostaliases-pod

# Entries added by HostAliases.
127.0.0.1	foo.local	bar.local
10.1.2.3	foo.remote	bar.remote
```

额外添加的条目的文件的底部。
<!--
## Why does the kubelet manage the hosts file? {#why-does-kubelet-manage-the-hosts-file}

The kubelet [manages](https://github.com/kubernetes/kubernetes/issues/14633) the
`hosts` file for each container of the Pod to prevent Docker from
[modifying](https://github.com/moby/moby/issues/17190) the file after the
containers have already been started.

{{< caution >}}
Avoid making manual changes to the hosts file inside a container.

If you make manual changes to the hosts file,
those changes are lost when the container exits.
{{< /caution >}}
 -->
## 为啥要 kubelet 管理 hosts 文件? {#why-does-kubelet-manage-the-hosts-file}

使用 kubelet [管理](https://github.com/kubernetes/kubernetes/issues/14633) the
Pod 中每个容器的 `hosts` 文件，是防止 Docker 在容器启动后再
[修改](https://github.com/moby/moby/issues/17190) 该文件。

{{< caution >}}
避免手动修改容器中的 `hosts` 文件

如果手动修改了 `hosts` 文件， 这些改动会在容器退出时丢失。
{{< /caution >}}
