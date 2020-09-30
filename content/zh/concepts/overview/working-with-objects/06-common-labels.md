---
title: 标签设置指导
weight: 2000306
date: 2020-07-03
publishdate: 2020-07-03
---
用户可以使用 kubectl 和 dashboard 外的可视化管理工具。一些通用的描述对象信息的标签配置可以让这些工具更好地工作
为了工具能更好的使用这些标签，建议标签以方便查询的方式来定义对象信息
元数据是围绕应用这个概念来组织的。k8s 并不是一个平台即服务(PaaS),也没有一个对应用有一个强制的格式规范。只是通过元数据来提供应用的描述和信息。关于应用所包含的信息的的规范是相关宽松的
注意: 只能推荐使用这些标签。以方便对应用的管理，这些都不是 k8s 核心工具所必要的
共享标签和注解需要共享一个共同的前缀 `app.kubernetes.io`, 没有前缀的标签是用户私有的。在共享标签上使用使用共享前缀是为了保证不会影响到用户私有的标签。

## 标签

为了给读者一个标签使用的整体印象，以下标签可以用在第一个资源对象上。

|标签键                        |说明                                 |示例值                  |值数据类型
|------------------------------|------------------------------------|-----------------------|-----
app.kubernetes.io/name	      |应用名称	                             |  `mysql`	            |string
app.kubernetes.io/instance	  |用于识别应用实例的唯一名称	             |  `wordpress-abcxzy`	|string
app.kubernetes.io/version	    |应用的当前版本 (如 版本号, 版本哈希, 等.) |	`5.7.21`	          |string
app.kubernetes.io/component	  |架构中的结构名	                       |  `database`	        |string
app.kubernetes.io/part-of	    |这个对象是哪个应用的一部分	             |  `wordpress`	        |string
app.kubernetes.io/managed-by	|用于管理这个对象的工具	                |  `helm`	              | string

以下是一个 `StatefulSet` 的实践示例

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  labels:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/instance: wordpress-abcxzy
    app.kubernetes.io/version: "5.7.21"
    app.kubernetes.io/component: database
    app.kubernetes.io/part-of: wordpress
    app.kubernetes.io/managed-by: helm
```

## 应用及应用实例

一个应用可能在 k8s 集群的一个命名空间中安装一次或多次。比如 安装多个 wordpress 的(不同)网站
应用名和其实例名都应该作区分， 比如 一个 wordpress 应用为 `app.kubernetes.io/name: wordpress`, 其实例可以设置为 `app.kubernetes.io/instance: wordpress-abcxzy` 这就应用和实例就比较好识别，当一个应用有多个实例是每个实例的名称都要唯一。

## 示例

以下示例展示这些标签的不同用法


### 一个简单的无状态 Service

以下应用场景为 用一个 Deployment 和 Service 对象部署一个简单的无状态服务。 以下为其标签的最简配置
Deployment 用于管理应用运行的 Pod
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: myservice
    app.kubernetes.io/instance: myservice-abcxzy
...
```
Service 用于应用接入
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: myservice
    app.kubernetes.io/instance: myservice-abcxzy
...

```

### 带数据库的Web应用

稍复杂一点的应用场景: 一个web 应用(WordPress)用到一个数据库(MySQL), 通过 Helm 安装

WordPress 的 Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: wordpress
    app.kubernetes.io/instance: wordpress-abcxzy
    app.kubernetes.io/version: "4.9.4"
    app.kubernetes.io/managed-by: helm
    app.kubernetes.io/component: server
    app.kubernetes.io/part-of: wordpress
...
```
WordPress 的 Service
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: wordpress
    app.kubernetes.io/instance: wordpress-abcxzy
    app.kubernetes.io/version: "4.9.4"
    app.kubernetes.io/managed-by: helm
    app.kubernetes.io/component: server
    app.kubernetes.io/part-of: wordpress
...
```

MySQL 的 StatefulSet
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  labels:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/instance: mysql-abcxzy
    app.kubernetes.io/version: "5.7.21"
    app.kubernetes.io/managed-by: helm
    app.kubernetes.io/component: database
    app.kubernetes.io/part-of: wordpress
...

```

MySQL 的 Service
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/instance: mysql-abcxzy
    app.kubernetes.io/version: "5.7.21"
    app.kubernetes.io/managed-by: helm
    app.kubernetes.io/component: database
    app.kubernetes.io/part-of: wordpress
...

```
在 StatefulSet 和 Service 可以找到它们自己的信息所属的应用的信息。这些在更大的应用中相当有用。
