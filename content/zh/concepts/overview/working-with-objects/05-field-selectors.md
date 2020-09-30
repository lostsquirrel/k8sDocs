---
title: 字段选择器
weight: 2000305
date: 2020-07-03
publishdate: 2020-07-03
---
用户可以通过字段选择器的以对象的一个或多个字段的值作为选择条件实现对 [k8s 对象](../00-kubernetes-objects/)的选择。示例如下:
- `metadata.name=my-service`
- `metadata.namespace!=default`
- `status.phase=Pending`

以下 `kubectl` 命令通过选择器，选择 `status.phase` 字段值是 `Running` 的对象:

```sh
kubectl get pods --field-selector status.phase=Running
```

注意: 字段选择器是基本的资源选择器。 默认没有设置选择条件。 也就是选择该类型所有的对象。这就让以下两个 `kubectl` 命令等效 `kubectl get pods`， `kubectl get pods --field-selector ""`


## 支持选择的字段

字段选择器支持的字段因 k8s 资源不同而不同。 但所有的资源都支持 `metadata.name` 和 `metadata.namespace`， 使用不支持的字段会报错。
比如以下示例:
```sh
kubectl get ingress --field-selector foo.bar=baz
```
错误信息如下
```
Error from server (BadRequest): Unable to find "ingresses" that match label selector "", field selector "foo.bar=baz": "foo.bar" is not a known field selector: only "metadata.name", "metadata.namespace"
```

## 支持的操作符

字段选择器支持的操作符有 `=`, `==,` 和 `!=`， 其中 (`=` 和 `==` 效果一样)。 例如以下示例表示，选择所有不属于 `default` 命名空间的 `Service`:

```sh
kubectl get services  --all-namespaces --field-selector metadata.namespace!=default
```

## 多个选择条件

与标签选择类似， 字段选择和多个条件也可以通过逗号分隔，例如以下示例表示选择所有 `status.phase` 不是 `Running` 且 `spec.restartPolicy` 字段值为 `Always` 的 Pod
```sh
kubectl get pods --field-selector=status.phase!=Running,spec.restartPolicy=Always
```

## 同时筛选多个类型的资源

字段选择器可以同时对多种类型对象进行筛选， 例如以下示例表示，选择所有不属于 `default` 命名空间的 `Statefulsets` 和 `Services`

```sh
kubectl get statefulsets,services --all-namespaces --field-selector metadata.namespace!=default
```
