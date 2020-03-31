---
title: vSphere主机模板配置参考
---

下表中描述了vSphere主机模板中可用的配置选项：

- [账户访问](#账户访问)
- [实例选项](#实例选项)
- [调度选项](#调度选项)

## 账户访问

帐户访问参数因Rancher版本而异。

 tabs 
 tab "Rancher v2.2.0+" 

| Parameter                | Required | Description |
|:----------------------|:--------:|:-----|
| Cloud Credentials   |   *      | 您的vSphere账户访问信息, 保存为[云凭证。](/docs/user-settings/cloud-credentials/)  |

 /tab 
 tab "Rancher v2.2.0之前的版本" 

| Parameter                | Required | Description |
|:------------------------|:--------:|:------------------------------------------------------------|
| vCenter or ESXi Server   |   *      | vCenter or ESXi服务使用的IP或FQDN。|
| Port                     |   *      | 连接服务时使用的端口，默认为`443`。 |
| Username                 |   *      | vCenter/ESXi服务的用户名 |
| Password                 |   *      | 用户密码。 |

 /tab 
 /tabs 

## 实例选项

根据您的Rancher版本，用于创建和配置实例的选项会有所不同。

 tabs 
 tab "Rancher v2.3.3+" 

| Parameter                | Required | Description |
|:----------------|:--------:|:-----------|
| CPUs                     |   *      | Number of vCPUS to assign to VMs. |
| Memory                   |   *      | Amount of memory to assign to VMs.  |
| Disk                     |   *      | Size of the disk (in MB) to attach to the VMs. |
| Creation method | * | The method for setting up an operating system on the node. The operating system can be installed from an ISO or from a VM template. Depending on the creation method, you will also have to specify a VM template, content library, existing VM, or ISO. For more information on creation methods, refer to the section on [configuring instances.](/docs/cluster-provisioning/rke-clusters/node-pools/vsphere/provisioning-vsphere-clusters/#c-configure-instances-and-operating-systems) |
| Cloud Init               |          | URL of a `cloud-config.yml` file or URL to provision VMs with. This file allows further customization of the operating system, such as network configuration, DNS servers, or system daemons. The operating system must support `cloud-init`. |
| Networks | | Name(s) of the network to attach the VM to. |
| Configuration Parameters used for guestinfo |          | Additional configuration parameters for the VMs. These correspond to the [Advanced Settings](https://kb.vmware.com/s/article/1016098) in the vSphere console. Example use cases include providing RancherOS [guestinfo]({{< baseurl >}}/os/v1.x/en/installation/running-rancheros/cloud/vmware-esxi/#vmware-guestinfo) parameters or enabling disk UUIDs for the VMs (`disk.EnableUUID=TRUE`). |

 /tab 
 tab "Rancher v2.3.3之前的版本" 

| Parameter                | Required | Description |
|:------------------------|:--------:|:------------------------------------------------------------|
| CPUs                     |   *      | 虚拟机使用的vCPUS数量。 |
| Memory                   |   *      | 虚拟机使用的内存数量。 |
| Disk                     |   *      | 虚拟机使用的磁盘大小（MB） |
| Cloud Init               |          | 用于配置VM的[RancherOS cloud-config]({{< baseurl >}}/os/v1.x/en/installation/configuration/)文件的URL。 该配置文件允许您进一步定制RancherOS操作系统，例如网络配置，DNS服务或系统守护程序。|
| OS ISO URL               |   *      | 创建VM使用的RancherOS或vSphere ISO文件的URL。 您可以在[Rancher OS GitHub Repo](https://github.com/rancher/os)中找到指定版本的URL。 |
| Configuration Parameters |          | VM的配置参数。 这些配置对应在vSphere控制台中的[高级设定](https://kb.vmware.com/s/article/1016098)。举例来说包括创建RancherOS [guestinfo]({{< baseurl >}}/os/v1.x/en/installation/running-rancheros/cloud/vmware-esxi/#vmware-guestinfo) 参数或为VM开启磁盘UUIDs(`disk.EnableUUID=TRUE`)。 |

 /tab 
 /tabs 

## 调度选项
根据您的Rancher版本，将虚拟机调度到虚拟机控制程序的选项有所不同。

 tabs 
 tab "Rancher v2.3.3+" 

| Parameter                | Required | Description |
|:------------------------|:--------:|:-------|
| Data Center              |   *      | 用于创建VM的数据中心的名称/路径。      |
| Resource Pool                     |          | 调度VM使用的资源池名称。 如果是独立的ESXi，请保留为空。若未指定，则使用默认的资源池。  |
| Data Store               |   *      | 如果您有数据存储集群，则可以切换**数据存储**选项。 这样，您可以选择将VM调度到的数据存储集群。 如果未切换该选项，则可以选择单个磁盘。|
| Folder                   |          | 要放置虚拟机的文件夹，该文件夹必须是已存在的。文件夹名称在vSphere配置文件中应该以`vm/`开头。 |
| Host                     |          | 用于调度VM的主机系统的IP。如果指定，将使用主机系统的池，并且**资源池**参数将被忽略。 |

 /tab 
 tab "Rancher v2.3.3之前的版本" 

| Parameter                | Required | Description |
|:------------------------|:--------:|:------------------------------------------------------------|
| Data Center              |   *      | 用于创建VM的数据中心的名称/路径。     |
| Pool                     |          | 调度VM使用的资源池名称/路径。若未指定，则使用默认的资源池。  |
| Host                     |          | 用于调度VM的主机系统的名称/路径。如果指定，将使用主机系统的池，并且**资源池**参数将被忽略。 |
| Network                  |   *      | 创建的VM使用的网络。 |
| Data Store               |   *      | 用于创建VM磁盘的数据存储 |
| Folder                   |          | 要放置虚拟机的文件夹，该文件夹必须是已存在的。文件夹名称在vSphere配置文件中应该以`vm/`开头。 |
 /tab 
 /tabs 