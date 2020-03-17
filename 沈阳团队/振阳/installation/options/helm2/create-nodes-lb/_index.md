---
title: '1. 创建节点与负载均衡'
---

使用您选择的提供商为您的RKE安装提供3个节点和一个负载均衡器端点。

> **注意事项:** 这些节点必须位于相同的区域/数据中心。您可以将这些服务器放在单独的可用区中。

#### 节点要求

查看支持的操作系统和运行Rancher节点的硬件/软件/网络需求 [节点需求](/docs/installation/requirements)

查看RKE的操作系统需求 [RKE需求]({{<baseurl>}}/rke/latest/en/os/)

#### 负载均衡

RKE将在每个节点上配置一个Ingress controller pod。

Ingress controller pods被绑定到主机网络的TCP/80和TCP/443端口上，并且是到Rancher Server的HTTPS流量的入口点。

将负载均衡器配置为基本的4层TCP转发器。确切的配置将取决于您的环境。

> **重要:**
> 安装后，请勿使用此负载平衡器（即`local`群集Ingress）对Rancher以外的应用程序进行负载平衡。与其他应用程序共享此Ingress可能会在其他应用的Ingress配置重新加载后导致Rancher出现websocket错误。我们建议将`local`群集专用于Rancher，而不应使用其他任何应用程序。

##### 例子

- [Nginx](/docs/installation/options/helm2/create-nodes-lb/nginx/)
- [Amazon NLB](/docs/installation/options/helm2/create-nodes-lb/nlb/)

#### [下一步: 使用RKE安装Kubernetes](/docs/installation/options/helm2/kubernetes-rke/)
