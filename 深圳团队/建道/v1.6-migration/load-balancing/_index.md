---
标题: '7. 负载均衡'
---

如果你的应用程序是面向公众的并且消耗大量流量，则应在集群之前放置一个负载平衡器，以便用户始终可以访问其应用程序而不会中断服务。 通常，你可以通过 [水平扩展](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) 你的deployment来满足大量服务请求，随着流量的增加它会拉长其应用程序容器。但是，此技术需要路由，以有效地在你的节点之间分配流量。 如果你需要适应可扩展的公共流量，则需要一个负载平衡器。

如概述 [它的文档]({{< baseurl >}}/rancher/v1.6/en/cattle/adding-load-balancers/),Rancher v1.6使用由HAProxy支持的自己的微服务为负载平衡提供了丰富的支持，该服务支持HTTP，HTTPS，TCP主机名和基于路径的路由。 v2.x中提供了大多数这些相同的功能。 但是，你于v1.6一起使用的负载平衡器无法迁移到v2.x。 你必须在v2.x中手动重新创建v1.6负载均衡器。

如果在将v1.6 Compose文件解析为Kubernetes清单后遇到下面的`output.txt`文本，则必须通过在v2.x中手动创建负载均衡器来解决它。

<figcaption><code>output.txt</code> Load Balancer Directive</figcaption>

![解决负载均衡器指令](/img/rancher/resolve-load-balancer.png)

### 在该文档

<!-- TOC -->

- [负载均衡器协议选项](#load-balancing-protocol-options)
- [负载均衡器部署](#load-balancer-deployment)
- [负载均衡器架构](#load-balancing-architecture)
- [Ingress警告](#ingress-caveats)
- [部署Ingress](#deploying-ingress)
- [Rancher v2.x负载均衡器限制](#rancher-v2-x-load-balancing-limitations)

<!-- /TOC -->

### 负载均衡器协议选项

默认情况下，Rancher v2.x用本地 [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) 替换了v1.6负载均衡器微服务，该服务由NGINX Ingress Controller支持 用于第7层负载均衡。 默认情况下，Kubernetes Ingress仅支持HTTP和HTTPS协议，不支持TCP。 使用Ingress时，负载均衡仅限于这两种协议。

> **TCP 要求?** 查看 [TCP 负载均衡选项](#tcp-load-balancing-options)

### 负载均衡器部署

在Rancher v1.6中，你可以添加端口/服务规则，配置HAProxy以实现目标服务的负载均衡。你还可以配置基于主机名/路径的路由规则。

Rancher v2.x提供了类似的功能，但是负载均衡由Ingress处理。 Ingress是控制器组件应用于负载均衡器规则的规范。实际的负载均衡器可以在集群之外或集群中运行。

默认情况下，Rancher v2.x在使用RKE（Rancher自己的Kubernetes安装程序）配置的集群上部署NGINX Ingress Controller，以处理Kubernetes Ingress规则。 默认情况下，NGINX Ingress Controller仅安装在RKE配置的集群中。 由云服务提供商（如GKE）配置的集群具有自己的入口控制器，用于配置负载均衡器。 对于本文档，我们的范围仅限于RKE安装的NGINX Ingress Controller。

RKE将NGINX Ingress Controller部署为 [Kubernetes守护程序](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/), 这意味着NGINX实例已部署在集群中的每个节点上。 NGINX就像一个Ingress Controller一样，在整个集群中侦听Ingress的创建，并且还将其自身配置为负载均衡器，以满足Ingress规则。 守护程序使用主机网络配置公开两个端口：80和443。

有关更多信息，NGINX Ingress Controller，它们的deployment作为守护程序，deployment配置选项， 查看 [RKE 文档]({{< baseurl >}}/rke/latest/en/config-options/add-ons/ingress-controllers/).

### 负载均衡器架构

将Ingress Controller作为守护程序部署在v2.x中，带来了v1.6用户应该了解的一些体系结构更改。

在Rancher v1.6中，你可以在堆栈中部署可伸缩的负载均衡器服务。 如果你在Cattle环境中有四个主机，则可以部署规模为2的负载均衡器服务，并通过将端口80附加到两个主机IP地址来指向你的应用程序。 你还可以在其余两台主机上启动另一个负载均衡器，以再次使用端口80均衡另一服务，因为你的负载均衡器正在使用不同的主机IP地址。

<!-- add comparison table-->

<figcaption>Rancher v1.6 Load Balancing Architecture</figcaption>

![Rancher v1.6 负载均衡](/img/rancher/cattle-load-balancer.svg)

Rancher v2.x Ingress Controller是一个守护程序，它全局部署在所有可调度节点上，以为整个Kubernetes集群服务。 因此，在对Ingress规则进行编程时，你必须使用唯一的主机名和路径来指向你的工作负载，因为负载均衡器节点IP地址以及端口80和443是所有工作负载的通用访问点。

<figcaption>Rancher v2.x Load Balancing Architecture</figcaption>

![Rancher v2.x 负载均衡](/img/rancher/kubernetes-load-balancer.svg)

### Ingress警告

尽管Rancher v2.x支持HTTP和HTTPS主机名以及基于路径的负载均衡，但是在配置工作负载时必须使用唯一的主机名和路径。 此限制源自：

- 将入口限制为端口80和443 (即端口 HTTP[S] 用于路由).
- 负载均衡器和Ingress Controller作为守护程序在集群中全局启动。

> **TCP 需求?** Rancher v2.x 仍然支持 TCP. 查看 [TCP负载均衡选项](#tcp-load-balancing-options) 解决方法。

### 部署Ingress

你可以启动新的负载均衡器以从v1.6替换你的负载均衡器。 使用Rancher v2.x UI，浏览到适用的项目并选择 **资源 > 工作负载 > 负载均衡.** (在v2.3.0之前的版本中，单击 **工作负载 > 负载均衡.**) 然后点击 **部署**. 在部署期间，你可以选择目标项目或名称空间。

> **前提条件:** 在部署Ingress之前，你必须部署的工作负载正在运行两个或更多pods的规模。

![工作负载规格]](/img/rancher/workload-scale.png)

为了在这两个pods之间保持平衡，你必须创建一个Kubernetes Ingress规则。 要创建此规则，请导航到你的集群和项目，然后单击**资源>工作负载>负载平衡。**（在v2.3.0之前的版本中，单击**工作负载>负载平衡**）然后单击**添加 入口**。 下面的GIF描述了如何将Ingress添加到你的一个项目中。

<figcaption>Browsing to Load Balancer Tab and Adding Ingress</figcaption>

![添加入口](/img/rancher/add-ingress.gif)

与Rancher v1.6中的服务/端口规则类似，你可以在此处指定针对工作负载的容器端口规则。 以下各节说明了如何创建Ingress规则。

#### 配置基于主机和基于路径的路由

使用Rancher v2.x，你可以添加基于主机名或URL路径的Ingress规则。 根据你创建的规则，NGINX Ingress Controller将流量路由到多个目标工作负载或Kubernetes服务。

例如，假设你有多个工作负载部署到一个名称空间。 你可以添加一个Ingress，以使用相同的主机名但使用不同的路径将流量路由到这两个工作负载，如下图所示。 对 `foo.com/name.html`的URL请求会将用户定向到`web`工作负载，而对`foo.com/login`的URL请求将用户定向到`chat`工作量。

<figcaption>Ingress: Path-Based Routing Configuration</figcaption>

![入口：基于路径的路由配置](/img/rancher/add-ingress-form.png)

Rancher v2.x还在Ingress记录上提供了指向工作负载的便捷链接。 如果你配置外部DNS以对DNS记录进行记录，则可以将该主机名映射到Kubernetes入口地址。

<figcaption>工作负载链接</figcaption>

![负载均衡器链接到工作负载](/img/rancher/load-balancer-links.png)

Ingress地址是Ingress Controller为你的工作负载分配的集群中的IP地址。 你可以通过浏览到该IP地址来处理你的工作负载。 使用下面的`kubectl`命令查看控制器分配的Ingress地址：

```
kubectl 获取 ingress
```

#### HTTPS /证书选项

Rancher v2.x Ingress功能支持HTTPS协议，但如果要使用它，则需要使用有效的SSL/TLS证书。在配置Ingress规则时，请使用**SSL/TLS 证书**部分来配置证书。

- 我们推荐 [上传证书](/docs/k8s-in-rancher/certificates/) 从已知的证书颁发机构（必须在配置Ingress之前执行此操作）。 然后，在配置负载均衡器时，使用**选择一个证书**选项，然后选择要使用的上载证书。
- 如果已配置 [NGINX默认证书]({{< baseurl >}}/rke/latest/en/config-options/add-ons/ingress-controllers/#configuring-an-nginx-default-certificate), 你能选择 **使用默认的ingress controller证书**.

<figcaption>Load Balancer Configuration: SSL/TLS Certificate Section</figcaption>

![SSL/TLS 证书部分](/img/rancher/load-balancer-ssl-certs.png)

#### TCP负载均衡选项

##### 四层负载均衡器

对于TCP协议，Rancher v2.x支持使用其中部署了Kubernetes集群的云提供商配置第4层负载均衡器。 一旦为集群配置了此负载均衡器设备，当你在工作负载部署期间选择`Layer-4 Load Balancer`选项进行端口映射时，Rancher会自动创建相应的负载均衡器服务。 该服务将呼叫相应的云提供商，配置负载平衡器设备以将请求路由到适当的pod。 有关如何为你的云提供商配置LoadBalancer服务的信息，查看 [云提供商](/docs/cluster-provisioning/rke-clusters/options/cloud-providers/)

例如，如果我们创建名为`myapp`的部署并在**端口映射**部分中指定第4层负载均衡器，则Rancher会自动将一个条目添加到名为`myapp-loadbalancer`的**负载均衡器**选项卡中。

<figcaption>Workload Deployment: Layer 4 Load Balancer Creation</figcaption>

![部署第4层负载均衡器](/img/rancher/deploy-workload-load-balancer.png)

负载均衡器的配置成功后，Rancher UI将提供指向工作负载的公共端点的链接。

##### ConfigMaps的NGINX Ingress Controller TCP支持

尽管NGINX支持TCP，但Kubernetes Ingress本身不支持TCP协议。 因此，无法进行NGINX Ingress Controller的现成配置来实现TCP均衡。

然而，有一种解决方法可以通过创建Kubernetes ConfigMap来使NGINX的TCP均衡，如 [Ingress GitHub自述文件](https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/exposing-tcp-udp-services.md). 你可以创建一个ConfigMap对象，该对象将pod配置参数存储为键值对，与pod镜像分开，如 [Kubernetes文档](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/).

要配置NGINX通过TCP公开你的服务，你可以添加`ingress-nginx`命名空间中应存在的ConfigMap `tcp-services`。 该命名空间还包含NGINX Ingress Controller容器。

![第4层负载平衡器：ConfigMap解决方法](/img/rancher/layer-4-lb-config-map.png)

ConfigMap条目中的密钥应该是要公开访问的TCP端口：`<namespace/service name>:<service port>`。 如上所示，在`Default`命名空间中列出了两个工作负载。 例如，上面的ConfigMap中的第一个条目指示NGINX通过外部端口`6790`暴露`myapp`工作负载（监听私有端口80的`default`名称空间中的工作负载）。 将这些条目添加到ConfigMap会自动更新NGINX容器，以配置这些工作负载以实现TCP均衡。 暴露的工作负载应在`<NodeIP>:<TCP Port>`中可用。 如果无法访问它们，则可能必须使用NodePort服务显式公开TCP端口。

### Rancher v2.x负载均衡限制

Cattle提供了功能丰富的负载均衡器支持，即 [well documented]({{< baseurl >}}/rancher/v1.6/en/cattle/adding-load-balancers/#load-balancers). 其中一些功能在Rancher v2.x中没有等效功能。 以下是这些功能的列表：

- 当前的NGINX Ingress Controller不支持SNI。
- TCP负载均衡需要集群中的云提供商启用的负载均衡器设备。 Kubernetes上没有对TCP的Ingress支持。
- 只能将端口80和443配置为通过Ingress进行HTTP/HTTPS路由。Ingress Controller也作为守护程序全局部署，而不作为可伸缩服务启动。 此外，用户不能分配随机的外部端口用于均衡。 因此，用户需要确保配置唯一的主机名/路径组合，以避免使用相同的两个端口而路由冲突。
- 无法指定端口规则优先级和顺序。
- Rancher v1.6添加了对耗尽后端连接和指定耗尽超时的支持。 Rancher v2.x不支持此功能。
- 从现在开始，Rancher v2.x中不支持指定自定义粘性策略和要附加到默认配置的自定义负载均衡器配置。但是，本机Kubernetes中提供了一些支持，用于自定义NGINX配置，如 [NGINX Ingress Controller自定义配置文档](https://kubernetes.github.io/ingress-nginx/examples/customization/custom-configuration/).

#### 已完成!
