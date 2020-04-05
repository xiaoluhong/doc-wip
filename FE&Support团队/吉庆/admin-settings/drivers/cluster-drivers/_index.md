---
标题：群集驱动程序
---

_自v2.2.0起可用_

群集驱动程序用于在[托管的Kubernetes提供程序](/docs/cluster-provisioning/hosted-kubernetes-clusters/)(例如Google GKE)中创建群集。创建群集时要显示哪个群集驱动程序的可用性由群集驱动程序的状态定义。将仅显示`活动`集群驱动程序作为创建集群的选项。默认情况下，Rancher与几个现有的云提供程序群集驱动程序打包在一起，但是您也可以将自定义群集驱动程序添加到Rancher。

如果不想让特定的群集驱动程序显示给用户，则可以在Rancher中停用这些群集驱动程序，它们不会作为创建群集的选项出现。

#### 管理群集驱动程序

> **先决条件：** 要创建，编辑或删除群集驱动程序，您需要具有以下权限之一：
>
> - [管理员全局权限](/docs/admin-settings/rbac/global-permissions/)
> - [自定义全局权限](/docs/admin-settings/rbac/global-permissions/＃custom-global-permissions)与[管理群集驱动程序](/docs/admin-settings/rbac/global-permissions/＃全局权限参考)角色分配。

### 激活/停用集群驱动程序

默认情况下，Rancher仅激活最受欢迎的云提供商，Google GKE，Amazon EKS和Azure AKS的驱动程序。如果要显示或隐藏任何节点驱动程序，则可以更改其状态。

1. 从`全局`视图中，在导航栏中选择`工具>驱动程序`。

2. 在`驱动程序`页面上，选择`集群驱动程序`选项卡。

3. 选择要`激活`或`禁用`的驱动程序，然后选择适当的图标。

### 添加自定义群集驱动程序

如果要使用Rancher不支持即用型的群集驱动程序，则可以添加提供程序的驱动程序，以便开始使用它们来创建_hosted_ kubernetes群集。

1. 从`全局`视图中，在导航栏中选择`工具>驱动程序`。

2. 从`驱动程序`页面中，选择`集群驱动程序`选项卡。

3. 单击`添加群集驱动程序`。

4. 完成`添加群集驱动程序`表格。然后点击`创建`。

#### 开发自己的集群驱动程序

为了开发群集驱动程序以添加到Rancher，请参阅我们的[example](https://github.com/rancher-plugins/kontainer-engine-driver-example)。
