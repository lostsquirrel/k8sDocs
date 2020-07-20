---
title: 镜像
date: 2020-07-19
draft: true
weight: 20200
---
<!-- overview -->

一个容器镜像就是构建应用的二进制数据和应用所以需要的依赖。 容器镜像是一个可独立运行的可执行软件包，并定义了清楚的运行环境。
通常是用户创建一个应用的容器镜像，推送到镜像仓库，在 Pod 中引用这个镜像。

本文介绍容器镜像的主要概念
<!-- body -->

## 镜像名称

镜像的名称通常是长成这些个样子的 `pause`, `example/mycontainer`, `kube-apiserver`。 镜像名也可以带上镜像仓库的主机名(自建或第三方镜像仓库默认都要带上)，如：`fictional.registry.example/imagename`。也可能有端口号，如: `fictional.registry.example:10443/imagename`

如果镜像名称上没有镜像仓库的主机名， 则 k8s 认为使用的是 Docker 公共镜像仓库。

在镜像名称的后面通常还会有一个标签(`tag`)(就和 `docker` 与 `podman` 命令用的的一样)。 标签用于区分同一系列镜像的不同版本。

标签名的命名规范为： 大小写字母，数字，下划线(`_`)，点(`.`)中划线(`-`)，但分隔符(`_`,`.`,`-`)有使用限制
如果镜像名称后没有标签，则默认使用 `latest` 作为标签。

{{ <warning> }}
在生产环境部署中尽量避免使用 latest 标签。 这样很难跟踪当前运行的是哪个版本，要做版本回退就更难了。
所以建议使用有含意的标签，如: `v1.42.0`
{{ </warning> }}

## 镜像更新策略

默认的更新策略为 `IfNotPresent`, 就是让 kubelet 在找不到镜像时才从仓库拉取。 如果想要每次都强制拉取，则可以通过以下任意一种方式实现：

- 将容器 `imagePullPolicy` 的值设置为 `Always`
- 不设置 `imagePullPolicy` 并让镜像使用 `latest` 标签
- 不设置 `imagePullPolicy` 并镜像也不设置标签
- 打开  [AlwaysPullImages](../../../5-reference/03-access-authn-authz/04-admission-controllers/#alwayspullimages) admission controller

如果配置了 `imagePullPolicy` 但没有设置值，则默认为 `Always`

## 带清单(Manifest)的多架构镜像

镜像仓库在提供二进制镜像的同时也可以提供 [容器镜像清单](https://github.com/opencontainers/image-spec/blob/master/manifest.md), 这个清单中列举了不同架构的镜像引用层。这样虽然镜像只有一个名字(如: `pause`, `example/mycontainer`, `kube-apiserve`), 但不同的操作系统架构可以拉取到对应架构的镜像。

在 k8s 中容器镜像名称是带有后缀 `-$(ARCH)`的。 为了向后兼容， 请为旧的镜像添加后缀。 比如生成包含所有架构清单的镜像叫 `pause`，`pause-amd64` 就是为向后兼容旧的配置或可能存在于YAML配置文件中带后续硬编码镜像名

## 使用私有仓库

从私有仓库拉取镜像时一般都需要提供凭据。
以下为提供凭据的几种方式:

- 在节点上配置仓库认证
  - 所有的 Pod 都对仓库有全部访问权限
  - 需要集群管理员对节点进行配置
- 预先下载镜像
  - 所有的 Pod 可以使用节点是缓存的镜像
  - 需要在节点上以 root 用户配置

- 基于提供商或本地插件
  - 如果使用自定义节点配置， 用户(或云提供商)可以自己在节点上实现向镜像仓库的认证

以下为对这些认证方式更详细的说明

### 在节点上配置仓库认证

如果节点上运行的是 Docker 用户可以通过配置 Docker 运行时向私有仓库进行认证
这种方式适用于用于能近控制节点配置

{{ <note> }}
k8s 只支持 Docker 配置的 `auths` 和 `HttpHeaders` 部分， Docker 凭据帮助工具(`credHelpers` 或 `credsStore`)是不支持的
{{ </note> }}

Docker 会将私有仓库的凭据存储在 `$HOME/.dockercfg` 或 `$HOME/.docker/config.json` 文件中，kubelet 在拉取镜像时也可以从以下列表中搜寻凭据配置文件:

- `{--root-dir:-/var/lib/kubelet}/config.json`
- `{cwd of kubelet}/config.json`
- `${HOME}/.docker/config.json`
- `/.docker/config.json`
- `{--root-dir:-/var/lib/kubelet}/.dockercfg`
- `{cwd of kubelet}/.dockercfg`
- `${HOME}/.dockercfg`
- `/.dockercfg`
{{ <note> }}
用户可以在 kubelet 进行的环境变量上显示地设置 HOME=/root
{{ </note> }}

以下为为节点设置私有仓库凭据的推荐方式。 本示例运行在控制机/笔记本上：
1. 为每一个想要使用的凭据执行命令 `docker login [server]`，该命令会更本机的 `$HOME/.docker/config.json` 文件
2. 在文本编辑器中查看 `$HOME/.docker/config.json` 文件，保证其中只包含需要使用到的凭据。
3. 获取节点列表; 例如:
    - 获取节点名称: `nodes=$( kubectl get nodes -o jsonpath='{range.items[*].metadata}{.name} {end}' )`
    - 获取节点IP: `nodes=$( kubectl get nodes -o jsonpath='{range .items[*].status.addresses[?(@.type=="ExternalIP")]}{.address} {end}' )`
4. 将本地 `.docker/config.json` 文件拷贝到以上列举的节点的凭据搜索目录中的一个
    - 例如: `for n in $nodes; do scp ~/.docker/config.json root@"$n":/var/lib/kubelet/config.json; done`

{{ <note> }}
在生成环境集群中，使用配置管理工具来让配置拷贝到所有需要的节点上
{{ </note> }}

要验证配置是否正确，则使用私有仓库中的镜像来创建一个Pod，如：
```sh
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: private-image-test-1
spec:
  containers:
    - name: uses-private-image
      image: $PRIVATE_IMAGE_NAME
      imagePullPolicy: Always
      command: [ "echo", "SUCCESS" ]
EOF
```

如果所有配置正确，则在一会之后可以通过命令
```sh
kubectl logs private-image-test-1
```
输出
```
SUCCESS
```
如果怀疑命令失败，则执行命令:
```sh
kubectl describe pods/private-image-test-1 | grep 'Failed'
```
如果有失败则输出类似如下
```
 Fri, 26 Jun 2015 15:36:13 -0700    Fri, 26 Jun 2015 15:39:13 -0700    19    {kubelet node-i2hq}    spec.containers{uses-private-image}    failed        Failed to pull image "user/privaterepo:v1": Error: image user/privaterepo:v1 not found
```

必须要保证所有节点都是使用的同一个 `.docker/config.json` 文件， 否则 Pod 可能在某些节点成功，某些节点则失败。 例如，在使用节点自动扩容时， 每个实例模板都需要包含 `.docker/config.json` 或挂载包含该文件的盘
当私有仓库的凭据被添加到`.docker/config.json`后 所有的 Pod 都可以对其中配置的任意私有仓库中拉取镜像。
