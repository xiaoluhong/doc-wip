---
标题: '3. 暴露您的服务'
---

在测试环境中，通常需要使用未发布的IP和端口号将外部流量路由到集群容器，以便用户能够访问其应用程序。您可以使用端口映射来实现此目标，只要您知道节点IP地址，该映射就会通过特定端口暴露工作负载（即服务）。 您可以使用HostPorts（在单个节点的指定端口上公开服务）或NodePorts（在单个端口的_所有_节点上公开服务）映射端口。

使用本文档修改在`output.txt`中列出`ports`的工作负载。 您可以通过设置HostPort或NodePort来修改它。

<figcaption>Resolve <code>ports</code> for the <code>web</code> Workload</figcaption>

![解析端口](/img/rancher/resolve-ports.png)

### In This Document

<!-- TOC -->

- [在Rancher v2.x中暴露服务有何不同？](#what-s-different-about-exposing-services-in-rancher-v2-x)
- [HostPorts](#hostport)
- [设置HostPort](#setting-hostport)
- [NodePorts](#nodeport)
- [设置NodePort](#setting-nodeport)

<!-- /TOC -->

### 在Rancher v2.x中公开服务有何不同?

在Rancher v1.6中，我们使用了_Port Mapping_来公开您和您的用户可以访问服务的IP地址和端口。

在Rancher v2.x中，暴露服务的机制和术语已更改和扩展。 现在，您有两个端口映射选项：_HostPorts_（与v1.6端口映射最相似，允许您将应用程序暴露在单个IP和端口上）和_NodePorts_（允许您在集群_所有_节点上映射端口， 不只是一个）。

可惜的是，迁移工具CLI无法解析端口映射。 如果要从v1.6迁移到v2.x的服务有端口映射，则必须设置[HostPort]（＃hostport）或[NodePort]（＃nodeport）作为替代。

### HostPort

_HostPort_是在运行一个或多个Pod的_specific node_上向公众暴露的端口。 到节点和公开端口（`<HOST_IP>：<HOSTPORT>`）的流量将路由到请求的容器的专用端口。 在Rancher v2.x中为Kubernetes Pod使用HostPort与在Rancher v1.6中为容器创建公共端口映射同义。

在下图中，用户试图访问Nginx实例，该实例在端口80的Pod中运行。但是，Nginx部署的HostPort为9890。用户可以通过浏览其主机IP地址加上正在使用的HostPort来连接到此Pod（当前例子是9890端口）。

![HostPort图解](/img/rancher/hostPort.svg)

##### HostPort 优点

- 主机上可用的任何端口都可以暴露。
- 配置很简单，并且可以在Kubernetes pod规范中直接设置HostPort。 与NodePort不同，无需创建其他对象即可公开您的应用程序。

##### HostPort 缺点

- 限制Pod的调度选项，仅能使用所选端口处于空闲状态的主机。
- 如果工作负载规模大于Kubernetes集群中的节点数，则部署将失败。
- 指定相同HostPort的任何两个工作负载无法部署到同一节点。
- 如果您的Pod运行所在的主机不可用，Kubernetes会将Pod重新调度到其他节点。 因此，如果您的工作负载的IP地址发生更改，则应用程序的外部客户端将失去对Pod的访问权限。 当您重新启动Pod-Kubernetes将其重新调度到另一个节点时也会发生同样的事情

### 设置 HostPort

您可以使用Rancher v2.x UI为已迁移的工作负载（即服务）设置HostPort。 要添加HostPort，请浏览到包含您的工作负载的项目，然后编辑要公开的每个工作负载，如下所示。 将服务容器公开的端口映射到目标节点上公开的HostPort。

例如，对于我们一直用作示例的从v1.6解析的 web-deployment.yml 文件，我们将编辑其Kubernetes清单，设置发布容器使用的端口，然后声明一个监听HostPort的端口。 所选择的端口(`9890`)，如下所示。 然后，您可以通过单击Rancher UI中创建的链接来访问工作负载。

<figcaption>Port Mapping: Setting HostPort</figcaption>

![设置HostPort](/img/rancher/set-hostport.gif)

### NodePort

_NodePort_是向_每一个_集群节点开放的端口。 当NodePort收到针对设置NodePort值的任何集群主机IP地址的请求时，NodePort（这是Kubernetes服务）会将流量路由到特定的Pod，而不管它在哪个节点上运行。 NodePort提供了一个静态端点，外部请求可以可靠地到达您的Pod。

NodePorts可帮助您规避IP地址的缺点。 尽管可以通过其IP地址访问Pod，但它们本质上是一次性的。 pod通常会被销毁并重新创建，每次复制都会获得一个新的IP地址。 因此，IP地址不是访问Pod的可靠方法。 NodePorts通过提供始终可以访问它们的静态服务来帮助您解决此问题。 即使您的Pod更改了其IP地址，依赖于它们的外部客户端也可以继续访问它们而不会受到干扰，所有这些都不会导致后端发生Pod的重新创建。

在下图中，用户试图连接到Rancher管理的Kubernetes集群中运行的Nginx实例。 尽管他知道NodePort Nginx正在运行（在这种情况下为30216），但他不知道Pod在其上运行的特定节点的IP地址。 但是，启用NodePort后，他可以使用集群中_任何_节点的IP地址连接到Pod。 Kubeproxy会将请求转发到正确的节点和Pod。

![NodePort图解](/img/rancher/nodePort.svg)

NodePort在内部IP的Kubernetes集群中可用。 如果要在集群外部公开Pod，请结合使用NodePort和外部负载均衡器。来自集群外部对 `<NodeIP>:<NodePort>` 的流量请求将定向到工作负载。`<NodeIP>` 可以是Kubernetes集群中任何节点的IP地址。

##### NodePort 优点

- 创建NodePort服务为您的工作负载容器提供了一个静态的公共端点。 在那里，即使Pod已被破坏，Kubernetes也可以在集群中的任何位置部署工作负载，而无需更改公共端点。
- pod的规模不受集群中节点数量的限制。 NodePort允许将公共访问与Pod的数量和位置分离。

##### NodePort 缺点

- 使用NodePort时， `<NodeIP>:<NodePort>` 仍在您的Kubernetes集群中所有节点上保留, 即使工作负载从未部署到其他节点。
- 您只能在可配置范围内指定端口 (默认, 是 `30000-32767` 之间).
- 需要一个额外的Kubernetes对象（类型为NodePort的Kubernetes服务）来暴露您的工作负载。 因此，找到应用程序的暴露方式并不容易。

### 设置 NodePort

您可以使用Rancher v2.x UI为迁移的工作负载（即服务）设置NodePort。 要添加NodePort，请浏览到包含您的工作负载的项目，然后编辑要公开的每个工作负载，如下所示。 将服务容器公开的端口映射到NodePort，您可以通过每个集群节点访问该端口。

例如，对于从我们一直用作示例的v1.6解析的`web-deployment.yml`文件，我们将编辑其Kubernetes清单，设置发布容器使用的端口，然后声明一个NodePort。 然后，您可以通过单击Rancher UI中创建的链接来访问工作负载。

> **注意:**
>
> - 如果设置NodePort时未为其提供值，则Rancher将从以下范围内随机选择一个端口： `30000-32767`。
> - 如果您手动设置NodePort，则端口必须在 `30000-32767` 之间。

<figcaption>Port Mapping: Setting NodePort</figcaption>

![设置NodePort](/img/rancher/set-nodeport.gif)

#### [下一步: 配置运行状况健康检查](/docs/v1.6-migration/monitor-apps)
