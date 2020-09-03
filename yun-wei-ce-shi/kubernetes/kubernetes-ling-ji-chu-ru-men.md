# Kubernetes 零基础入门

原文链接： [https://juejin.im/post/5d1dec746fb9a07eb67dae0f](https://juejin.im/post/5d1dec746fb9a07eb67dae0f)

Kubernetes 是 Google 团队发起并维护的基于 Docker 的开源容器集群管理系统，它不仅支持常见的云平台，而且支持内部数据中心。它的目标是管理跨多个主机的容器，提供基本的部署，维护以及运用伸缩，主要实现语言为 Go 语言。

最初，Google 开发了一个叫 Borg 的系统（现在命令为 Omega）来调度如此庞大数量的容器和工作负载。在积累了这么多年的经验后，Google 决定重写这个容器管理系统，并将其贡献到开源社区，让全世界都能受益。它就是 Kubernetes。

建于 Docker 之上的 Kubernetes 可以构建一个容器的调度服务，其目的是让用户透过 Kubernetes 集群来进行云端容器集群的管理，而无需用户进行复杂的设置工作。它是当前最流行的容器编排引擎。

## 1. 安装环境

安装 Kubernetes 需要安装

* `kubeadm`: 用来初始化集群的指令。
* `kubelet`: 在集群中的每个节点上用来启动 pod 和 container 等。
* `kubectl`: 是 Kubernetes 命令行工具，用来与集群通信的命令行工具。

我们可以使用 [minikube](https://user-gold-cdn.xitu.io/2019/7/5/16bc14ff8072eb3b) 来快速安装开发和学习环境。

安装好 `minikube` 可以执行如下命令创建虚拟机。

```text
$ minikube start 
    # --vm-driver hyperv \
    # --hyperv-virtual-switch PVS \
    --image-mirror-country cn \
    --registry-mirror https://dockerhub.azk8s.cn \
    --v 7 \
    --alsologtostderr
```

> 如果是 windows 10 可以使用 hyperv， 需要设置 `--vm-driver` 和 `--hyperv-virtual-switch` 参数，需要使用管理员权限。

```text
$ minikube status # 安装完成后，查看一下状态

$ kubectl cluster-info # 查看一下集群信息

$ minikube dashboard # 开启 Kubernetes web UI 控制台
```

还可以使用在线的 [play with k8s](https://labs.play-with-k8s.com/)。

或者去 [官网交互教程](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-interactive/)。

## 2. 概念

创建一个集群需要使用到 `kubeadm`，它会生成证书，配置文件，安装附加组键，如 `kube-proxy` 和 `kube-dns`。几乎所有的 `Kubernetes` 组件本身也运行在 `Pod` 里。

### 2.1 Pod

Kubernetes 中我们不直接管理容器，而是 `Pod`。它是最小工作单元。

一个 `Pod` 是一个或一组运行非常紧密的容器， `Pod` 中的所有容器使用同一个网络 `namespace`，即相同的 `IP` 地址和 `Port` 空间。它们可以直接用 `localhost` 通信。同样的，这些容器可以共享存储，当 `Kubernetes` 挂载 `volume` 到 `Pod`，本质上是将 `volume` 挂载到 `Pod` 中的每一个容器。

`Pod` 有 `Pending`，`Running`，`Succeeded`，`Failed` 和 `Unknown` 这几个阶段。

我们一般不自己创建 `Pod`, 而是创建 `Deployment`。

### 2.2 Master

Kubernetes 集群中主机分为 `master` 和 `node`。

`Master` 是 `Cluster` 的大脑，它负责管理集群，主要职责是调度，决定将应用放在哪里运行。一个主机可以同时拥有 `Master` 和 `Node` 两种身份。

`Master` 上运行着

#### 2.2.1 kube-apiserver

API Server 提供 HTTP/HTTPS RESTful API。API Server 是 Kubernetes Cluster 的前端接口，各种客户端工具（CLI 或 UI）以及 Kubernetes 其他组件可以通过它管理 Cluster 的各种资源。

#### 2.2.2 kube-controller-manager

`Controller Manager` 负责管理 `Cluster` 各种资源，保证资源处于预期的状态。`Controller Manager` 由多种 `controller` 组成，包括 `replication controller`、`endpoints controller`、`namespace controller`、`serviceaccounts controller` 等。不同的 `controller` 管理不同的资源。

#### 2.2.3 etcd

etcd 负责保存 Kubernetes Cluster 的配置信息和各种资源的状态信息。当数据发生变化时，etcd 会快速地通知 Kubernetes 相关组件。

#### 2.2.4 Pod 网络

Pod 网络让 Kubernetes Cluster 中 Pod 能够相互通信。

### 2.3 Node

`Node` 的职责是运行容器应用。`Node` 由 `Master` 管理，`Node` 负责监控并汇报容器的状态，并根据 Master 的要求管理容器的生命周期。

每个工作节点都有一个 `Kubelet`，它是管理 节点 并与 `Kubernetes Master` 节点进行通信的代理。节点 上还应具有处理容器操作的工作，例如 `Docker` 或 `rkt`。一个 `Kubernetes` 工作集群至少有三个节点。

`Node` 上运行的 Kubernetes 组件有

#### 2.3.1 kubelet

`kubelet` 是 `Node` 的 `agent`，当 `Scheduler` 确定在某个 `Node` 上运行 `Pod` 后，会将 `Pod` 的具体配置信息（image、volume 等）发送给该节点的 `kubelet`，`kubelet` 根据这些信息创建和运行容器，并向 `Master` 报告运行状态。

#### 2.3.2 kube-proxy

每个 Node 都会运行 kube-proxy 服务，它负责将访问 service 的 TCP/UPD 数据流转发到后端的容器。如果有多个副本，kube-proxy 会实现负载均衡。

#### 2.3.3 Pod 网络

Pod 网络让 Kubernetes Cluster 中 Pod 能够相互通信。

![2020-05-17-QCoVAY](https://image.ldbmcs.com/2020-05-17-QCoVAY.jpg)

### 2.4 Controller

`Kubernetes` 通常不会直接创建 `Pod`，而是通过 `Controller` 来管理 `Pod` 的。`Controller` 中定义了 `Pod` 的部署特性，比如有几个副本，在什么样的 `Node` 上运行等。`Kubernetes` 中有多种 `Controller`。

#### 2.4.1 Deployment

`Deployment` 可以管理 `Pod` 的多个副本。`Deployment` 负责创建和更新应用程序实例。创建 `Deployment` 后， `Kubernetes master` 会将 `Deployment` 创建的应用程序实例调度到集群中的各个节点。

#### 2.4.2 ReplicaSet

实现了 `Pod` 的多副本管理。使用 `Deployment` 时会自动创建 `ReplicaSet`，也就是说 `Deployment` 是通过 `ReplicaSet` 来管理 `Pod` 的多个副本，我们通常不需要直接使用 `ReplicaSet`。

#### 2.4.3 DaemonSet

用于每个 `Node` 最多只运行一个 `Pod` 副本的场景。比如运行每台机器上运行一个监测系统。

#### 2.4.4 StatefuleSet

能够保证 `Pod` 的每个副本在整个生命周期中名称是不变的。而其他 `Controller` 不提供这个功能，当某个 `Pod` 发生故障需要删除并重新启动时，`Pod` 的名称会发生变化。同时 `StatefuleSet` 会保证副本按照固定的顺序启动、更新或者删除。

#### 2.4.5 Job

用于运行结束就删除的应用。而其他 `Controller` 中的 `Pod` 通常是长期持续运行。

### 2.5 Service

`Pod` 在 `Kubernetes` 中是不稳定的，它可能被销毁并重新创建，或者重新放置到了不同的 `Node`，它们的 IP 可能就不相同，所以为了让应用稳定的访问到 `Pod` 我们就需要使用到 `Service`。

`Kubernetes` 中的服务是一个抽象对象，它定义了一组逻辑的 `Pods` 和一个访问它们的策略。服务让互相依赖的 Pod 之间的耦合松动。

我们部署了 `Deployment` 外界要访问我们的应用，或者 `Pod` 到 `Pod` 之间的访问，就要用到 `Service`。

`Service` 有自己的 IP 和端口，`Service` 为 `Pod` 提供了负载均衡。

它有几个子类型

#### 2.5.1 ClusterIP

\(默认\) - 在集群中的内部IP上公开服务。此类型使服务只能从集群中访问。

#### 2.5.2 NodePort

使用NAT在群集中每个选定的节点的同一端口上显示该服务。使用 `:`可以从群集外部访问服务。建立 ClusterIP 的超集。它类似于 `docker run` 中的 `-p` 参数。

#### 2.5.3 LoadBalancer

在当前云中创建外部负载平衡器\(如果支持\)，并为服务分配固定的外部IP。建立 `NodePort` 的超集。

#### 2.5.4 ExternalName

用任意名称显示该服务，本过程通过使用该名称返回 CNAME 记录达成。无须使用代理。这种类型需要 v1.7 或更高版本的 kube-dns.

### 2.6 Namespace

如果有多个用户或项目组使用同一个 `Kubernetes Cluster`，可以使用 `Namespace`。

`Namespace` 可以将一个物理的 `Cluster` 逻辑上划分成多个虚拟 `Cluster`，每个 `Cluster` 就是一个 `Namespace`。不同 `Namespace` 里的资源是完全隔离的。

默认会创建两个 `Namespace`

* `default` 创建资源时如果不指定，将被放到这个 Namespace 中。
* `kube-system` Kubernetes 自己创建的系统资源将放到这个 Namespace 中。

### 2.7 Taint 和 Toleration

`Taint` 是设置到节点上类似于标签，它分为三个部分 `键=值:效果`，它好像在描述这个节点有这些污点，`Pod` 要会根据自己的 `Toleration`（容忍）来判断自己要不要运行在这个节点。

`Taint` 和 `toleration` 相互配合，可以用来避免 `pod` 被分配到不合适的节点上。每个节点上都可以应用一个或多个 `taint` ，这表示对于那些不能容忍这些 `taint` 的 `p`od，是不会被该节点接受的。

```text
$ kubectl taint nodes node1 key=value:NoSchedule
# 给节点 node1 设置 taint
$ kubectl taint nodes node1 key=value:NoSchedule-
# 后面加个减号可以删除这个 taint
```

#### 2.7.1 效果

`taint` 一共有三种效果 `NoSchedule` `PreferNoSchedule` 和 `NoExecute`。

* 如果 `node` 上有一个 `pod` 不能容忍的 `NoSchedule` 的 `taint`，则 Kubernetes 不会将 pod 分配到该节点。
* 如果 `node` 上有一个 `pod` 不能容忍的 `PreferNoSchedule` 的 `taint`，则 Kubernetes 会_尝试_将 `pod` 分配到该节点。
* 如果 `node` 上有一个 `pod` 不能容忍的 `NoExecute` 的 `taint`，则 Kubernetes 不会将 `pod` 分配到该节点，和 `NoSchedule` 不同如果 `pod` 已经在节点上运行它会将 `pod` 从该节点驱逐。

#### 2.7.2 tolerations

`Pod` 可以通过配置文件它的 `toleration`

```yaml
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
# operator 是 Equal，需要 键 值 效果完全匹配
- key: "key2"
  operator: "Exists"
  effect: "NoSchedule"
# 如果 operator 是 Exists 则不能指定 value
- operator: "Exists"
# 表示这个 toleration 能容忍任意 taint
tolerations:
- key: "key3"
  operator: "Exists"
# key 要匹配，效果任意
- key: "key4"
  operator: "Equal"
  value: "value4"
  effect: "NoExecute"
  tolerationSeconds: 3600
# tolerationSeconds 代表 node 设置了上面的 taint，
# Pod 还可以在上面运行 `tolerationSeconds` 秒，如果不设置则可以一直运行
```

新版本的 Kubernetes 可以自动给 node 设置 `taint`，如 `node.kubernetes.io/not-ready`

出于安全考虑，默认配置下 Kubernetes 不会将 Pod 调度到 Master 节点。如果希望将 master 也当作 Node 使用，可以执行。

```text
$ kubectl taint node node1 node-role.kubernetes.io/master-
# 去除
$ kubectl taint node k8s-master node-role.kubernetes.io/master='':NoSchedule
# 恢复
```

```text
$ kubectl describe node node1
# 我们可以通过 kubectl describe 查看节点的 taint
```

## 3. 使用

我们现在使用 Kubernetes 创建一个小应用。

![2020-05-17-WPhkGY](https://image.ldbmcs.com/2020-05-17-WPhkGY.jpg)

我们在 `deployment` 中运行 4 个 `Pod` 副本，使用 `NodePort` 类型的 `service` 暴露我们的 `deployment`，让外部可以访问到。

可以使用本地的 `minikube` 或 Kubernetes 交互教程中的虚拟机。

```text
$ kubectl run es --image=elasticsearch:2 --port=9200
# kubectl run 有点类似 docker run 它会创建并运行指定类型的镜像
# 它也支持 `--rm` `--env` `--command` `--restart` 等这些参数
# 上面我们创建一个 Deployment，Deployment 会创建 ReplicaSet
# 由 ReplicaSet 创建 Pod
# `--image` 用来指定镜像
# `--port` 是容器需要暴露的端口
# `--replicas` 参数为创建多少个副本，默认是 1
```

```text
$ kubectl get deploy
# deploy 也可以写成 deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
es                    1/1     1            1           30s
$ kubectl get po
# po 也可以写成 pods
NAME                                   READY   STATUS    RESTARTS   AGE
es-6994df7bf8-cchzb                    1/1     Running   0          51s
```

> `kubectl get` 可以让我们获取集群的各种资源信息，比如 `kubectl get nodes` 获取集群节点信息，`kubectl get namespaces` 来获取命令空间信息。

我们现在启动了一个 `deployment` 它里面有一个运行着 `elasticsearch` 容器的 `pod`。但是我们现在还不能访问到它，想要访问到它我们需要 `service` 的帮助。

```text
$ kubectl expose deploy es --port=9200 --target-port=9200 --type=NodePort
service/es exposed
# kubectl expose 将资源暴露成新的 service
# 上面我们将名为 es 的 deployment 资源暴露出来
# `--type` 将类型设置为 NodePort 
# `--port` 指定服务的端口，其他 deplyment 可以通过这个端口访问到我们 es deplyment
# `--target-port` 目标端口 service 会将流量导入到这个端口
$ kubectl get services
NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
es                    NodePort    10.99.113.186   <none>        9200:32687/TCP   11s
# 可以看到我们的服务被随机映射到了 32687 端口
$ curl `hostname -i`:32687
{
  "name" : "Firelord",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "-QGqxWm3RTCrCv0wDAQJ9A",
  "version" : {
    "number" : "2.4.6",
    "build_hash" : "5376dca9f70f3abef96a77f4bb22720ace8240fd",
    "build_timestamp" : "2017-07-18T12:17:44Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.4"
  },
  "tagline" : "You Know, for Search"
}
# 现在我们就可以访问到 es deployment 了
```

现在想把我们的 pod 增加 4 个，就需要用到 `kubectl scale` 命令。

```text
$ kubectl scale --replicas=4 deployment/es
deployment.extensions/es scaled
# kubectl scale 用来伸缩 
# Deployment, ReplicaSet, Replication Controller, 或 StatefulSet 的大小
# 我们这里将 es 应用创建 4 个副本
$ kubectl get deploy
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
es                    4/4     4            4           3m49s
$ kubectl get po
NAME                                   READY   STATUS    RESTARTS   AGE
es-6994df7bf8-brhfw                    1/1     Running   0          3m59s
es-6994df7bf8-bvmkk                    1/1     Running   0          19s
es-6994df7bf8-ffv7c                    1/1     Running   0          19s
es-6994df7bf8-z4kkf                    1/1     Running   0          19s
$ curl `hostname -i`:32687
{
  "name" : "Claudette St. Croix"
}
$ curl `hostname -i`:32687
{
  "name" : "Firelord"
}
# 请求会被负载均衡到 4 个节点中的一个
```

我们还可以滚动更新。

```bash
$ kubectl set image dployment/es es=elasticsearch:5
# 滚动跟新 elasticsearch:2 到 elasticsearch:5
# kubectl set 可以用来配置更新资源，它有多个子命令如 `image` `env` 等
# 上面命令我们将 es deployment 的 es 容器的镜像更新到 elasticsearch:5
$ kubectl describe pod es
# kubectl describe 用来查看资源详情类似于 docker inspect
# kubectl describe 类型 前缀
Containers:
  es:
    Image:          elasticsearch:5 # 镜像现在已经是版本 5 了
    Port:           9200/TCP
    Host Port:      0/TCP
    State:          Running
```

还可以撤销回去。

```bash
$ kubectl rollout undo deployment/es
# kubectl rollout 用来管理回滚
# kubectl rollout undo 回滚到之前的版本
# `--to-revision=0` 回滚版本 默认 0（上一个版本）
$ kubectl describe po es
Containers:
  es:
    Image:          elasticsearch:2 # 变回版本 2 了
    Port:           9200/TCP
    Host Port:      0/TCP
    State:          Running
```

## 4. 配置文件

我们除了使用 命令式 来创建资源，还可以使用 `yaml` 配置文件。不同于 docker-compose，Kubernetes 中一个配置文件只创建一个对象。使用配置文件我们创建或更新资源只用 `kubectl apply` 这一个命令就可以了。

我们现在使用配置文件创建一个 [drupal](https://www.drupal.org/) 网站。

![2020-05-17-tKpf4t](https://image.ldbmcs.com/2020-05-17-tKpf4t.jpg)

首先新建一个 `k8s` 目录，然后再里面创建一个 `drupal-deploy.yml` 文件。

```yaml
apiVersion: apps/v1
Kind: Deployment # 要创建对象的类型
metadata: # 元数据
    name: drupal-deployment  # 对象名称
    namespace: default # 命名空间，默认 default
    labels: # 标签
        app: drupal-deployment
spec: # 配置
    replicas: 1 # 期望的副本，默认 1
    minReadySeconds: 0 # 应用准备时间，默认 0
    selector: # 选择要管理的 Pod
        matchLabels: # 需要与 Pod 的 label 匹配
            app: drupal-pod
    template: # pod 的配置
        metadata: # pod 的元数据
            labels: # 标签
                app: drupal-pod
        spec: # 配置
            containers: # Pod 中的容器
                - name: drupal # 名称
                  image: drupal # 镜像
                  imagePullPolicy: IfNotPresent
                  # IfNotPresent（默认）Always 或 Never
                  ports: # 暴露的端口
                    - containerPort: 80
                      protocol: TCP
                      # TCP（默认）, UDP, 或 SCTP
            tolerations: # Pod 容忍
                - key: key
                  operator: Exists
            restartPolicy: Always # Pod 重启规则
            # Always（默认）, OnFailure, 或 Never
```

`apiVersion` 是 api 的版本，它控制能创建什么样的对象，比如我们要创建一个 `Deployment` 对应的 `apiVersion` 是 `apps/v1`，所以一般我们一般根据 `Kind` 去选择 `apiVersion`。可以在 [API Reference](https://kubernetes.io/docs/reference/#api-reference) 查看更多信息。

```bash
$ kubectl api-versions # 查看当前 kubernetes 支持的 api 版本
```

API 版本号分为 3 种：

* `Alpha 测试版本` 可能是有缺陷的和随时被删除，默认情况是关闭的。
* `Beta 测试版本` 已经测试过，支持的功能不会删除，细节可能发生变化，默认开启。
* `稳定版本` 版本名称是 `vX`，其中 `X` 是整数。

`imagePullPolicy` 定义了镜像拉取的规则，默认是 `IfNotPresent`，就是有本地缓存就用本地缓存，没有就去远程拉取，如果镜像名后的标签是`:latest` 则默认为 `Always`。总是会去远程拉取。

### 4.1 Service

`Pod` 是不稳定的它可以被创建，也可以被销毁，每个 `Pod` 都会获取它自己的 `IP` 地址，当它被动态地创建和销毁，可能获取到不同的 `IP`，为了我们可以稳定的访问到 `Pod` 就需要使用到 `Service`。

有了 `Service` 我们就无需关系后端 `Pod`，`Service` 会把请求代理到合适的 `Pod`。

默认的 `Service` 是 `ClusterIP`（虚拟IP），`ClusterIP` 服务只能够在集群内部可以访问，无法被外部直接访问。

![2020-05-17-jW6aAC](https://image.ldbmcs.com/2020-05-17-jW6aAC.png)

`kube-proxy` 负责为 `ExternalName` 以外的类型的服务实现一种形式的虚拟IP

现在我们再创建一个 `drupal-cluster-ip.yml`

```text
apiVersion: v1
kind: Service
metadata:
    name: drupal-cluster-ip
spec:
    type: ClusterIP
    # ClusterIP（默认）, ExternalName, NodePort, 或 LoadBalancer
    sessionAffinity: None
    # None（默认） 或 ClientIP
    # 如果想让同一个客户端的请求连接到同一个 Pod 可以设置为 ClientIP
    selector: # 通过标签选择 Pods
       app: drupal-pod
    ports:
        - port: 80
          # Service 的 port
          targetPort: 80
          # 目标（pod）的 port
          protocol: TCP
          # TCP（默认）, UDP 或 SCTP
          name: http
          # 名称，当有多个端口时必填
```

然后我们创建 `postgres-cluster-ip.yml`

```text
apiVersion: v1
kind: Service
metadata:
    name: postgres-cluster-ip
spec:
    selector:
        app: postgres-pod
    ports:
        - port: 5432
          targetPort: 5432
```

### 4.2 Volumes

当运行在 `Pod` 中的容器崩溃，重启时，容器中保存在磁盘上的数据都会被清空，在 `Docker` 中我们使用 `Volume` 来解决这个问题。

在 `Kubernetes` 中也有 `Volume` 当容器重启时，`Volume` 中的数据不会被清除，而且 `Pod` 中的容器可以共享 `Volume`。但是 `Kubernetes Volume` 也有生命周期，当 `Pod` 不存在时，`Kubernetes Volume` 也会不存在，但是使用如 `awsElasticBlockStore` 这种类型的 `Volume`，删除 `Pod` 时不会删除 `Volume` 中的数据只是卸载数据卷。

`Kubernetes Volume` 支持非常多的类型，如 `azureDisk` `configMap` `secret` 等等。

#### 4.2.1 emptyDir

`emptyDir` 类型是一个空的文件夹，当 `Pod` 被创建时，它也被创建，当 `Pod` 被删除时，它里面的数据将被永久删除。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts: # 挂在到容器中的位置
    - mountPath: /cache
      name: cache-volume
  volumes: # 定义数据卷
  - name: cache-volume # 数据卷名称
    emptyDir:
        medium: Memory
        # 空字符串（默认） 或 Memory
        # 该存储介质的是什么空字符串代表节点默认的媒介
        sizeLimit: 1Gi # 数据卷大小，默认未定义
```

#### 4.2.2 hostPath

`hostPath` 类型将节点中的目录挂在到容器中。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      path: /data
      # 节点中的路径
      type: DirectoryOrCreate # 是目录，当不存在时自动创建
      # 默认空字符串，代表挂载 hostPath 之前不会执行任何检查
      # File, Directory, FileOrCreate, DirectoryOrCreate 等等
```

### 4.3 Persistent Volumes

`Persistent Volumes`（PV）是群集中的一块存储，它生命周期独立于任何 `pod`。

`Persistent Volume Claim`（PVC）是用户对存储的请求，`PVC` 消耗 `PV` 资源，`PVC` 可以要求特定的大小和访问模式的存储。

我们使用 `PVC` 来请求不同大小的 `PV`，集群管理员会根据请求找到符合的 `PV`，如果不存在则会创建符合要求的 `PV`。

我们的应用的 `postgres` 使用 `PV`，创建一个 `db-pvc.yml`。

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-pvc
spec:
  accessModes: # 访问模式
    - ReadWriteOnce
  volumeMode: Filesystem # 默认 Filesystem
  resources:
    requests:
      storage: 1Gi # 请求的存储大小
  storageClassName: standard # 选择属于这个类的 PV
  # 特定类的 `PV` 只能绑定到请求该类的 `PVC`
  selector:
    matchLabels:
      release: stable # 选择特定标签的 PV
```

一共有三中访问模式

* `ReadWriteOnce` 被一个节点挂载读写
* `ReadOnlyMany` – 被多个节点挂载可读
* `ReadWriteMany` – 被多个节点挂载读写

我们也可以手动创建一个 `PersistentVolume`，配置它的 `persistentVolumeReclaimPolicy` 它的值 `Delete` 或 `Retain` ，也就是当我们删除 `PVC` 时，`PV` 的中的数据是全部删除还是保留。

然后再创建 `postgres-deploy.yml` 文件

```text
apiVersion: apps/v1
kind: Deployment 
metadata:
    name: postgres-deployment
spec:
    selector:
        matchLabels:
            app: postgres-pod
    template:
        metadata:
            labels:
                app: postgres-pod
        spec:
            volumes:
                - name: postgres-volume
                  persistentVolumeClaim:
                    claimName: db-pvc
            containers:
                - name: postgres
                  image: postgres
                  env: # 设置容器的环境变量
                    - name: POSTGRES_PASSWORD
                      value: password
                  ports:
                    - containerPort: 5432
                  volumeMounts:
                    - name: postgres-volume
                      mountPath: /var/lib/postgresql/data
                      subPath: postgres
                      # Volume 中的子目录
                      # 防止将 Volume 中其他文件也挂载进来
```

### 4.4 Ingress

`Ingress` 可以将集群外部的 `HTTP` 和 `HTTPS` 路由到集群中的服务。

要想 `ingress` 生效，必须有一个 `ingress controller` 满足它。

我们使用 [ingress-nginx](https://github.com/kubernetes/ingress-nginx/) 这个 `ingress controller`，我们可以使用如下命令安装它。

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml
```

如果使用 minikube 可以执行：

```bash
minikube addons enable ingress
```

也可以使用 [Helm](https://github.com/helm/helm) 安装（Kubernetes 包管理器，功能强大）。

```bash
helm install stable/nginx-ingress --name my-nginx
```

现在来创建 `ingress-service.yml` 文件。

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ingress-service
  annotations: # 和资源一起存储的键值对，它相当于一个额外的配置
    kubernetes.io/ingress.class: nginx
    # 表示我们使用 ingress-nginx
    nginx.ingress.kubernetes.io/rewrite-target: /
    # 将 url 前缀重写为 /，如 /api/ 将会变成 /
spec:
  rules:
  - http:
      paths:
      - path: / # 该路径的 url 全部代理到我们的 drupal-cluster-ip
        backend:
          serviceName: drupal-cluster-ip
          servicePort: 80
```

好了现在我们就可以创建应用了。

```bash
$ kubectl apply -f k8s
```

`kubectl apply` 来应用配置文件，`-f` 后面可以是文件，文件夹 或 url。

我们可以使用命令查看创建状态：

```bash
$ kubectl get svc # 查看 services
$ kubectl get deploy 查看 deployments
$ kubectl get po # 查看 pods
$ kubectl get pv # 查看 Persistent Volumes
$ kubectl get pvc # 查看 Persistent Volume Claim
```

发现全部创建成功后我们就可以去浏览器访问我们的应用了。

```bash
$ minikube ip # 获取虚拟机 ip，输入到浏览器中
```

当我们打开我们的浏览器这个地址，会发现会跳转到 `https` 连接，然后浏览器报不安全的错误，我们要点高级继续访问我们的网站才能浏览我们的应用。

因为我们 ingress 默认是 `https` 的，我们可以看见证书是一个假证书。

![2020-05-17-MGwtyz](https://image.ldbmcs.com/2020-05-17-MGwtyz.jpg)

我们可以使用 [cert manager](https://github.com/jetstack/cert-manager) 来自动配置和管理 TLS 证书。

![2020-05-17-bRPkFc](https://image.ldbmcs.com/2020-05-17-bRPkFc.jpg)

### 4.5 DNS

可以看到我们的 Host 可以直接填写 postgres service 的名称。

`kubeadm` 部署时会默认安装 `kube-dns` 组件，`kube-dns` 是一个 `DNS` 服务器。每当有新的 `Service` 被创建，`kube-dns` 会添加该 `Service` 的 `DNS` 记录。`Cluster` 中的 `Pod` 可以通过 `<SERVICE_NAME>.<NAMESPACE_NAME>` 访问 `Service`。

所以我们以可以通过 `postgres-cluster-ip.default` 访问，`default` 命名空间可以省略，我们就可以写成 `postgres-cluster-ip` 。

## 5. 控制 Pod 运行的 Node

Kubernetes 中可以使用 `label` 来控制 `Pod` 运行的节点。

```bash
$ kubectl label pods foo unhealthy=true
# 给节点 foo 设置 label unhealthy=true
$ kubectl get node --show-labels
# 查看节点的 label
$ kubectl label --overwrite pods foo status=unhealthy
# 修改 node foo 的label
$ kubectl label pods --all status=unhealthy
# 给所有这个 namespace 中的节点添加指定 label
$ kubectl label pods foo bar-
# 后面加 - 号可以删除节点上的 label
```

给节点设置了 label 后，我们就可以使用配置文件中的 `nodeSelector` 来指定。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
      nodeSelector: # 选择要运行的节点
        env: test
```

如果我们现在把节点的这个标签删除了，会发现 Pod 还运行在这个节点，不会重新部署。

## 6. Secret

`Secret` 对象类型用来保存敏感信息，例如密码、OAuth 令牌和 ssh key。将这些信息放在 `secret` 中比放在 `pod` 的定义或者 `docker` 镜像中来说更加安全和灵活。

`Secret` 会以密文的方式存储数据，避免了直接在配置文件中保存敏感信息。`Secret` 会以 `Volume` 的形式被 `mount` 到 `Pod`，容器可通过文件的方式使用 `Secret` 中的敏感数据。

我们可以通过文件来创建 `Secret`。

```bash
$ kubectl create secret generic my-secret \
    --from-file=./username.txt \
    --from-file=./password.txt
```

也可以通过 `yaml` 文件创建。

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
data: # 每一项必须是 base64 编码
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```

```bash
kubectl create -f ./secret.yaml # 创建 secret
```

然后我们就可以在 `Pod` 中使用它。

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: secret-test-pod
  labels:
    name: secret-test
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: mysecret
      items: # 可选，自定义访问的 secret
        - key: password
          path: a/pass
  containers:
  - name: ssh-test-container
    image: mySshImage
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: /etc/secret # 挂载的目录
```

现在我们就可以通过 `/etc/secret/username` 和 `/etc/secret/a/pass` 访问我们的 Secret 了。

我们还可以通过环境变量访问我们的 Secret。

```yaml
apiVersion: v1
metadata:
  name: secret-test-pod
spec:
  containers:
  - name: ssh-test-container
    image: mySshImage
    env:
      - name: SECRET
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
```

## 7. configMap

`ConfigMap` 可以用来保存应用配置信息，`ConfigMap` 和 `Secret` 非常类似，主要的不同是数据以明文的形式存放。

```bash
$ kubectl create configmap myconfigmap \
    --from-file=./config1 \
    --from-file=./config2
```

通过文件创建 `configMap`。每个文件内容对应一个信息条目。

我们也可以使用 `yaml` 文件。

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
data:
  game.properties: | # | 表示保留换行符
    enemies: aliens
    lives: 3
    enemies.cheat: true
    enemies.cheat.level: noGoodRotten
    secret.code.passphrase: UUDDLRLRBABAS
    secret.code.allowed: true
    secret.code.lives: 30

--- # 文件的开头，可以将多个配置写入一个文件中

apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "ls /etc/config/" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: special-config
  restartPolicy: Never
```

和 `secret` 一样也可以通过环境变量来使用，只是将 `secretKeyRef` 换为 `configMapKeyRef`。

## 8. 可视化监控

除了上面介绍的 [kubernetes dashboard](https://github.com/kubernetes/dashboard)，还有很多监控工具。

* [Kuboard官网_Kubernetes教程\_K8S安装_管理界面](https://kuboard.cn/)：一款基于 Kubernetes 的微服务管理界面。目的是帮助用户快速在 Kubernetes 上落地微服务。

  ![2020-05-17-4vz0XL](https://image.ldbmcs.com/2020-05-17-4vz0XL.png)

* [Weave Scope](https://github.com/weaveworks/scope) Docker 和 Kubernetes 的监控工具

  ![2020-05-17-28DI4P](https://image.ldbmcs.com/2020-05-17-28DI4P.jpg)

* [Prometheus Operator](https://github.com/coreos/prometheus-operator) 是 CoreOS 开发的基于 Prometheus 的 Kubernetes 监控方案，也可能是目前功能最全面的开源方案。

  ![2020-05-17-jUrPEW](https://image.ldbmcs.com/2020-05-17-jUrPEW.jpg)

## 9. 更多

[Docker 零基础入门](https://juejin.im/post/5d1418ea6fb9a07ef444175c)

[Docker Compose 零基础入门](https://juejin.im/post/5d17442e518825559f46ed92)

[Docker Swarm 零基础入门](https://juejin.im/post/5d187db1e51d455d6c0ad96f)

