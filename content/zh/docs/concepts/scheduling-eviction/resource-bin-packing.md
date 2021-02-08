---
title: Resource Bin Packing for Extended Resources
content_type: concept
weight: 50
---
<!--
---
reviewers:
- bsalamat
- k82cn
- ahg-g
title: Resource Bin Packing for Extended Resources
content_type: concept
weight: 50
---
 -->
<!-- overview -->

{{< feature-state for_k8s_version="v1.16" state="alpha" >}}
<!--
The kube-scheduler can be configured to enable bin packing of resources along
with extended resources using `RequestedToCapacityRatioResourceAllocation`
priority function. Priority functions can be used to fine-tune the
kube-scheduler as per custom needs.
 -->

`kube-scheduler` 可以通过使用 `RequestedToCapacityRatioResourceAllocation` 优先级函数
配置启用对扩展资源打包。 优先级函数可以用来根据每个自定义的需求对 `kube-scheduler` 进行微调

<!-- body -->
<!--
## Enabling Bin Packing using RequestedToCapacityRatioResourceAllocation

Kubernetes allows the users to specify the resources along with weights for
each resource to score nodes based on the request to capacity ratio. This
allows users to bin pack extended resources by using appropriate parameters
and improves the utilization of scarce resources in large clusters. The
behavior of the `RequestedToCapacityRatioResourceAllocation` priority function
can be controlled by a configuration option called
`requestedToCapacityRatioArguments`. This argument consists of two parameters
`shape` and `resources`. The `shape` parameter allows the user to tune the
function as least requested or most requested based on `utilization` and
`score` values.  The `resources` parameter consists of `name` of the resource
to be considered during scoring and `weight` specify the weight of each
resource.


Below is an example configuration that sets
`requestedToCapacityRatioArguments` to bin packing behavior for extended
resources `intel.com/foo` and `intel.com/bar`.

```yaml
apiVersion: v1
kind: Policy
# ...
priorities:
  # ...
  - name: RequestedToCapacityRatioPriority
    weight: 2
    argument:
      requestedToCapacityRatioArguments:
        shape:
          - utilization: 0
            score: 0
          - utilization: 100
            score: 10
        resources:
          - name: intel.com/foo
            weight: 3
          - name: intel.com/bar
            weight: 5
```

**This feature is disabled by default**
 -->

## Enabling Bin Packing using RequestedToCapacityRatioResourceAllocation

k8s 允许用户在设置资源时给每个资源指定权重，这个权重会被用在基于请求容量比的节点计分。
这使得用户可以使用适当的参数来对扩展资源打包，并改善大型集群中稀缺资源的利用率。
`RequestedToCapacityRatioResourceAllocation` 优先级函数 的行为可以由一个叫做
`requestedToCapacityRatioArguments` 的选项控制。 这个参数由 `shape` 和 `resources`
组成. 其中 `shape` 让用户可以基于 `utilization` 和 `score` 值的调整函数最低要求，或请求上限。
`resources` 中的 `name` 是计分时考量的资源名称，`weight` 指定每种资源的权重


下面的示例配置中，通过 `requestedToCapacityRatioArguments` 设置
`intel.com/foo` 和 `intel.com/bar` 扩展资源的打包行为。

```yaml
apiVersion: v1
kind: Policy
# ...
priorities:
  # ...
  - name: RequestedToCapacityRatioPriority
    weight: 2
    argument:
      requestedToCapacityRatioArguments:
        shape:
          - utilization: 0
            score: 0
          - utilization: 100
            score: 10
        resources:
          - name: intel.com/foo
            weight: 3
          - name: intel.com/bar
            weight: 5
```

**该特性默认是禁用的**
<!--
### Tuning the Priority Function

`shape` is used to specify the behavior of the
`RequestedToCapacityRatioPriority` function.

```yaml
shape:
 - utilization: 0
   score: 0
 - utilization: 100
   score: 10
```

The above arguments give the node a `score` of 0 if `utilization` is 0% and 10 for
`utilization` 100%, thus enabling bin packing behavior. To enable least
requested the score value must be reversed as follows.

```yaml
shape:
  - utilization: 0
    score: 10
  - utilization: 100
    score: 0
```

`resources` is an optional parameter which defaults to:

``` yaml
resources:
  - name: CPU
    weight: 1
  - name: Memory
    weight: 1
```

It can be used to add extended resources as follows:

```yaml
resources:
  - name: intel.com/foo
    weight: 5
  - name: CPU
    weight: 3
  - name: Memory
    weight: 1
```

The `weight` parameter is optional and is set to 1 if not specified. Also, the
`weight` cannot be set to a negative value.
 -->

### 调整优先级函数 {#tuning-the-priority-function}

`shape` is used to specify the behavior of the
`RequestedToCapacityRatioPriority` function.

`shape` 是用来指定 `RequestedToCapacityRatioPriority` 函数的行为。

```yaml
shape:
 - utilization: 0
   score: 0
 - utilization: 100
   score: 10
```

上面的参数表示，如果这个资源的利用率(`utilization`) 是 `0%` 节点的一个 `score` 就是 `0`,
利用率(`utilization`) 是 `100%`, 则 `score` 是 `10`，这就是启用了打包行为。
要启用分数值的最小值则要像下面这样反过来。

```yaml
shape:
  - utilization: 0
    score: 10
  - utilization: 100
    score: 0
```

`resources` 是一个可靠参数，默认值如下:
``` yaml
resources:
  - name: CPU
    weight: 1
  - name: Memory
    weight: 1
```

也可以像下面这样添加扩展资源:

```yaml
resources:
  - name: intel.com/foo
    weight: 5
  - name: CPU
    weight: 3
  - name: Memory
    weight: 1
```

`weight` 参数为可选，如果没有设置则默认为 1。 还有就是 `weight` 的值不能设置为负数
<!--
### Node scoring for capacity allocation

This section is intended for those who want to understand the internal details
of this feature.
Below is an example of how the node score is calculated for a given set of values.

Requested resources:

```
intel.com/foo : 2
Memory: 256MB
CPU: 2
```

Resource weights:

```
intel.com/foo : 5
Memory: 1
CPU: 3
```

FunctionShapePoint {{0, 0}, {100, 10}}

Node 1 spec:

```
Available:
  intel.com/foo: 4
  Memory: 1 GB
  CPU: 8

Used:
  intel.com/foo: 1
  Memory: 256MB
  CPU: 1
```

Node score:

```
intel.com/foo  = resourceScoringFunction((2+1),4)
               = (100 - ((4-3)*100/4)
               = (100 - 25)
               = 75                       # requested + used = 75% * available
               = rawScoringFunction(75)
               = 7                        # floor(75/10)

Memory         = resourceScoringFunction((256+256),1024)
               = (100 -((1024-512)*100/1024))
               = 50                       # requested + used = 50% * available
               = rawScoringFunction(50)
               = 5                        # floor(50/10)

CPU            = resourceScoringFunction((2+1),8)
               = (100 -((8-3)*100/8))
               = 37.5                     # requested + used = 37.5% * available
               = rawScoringFunction(37.5)
               = 3                        # floor(37.5/10)

NodeScore   =  (7 * 5) + (5 * 1) + (3 * 3) / (5 + 1 + 3)
            =  5
```

Node 2 spec:

```
Available:
  intel.com/foo: 8
  Memory: 1GB
  CPU: 8
Used:
  intel.com/foo: 2
  Memory: 512MB
  CPU: 6
```

Node score:

```
intel.com/foo  = resourceScoringFunction((2+2),8)
               =  (100 - ((8-4)*100/8)
               =  (100 - 50)
               =  50
               =  rawScoringFunction(50)
               = 5

Memory         = resourceScoringFunction((256+512),1024)
               = (100 -((1024-768)*100/1024))
               = 75
               = rawScoringFunction(75)
               = 7

CPU            = resourceScoringFunction((2+6),8)
               = (100 -((8-8)*100/8))
               = 100
               = rawScoringFunction(100)
               = 10

NodeScore   =  (5 * 5) + (7 * 1) + (10 * 3) / (5 + 1 + 3)
            =  7

``` -->


### 容量分配的节点计分 {#node-scoring-for-capacity-allocation}

本节是为那些想要理解该特性内部细节的用户提供的。
下面示例展示了基于示例中设置的值，节点分数是怎么计算出来的。

资源要求:

```
intel.com/foo : 2
Memory: 256MB
CPU: 2
```

资源权重:
```
intel.com/foo : 5
Memory: 1
CPU: 3
```

FunctionShapePoint {{0, 0}, {100, 10}}

节点 1 `spec`:

```
Available:
  intel.com/foo: 4
  Memory: 1 GB
  CPU: 8

Used:
  intel.com/foo: 1
  Memory: 256MB
  CPU: 1
```

节点 1 得分计算过程:

```
intel.com/foo  = resourceScoringFunction((2+1),4)
               = (100 - ((4-3)*100/4)
               = (100 - 25)
               = 75                       # requested + used = 75% * available
               = rawScoringFunction(75)
               = 7                        # floor(75/10)

Memory         = resourceScoringFunction((256+256),1024)
               = (100 -((1024-512)*100/1024))
               = 50                       # requested + used = 50% * available
               = rawScoringFunction(50)
               = 5                        # floor(50/10)

CPU            = resourceScoringFunction((2+1),8)
               = (100 -((8-3)*100/8))
               = 37.5                     # requested + used = 37.5% * available
               = rawScoringFunction(37.5)
               = 3                        # floor(37.5/10)

NodeScore   =  (7 * 5) + (5 * 1) + (3 * 3) / (5 + 1 + 3)
            =  5
```

节点 2 `.spec`:

```
Available:
  intel.com/foo: 8
  Memory: 1GB
  CPU: 8
Used:
  intel.com/foo: 2
  Memory: 512MB
  CPU: 6
```

节点 2 得分计算过程:

```
intel.com/foo  = resourceScoringFunction((2+2),8)
               =  (100 - ((8-4)*100/8)
               =  (100 - 50)
               =  50
               =  rawScoringFunction(50)
               = 5

Memory         = resourceScoringFunction((256+512),1024)
               = (100 -((1024-768)*100/1024))
               = 75
               = rawScoringFunction(75)
               = 7

CPU            = resourceScoringFunction((2+6),8)
               = (100 -((8-8)*100/8))
               = 100
               = rawScoringFunction(100)
               = 10

NodeScore   =  (5 * 5) + (7 * 1) + (10 * 3) / (5 + 1 + 3)
            =  7

```

## {{% heading "whatsnext" %}}
<!--
- Read more about the [scheduling framework](/docs/concepts/scheduling-eviction/scheduling-framework/)
- Read more about [scheduler configuration](/docs/reference/scheduling/config/)
 -->
- 概念 [调度框架](/k8sDocs/docs/concepts/scheduling-eviction/scheduling-framework/)
- 参考 [调度器配置](https://kubernetes.io/docs/reference/scheduling/config/)
