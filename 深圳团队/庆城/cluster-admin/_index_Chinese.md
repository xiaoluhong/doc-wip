---
标题: 集群管理
---

在Rancher中创建集群之后，您便可以开始使用强大的Kubernetes特性在开发、测试或生产环境中部署和扩展您的容器化应用程序。

本页面涵盖以下主题:

- [集群之间切换](#switching-between-clusters)
- [在Rancher中管理集群](#managing-clusters-in-rancher)
- [配置工具](#configuring-tools)

> 这部分内容会假设您基本熟悉Docker和Kubernetes。有关Kubernetes组件是如何协同工作的简要说明，请参阅 [概念](/docs/overview/concepts) 页面.

### 集群之间切换

要在集群之间切换，可以使用导航栏中的下拉菜单。

或者，您可以在导航栏中直接在项目和集群之间切换。打开**全局**视图，从主菜单中选择**集群**。然后选择要打开集群的名称。

### 在Rancher中管理集群

当在Rancher中创建集群完成之后，集群所有者将需要管理这些集群

当在Rancher中 [创建集群](/docs/cluster-provisioning/) 完成之后, [集群所有者](/docs/admin-settings/rbac/cluster-project-roles/#cluster-roles) 将需要管理这些集群. 关于如何管理集群，有许多不同的功能选择。

| 功能                                                         | [Rancher启动的Kubernetes集群](/docs/cluster-provisioning/rke-clusters/) | [托管的Kubernetes集群](/docs/cluster-provisioning/hosted-kubernetes-clusters/) | [导入的Kubernetes集群](/docs/cluster-provisioning/imported-clusters) |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [使用kubectl和kubeconfig文件访问集群](/docs/cluster-admin/cluster-access/kubectl/) | \*                                                           | \*                                                           | \*                                                           |
| [添加集群成员](/docs/cluster-admin/cluster-access/cluster-members/) | \*                                                           | \*                                                           | \*                                                           |
| [编辑集群](/docs/cluster-admin/editing-clusters/)            | \*                                                           | \*                                                           | \*                                                           |
| [管理节点](/docs/cluster-admin/nodes)                        | \*                                                           | \*                                                           | \*                                                           |
| [管理持久卷和存储类](/docs/cluster-admin/volumes-and-storage/) | \*                                                           | \*                                                           | \*                                                           |
| [管理项目和命名空间](/docs/cluster-admin/projects-and-namespaces/) | \*                                                           | \*                                                           | \*                                                           |
| [配置工具](#configuring-tools)                               | \*                                                           | \*                                                           | \*                                                           |
| [克隆集群](/docs/cluster-admin/cloning-clusters/)            |                                                              | \*                                                           | \*                                                           |
| [证书轮换的能力](/docs/cluster-admin/certificate-rotation/)  | \*                                                           |                                                              |                                                              |
| [备份您的Kubernetes集群的能力](/docs/cluster-admin/backing-up-etcd/) | \*                                                           |                                                              |                                                              |
| [恢复和还原etcd的能力](/docs/cluster-admin/restoring-etcd/)  | \*                                                           |                                                              |                                                              |
| [当集群不再能从Rancher访问时，清理Kubernetes组件](/docs/cluster-admin/cleaning-cluster-nodes/) | \*                                                           |                                                              |                                                              |

### 配置工具

Rancher包含许多Kubernetes中没有的工具来帮助您的DevOps运作。Rancher可以与外部服务集成，以帮助您的集群更有效地运行。工具分为以下几类:

- 告警
- 通知
- 日志
- 监控

更多信息，请参见 [工具](/docs/cluster-admin/tools/)
