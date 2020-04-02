---
title: 使用Rancher启动Kubernetes
---

您可以让Rancher使用您想要的任何节点启动Kubernetes集群。 当Rancher将Kubernetes部署到这些节点时，它使用[Rancher Kubernetes Engine]({{<baseurl>}}/rke/latest/en/)(RKE)，这是Rancher自己的轻量级Kubernetes安装程序。 它可以在任何计算机上启动Kubernetes，包括:

- 裸金属服务器
- 本地虚拟机
- 由云服务商托管的虚拟机

Rancher可以在现有节点上安装Kubernetes，也可以在云服务商中动态配置节点并在其上安装Kubernetes。

RKE集群包括Rancher在Windows节点或其他现有自定义节点上启动的集群，以及Rancher在Azure、Digital Ocean、EC2或vSphere上使用新节点启动的集群。

#### 要求

如果您使用RAKE设置集群，您的节点必须满足下游用户集群中节点的[要求](/docs/cluster-provisioning/node-requirements)。

#### 在云服务商的新节点上启动Kubernetes

使用Rancher，您可以基于[节点模板](/docs/cluster-provisioning/rke-clusters/node-pools/#node-templates)创建节点池。 此节点模板定义要用于在云提供商中启动节点的参数。

在云提供商托管的节点池上安装Kubernetes的一个好处是，如果节点与集群失去连接，Rancher可以自动创建另一个节点加入集群，以确保节点池的计数符合预期。

有关更多信息，请参阅[在新节点上启动Kubernetes。](/docs/cluster-provisioning/rke-clusters/node-pools/)

#### 在现有的自定义节点上启动Kubernetes

在这种情况下，您希望在裸金属服务器、内部部署虚拟机或云提供商中已存在的虚拟机上安装Kubernetes。 使用此选项，您将在计算机上运行Rancher agent Docker容器。

如果要重复使用以前的自定义集群中的节点，请在再次在集群中使用之前[清理节点](/docs/admin-settings/removing-rancher/rancher-cluster-nodes/)。 如果重复使用尚未清理的节点，则集群设置可能会失败。

有关更多信息，请参阅[自定义节点。](/docs/cluster-provisioning/rke-clusters/custom-nodes/)
