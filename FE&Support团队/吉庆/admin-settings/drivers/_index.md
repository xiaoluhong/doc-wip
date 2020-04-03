---
标题：预配驱动程序
---

使用Rancher中的驱动程序，您可以管理可以使用哪些提供程序来部署[托管的Kubernetes群集](/docs/cluster-provisioning/hosted-kubernetes-clusters/)或[基础架构提供程序中的节点](/docs/cluster-provisioning/rke-clusters/node-pools/)，以允许Rancher部署和管理Kubernetes。

#### Rancher驱动程序

使用Rancher驱动程序，您可以启用/禁用Rancher中打包的现有内置驱动程序。另外，如果Rancher尚未实现，则可以添加自己的驱动程序。

Rancher中有两种驱动程序：

- [群集驱动程序](＃cluster-drivers)
- [节点驱动程序](＃node-drivers)

### 群集驱动程序

_自v2.2.0起可用_

群集驱动程序用于配置[托管的Kubernetes群集](/docs/cluster-provisioning/hosted-kubernetes-clusters/)，例如GKE，EKS，AKS等。在创建群集时显示哪个群集驱动程序的可用性是根据群集驱动程序的状态定义的。将仅显示`活动`集群驱动程序作为为托管Kubernetes集群创建集群的选项。默认情况下，Rancher与几个现有的群集驱动程序打包在一起，但是您也可以创建自定义群集驱动程序以添加到Rancher。

默认情况下，Rancher已激活了多个托管的Kubernetes云提供商，包括：

- [Amazon EKS](/docs/cluster-provisioning/hosted-kubernetes-clusters/eks/)
- [Google GKE](/docs/cluster-provisioning/hosted-kubernetes-clusters/gke/)
- [Azure AKS](/docs/cluster-provisioning/hosted-kubernetes-clusters/aks/)

还有其他一些托管的Kubernetes云提供商默认情况下被禁用，但打包在Rancher中：

- [阿里巴巴ACK](/docs/cluster-provisioning/hosted-kubernetes-clusters/ack/)
- [华为CCE](/docs/cluster-provisioning/hosted-kubernetes-clusters/cce/)
- [腾讯TKE](/docs/cluster-provisioning/hosted-kubernetes-clusters/tke/)

### 节点驱动程序

节点驱动程序用于配置主机，Rancher用于启动和管理Kubernetes集群。节点驱动程序与[Docker Machine driver](https://docs.docker.com/machine/drivers/)相同。根据节点驱动程序的状态定义在创建节点模板时显示哪个节点驱动程序的可用性。将仅显示`活动`节点驱动程序作为创建节点模板的选项。默认情况下，Rancher与许多现有的Docker Machine驱动程序打包在一起，但是您也可以创建自定义节点驱动程序以添加到Rancher。

如果您不想向用户显示特定的节点驱动程序，则需要停用这些节点驱动程序。

Rancher支持几个主要的云提供程序，但是默认情况下，这些节点驱动程序是活动的并可用于部署：

- [Amazon EC2](/docs/cluster-provisioning/rke-clusters/node-pools/ec2/)
- [Azure](/docs/cluster-provisioning/rke-clusters/node-pools/azure/)
- [Digital Ocean](/docs/cluster-provisioning/rke-clusters/node-pools/digital-ocean/)
- [vSphere](/docs/cluster-provisioning/rke-clusters/node-pools/vsphere/)
