---
title: "存活(liveness), 就绪(readiness), 启动(startup)探针配置"
date: 2020-05-22T10:52:35+08:00
---

本文介结怎么配置容器的 存活, 就绪, 启动探针配置

kubelet 通过存活探针的结果来决定何时应该重启容器。例如，可以发现一个应用在运行但不能继续干活比如发生死锁，在这种情况下，如果不是有bug 那重启便可以使应用恢复，以提高应用的可用性。

kubelet 通过就绪探针的结果来决定何时容器可以提供服务。当一个Pod所有的容器都就绪后才认为其就绪。就绪探针的一个应用场景为Pod 作为 Service 后台时，当 Pod 不是就绪状态时，会从 Service 的负载均衡列表中移除

kubelet 通过启动探针的结果为确定应用容器的否启动。当配置启动探针时， 存活探针和就绪探针会在启动探针成功后才开始工作。以保证这些探针不会影响应用正常启动。特别针对启动较慢的应用，防止其在启动之前因为存活探针导致重启

## 基于命令的存活探针[实验一]

大多数应用长时间运行后最终都会因各种原因而挂掉，不得不重启。k8s 提供存活探针来发现和补救这个问题

- 实验镜像 `k8s.gcr.io/busybox` (可以是其它任意常见基础镜像)

- 实验配置(`exec-liveness.yaml`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```
- 配置说明

  - `periodSeconds` kubelet 会每5秒执行一次存活探针
  - `initialDelaySeconds` kubelet 应该在容器启动5秒后，执行第一次存活探针
  - `cat /tmp/healthy` kubelet 在目标容器中执行该命令作为探针，如果命令成功返回0，则认为容器存活，否则 kubelet 认为容器已经不能正常工作，会杀死并重启该容器
  - `/bin/sh -c "touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600"` 容器启动30秒内，`cat /tmp/healthy` 返回成功， 之后 `cat /tmp/healthy` 返回失败

### 操作步骤

- 开启容器

```sh
kubectl apply -f exec-liveness.yaml
```

- 查看容器状态

```sh
kubectl get pod liveness-exec
```
查看当前的重启次数

```sh
kubectl describe pod liveness-exec
```

30 秒内结果类似
```
FirstSeen    LastSeen    Count   From            SubobjectPath           Type        Reason      Message
--------- --------    -----   ----            -------------           --------    ------      -------
24s       24s     1   {default-scheduler }                    Normal      Scheduled   Successfully assigned liveness-exec to worker0
23s       23s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Pulling     pulling image "k8s.gcr.io/busybox"
23s       23s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Pulled      Successfully pulled image "k8s.gcr.io/busybox"
23s       23s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Created     Created container with docker id 86849c15382e; Security:[seccomp=unconfined]
23s       23s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Started     Started container with docker id 86849c15382e
```

30 秒后

```
FirstSeen LastSeen    Count   From            SubobjectPath           Type        Reason      Message
--------- --------    -----   ----            -------------           --------    ------      -------
37s       37s     1   {default-scheduler }                    Normal      Scheduled   Successfully assigned liveness-exec to worker0
36s       36s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Pulling     pulling image "k8s.gcr.io/busybox"
36s       36s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Pulled      Successfully pulled image "k8s.gcr.io/busybox"
36s       36s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Created     Created container with docker id 86849c15382e; Security:[seccomp=unconfined]
36s       36s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Started     Started container with docker id 86849c15382e
2s        2s      1   {kubelet worker0}   spec.containers{liveness}   Warning     Unhealthy   Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
```

再过 30 秒

执行 `kubectl get pod liveness-exec`， 输出类似

```
NAME            READY     STATUS    RESTARTS   AGE
liveness-exec   1/1       Running   1          1m
```

注意查看重启次数的增加

## 基于HTTP请求的存活探针[实验二]

另一种存活探针基于 HTTP GET 请求来实现

- 实验镜像 `k8s.gcr.io/liveness`(墙内请用 `registry.cn-hangzhou.aliyuncs.com/lisong/liveness`)

- 实验配置(`http-liveness.yaml`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```

- 配置说明
  - `periodSeconds` kubelet 会每3秒执行一次存活探针
  - `initialDelaySeconds` kubelet 应该在容器启动3秒后，执行第一次存活探针
  - `port: 8080` kubelet HTTP请求目标端口为 `8080`
  - `path: /healthz` kubelet HTTP请求目标路径为 `8080`
  - kubelet 基于HTTP响应码作为探测依据，若响应码 大于等于`200`且小于`400`认为成功，其它失败

- 实验容器关键代码(`server.go`)

```go
http.HandleFunc("/healthz", func(w http.ResponseWriter, r *http.Request) {
    duration := time.Now().Sub(started)
    if duration.Seconds() > 10 {
        w.WriteHeader(500)
        w.Write([]byte(fmt.Sprintf("error: %v", duration.Seconds())))
    } else {
        w.WriteHeader(200)
        w.Write([]byte("ok"))
    }
})

```

代码说明，在容器开启后 `10`秒内返回响应码 `200`(成功)，之后返回响应码 `500`(错误)

### 操作步骤

- 开启

```sh
kubectl apply -f http-liveness.yaml
```
- 查看容器状态

开启后马上查看重启次数
```sh
kubectl get pod liveness-http
```

在开启 10秒内查看容器信息
```sh
kubectl describe pod liveness-http
```

在开启 10秒后再次查看容器信息
```sh
kubectl describe pod liveness-http
```
在开启 10秒后再次查看容器重启次数
```sh
kubectl get pod liveness-http
```

## 基于 TCP 的存活探针和就绪探针[实验三]

- 实验镜像 `k8s.gcr.io/goproxy:0.1` (墙内请用 `registry.cn-hangzhou.aliyuncs.com/lisong/goproxy:0.1`)

- 实验配置(`tcp-liveness-readiness.yaml`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: goproxy
  labels:
    app: goproxy
spec:
  containers:
  - name: goproxy
    image: k8s.gcr.io/goproxy:0.1
    ports:
    - containerPort: 8080
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```
- 配置说明

  由配置可见TCP探针与HTTP探针类似，且本练习中同时配置了存活探针和就绪探针。
  - `readinessProbe.initialDelaySeconds` 就绪探针于容器开启5秒后开始执行
  - `readinessProbe.periodSeconds` 就绪探针每10秒执行一次
  - `livenessProbe.initialDelaySeconds` kubelet 应该在容器启动15秒后，执行第一次存活探针
  - `livenessProbe.periodSeconds` kubelet 会每20秒执行一次存活探针

### 操作步骤

- 开启

```sh
kubectl apply -f tcp-liveness-readiness.yaml
```

开启 15秒后，查看状态
```sh
kubectl describe pod goproxy
```

## 在探针中使用命名端口

命令端口可用于 HTTP和TCP 配置存活探针
```yaml
ports:
- name: liveness-port
  containerPort: 8080
  hostPort: 8080

livenessProbe:
  httpGet:
    path: /healthz
    port: liveness-port
```

## 通过启动探针来保护启动时间长的容器

有时为解决传统项目在启动时可能需要额外的初始化需要花费较长时间，此时可以配置与存活探针相同命令的启动探针，示例配置如下

```yaml
ports:
- name: liveness-port
  containerPort: 8080
  hostPort: 8080

livenessProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 1
  periodSeconds: 10

startupProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 30
  periodSeconds: 10
```

- `startupProbe.failureThreshold` 启动探针的尝试次数，这里是 30
- `startupProbe.periodSeconds` 启动探针每次尝试间隔时间，这里是 10

因此 应用可以有 `failureThreshold * periodSeconds` (30 * 10 = 300s)， 即5分针的时间来完成启动，在这个过程中如果启动探针成功一次，则不再继续，转而由存活探针开始工作，如果启动探针最终没有一次成功，则根据容器的重启策略(`restartPolicy`)来决定下一步动作

## 定义就绪探针

### 应用场景

- 应用在启动是需要加载大量数据或配置文件
- 应用需要信赖外部服务先启动

以上场景应用暂时不能提供服务，但并不需要重启，而又不想转发流量到这个应用
注意: 就绪探针在应用整个生命周期都在运行

```yaml
readinessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```
就绪探针与存活探针配置类似
就绪探针与存活搾在同一个容器中，可以并行执行。同时使用它们能保证流量不会到未准备好的容器上，容器挂掉是能重启

## 探针配置

### 探针有如下配置项，以便能更精确的控制检测方式

- `initialDelaySeconds` 容器开启后多少秒，存活/就绪探针开始工作。默认 `0` 最小 `0`
- `periodSeconds` 探针每次执行的时间间隔，单位秒， 默认 `10`， 最小 `1`
- `timeoutSeconds` 探针检测超时时间，单位秒，默认 1， 最小 1
- `successThreshold` 在失败后，连续多少次成功决定状态为成功， 默认 1， 存活探针必须为 1 ， 最小值 1
- `failureThreshold` 放弃尝试前的失败次数， 存活探针失败则重启容器。 就绪探针则标记容器为未就绪， 默认 3， 最小 1

### HTTP 类探针额外配置项

- `host` 请求主机名， 默认为 pod IP, 可以 `httpHeaders` 设置 `Host`
- `scheme` 请求协议 (HTTP / HTTPS) 默认 HTTP
- `path` 请求路径
- `httpHeaders` 请求头
- `port` 请求端口号或命令端口名称 端口号范围 `1 - 65535`

如果协议为 https 请求不会验证证书，大多数场景不需要配置 `host` 字段。吸的地 应用监听地址为 `127.0.0.1` Pod 使用 `hostNetwork`, 如果Pod 信赖虚拟主机(比较常见)，此时不应该配置 `host` 字段，而通过 `httpHeaders` 设置 `Host`
如果是 TCP 探针，k8s通过 `node` 来连接，因此不能使用 sevice name 作为 `host` 字段的值

## 总结

|探针类型|探针目的|探测成功作用|探测失败作用|生命周期
|-------|-------|---------|----------|--------
|启动    |保障启动过程中不被存活探针重启|表示容器启动成功，并停止运行|在达到 `failureThreshold`之前，无操作，达到后由重启策略决定|成功后结束或尝试直至达到 `failureThreshold` 配置的次数|
|就绪    |保证容器能提供服务才引入流量|标记容器就绪，service 可以分配流量|标记容器未就绪，service 不能引入流量|容器整个生命
|存活    |让容器不能提供服务时重启|无操作|达到 `failureThreshold` 配置次数后重启容器 |容器整个生命
