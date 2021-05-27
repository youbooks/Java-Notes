# 什么是无服务器(what is serverless)？

> 转载：[什么是无服务器？](https://www.redhat.com/zh/topics/cloud-native-apps/what-is-serverless)

无服务器是一种[云原生](https://www.redhat.com/zh/topics/cloud-native-apps)开发模型，可使开发人员专注构建和运行应用，而无需管理服务器。

无服务器方案中仍然有服务器，但它们已从应用开发中抽离了出来。[云提供商](https://www.redhat.com/zh/topics/cloud-computing/what-are-cloud-providers)负责[置备](https://www.redhat.com/zh/topics/automation/what-is-provisioning)、维护和扩展服务器[基础架构](https://www.redhat.com/zh/topics/cloud-computing/what-is-it-infrastructure)等例行工作。开发人员可以简单地将代码打包到[容器](https://www.redhat.com/zh/topics/containers)中进行部署。

部署之后，无服务器应用即可响应需求，并根据需要[自动](https://www.redhat.com/zh/topics/automation)扩容。[公共云](https://www.redhat.com/zh/topics/cloud-computing/what-is-public-cloud)提供商的无服务器产品通常通过一种[事件驱动](https://www.redhat.com/zh/topics/integration/what-is-event-driven-architecture)执行模型来按需计量。因此，当无服务器功能闲置时，不会产生费用。

## 1. 无服务器架构概述

无服务器与其他[云计算](https://www.redhat.com/zh/topics/cloud)模型的区别在于，它是由云提供商负责管理[云基础架构](https://www.redhat.com/zh/topics/cloud-computing/what-is-cloud-infrastructure)和应用扩展。无服务器应用部署在容器中，这些容器在被调用时会自动按需启动。

在标准的[基础架构即服务（IaaS）](https://www.redhat.com/zh/topics/cloud-computing/what-is-iaas)云计算模型中，用户需要预先购买容量单元；也就是说，您要先向公共云提供商支付始终可用的服务器组件的费用，才能运行您的应用。 用户自行负责在需求高时扩展服务器容量，并在不再需要时缩减容量。即使在应用闲置不用期间，运行该应用所需的云基础架构也要保持就绪。

无服务器架构则与之相反，应用仅在需要时启动。有事件触发应用代码运行时，公共云提供商才会为这一代码分配资源。该代码执行结束后，用户便不再付费。除了成本与效率上的优势外，无服务器也能将开发人员从应用扩展和服务器置备相关的琐碎日常任务中解放出来。

使用无服务器时，[管理](https://www.redhat.com/zh/topics/management)操作系统和文件系统、[安全补丁](https://www.redhat.com/zh/topics/management/what-patch-management-and-automation)、负载平衡、容量管理、扩展、日志和监控等例行任务都由云服务提供商分担。

您可以构建完全无服务器化的应用，也可以打造包含部分无服务器、部分传统[微服务](https://www.redhat.com/zh/topics/microservices/what-are-microservices)组件的应用。

## 2. 云提供商在无服务器计算中有什么作用？

在无服务器模型中，云提供商负责运行物理服务器并代表用户动态分配资源，用户则可以直接将代码部署到生产环境中。

无服务器计算产品通常分为两类，分别是后端即服务（BaaS）和[功能即服务（FaaS）](https://www.redhat.com/zh/topics/cloud-native-apps/what-is-faas)。 

BaaS 可让开发人员访问各种各样的第三方服务和应用。例如，云提供商可以提供认证服务、额外加密、云访问数据库以及高置信度使用数据。在 BaaS 中，无服务器功能通常通过[应用编程接口（API）](https://www.redhat.com/zh/topics/api/what-are-application-programming-interfaces)调用。

在大多数情况下，当开发人员提到无服务器时，他们所指的基本是 FaaS 模型。在 FaaS 下，开发人员仍然要编写自定义服务器端逻辑，但它可以在完全由云服务提供商管理的容器中运行。 

主流公共云提供商全都拥有一种或多种 FaaS 产品。比如 [Amazon Web Services](https://www.redhat.com/zh/partners/amazon-web-services) 的 AWS Lambda、[Microsoft Azure](https://www.redhat.com/zh/partners/microsoft) 的 Azure Functions、[Google Cloud](https://www.redhat.com/zh/partners/google) 的多款产品，以及 [IBM Cloud](https://www.redhat.com/zh/partners/red-hat-and-ibm) 的 IBM Cloud Functions 等。 

一些组织选择使用开源服务器平台运行自己的 FaaS 环境，例如基于 [Kubernetes](https://www.redhat.com/zh/topics/containers/what-is-kubernetes) 的 [Knative](https://www.redhat.com/zh/topics/microservices/what-is-knative) 项目构建的 [红帽® OpenShift Serverless](https://www.redhat.com/zh/topics/microservices/why-choose-openshift-serverless)。

## 3. 什么是功能即服务 (FaaS)？

功能即服务（FaaS）是一种事件驱动计算执行模型；开发人员编写逻辑，部署到完全由平台管理的容器中，然后按需执行。

与 BaaS 不同，FaaS 可让开发人员拥有更大的掌控权力，他们可以创建自定义应用，而不依赖于包含预编写服务的库。 

代码则部署到云提供商管理的容器中。具体而言，这些容器具有以下特点：

- [无状态](https://www.redhat.com/zh/topics/cloud-native-apps/stateful-vs-stateless) - 让[数据集成](https://www.redhat.com/zh/topics/integration)变得更加简单。
- 寿命短 - 可以只运行非常短的时间。
- 由事件触发 - 可在需要时自动运行。
- 完全由云提供商管理；这样，您只用为所需的计算能力付费，而不必管"闲置"的应用和服务器。 

使用 FaaS 时，开发人员可以通过 API 调用无服务器应用，FaaS 提供商则通过 [API 网关](https://www.redhat.com/zh/topics/api/what-does-an-api-gateway-do)来处理 API。

>  [继续了解 FaaS](https://www.redhat.com/zh/topics/cloud-native-apps/what-is-faas)

## 4. 无服务器用例有哪些？

对于能够瞬时启动的异步、无状态应用，无服务器架构是十分理想的选择。同样，无服务器适合那些有不频繁、无法预知的激增需求的用例。

比如有一个批处理传入图像文件的任务，它的运行频率也许并不高，但时不时就会有大量图像一次性到达。或者例如监控数据库传入的更改，再应用一系列功能，比如对照质量标准检查更改或进行自动转换。

无服务器应用也适合涉及传入数据流、聊天机器人、计划任务或业务逻辑的用例。

其他一些常见的无服务器用例有后端 API 和 Web 应用、[业务流程自动化](https://www.redhat.com/zh/topics/automation/whats-business-automation)、无服务器网站，以及跨多系统集成。

## 5. 什么是 Knative 和无服务器 Kubernetes？

作为一种在[自动化基础架构](https://www.redhat.com/zh/topics/automation/whats-it-automation)中运行容器化应用的方式，Kubernetes 容器编排平台不出意料地成为了运行无服务器环境的热门选择。然而，Kubernetes 本身并不足以原生运行无服务器应用。

[Knative](https://www.redhat.com/zh/topics/microservices/what-is-knative) 是一个[开源](https://www.redhat.com/zh/topics/open-source/what-is-open-source)社区项目，可以添加组件，从而在 Kubernetes 上部署、运行和管理无服务器应用。

利用 Knative 无服务器环境，您可以将代码部署到 Kubernetes 平台，如[红帽 OpenShift](https://www.redhat.com/zh/technologies/cloud-computing/openshift)。借助 Knative，您可以将代码打包为容器镜像并交给系统，以此来创建相应的服务。您的代码仅在需要时才会运行，并由 Knative 来自动启动和停止实例。

Knative 主要由 3 个组件构成：

- **构建** - 一种灵活地将源代码构建到容器中的方法。
- **服务** - 通过请求驱动模型实现容器的快速部署和自动扩展，以根据需要为工作负载提供服务。
- **事件** - 用于使用和发起事件以触发应用的基础架构。应用可能由多种源触发，例如您自己应用的事件、来自多个提供商的[云服务](https://www.redhat.com/zh/topics/cloud-computing/what-are-cloud-services)、[软件即服务 (SaaS) ](https://www.redhat.com/zh/topics/cloud-computing/what-is-saas)系统，以及[红帽 AMQ](https://www.redhat.com/zh/technologies/jboss-middleware/amq) 流。

与早期的无服务器框架不同，Knative 能够部署任何现代应用工作负载，包括单体应用以及微服务和微小功能等。

作为由单一服务提供商控制的 FaaS 解决方案的替代选择，Knative 可以在运行 Kubernetes 的任何云平台中运行，包括在本地数据中心运行。这让组织在运行无服务器工作负载时拥有了更大的敏捷性和灵活性。

> [进一步了解 Knative](https://www.redhat.com/zh/topics/microservices/what-is-knative)

## 6. 无服务器计算的利弊

**优点**：

- 无服务器计算可以提高开发人员的工作效率，降低运营成本。通过摆脱诸如服务器置备和管理等例行任务，开发人员就会有更多的时间专注于自己的应用。 
- 无服务器有助于 [DevOps](https://www.redhat.com/zh/topics/devops) 的采用，可以减少开发人员明确描述基础架构（需要相应的置备操作）的需要。 
- 可以通过整合第三方 BaaS 产品的完整组件来进一步简化应用开发。 
- 在无服务器模型中，由于您只需为所需的云计算时间付费，而不用全程运行和管理自己的服务器，因此大大降低了运营成本。

**缺点**：

- 不运行自己的服务器或控制自己的服务器端逻辑也有弊端。
- 云提供商可能对其组件的交互方式有着严格的限制，从而影响您系统的灵活性和定制能力。采用 BaaS 环境时，开发人员可能要为代码不受其控制的服务负责。
- 放弃对 IT 堆栈这些方面的控制，也同时意味着您会受制于供应商技术锁定。即便您决定要更换提供商，也可能需要升级系统以符合新供应商的规范，而这无疑会增加成本。

## 7. 无服务器的发展

随着容器和按需云产品的普及，无服务器架构和 FaaS 也发展起来。根据红帽和 [451 Research 携手开展的调研](https://www.redhat.com/zh/resources/451-research-red-hat-openshift-serverless)，无服务器的发展可以追溯为三个阶段。 

无服务器的"1.0"阶段存在诸多局限，对于常规计算而言不甚理想。无服务器 1.0 具有以下特征：

- HTTP 和少许其他代码
- 仅限功能
- 有限执行时间（5-10 分钟）
- 无编排
- 有限本地开发体验

Kubernetes 在"无服务器 1.5"时代面世，此时许多无服务器框架都开始自动缩放容器。无服务器 1.5 具有以下特征：

- Knative
- 基于 Kubernetes 的自动扩展
- 微服务和功能
- 可在本地轻松调试和测试
- 多语言并可移植

如今，"无服务器 2.0"方兴正艾，增加了集成和状态。提供商已开始逐渐添加缺少的部件，使无服务器适合一般用途的业务工作负载。无服务器 2.0 具有以下特征：

- 基本状态处理
- 采用企业级启程模式
- 高级消息传递功能
- 与企业级 PaaS 混合
- 企业就绪型事件来源
- 状态与集成