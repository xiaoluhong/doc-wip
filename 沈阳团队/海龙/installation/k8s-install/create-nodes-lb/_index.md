---
title: '1. 创建节点和负载均衡器'
---

使用您选择的基础架构提供商为RKE安装设置三个节点和一个负载均衡器端点。

> **注意：** 这些节点必须位于相同的区域/数据中心中，您可以将这些服务器放在单独的可用区中。

#### 操作系统，Docker，硬件和网络的要求

确保您的节点满足[安装要求。](/docs/installation/requirements/)

在[RKE要求]({{<baseurl>}}/rke/latest/en/os/)中查看RKE的操作系统要求。

#### 负载均衡器

RKE将在每个节点上配置一个Ingress控制器pod。 Ingress控制器pod绑定到主机网络上的TCP/80和TCP/443端口，并且是到Rancher server的HTTPS流量的入口点。

将负载均衡器配置为基本的4层TCP转发器。确切的配置将根据您的环境而有所不同。

> **重要提示：**
> 安装后，请不要使用这个负载均衡器（即`local`集群Ingress）对Rancher以外的应用程序进行负载均衡。与其他应用程序共享这个Ingress可能会在其他应用的Ingress配置重新加载后导致Rancher出现websocket错误。我们建议将`local`集群专用于Rancher，不要使用其他应用程序。

##### 入门指南

- 有关如何设置NGINX负载均衡器的示例，请参阅[此页面。](/docs/installation/k8s-install/create-nodes-lb/nginx/)
- 有关显示如何设置Amazon NLB负载均衡器的示例，请参阅[此页面。](/docs/installation/k8s-install/create-nodes-lb/nlb/)

#### [下一步：使用RKE安装Kubernetes](/docs/installation/k8s-install/kubernetes-rke/)
