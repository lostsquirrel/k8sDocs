---
title: 注解 (Annotations)
weight: 2000304
date: 2020-06-29
publishdate: 2020-07-02
---
用户可以将任非标识符元素关联到对象上，其它库或工具可以读取这些元数据
## 关联元数据到对象
用户可以通过标签或注意的方式将元数据关联到对象上，标签用于选择或查找符合条件的对象， 而在注解中的元数据则不是用于选择或查找对象的，其中的内容可以很小也可以很大，可以是结构化数据也可以是非结构化数据，还可以使用标签不允许的字符
注解与标签类似，也是键值对形式的字典，例如:
```json
"metadata": {
  "annotations": {
    "key1" : "value1",
    "key2" : "value2"
  }
}
```
可以用注解存储的常见数据示例：
- 由声明式配置层管理的字段. 把这些字段作为注解关联到对象是为能与诸如以下字段做区分: 由客户端或服务端设置的默认值; 由自动扩容系统自动生成的字段
- 构建，发布，镜像相关信息如 时间戳，发布编号，git 分支，PR 编号，镜像 hash, 镜像库地址
- 日志, 监控, 分析, 审计仓库的信息.
- 关于客户端库或工具可用于调试目的的信息 例如：名称, 版本, 构建信息.
- 来自用户或 工具/系统 信息, 例如 对象在外部系统中的URLs地址
- 轻量级回滚帮助信息， 例如, 配置或检查点.
- 对使用对象的用户提供修改指导或非标准特性的使用说明

当然这些信息可以保存在外部的数据库或系统中，但这样制作部署，管理，自省的工具库就不那么容易了

## 语法和字符集

标签由 键值对组成。 合法的键可以由两上部分组成， 一个可选的前级加上本身的名称中间用斜线(`/`)分隔。
名称部分 不得多于63个字符，必须以字母或数字 ([a-z0-9A-Z])开头和结束，中间部分可以包含中划线(`-`)，下划线 `(_)`，点 (`.`),字母,数字。
如果要使用前缀， 前端必须是一个合法的 DNS 字域名，由多个 DNS 标签组成，中间由点(`.`)分隔， 总长度不超过 253 个字符

如果一个标签键没有前缀则假定它是属于用于私有的，
由系统自动化组件(e.g. `kube-scheduler`, `kube-controller-manager`, `kube-apiserver`, `kubectl`, 或其它第三方自动化工具), 在给用户对象加标签时必须加前缀， `kubernetes.io/`和 `k8s.io/` 为 k8s 核心组件保留前缀

以下示例一个包含一个注解 `imageregistry: https://hub.docker.com/` 的 Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: annotations-demo
  annotations:
    imageregistry: "https://hub.docker.com/"
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```
