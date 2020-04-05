---
标题：节点驱动程序
---

节点驱动程序用于配置主机，Rancher用于启动和管理Kubernetes集群。节点驱动程序与[Docker Machine driver](https://docs.docker.com/machine/drivers/)相同。根据节点驱动程序的状态定义在创建节点模板时显示哪个节点驱动程序的可用性。将仅显示`活动`节点驱动程序作为创建节点模板的选项。默认情况下，Rancher与许多现有的Docker Machine驱动程序打包在一起，但是您也可以创建自定义节点驱动程序以添加到Rancher。

如果您不想向用户显示特定的节点驱动程序，则需要停用这些节点驱动程序。

##### 管理节点驱动程序

> **先决条件：** 要创建，编辑或删除驱动程序，您需要具有以下权限之一：
>
> - [管理员全局权限](/docs/admin-settings/rbac/global-permissions/)
> - 使用[管理节点驱动程序]的[自定义全局权限](/docs/admin-settings/rbac/global-permissions/＃custom-global-permissions)(/docs/admin-settings/rbac/global-permissions/＃全局权限参考)角色分配。

### 激活/停用节点驱动程序

默认情况下，Rancher仅激活最受欢迎的云提供商，Amazon EC2，Azure，DigitalOcean和vSphere的驱动程序。如果要显示或隐藏任何节点驱动程序，则可以更改其状态。

1. 从`全局`视图中，在导航栏中选择`工具>驱动程序`。在`驱动程序`页面上，选择`节点驱动程序`选项卡。在v2.2.0之前的版本中，您可以直接在导航栏中选择`节点驱动程序`。

2. 选择要`激活`或`停用`的驱动程序，然后选择适当的图标。

### 添加自定义节点驱动程序

如果要使用Rancher不支持即用型的节点驱动程序，则可以添加该提供程序的驱动程序，以便开始使用它们来为Kubernetes集群创建节点模板并最终创建节点池。

1. 从`全局`视图中，在导航栏中选择`工具>驱动程序`。在`驱动程序`页面上，选择`节点驱动程序`选项卡。在v2.2.0之前的版本中，您可以直接在导航栏中选择`节点驱动程序`。

2. 单击`添加节点驱动程序`。

3. 完成`添加节点驱动程序`表格。然后点击`创建`。

#### 开发自己的节点驱动程序

节点驱动程序是通过[Docker Machine](https://docs.docker.com/machine/)实现的。
