---
title: 镜像
date: 2020-07-19
publishdate: 2020-07-24
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


### 预先拉取镜像

{{ <note> }}
这种方式适用于用于用户有权限对节点配置的情况，而且不适用于节点由云提供商管理且能够自动扩容的情况
{{ </note> }}

默认情况下， kubelet 会尝试从指定镜像仓库拉取每一个镜像， 但是在容器 imagePullPolicy 属性的值被设置为 `IfNotPresent` 或 `Never`时，则会使用本地镜像。 (preferentially or exclusively, respectively).

如果想要用预先摘取的方式取代镜像仓库认证， 需要保证集群中所有节点上预先摘取到的所有镜像都必须一致。
这种方式可用于预先取镜像来达到提速的目的或代替私有镜像仓库认证。
所有的 Pod 都有权访问任意预先拉取的镜像

### 在 Pod 上设置 `ImagePullSecrets`

{{ <note> }}
这是使用私有仓库镜像的推荐方式
{{ </note> }}

k8s 支持在 Pod 上配置私有镜像仓库凭据

#### 创建带 Docker 配置的 Secret

替换命令中的大写值为对应的配置，并运行此命令:
```sh
kubectl create secret docker-registry <name> --docker-server=DOCKER_REGISTRY_SERVER --docker-username=DOCKER_USER --docker-password=DOCKER_PASSWORD --docker-email=DOCKER_EMAIL
```

如果已经有 Docker 凭据，可以不用以上命令，直接使用凭据文件创建对应的 Secret.
具体配置见[这里](../../../3-tasks/02-configure-pod-container/11-pull-image-private-registry/#registry-secret-existing-credentials)
这种配置方式尤其适用有多个私有镜像仓库的情况，因为 `kubectl create secret docker-registry` 创建 Secret 的方式只适用于单个私有镜像仓库的情况。
{{ <note> }}
Pod 只能引用当前名字空间内的 `Secret` , 因此需要在每个名字空间都需要创建 `Secret`
{{ </note> }}

### 在 Pod 中使用 `imagePullSecrets`

当 `Secret` 创建好后，可以配置 `imagePullSecrets` 使用该 `Secret`, 例如：
```sh
cat <<EOF > pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: foo
  namespace: awesomeapps
spec:
  containers:
    - name: foo
      image: janedoe/awesomeapp:v1
  imagePullSecrets:
    - name: myregistrykey
EOF

cat <<EOF >> ./kustomization.yaml
resources:
- pod.yaml
EOF
```
需要在每个用到私有镜像仓库的 Pod 都需要该配置。
也可以在 `ServiceAccount` 设置 `imagePullSecrets` 可以使用对应的 Pod 自动添加该属性
在 `ServiceAccount` 设置 `imagePullSecrets`具体见[这里](../../../3-tasks/02-configure-pod-container/10-configure-service-account/#add-imagepullsecrets-to-a-service-account)
这个配置可与节点上的 `.docker/config.json` 配合使用，两个配置的凭据全合并到一起。


## 应用场景

配置私有仓库镜像的方式有多种，以下为常用应用场景和推荐方案。

  1. 集群只使用开放(如：开源)镜像， 不需要私有仓库。
    - 使用 Docker Hub 上的公有镜像
      - 不需要配置
      - 一些云提供商会自动缓存或镜像开放镜像， 可以提高可用性并减少拉取镜像的时间
  2. 集群使用到的镜像对外私有，对内公开
    - 使用自建私有镜像仓库
      - 可以托管在 Docker Hub 或其它地方
      - 使用上面介结的在每个节点配置 `.docker/config.json` 的方式
    - 在内网运行一个开放的内部私有镜像仓库
      - 不需要在 k8s 上做配置
    - 使用一个有访问控制的镜像仓库
      - 在节点自动扩容的场景下会比手动更佳
    - 在节点配置不方便的集群中使用 `imagePullSecrets`
  3. 镜像仓库需要更严格的访问控制
    - 需要打开  [AlwaysPullImages](../../../5-reference/03-access-authn-authz/04-admission-controllers/#alwayspullimages) admission controller， 否则所有 Pod 默认对所有镜像有访问权限
    - 将敏感数据放的 Secret 中， 还是是打在镜像中
  4. 多租户集群，每个租户需要独立的私有镜像仓库
    - 需要打开  [AlwaysPullImages](../../../5-reference/03-access-authn-authz/04-admission-controllers/#alwayspullimages) admission controller， 否则所有租户的所有 Pod 默认对所有镜像有访问权限
    - 私有镜像仓库需要有认证系统
    - 为每个租户生成私有镜像仓库凭据，并在每个租户的名字空间中创建对应 Secret.
    - 各名字空间的租户将名称的 Secret 配置到 `imagePullSecrets`

如果用到的多个私有镜像仓库， 可以对每个仓库创建一个 Secret, kubelet 会将所有 imagePullSecrets 合并到一个虚拟的 `.docker/config.json` 中。
