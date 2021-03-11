## 1. 什么是Kubernetes

**[Kubernetes](https://kubernetes.io/zh/docs/concepts/overview/what-is-kubernetes/) 是用于自动部署，扩展和管理容器化应用程序的开源系统。**

使用Kubernetes，您可以快速有效地回应客户需求:

- **快速部署**：快速、可预测地部署应用。

- **弹性伸缩**：动态缩放您的应用。

- 无缝地推出新功能。

- 仅对需要的资源限制硬件的使用

**我们的目标是构建一个生态系统，提供组件和工具以减轻在公共和私有云中运行应用程序的负担。**

**Kubernetes是：**

- **可移植**: 共有、私有、混合、多云  

- **可扩展**: 模块化、可插拔、提供Hook、可组合

- **自愈**: 自动放置、自动重启、自动复制、自动缩放

Google于2014年启动了Kubernetes项目。Kubernetes建立在Google在大规模运行生产工作负载方面 十几年的经验之
上，并结合了社区中最佳的创意和实践。

### 1.1 为什么使用容器

<img src="https://image.ldbmcs.com/2020-05-14-6jSpBb.png" title="" alt="" width="483">

- 部署应用程序的旧方法是使用操作系统的软件包管理器在主机上安装应用程序。这种方式，存在可执行文件、配置、库
  和生命周期与操作系统相互纠缠的缺点（耦合度高）。人们可构建不可变的虚拟机映像，从而实现可预测的升级和回滚，但VM是重
  量级、不可移植的。

- 新方法是部署容器，容器基于操作系统级别的虚拟化而不是硬件虚拟化。这些容器彼此隔离并且与宿主机隔离:它们有
  自己的文件系统，看不到对方的进程，并且它们的计算资源使用可以被界定。它们比VM更容易构建，并且由于它们与
  底层基础架构和宿主机文件系统解耦了，可实现跨云、跨操作系统的移植（耦合度低）。

- 由于容器小而快，因此可在**每个容器镜像中包装一个应用程序**。这种一对一的应用到镜像关系解锁了容器的全部优势。
  **使用容器，可以在构建/发布期间(而非部署期间)创建不可变的容器镜像**，因为每个应用程序无需与其余的应用程序
  栈组合，也无需与生产基础架构环境结合。 在构建/发布期间生成容器镜像使得从开发到生产都能够保持一致的环境。
  同样，**容器比虚拟机更加透明、便于监控和管理**——特别是当容器进程的生命周期由基础架构管理而非容器内隐藏的进
  程监控程序管理时。 最后，**通过在每个容器中使用单个应用程序的方式，管理容器无异于管理应用程序的部署**。

**容器好处概要:**

- **灵活的应用创建和部署** : 与VM映像相比，容器镜像的创建更加容易、有效率。  

- **持续开发，集成和部署** : 通过快速轻松的回滚(由于镜像的不可变性)提供可靠且频繁的容器镜像构建和部署。 

- **Dev和Ops分离问题** : 在构建/发布期间而非部署期间创建镜像，从而将应用程序与基础架构分离。 

- **开发、测试和生产环境一致** : 在笔记本电脑运行与云中一样。  

- **云和操作系统可移植性** : 可运行在Ubuntu、RHEL、CoreOS、内部部署，Google Container Engine以及任何其他
  地方。

- **以应用为中心的管理**: 从在虚拟硬件上运行操作系统的抽象级别，提升到使用逻辑资源在操作系统上运行应用程序
  的级别。 

- **松耦合，分布式，弹性，解放的微服务**: 应用程序分为更小、独立的部件，可动态部署和管理——而不是一个运行
  在一个大型机上的单体。

- **资源隔离**: 可预测的应用程序性能。 

- **资源利用**: 效率高，密度高。

### 1.2 为什么使用 Kubernetes

**Kubernetes可在物理机或虚拟机集群上调度和运行应用容器**。然而，Kubernetes还允许开发人员将物理
机以及虚拟机 “从主机为中心的基础设施转移到以容器为中心的基础设施”，从而提供容器固有的全部优势。 

**Kubernetes提供了构建以容器为中心的开发环境的基础架构。**

### 1.3 Kubernetes 是什么

尽管Kubernetes提供了大量功能，但总有新的场景从新功能中受益。**应用程序特定的工作流程可被简化，从而加快开发
人员的速度。可接受的特别编排最初常常需要大规模的自动化**。这就是为什么Kubernetes也被设计为提供构建组件和工
具的生态系统，使其更容易部署，扩展和管理应用程序。

**Label 允许用户随心所欲地组织他们的资源。Annotation 允许用户使用自定义信息来装饰资源以方便他们的工作流程，
并为管理工具提供检查点状态的简单方法。**

此外， `Kubernetes control plane` 所用的API 与开发人员和用户可用的API相同。用户可以使用 their own API 编写自己
的控制器，例如 scheduler ，这些API可由通用 command-line tool 定位。

这种 design 使得许多其他系统可以构建在Kubernetes上。

### 1.4 Kubernetes 不是什么

Kubernetes不是一个传统的，全面的PaaS系统。 它保留了用户的重要选择。 

**Kubernetes**:

- **不限制支持的应用类型**。不规定应用框架(例如 Wildfly )，不限制支持的语言运行时(例如Java，Python， Ruby)，不局限于 12-factor applications ，也不区分应用程序和服务 。 Kubernetes旨在支持各种各样的工作负载，包括无状态、有状态以及数据处理工作负载。 如果应用程序可在容器中运行，那么它应该能够很好地在 Kubernetes上运行。

- **不提供中间件**(例如消息总线)、数据处理框架(例如Spark)、数据库(例如MySQL)，也不提供分布式存储系统(例如Ceph)作为内置服务。 这些应用可在Kubernetes上运行。

- **没有点击部署的服务市场**。

- **不部署源代码，并且不构建应用**。持续集成(CI)工作流是一个不同用户/项目有不同需求/偏好的领域，因此它支持在Kubernetes上运行CI工作流，而不强制工作流如何工作。

- **允许用户选择其日志记录、监视和警报系统**。(它提供了一些集成。)

- **不提供/授权一个全面的应用配置语言/系统**(例如 jsonnet )。

- **不提供/不采用任何综合的机器配置、维护、管理或自愈系统**。

另一方面，一些PaaS系统可运行在 Kubernetes上，例如 Openshift 、 Deis 、Eldarion 等。 您也可实现自己的定制 PaaS，与您选择的CI系统集成，或者仅使用Kubernetes部署容器。

由于Kubernetes在应用层面而非硬件层面上运行，因此它提供了PaaS产品通用的功能，例如**部署，扩展，负载均衡，日志和监控**。然而，Kubernetes并不是一个单体，这些默认解决方案是可选、可插拔的。

另外Kubernetes不仅仅是一个编制系统 。实际上，它消除了编制的需要。编制的技术定义，就是执行定义的工作流:首先执行A，然后B，然后执行C。相反，**Kubernetes由一组独立、可组合的控制进程组成，这些控制进程可将当前状态持续地驱动到所需的状态**。 如何从A到C不要紧，集中控制也不需要;这种做法更类似于编排 。 这使系统更易用、更强大，更具弹性和可扩展性。

> 编排和编制: https://wenku.baidu.com/view/ad063ef2f61fb7360b4c65cd.html

### 1.5 Kubernetes VS k8s

Kubernetes源自希腊语，意思是舵手或⻜行员 ，是governor(掌舵人) 和cybernetic(控制论) 的根源。 K8s是将8 个字母“ubernete”替换为“8”的缩写。

### 1.6 Swarm VS Kubernetes VS Mesos

当前主流的容器集群管理技术，包括了 Docker 官方的 **Docker Swarm**、Twitter 背书的 **Mesos** 和 Google 背书的 **Kubernetes**。由于Apache Mesos 只是一个分布式内核，目前的发展方向是数据中心操作系统（DCOS），它同时支持 Marathon、Kubernetes 和 Swarm 等多种框架，连 Mesosphere 也是 Kubernetes 生态的一员，从编排的角度，讨论 Mesos 意义不大，故而只对比 Docker Swarm 和 Kubernetes。

- **Kubernetes 功能完善**：**资源调度、服务发现、运行监控、扩容缩容、负载均衡、灰度升级、失败冗余、容灾恢复、DevOps**等样样精通，可实现**大规模、分布式、高可用的 Docker 集群**，Kubernetes面向 PaaS，它直接为解决业务的分布式架构、服务化设计，完整定义了构建业务系统的标准化架构层，即Cluster、Node、Pod、Label等一系列的抽象都是定义好的，为服务编排提供了一个简单、轻量级的方式。

- **产品成熟，经验丰富**：Kubernetes 是基于 google 自身多年使用 linux 容器的经验创建出来的，所以可以说它是 **Google自身多年操作经验的一个复制**，使用Kubernetes 来管理容器在多个方面都将带来很大的好处，而其中最重要的就是 Google 把他们多年使用容器的经验带入了这个工具。Kubernetes 解决了许多 Docker 自身的问题。通过 Kubernetes，你可以在容器中使用实际的物理存储单元，从而我们可以很方便的把容器移动到其他机器上，而不丢失任何数据。Kubernetes 使用 flannel 来创建容器之间的网络，Kubernetes 集成了 load balancer，Kubernetes 使用 etcd 来实现服务发现，诸如此类的东西还有很多。

- 但是使用 Kubernetes 是要付出一定的代价爱的，比如，Kubernetes 使用了一个完全不同于 Docker 的 CLI (Command Line Interface)，不同的 API 以及不同的 YAML 配置定义。换句话说，如果你使用Kubernetes，那么你**将完全不能使用 Docker 自带的CLI**，也不能使用 Docker Compose 来定义(创建)容器。**使用 Kubernetes，你必须为 Kubernetes 重新创建所有和Docker 容器管理相关的东西**，因此我们可以认为 Kubernetes 并不是为了 Docker创建的(从一定意义上说是正确的)。Kubernetes 提升了 Docker 容器集群的管理层次，但同时它的学习曲线也是非常陡峭的。

- Docker Swarm 就截然不同， 它就是针对 Docker 容器技术创建的集群工具。最关键的是 **Dcoker Swarm 对外提供的是完全标准的 Docker API**，因此任何使用 Docker API 与 Docker 进行通讯的工具(Docker ClI, Docker Compose, Dokku, Krane)都可以完全无缝地和 Docker Swarm协同工作。这一点对 Docker Swarm 来说既是一个优点，也是一个缺点。有点就是你可以使用熟悉的工具集，缺点也显而易见，就是你只能做 Docker API 规定的事情。如果 Docker API 不支持某个你要的功能，你就不能直接使用 Docker Swarm 来实现，你可能需要使用一些特别的技巧来实现(也可能完全不能实现)。

**所以，从设计模式、工具链、最佳实践和商业模式来看，Kubernetes 都是目前更加让人放心的容器编排管理技术。**

### 1.7 k8s VS k3s

k3s 是 [Rancher](https://rancher.com/) 推出的轻量级 k8s。k3s 本身包含了 k8s 的源码，所以本质上和 k8s 没有区别。但为了降低资源占用，k3s 和 k8s 还是有一些区别的，主要是：

- 使用了相比 Docker 更轻量的 [containerd](https://containerd.io/) 作为容器运行时（Docker 并不是唯一的容器选择）。
- 去掉了 k8s 的 Legacy, alpha, non-default features。
- 用 sqlite3 作为默认的存储，而不是 etcd。
- 其他的一些优化，最终 k3s 只是一个 binary 文件，非常易于部署。

所以 k3s 适用于边缘计算，IoT 等资源紧张的场景。同时 k3s 也是非常容易部署的，官网上提供了[一键部署的脚本](https://raw.githubusercontent.com/rancher/k3s/master/install.sh)。

**k3s的优点：**

- k3s将安装Kubernetes所需的一切打包进仅有60MB大小的二进制文件中，并且完全实现了Kubernetes API。为了减少运行Kubernetes所需的内存，Rancher删除了很多不必要的驱动程序，并用附加组件对其进行替换。

- k3s是一款完全通过CNCF认证的Kubernetes发行版，这意味着你可以编写YAML来对完整版的Kubernetes进行操作，并且它们也将适用于k3s集群。

- 由于它只需要极低的资源就可以运行，因此它能够在任何512MB RAM以上的设备上运行集群，换言之，我们可以让pod在master和节点上运行。

**k3s的缺点：**

- 首先，当前k3s的版本（k3s v0.8.1）仅能运行单个master，这意味着如果你的master宕机，那么你就无法管理你的集群，即便已有集群要继续运行。但是在k3s v0.10的版本中，多主模式已经是实验性功能，也许在下一个版本中能够GA。

- 其次，在单个master的k3s中，默认的数据存储是SQLite，这对于小型数据库十分友好，但是如果遭受重击，那么SQLite将成为主要痛点。但是，Kubernetes控制平面中发生的更改更多是与频繁更新部署、调度Pod等有关，因此对于小型开发/测试集群而言，数据库不会造成太大负载。

**结论：**

K8s和k3s各有优劣，使用场景也有所区别，因此不能一概而论。如果你要进行大型的集群部署，那么我建议你选择使用K8s；

如果你处于边缘计算等小型部署的场景或仅仅需要部署一些非核心集群进行开发/测试，那么选择k3s则是性价比更高的选择。

## 2. 安装 k8s

### 2.1 安装单机版 k8s

#### 2.1.1 对于Mac/Windows 10

对于macOS或者Windows 10，Docker已经原生支持了Kubernetes。你所要做的只是启用Kubernetes即可，如下图：

<img src="https://image.ldbmcs.com/2020-05-14-UUSEFJ.png" title="" alt="2020-05-14-UUSEFJ" width="485">

#### 2.1.2 Minikube

一些场景下，安装Minikube是个不错的选择，该方式适用于Windows 10、Linux、macOS 。

- 官方安装说明文档： https://github.com/kubernetes/minikube

- 如何在Windows 10上运行Docker和Kubernetes： http://dockone.io/article/8136

#### 2.1.3 启用Kubernetes Dashboard

```shell
kubectl proxy
```

访问:

http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/overview?namespace=default

参考:

https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

### 2.2 生产环境部署 k8s(略)

## 3. k8s 组件

### 3.1 Master组件

**Master组件提供K8s集群的控制面板**。Master对集群进行全局决策(例如调度)，以及检测和响应集群事件(例如:当replication controller所设置的 replicas 不够时，启动一个新的Pod)。 Master可在集群中的任意节点上运行。

然而，简单起⻅，设置脚本通常在同一个VM上启动所有Master组件，并且不会在该VM上运行用户的容器。请阅读 **Building High-Availability Clusters** 以实现多主机VM配置。

#### 3.1.1 kube-apiserver

**kube-apiserver 暴露Kubernetes的API**。它是Kubernetes控制能力的前端。它被设计为可水平扩展——也就是通过部署更多实例来实现扩容。详⻅ Building High-Availability Clusters 。

#### 3.1.2 etcd

**etcd 用作Kubernetes的后端存储**。集群的所有数据都存储在此。请为你Kubernetes集群的etcd数据提供备份计划。

#### 3.1.3 kube-controller-manager

**kube-controller-manager 运行Controller，它们是处理集群中常规任务的后台线程**。逻辑上来讲，每个Controller都是一个单独的进程，但为了降低复杂性，它们都被编译成独立的二进制文件并运行在一个进程中。

这些控制器包括:

- **Node Controller**：当节点挂掉时，负责响应。  

- **Replication Controller**：负责维护系统中每个replication controller对象具有正确数量的Pod。

- **Endpoints Controller**：填充Endpoint对象(即:连接Service&Pod)。  

- **Service Account & Token Controllers**：为新的namespace创建默认帐户和API access tokens。

#### 3.1.4 cloud-controller-manager

**cloud-controller-manager运行着与底层云提供商交互的Controller**。cloud-controller-manager是在Kubernetes 1.6版中引入的，处于Alpha阶段。

cloud-controller-manager仅运行云提供商特定的Controller循环。您必须在kube-controller-manager中禁用这些 Controller循环。可在启动kube-controller-manager时将 --cloud-provider 标志设为 external 来禁用控制器循环。

cloud-controller-manager允许云供应商代码和Kubernetes内核独立发展。在以前的版本中，核心的Kubernetes代码依赖于特定云提供商的功能代码。在未来的版本中，云供应商的特定代码应由云供应商自己维护，并在运行Kubernetes时与cloud-controller-manager相关联。

以下控制器存在云提供商依赖：

- **Node Controller**：用于检查云提供商，从而确定Node在停止响应后从云中删除  

- **Route Controller**：用于在底层云基础设施中设置路由  

- **Service Controller**：用于创建、更新以及删除云提供商负载均衡器  

- **Volume Controller**：用于创建、连接和装载Volume，并与云提供商进行交互，从而协调Volume

#### 3.1.5 kube-scheduler

kube-scheduler 监视新创建的、还没分配Node的Pod，并选择一个Node供这些Pod运行。

### 3.2 Node组件

**Node组件在每个Node上运行，维护运行的Pod并提供Kubernetes运行时环境。**

#### 3.2.1 kubelet

kubelet 是主要的Node代理。

- **监视已分配到其Node上的Pod**（通过apiserver或本地配置文件）

- 装载Pod所需的Volume。  

- 下载Pod的secret。

- 通过Docker(或实验时使用rkt)运行Pod的容器。

- 定期执行任何被请求容器的活动探针(liveness probes)。

- 在必要时创建mirror pod ，从而将pod的状态报告回系统的其余部分。

- 将节点的状态报告回系统的其余部分。

#### 3.2.2 kube-proxy

kube-proxy 在主机上维护网络规则并执行连接转发，从而来实现Kubernetes服务抽象。

#### 3.2.3 docker

docker 用于运行容器。

#### 3.2.4 rkt

rkt 是一个Docker的替代品，支持在实验中运行容器。

#### 3.2.5 supervisord

supervisord 是一个轻量级的进程监控/控制系统，可用于保持kubelet和docker运行。

#### 3.2.6 fluentd

fluentd 是一个守护进程，利用它可实现 cluster-level logging 。

3.1.6 addons(插件)

**Addon是实现集群功能的Pod和Service**。Pod可由Deployment、ReplicationController等进行管理。Namespace的插件对象则是在 kube-system 这个namespace中被创建的。

Addon manager创建并维护addon的资源。详⻅这里: [here](https://github.com/kubernetes/kubernetes/tree/HEAD/cluster/addons) 。

### 3.3 Addons

Addons 使用 Kubernetes 资源（DaemonSet、Deployment等）实现集群的功能特性。由于他们提供集群级别的功能特性，addons使用到的Kubernetes资源都放置在 `kube-system` 名称空间下。

#### 3.3.1 DNS

虽然其他Addon不是严格要求的，但**所有Kubernetes集群都应该有 cluster DNS**，许多用例都依赖于它。

Cluster DNS是一个DNS服务器，它为Kubernetes服务提供DNS记录。

**Kubernetes启动的容器会自动将该DNS服务器包含在DNS搜索中**。

#### 3.3.2 Web UI (Dashboard)

Dashboard 是一个Kubernetes集群通用、基于Web的UI。它允许用户管理/排错集群中应用程序以及集群本身。

![20200517Z8pRiI](https://image.ldbmcs.com/2020-05-17-Z8pRiI.jpg)

#### 3.3.3 Container Resource Monitoring(容器资源监控)

Container Resource Monitoring 将容器的通用时序指标记录到一个中心化的数据库中，并提供一个UI以便于浏览该数
据。

#### 3.3.4 Cluster-level Logging(集群级别的日志)

Cluster-level logging 机制负责将容器的日志存储到具有搜索/浏览界面的中央日志存储中去。

## 4. Kubernetes API

[API conventions doc](https://git.k8s.io/community/contributors/devel/api-conventions.md) 中描述了API的总体规范。  

[API Reference]()https://kubernetes.io/docs/reference 中描述了API端点、资源类型和样本。  

[access doc](https://kubernetes.io/docs/admin/accessing-the-api) 讨论了API的远程访问。  

Kubernetes API也是系统声明式配置模式的基础。 Kubectl 命令行工具可用于创建、更新、删除以及获取API对象。 

Kubernetes也会存储其API资源方面的序列化状态(目前存在 etcd 中)。 

Kubernetes本身被分解成了多个组件，通过其API进行交互。

## 5. 理解K8s对象
