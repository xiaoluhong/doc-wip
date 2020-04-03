---
title: 在vSphere中创建Kubernetes集群
---

本节说明了如何在Rancher中配置vSphere凭证，在vSphere中创建主机以及在这些主机上启动Kubernetes集群。

## 前提条件

本节介绍了使用vSphere在Rancher中创建主机和集群需要的必要条件。

该主机模板的文档使用vSphere Web Services API 6.5版本中进行了测试。

- [在vSphere中创建凭证](#在vSphere中创建凭证)
- [网络权限](#网络权限)
- [适用于vSphere API访问的有效ESXi许可证](#适用于vSphereAPI访问的有效ESXi许可证)

#### 在vSphere中创建凭证

在继续创建集群之前，必须确保您拥有具有足够权限的vSphere用户。 当您设置主机模板时，该模板将需要使用这些vSphere凭证。

有关如何在vSphere中创建具有所需权限的用户，请参考此[使用指南](/docs/cluster-provisioning/rke-clusters/node-pools/vsphere/provisioning-vsphere-clusters/creating-credentials)。 通过这些步骤您将创建出需要提供给Rancher的用户名和密码，从而允许Rancher在vSphere中创建资源。

#### 网络权限

必须确保运行Rancher的主机能够建立以下网络连接：

- 能够访问vCenter服务中的vSphere API(通常使用端口： 443/TCP)。
- 能够访问位于实例化虚拟机的所有ESXi主机上的Host API（端口 443/TCP）(_使用Rancher v2.3.3之前版本或在之后的版本中使用ISO创建_).
- 能够访问虚拟机的端口22/TCP和2376/TCP

请参照[主机网络需求](/docs/cluster-provisioning/node-requirements/#networking-requirements) 来获取详细的端口需求。

#### 适用于vSphereAPI访问的有效ESXi许可证

免费的ESXi许可证不支持API访问。 vSphere服务器必须具有效或评估的ESXi许可证。

## 在Rancher中使用vSphere创建集群

本节介绍了如何使用Rancher UI设置vSphere凭证，主机模板和vSphere集群。

您需要执行以下步骤：

1. [使用vSphere凭证创建主机模板](#1-使用vSphere凭证创建主机模板)
2. [使用主机模板创建Kubernetes集群](#2-使用主机模板创建Kubernetes集群)
3. [可选: 创建存储](#3-可选-创建存储)

- [为集群启用vSphere cloud provider](#为集群启用vSphere-cloud-provider)

#### Configuration References 配置参考

详细的主机模板配置, 请参照[主机模板配置参考](/docs/cluster-provisioning/rke-clusters/node-pools/vsphere/provisioning-vsphere-clusters/node-template-reference/)

Rancher使用RKE来创建Kubernetes集群。详细的vSphere中集群配置, 请参照[RKE文档中的集群配置参考]({{<baseurl>}}/rke/latest/en/config-options/cloud-providers/vsphere/config-reference/)

请注意，必须[启用](#enable-the-vsphere-cloud-provider-for-the-cluster)vSphere cloud provider才能够动态配置数据卷。

## 1. 使用vSphere凭证创建主机模板

为了创建集群，您至少创建一个vSphere[主机模板](/docs/cluster-provisioning/rke-clusters/node-pools/#node-templates) 来配置如何在vSphere创建虚拟机。
创建并保存的主机模板可在创建其他vSphere集群时重复使用它。

为了创建主机模板,

1. 在Rancher UI中通过管理员账号登录。

1. 在用户设定菜单中, 选择**主机模板**。

1. 点击**添加模板**然后点击**vSphere**图标。

然后配置您的主机模板:

- [A. 配置vSphere凭证](#a-配置vSphere凭证)
- [B. 配置主机调度](#b-配置主机调度)
- [C. 配置实例和操作系统](#c-配置实例和操作系统)
- [D. 添加网络](#d-添加网络)
- [E. 启用磁盘UUIDs](#e-启用磁盘UUIDs)
- [F. 可选: 配置主机标签和自定义属性](#f-可选-配置主机标签和自定义属性)
- [G. 可选: 配置cloud-init](#g-可选-配置cloud-init)
- [H. 保存主机模板](#h-保存主机模板)

#### A. 配置vSphere凭证

根据您的Rancher版本，为集群配置vSphere凭证的步骤有所不同。

 tabs 
 tab "Rancher v2.2.0+" 

您的账户访问信息保存在[云凭证。](/docs/user-settings/cloud-credentials/) 云凭证会保存为Kubernetes secrets。

您可以使用现有的云凭证或创建新的云凭证。要创建新的云凭证：

1. 点击**添加凭证**。
1. 在**名称**字段, 输入vSphere凭证的名称。
1. 在**vCenter or ESXi服务** 字段, 输入vCenter或ESXi主机名/IP。 ESXi是用于创建和运行虚拟机和虚拟设备的虚拟化平台。 vCenter Server是一项服务，通过它可以管理网络中连接的多个主机池中的主机资源。
1. 可选: 在**端口**字段，配置vCenter或ESXi服务的端口。
1. 在**用户名** 和 **密码** 字段, 输入您vSphere的用户名和密码。
1. 点击**创建**。

**结果:** 主机模板成功添加了vSphere的云凭证。

 /tab 
 tab "Rancher v2.2.0之前的版本" 
在 **账户访问** 选项中, 输入vCenter的FQDN或IP地址和vSphere用户账户的凭证。
 /tab 
 /tabs 

#### B. 配置主机调度

选择虚拟机将被调度到的虚拟机管理程序。 配置选项取决于您的Rancher版本。

 tabs 
 tab "Rancher v2.3.3+" 

**调度**部分中的字段会自动填充为数据中心和vSphere中可用的选项。

1. 在**数据中心** 选项中, 选择哪个数据中心用于虚拟机调度。
1. 可选: 选择**资源池。** 资源池可用于对独立主机或集群中的CPU和内存资源进行分区，也可以嵌套。
1. 如果您有数据存储集群，则可以切换**数据存储**选项。 这样，您可以选择将VM调度到哪个数据存储集群。 如果选择该选项，则可以选择单个磁盘。
1. 可选: 选择要放置虚拟机的文件夹。 此下拉菜单中的VM文件夹直接与vSphere中的VM文件夹相对应。 注意：文件夹名称在vSphere配置文件中应该以`vm/`开头。
1. 可选: 选择一个特定的主机用于创建VM。 对于独立ESXi或具有DRS（分布式资源调度程序）的集群，请将此字段设置为空。 如果指定了该字段，将使用主机系统的池，并且**资源池**参数将被忽略。
    /tab 
    tab "Rancher v2.3.3之前的版本" 

在**调度**选项卡中, 输入:

- 用于创建VM的**数据中心**的名称或路径。
- **虚拟机网络**的名称。
- 保存磁盘的**数据存储**名称或路径。

  ![image](/img/rancher/vsphere-node-template-2.png")

 /tab 
 /tabs 

#### C. 配置实例和操作系统

根据Rancher版本的不同，可以使用不同的选项来配置实例。

 tabs 
 tab "Rancher v2.3.3+" 

在**实例选项**选项卡中, 配置该主机模板创建主机的CPU数量，内存和磁盘大小。

在**Creation method**选项中, 配置在vSphere中创建VMs的创建方法。 可用选项包括通过RancherOS ISO创建VMs或通过一个已有的VM或[VM模板](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.vm_admin.doc/GUID-F7BF0E6B-7C4F-4E46-8BBF-76229AEA7220.html).克隆VMs。

已有的VM或VM模板可以使用配置了[cloud-init](https://cloudinit.readthedocs.io/en/latest/)并使用[NoCloud datasource](https://cloudinit.readthedocs.io/en/latest/topics/datasources/nocloud.html)的任何现代Linux操作系统。

选择创建虚拟机的方式：

- **Deploy from template: Data Center:** 选择一个数据中心中的VM模板。
- **Deploy from template: Content Library:** 首先选择包含您的模板的[Content Library](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.vm_admin.doc/GUID-254B2CE8-20A8-43F0-90E8-3F6776C2C896.html), 然后在`Library templates`选项中选择模板。
- **Clone an existing virtual machine:** 在**Virtual machine**选项中, 选择一个用于克隆的虚拟机。
- **Install from boot2docker ISO:** 确保`操作系统ISO下载地址`选项填写正确的VMware ISO或RancherOS (rancheros-vmware.iso)地址。 请注意这个URL必须在Rancher Server的主机中能够访问。

 /tab 
 tab "Rancher v2.3.3之前的版本" 

在**实例选项**选项卡中, 配置该主机模板创建主机的CPU数量，内存和磁盘大小。

仅支持从RancherOS ISO创建VM。

确保`操作系统ISO下载地址`选项填写正确的VMware ISO或RancherOS (rancheros-vmware.iso)地址。

    ![image](/img/rancher/vsphere-node-template-1.png)

 /tab 
 /tabs 

#### D. 添加网络

_从Rancher v2.3.3开始可用_

The node template now allows a VM to be provisioned with multiple networks. In the **Networks** field, you can now click **Add Network** to add any networks available to you in vSphere.

现在，主机模板允许VM配置多个网络。 在**网络**字段中，您可以点击**添加网络**来添加任何vSphere中可用的网络。

#### E. 启用磁盘UUIDs

为了使用RKE创建集群，必须为所有主机配置磁盘UUIDs。

从Rancher v2.0.4开始，在vSphere主机模板中默认启用了磁盘UUIDs。

如果您在使用Rancher v2.0.4之前的版本，请参照以下[说明](/docs/cluster-provisioning/rke-clusters/node-pools/vsphere/provisioning-vsphere-clusters/#enabling-disk-uuids-with-a-node-template)以获取如何通过Rancher主机模板启用UUID的详细信息。

#### F. 可选：配置主机标签和自定义属性

将元数据附加到VM的方式因Rancher版本而异。

 tabs 
 tab "Rancher v2.3.3+" 

**可选:** 添加vSphere标记和自定义属性。 标记使您可以将元数据附加到vSphere清单中的对象，以使排序和搜索这些对象更加容易。

对于标记，所有vSphere标记将显示为在主机模板中可选择的选项。

在自定义属性中，Rancher将允许您选择已在vSphere中设置的所有自定义属性。 自定义属性作为keys，您可以为每个属性输入value。

> **注意:** 自定义属性是一项传统功能，最终将会从vSphere中删除。 这些属性使您可以将元数据附加到vSphere清单中的对象，以使排序和搜索这些对象更加容易。

 /tab 
 tab "Rancher v2.3.3之前的版本" 

**可选:**

- 为虚拟机提供一组配置参数（实例选项）。
- 为VM分配标签，这些标签可用作在集群中调度规则的依据。
- 自定义将要创建的VM中的Docker守护程序的配置。

> **注意:** 自定义属性是一项传统功能，最终将会从vSphere中删除。 这些属性使您可以将元数据附加到vSphere清单中的对象，以使排序和搜索这些对象更加容易。

 /tab 
 /tabs 

#### G. 可选: 配置cloud-init

[Cloud-init](https://cloudinit.readthedocs.io/en/latest/) 允许您在第一次启动时通过应用配置来初始化主机。 这可能包括诸如创建用户，授权SSH密钥或设置网络等事情。

VM的cloud-init支持范围取决于Rancher的版本。

 tabs 
 tab "Rancher v2.3.3+" 

To make use of cloud-init initialization, create a cloud config file using valid YAML syntax and paste the file content in the the **Cloud Init** field. Refer to the [cloud-init documentation.](https://cloudinit.readthedocs.io/en/latest/topics/examples.html) for a commented set of examples of supported cloud config directives.

要使用cloud-init初始化，请使用有效的YAML语法创建一个配置文件，然后将文件内容粘贴到**Cloud Init**字段中。 请参照[cloud-init文档。](https://cloudinit.readthedocs.io/en/latest/topics/examples.html)以获取cloud config配置示例。

_注意，通过ISO创建VM时不支持使用cloud-init选项_

 /tab 
 tab "Rancher v2.3.3之前的版本" 

您可以在**Cloud Init**字段中指定RancherOS cloud-config.yaml文件的URL。 有关受支持的配置的详细信息，请参考[RancherOS文档](https://rancher.com/docs/os/v1.x/en/installation/configuration/#cloud-config)。 请注意，创建的VM中必须可以访问该URL。

 /tab 
 /tabs 

#### H. 保存主机模板

为此模板分配一个描述性的**名称**，然后点击**创建**。

#### 主机模板配置参考

适用于vSphere主机模板的配置选项的参考，请参考[本节](/docs/cluster-provisioning/rke-clusters/node-pools/vsphere/provisioning-vsphere-clusters/node-template-reference/)。

## 2. 使用主机模板创建Kubernetes集群

创建模板后，可以使用它启动vSphere集群。

要在vSphere主机上安装Kubernetes，您需要通过修改集群YAML文件来启用vSphere cloud provider。 此项既适用于预先创建的[自定义节点](/docs/cluster-provisioning/rke-clusters/custom-nodes/)，也适用于使用vSphere主机驱动程序在Rancher中创建的主机。

要创建集群并为集群启用vSphere提供程序，请执行以下步骤：To create the cluster and enable the vSphere provider for cluster, follow these steps:

- [A. 设置集群名称和成员角色](#a-设置集群名称和成员角色)
- [B. 配置Kubernetes选项](#b-配置Kubernetes选项)
- [C. 为集群添加主机池](#c-为集群添加主机池)
- [D. 可选: 添加自我修复的主机池](#d-可选-添加自我修复的主机池)
- [E. 创建集群](#e-创建集群)

#### A. 设置集群名称和成员角色

1. 以管理员身份登录到Rancher UI。
2. 进入到**全局**中的**集群列表**页面。
3. 点击**添加集群** 选择 **vSphere** 基础设施供应商。
4. 填写 **集群名称**。
5. 指定需要的**成员角色**。 {{< step_create-cluster_member-roles >}}

> **注意:**
>
> 如果您的集群启用了DRS，推荐设置[VM-VM关联性规则](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.resmgmt.doc/GUID-7297C302-378F-4AF2-9BD6-6EDB1E0A850A.html)。 这些规则使分配了etcd和控制平面角色的VM在分配给不同的主机池时可以在单独的ESXi主机上运行。 这种做法可确保单个物理机的故障不会影响这些平面的可用性。

#### B. 配置Kubernetes选项

{{<step_create-cluster_cluster-options>}}

#### C. 为集群添加主机池

{{<step_create-cluster_node-pools>}}

#### D. 可选: 添加自我修复的主机池

要使主机池能够自我修复，请在**自动替换**列中输入一个大于零的数字。 如果主机池在指定的分钟内处于非活动状态，Rancher将使用该主机池使用的主机模板来重新创建该主机。

> **注意:** 自我修复主机池旨在帮助您为**无状态**应用程序替换工作主机。 不建议在主主机或具有持久卷连接的主机的主机池上启用主机自动替换。 当主机池中的主机失去与集群的连接时，其持久卷将被破坏，从而导致有状态应用程序丢失数据。

#### E. 创建集群

点击 **创建** 开始创建虚拟机和Kubernetes集群。

{{< result_create-cluster >}}

## 3. 可选: 创建存储

有关如何使用Rancher在vSphere中创建存储的，请参照
[集群管理小节。](/docs/cluster-admin/volumes-and-storage/examples/vsphere)

为了在vSphere中创建存储，必须启用vSphere provider。

#### 为集群启用vSphere cloud provider

1. 设置 **Cloud Provider** 选项为`Custom`.

   ![vsphere-node-driver-cloudprovider](/img/rancher/vsphere-node-driver-cloudprovider.png")

1. 点击**编辑YAML**
1. 将以下结构内容插入到预先配置的集群YAML中。 从Rancher v2.3+开始，此结构必须放在`rancher_kubernetes_engine_config`下。 在v2.3之前的版本中，必须将其定义为顶级字段。 注意，`name` _必须_设置为`vsphere`。

   ```yaml
   rancher_kubernetes_engine_config: # Required as of Rancher v2.3+
     cloud_provider:
       name: vsphere
       vsphereCloudProvider: [Insert provider configuration]
   ```

   Rancher使用RKE (the Rancher Kubernetes Engine)来创建Kubernetes集群. 请参照[RKE文档中的vSphere配置参考]({{<baseurl>}}/rke/latest/en/config-options/cloud-providers/vsphere/config-reference/) 以获取`vsphereCloudProvider`命令中属性的详细信息。

## 可选步骤

创建集群后，您可以通过Rancher UI访问它。 作为最佳实践，我们建议设置以下替代方法来访问集群：

- **通过kubectl CLI访问集群:** 请按照[这些步骤](/docs/cluster-admin/cluster-access/kubectl/#accessing-clusters-with-kubectl-on-your-workstation)来通过kubectl访问您的集群. 在这种情况下，您将通过Rancher服务器的身份验证代理进行身份验证，然后Rancher会将您连接到下游集群。 此方法使您无需Rancher UI即可管理集群。

- **通过kubectl CLI和授权的集群地址访问您的集群:** 请按照[这些步骤](/docs/cluster-admin/cluster-access/kubectl/#authenticating-directly-with-a-downstream-cluster)来通过kubectl直接访问您的集群,而不需要通过Rancher进行认证. 我们建议您设定此方法访问集群，这样在您无法连接Rancher时您仍然能够访问集群。

