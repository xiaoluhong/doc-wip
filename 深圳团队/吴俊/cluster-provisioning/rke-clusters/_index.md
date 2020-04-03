---
title: Launching Kubernetes with Rancher
---

您可以让Rancher使用您想要的任何节点启动Kubernetes集群. 当Rancher将Kubernetes部署到这些节点上时, 它使用[Rancher Kubernetes Engine]({{<baseurl>}}/rke/latest/en/) (RKE), 这是Rancher自己的轻量级Kubernetes安装程序. 它可以在任何计算机上启动Kubernetes, 包括:

- 裸机服务器
- 本地虚拟机
- 由设备提供商托管的虚拟机

Rancher可以在现有节点上安装Kubernetes, 也可以在设备提供商中动态配置节点并在这些节点上安装Kubernetes.

RKE 集群包括 Rancher 在 Windows 节点或其他现有自定义节点上启动的集群, 以及 Rancher 在 Azure、Digital Ocean, EC2或 vSphere 上使用新节点启动的集群.

#### 要求

如果您使用RKE来设置集群,那么您的节点必须满足下游用户集群中的节点的[需求](/docs/cluster-provisioning/node-requirements).

#### 在设备提供商的新节点上启动Kubernetes

使用Rancher，您可以基于[节点模板](/docs/cluster-provisioning/rke-clusters/node-pools/#node-templates)创建节点池. 这个节点模板定义了您希望用来启动云提供商中的节点的参数.

在设备提供商托管的节点池上安装Kubernetes的一个好处是, 如果一个节点与集群失去连接, Rancher可以自动创建另一个节点加入集群, 以确保节点池的计数与预期一致.

有关更多信息,请参阅[在新节点上启动Kubernetes](/docs/cluster-provisioning/rke-clusters/node-pools/)一节

#### 在现有的自定义节点上启动Kubernetes

在这个场景中,您希望在裸机服务器、本地虚拟机或已经存在于云提供商中的虚拟机上安装Kubernetes. 使用此选项, 您将在机器上运行一个由Rancher代理Docker容器.

如果要重用以前自定义集群中的节点, 请先在集群中再次使用节点之前[清理该节点](/docs/admin-settings/removing-rancher/rancher-cluster-nodes/). 如果重用尚未清理的节点,集群配置可能会失败.

有关更多信息, 请参考[自定义节点](/docs/cluster-provisioning/rke-clusters/custom-nodes/)一节.
