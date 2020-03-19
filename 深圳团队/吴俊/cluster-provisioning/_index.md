---
title: 在 Rancher 中设置 Kubernetes 集群
---

Rancher允许您通过Rancher UI(而不是更复杂的方案)创建集群, 从而简化了集群的创建. Rancher提供了启动集群的多种选项. 使用最适合您的用例的选项.

本节假设您基本熟悉Docker和Kubernetes. 有关Kubernetes组件如何协同工作的简要说明, 请参阅 [概念](/docs/overview/concepts) 页面.

有关Rancher服务器如何配置集群以及使用什么工具来配置集群的概念概述, 请参阅 [架构](/docs/overview/architecture/) 页面.

本节讨论以下主题:

<!-- TOC -->

- [在托管的Kubernetes提供商中设置集群](#setting-up-clusters-in-a-hosted-kubernetes-cluster)
- [使用Rancher启动Kubernetes](#launching-kubernetes-with-rancher)
  - [在基础设施提供商中启动Kubernetes和配置节点](#launching-kubernetes-and-provisioning-nodes-in-an-infrastructure-provider)
  - [在现有的自定义节点上启动Kubernetes](#launching-kubernetes-on-existing-custom-nodes)
- [导入现有集群](#importing-existing-cluster)
  <!-- /TOC -->

下表总结了每种群集类型的可用选项和设置:

| Rancher 功能   | RKE 启动 | 托管Kubernetes集群| 导入群集 |
| -------------------- | ------------ | ------------------------- | ---------------- |
| 管理成员角色  | ✓            | ✓                         | ✓                |
| 编辑群集选项  | ✓            |                           |
| 管理节点池    | ✓            |                           |

## 在托管的Kubernetes提供商中设置集群

在这个场景中, Rancher不提供Kubernetes, 因为它是由供应商安装的,例如Google Kubernetes引擎（GKE）,用于Kubernetes的Amazon 弹性容器服务或Azure Kubernetes服务.

如果您使用Kubernetes提供商(例如:谷歌GKE)， Rancher将与其云api集成, 允许您在Rancher UI中创建和管理托管群集的基于角色的访问控制.

有关更多信息, 请参阅[托管Kubernetes集群](/docs/cluster-provisioning/hosted-kubernetes-clusters)一节.

## 使用Rancher启动Kubernetes

Rancher在您自己的节点上配置Kubernetes时,使用[Rancher Kubernetes Engine (RKE)]({{<baseurl>}}/rke/latest/en/)作为库时. RKE是Rancher自己的轻量级Kubernetes安装程序.

在RKE集群中, Rancher管理Kubernetes的部署. 这些集群可以部署在任何裸机服务器、云提供商或虚拟化平台上.

这些节点可以通过Rancher UI进行动态配置, Rancher UI 调用[Docker Machine](https://docs.docker.com/machine/)来启动各种云提供商上的节点.

如果已经有了要添加到RKE集群中的节点, 则可以通过在其上运行Rancher代理容器将其添加到集群中.

有关详细信息, 请参阅有关 [RKE 群集](/docs/cluster-provisioning/rke-clusters/)的部分.

#### 在基础设施提供商中启动Kubernetes和配置节点

Rancher可以在Amazon EC2、DigitalOcean、Azure或vSphere等基础设施提供商中动态地提供节点,然后在这些节点上安装Kubernetes .

使用Rancher, 您可以基于[节点模板](/docs/cluster-provisioning/rke-clusters/node-pools/#node-templates)创建节点池. 此模板定义用于启动云提供商中的节点的参数.

使用基础设施提供者托管的节点的一个好处是, 如果节点失去与集群的连接, Rancher可以自动替换它, 从而维护预期的集群配置.

创建一个可用的云提供商节点模板是由Rancher UI中活动的[节点驱动](/docs/cluster-provisioning/rke-clusters/node-pools/#node-drivers)决定的.

有关更多信息, 请参阅关于 [由设备提供商托管的节点](/docs/cluster-provisioning/rke-clusters/node-pools/)

#### 在现有的自定义节点上启动Kubernetes

在设置这种类型的集群时, Rancher会在现有的[自定义节点](/docs/cluster-provisioning/rke-clusters/custom-nodes/)上安装Kubernetes, 这将创建一个自定义集群.

您可以使用任何节点, 在Rancher中创建一个集群.

这些节点包括本地裸机服务器、云托管虚拟机或本地虚拟机.

## 导入现有集群

在这种类型的集群中,Rancher连接到一个已经建立的Kubernetes集群. 因此, Rancher不提供Kubernetes, 而只设置Rancher代理来与集群通信.

请注意, Rancher不会自动配置、扩展或升级导入的集群. 所有其他Rancher特性, 包括集群管理、策略和工作负载, 都可用于导入的集群.

有关更多信息，请参阅[导入现有群集](/docs/cluster-provisioning/imported-clusters/)一节
