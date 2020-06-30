---
title: 对象命令与ID
weight: 200031
date: 2020-06-29
draft: true
---
在 k8s 的对象中，同一类型的对象名称唯一， 比如在一个 [名字空间](./02-namespace)中只能有一个叫 `myapp-1234` 在 Pod， 但不同类型的资源可以有相同的名称，比如还可以定义一个叫 `myapp-1234` 在 Deployment。
每个对象也有一个 `UID` 这个 `UID` 整个集群全局唯一
用户需要定义非唯一的，用户自定义的属性，则可通过 [标签](./03-label-selectors) 和 [注解](04-annotation) 实现

# 对象名称

对象的名称是一个字符串，体现在对象的 URL中， 比如 `/api/v1/pods/some-name`
在同一时间，一个类型的对象名称必须唯一，但如果用户删除的这个对象，则可以用这个名称再创建一个新的对象

以下是命令规范三类限制

### DNS 子域名

多数类型的资源命令必须可以作为 DNS 子域名，定义在这里(https://tools.ietf.org/html/rfc1123), 总结如下:

- 不能多于 253个字符
- 只能包含小写字母，数字， `-`(中划线)，`.`(点)
- 只能以字母数字开头
- 只能以字母数字结尾

更多资料见 [这里](https://en.wikipedia.org/wiki/Subdomain)
### DNS 标签名

有些资源类型命令遵循 DNS 标签名称， 规则如下：

- 最多 63 个字符
- 只能包含小写字母，数字， `-`(中划线)
- 只能以字母数字开头
- 只能以字母数字结尾

### 路径分段名

有些资源类型的命令必须要能够编码到 路径的一段上，所以名称不能包含 `.` 或 `..`, 也不能包含 `/` 或 `%`

以下为一个命令为 `nginx-demo` 的 Pod 示例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-demo
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

注意: 还有一些资源类型有更严格的命名规则

## UIDs

由系统创建的对象的唯一标识，类型为字符串
k8s 集群整个生命周期内，创建的每一个对象的UID都是唯一的，用于区分可能存在或曾今过的相似对象
k8s UID 是 UUID， 使用标准为  `ISO/IEC 9834-8`  和 `ITU-T X.667`

## 引申阅读

[k8s 标签](./03-label-selectors)
[k8s 标识符与命令设计文档](https://git.k8s.io/community/contributors/design-proposals/architecture/identifiers.md)
