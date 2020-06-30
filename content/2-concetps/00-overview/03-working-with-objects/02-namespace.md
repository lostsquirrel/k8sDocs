---
title:
weight:
date: 2020-06-29
draft: true
---

# 名字空间(Namespaces)
k8s 可以通过 namespace 在一个物理集群上创建多个虚拟 集群

## 啥时候用名字空间
    名字空间是为分散有多个组或项目下的多用户设计的
    如果只有二三十个用户就不用想了
    名字空间是对象名称的一个作用域，对象命名只需要在一个名字空间唯一即可
    用于在多个用户之间区分资源
    未来版本，一个名字空间下的对象可能有相同的默认访问控制
    不必要用名字空间来区分差异较小的资源，可以使用标签(labels)区分同一个名字空间下的对象

## 管理名字空间
    名字空间的创建和删除请见(https://kubernetes.io/docs/admin/namespaces)

### 查看
    ```sh
    kubectl get namespaces
    NAME          STATUS    AGE
    default       Active    1d
    kube-system   Active    1d
    kube-public   Active    1d
    ```
    k8s 启动时会创建3个名字空间
    - default： 当对象不指定名字空间时
    - kube-system： 由系统创建对象所在的名字空间
    - kube-public：可以被所有人访问，一般保留为系统使用，公开只是为了方便，不是强制要求

## 设定请求的名字空间

    使用 `--namespace`
    例：
    ```sh
    $ kubectl --namespace=<insert-namespace-name-here> run nginx --image=nginx
    $ kubectl --namespace=<insert-namespace-name-here> get pods
    ```
## 设置默认名字空间
    持久化配置 kubectl 默认操作的名字空间
    ```sh
    $ kubectl config set-context $(kubectl config current-context) --namespace=<insert-namespace-name-here>
    # Validate it
    $ kubectl config view | grep namespace:
    ```

## 名字空间和DNS
    创建 Service 时生成 DNS 记录会包含当前所在的 名字空间，如果跨名字空间访问 Service 解析DNS时会带上名字空间， [详见](k8s_concepts/k8s_concepts_620_dns_service_pod.md)

## 不属于任何名字空间的对象
    - Node
    - persistentVolumes
    - Events: 根据事件关联的对象，可能有属于名字空间也可能不属于名字空间
