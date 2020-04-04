---
标题: 在现有的自定义节点上启动Kubernetes
---

当您创建自定义集群时，Rancher使用RKE（Rancher Kubernetes Engine）在本地裸金属服务器、本地虚拟机或云提供商托管的任何节点中创建Kubernetes集群。

要使用此选项，您需要访问将要在Kubernetes集群中使用的服务器。根据[要求](/docs/cluster-provisioning/node-requirements)配置每个服务器，其中包括一些硬件规格和Docker。 在每台服务器上安装Docker后，运行Rancher UI中提供的命令，将每台服务器转换为Kubernetes节点。

本节介绍如何设置自定义集群。

## 创建具有自定义节点的集群

> **想要使用Windows主机作为Kubernetes workers?**
>
> 在开始之前，请参阅[为Windows配置自定义集群](/docs/cluster-provisioning/rke-clusters/windows-clusters/)。

<!-- TOC -->

- [1. 设置Linux主机](#1-provision-a-linux-host)
- [2. 创建自定义集群](#2-create-the-custom-cluster)
- [3. 仅限Amazon：标签资源](#3-amazon-only-tag-resources)

<!-- /TOC -->

#### 1. 设置Linux主机

通过配置Linux主机开始创建自定义集群。 您的主机可以成为:

- 云主机虚拟机 (VM)
- 内部部署VM
- 裸金属服务器

如果要重复使用以前的自定义集群中的节点，再次在集群中使用之前请[清理节点](/docs/admin-settings/removing-rancher/rancher-cluster-nodes/)。 如果重复使用尚未清理的节点，则集群设置可能会失败。

根据[安装要求](/docs/cluster-provisioning/node-requirements)和[生产就绪集群的核对清单配置主机。](/docs/cluster-provisioning/production)

#### 2. 创建自定义集群

1. 在**集群**页面中，单击**添加集群**。

2. 选择**Custom**.

3. 输入**集群名称**.

4. {{< step_create-cluster_member-roles >}}

5. {{< step_create-cluster_cluster-options >}}

   > **使用Windows主机作为Kubernetes workers?**
   >
   > - 请参阅[启用Windows支持选项](/docs/cluster-provisioning/rke-clusters/windows-clusters/#enable-the-windows-support-option).
   > - 唯一可用于支持Windows的集群的网络插件是Flannel。 请参阅[网络选项](/docs/cluster-provisioning/rke-clusters/windows-clusters/#networking-option)

6. <a id="step-6"></a>点击 **下一步**.

7. 从**节点角色**中，选择要集群节点角色。

   > **注意:**
   >
   > - 使用Windows主机作为Kubernetes workers? 请参阅[节点配置](/docs/cluster-provisioning/rke-clusters/windows-clusters/#node-configuration).
   > - 裸金属服务器提醒：如果您计划将裸金属服务器专用于每个角色，则必须为每个角色配置裸金属服务器（即配置多个裸金属服务器）。

8. <a id="step-8"></a>**可选**: 点击 **[显示高级选项](/docs/admin-settings/agent-options/)** 以指定注册节点时要使用的IP地址、重写节点的主机名或添加[标签](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) 或[taints](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)到节点上.

9. 复制屏幕上显示的命令到剪贴板。

10. 使用您首选的shell登录到您的Linux主机，如PuTTy或远程终端连接。 运行复制到剪贴板的命令。

    > **注意:** 如果要将特定主机专用于特定节点角色，请重复步骤7-10。 根据需要多次重复这些步骤。

11. 当您完成在Linux主机上运行该命令时, 点击 **完成**.

{{< result_create-cluster >}}

#### 3. 仅限Amazon：标签资源

如果您已将集群配置为使用Amazon作为**云提供商**，请使用ClusterID标记您的AWS资源。

[Amazon文档: 标记您的 Amazon EC2 资源](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/Using_Tags.html)

> **注意:** 您无需在Kubernetes中配置云提供商即可使用Amazon EC2实例。 如果您想使用特定的Kubernetes云提供商功能，则只需配置云提供商。 有关更多信息，请参阅[Kubernetes云提供商](https://kubernetes.io/docs/concepts/cluster-administration/cloud-providers/)

以下资源需要标记为`ClusterID`:

- **节点**: 在Rancher中添加的所有主机。
- **子网**: 用于集群的子网
- **安全组**: 用于集群的安全组。

      	>**注意:** 不要标记多个安全组。 创建弹性负载均衡器时，标记多个组会产生错误。

应该使用的标签是:

```
Key=kubernetes.io/cluster/<CLUSTERID>, Value=owned
```

`<CLUSTERID>`可以是你选择的任何字符串。 但是，必须在您标记的每个资源上使用相同的字符串。 将标记值设置为`owned`会通知集群，使用`<CLUSTERID>`标记的所有资源都由该集群拥有和管理。

如果在集群之间共享资源，则可以将标记更改为:

```
Key=kubernetes.io/cluster/CLUSTERID, Value=shared
```

## 可选的后续步骤

创建集群后，您可以通过Rancher UI访问它。 作为最佳实践，我们建议设置以下备用方式来访问集群:

- **使用kubectl CLI访问您的集群:** 按照[这些steps](/docs/cluster-admin/cluster-access/kubectl/#accessing-clusters-with-kubectl-on-your-workstation)在您的工作站上使用kubectl访问集群。 在这种情况下，您将通过Rancher服务器的身份验证代理进行身份验证，然后Rancher将连接到下游集群。 此方法允许您在没有Rancher UI的情况下管理集群。
- **使用授权的集群端点使用kubectl CLI访问您的集群:** 请按照[以下步骤](/docs/cluster-admin/cluster-access/kubectl/#authenticating-direct-with-a-downstream-cluster)使用kubectl直接访问您的集群，无需通过Rancher进行身份验证。 我们建议设置此替代方法来访问您的集群，以便在您无法连接到Rancher的情况下，仍然可以访问集群。
