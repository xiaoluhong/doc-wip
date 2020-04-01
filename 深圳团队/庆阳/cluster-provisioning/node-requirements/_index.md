---
标题: 用户集群节点要求
---

本页描述了安装您的应用和服务所在节点的要求。

在本章节, "用户集群" 是指运行您的应用程序的集群，它应该与运行Rancher的集群（或单个节点）分开。

> 如果Rancher安装在高可用性Kubernetes集群上，Rancher服务器集群和用户集群有不同的要求。 有关Rancher安装要求，请参阅 [安装章节](/docs/installation/requirements/)

请确保Rancher服务器的节点满足以下要求:

- [操作系统和Docker要求](#operating-systems-and-docker-requirements)
- [硬件要求](#hardware-requirements)
- [网络要求](#networking-requirements)
- [可选: 安全注意事项](#optional-security-considerations)

## 操作系统和Docker要求

Rancher应该与任何现代Linux发行版和任何现代Docker版本一起工作。 所有下游集群的etcd和controlplane节点都需要运行在Linux上。 Worker节点可以运行在Linux或[Windows服务器](#requirements-for-windows-nodes)上。 在Rancher v2.3.0中添加了在下游集群中使用Windows worker节点的功能。

Rancher已经过测试，并支持下游集群运行在Ubuntu，CentOS，Oracle Linux，RancherOS和RedHat Enterprise Linux上。 关于每个Rancher版本测试过的操作系统和Docker版本的详细信息，请参阅[支持维护条款](https://rancher.com/support-maintenance-terms/)

支持所有的64位x86操作系统。

如果您计划使用ARM64, 请参阅 [在ARM64上运行（实验）。](/docs/installation/options/arm64-platform/)

有关如何安装Docker的信息，请参阅官方[Docker文档。](https://docs.docker.com/)

一些从RHEL派生的Linux发行版，包括Oracle Linux，可能有默认防火墙规则阻止与Helm通信。这个[操作指南](/docs/installation/options/firewall)展示了如何检查默认的防火墙规则，以及如何在必要时使用`firewalld`开放端口。

SUSE Linux可能有默认阻止所有端口的防火墙。 在这种情况下，请按照[以下步骤](#opening-suse-linux-ports)开放将主机添加到自定义集群所需的端口。

#### Windows节点要求

_Rancher v2.3.0可以使用Windows worker节点_

Windows Server的节点必须运行在Docker企业版。

Windows节点只能用于工作节点。 请参阅[配置自定义Windows集群](/docs/cluster-provisioning/rke-clusters/windows-clusters/)

## 硬件要求

具有`worker`角色的节点的硬件要求主要取决于您的工作负载。 运行Kubernetes节点组件的最小值是1个CPU（核心）和1GB内存。

关于CPU和内存，建议将不同平面的Kubernetes集群（etcd、controlplane和worker）托管在不同的节点上，以便它们可以彼此分开扩展。

有关大型Kubernetes集群的硬件建议，请参阅关于[构建大型集群](https://kubernetes.io/docs/setup/best-practices/cluster-large/)的官方Kubernetes文档。

有关生产中etcd集群的硬件建议，请参阅官方[etcd文档。](https://etcd.io/docs/v3.4.0/op-guide/hardware/)

## 网络要求

对于生产集群，我们建议您仅开放下文端口要求中定义的端口来限制流量。

需要开放的端口根据用户集群的启动方式而有所不同。 下面的每个部分列出了在不同的[集群创建选项](/docs/cluster-provisioning/#cluster-creation-options)下需要开放的端口。

有关kubernetes集群中etcd节点、controlplane节点和worker节点的端口要求的详细信息，请参阅[Rancher Kubernetes引擎的端口要求。]({{<baseurl>}}/rke/latest/en/os/#ports)

有关在每种情况下使用哪些端口的详细信息，请参阅以下章节:

- [常用端口](#commonly-used-ports)
- [自定义集群的端口要求](#port-requirements-for-custom-clusters)
- [集群托管在云服务商的端口要求](#port-requirements-for-clusters-hosted-by-an-infrastructure-provider)
  - [AWS EC2上节点的安全组](#security-group-for-nodes-on-aws-ec2)
- [集群托管在Kubernetes服务商的端口要求](#port-requirements-for-clusters-hosted-by-a-kubernetes-provider)
- [导入集群的端口要求](#port-requirements-for-imported-clusters)
- [本地流量的端口要求](#port-requirements-for-local-traffic)

#### 常用端口

如果安全性不是一个大问题，并且您可以开放一些额外的端口，则可以使用此表作为端口参考，而不是后续部分中的综合表。

这些端口通常在Kubernetes节点上是开放的，无论它是什么类型的集群。

 accordion id="common-ports" label="点击展开" 

<figcaption>常用端口参考</figcaption>

| 协议 |    端口     | 描述                                         |
| :------: | :---------: | --------------------------------------------------- |
|   TCP    |     22      | 节点驱动程序SSH设置                                    |
|   TCP    |    2376     | 节点驱动程序Docker守护进程TLS端口                        |
|   TCP    |    2379     | etcd客户端请求                                        |
|   TCP    |    2380     | etcd对等通信                                          |
|   UDP    |    8472     | Canal/Flannel VXLAN overlay网络                      |
|   UDP    |    4789     | windows集群上的Flannel VXLAN overlay 网络             |
|   TCP    |    9099     | Canal/Flannel livenessProbe/readinessProbe          |
|   TCP    |    6783     | Weave 端口                                          |
|   UDP    |  6783-6784  | Weave UDP 端口                                     |
|   TCP    |    10250    | kubelet API                                         |
|   TCP    |    10254    | Ingress controller livenessProbe/readinessProbe     |
| TCP/UDP  | 30000-32767 | NodePort 端口范围                                 |

 /accordion 

#### 自定义集群的端口要求

如果您要在现有的云服务商上启动Kubernetes集群，请参阅这些端口要求。

 accordion id="port-reqs-for-custom-clusters" label="点击展开" 

下表描述了[Rancher启动Kubernetes](/docs/cluster-provisioning/rke-clusters/)与[自定义节点](/docs/cluster-provisioning/rke-clusters/custom-nodes/)的端口要求。

{{< ports-custom-nodes >}}

 /accordion 

#### 集群托管在云服务商的端口要求

如果您要在云服务商（如Amazon EC2、Google Container Engine、DigitalOcean、Azure或vSphere）中的节点上启动Kubernetes集群，请应用这些端口要求。

在使用云服务商创建集群期间，Rancher会自动开放这些必需的端口。

 accordion id="port-reqs-for-infrastructure-providers" label="点击展开" 

下表描述了[Rancher启动Kubernetes](/docs/cluster-provisioning/rke-clusters/)在[云服务商](/docs/cluster-provisioning/rke-clusters/node-pools/)创建节点的端口要求。

> **注意:**
> 在Amazon EC2或DigitalOcean等云服务商创建集群期间，Rancher会自动开放所需的端口。

{{< ports-iaas-nodes >}}

 /accordion 

##### AWS EC2上节点的安全组

使用[AWS EC2节点驱动程序](/docs/cluster-provisioning/rke-clusters/node-pools/ec2/)在Rancher中配置集群节点时，您可以选择让Rancher创建名为"rancher-nodes"的安全组。 以下规则将自动添加到此安全组中。

| 类型            | 协议 | 端口范围  | 源/目的     | 规则类型 |
| --------------- | :------: | :---------: | ---------------------- | :-------: |
| SSH             |   TCP    |     22      | 0.0.0.0/0              |  Inbound  |
| HTTP            |   TCP    |     80      | 0.0.0.0/0              |  Inbound  |
| Custom TCP Rule |   TCP    |     443     | 0.0.0.0/0              |  Inbound  |
| Custom TCP Rule |   TCP    |    2376     | 0.0.0.0/0              |  Inbound  |
| Custom TCP Rule |   TCP    |  2379-2380  | sg-xxx (rancher-nodes) |  Inbound  |
| Custom UDP Rule |   UDP    |    4789     | sg-xxx (rancher-nodes) |  Inbound  |
| Custom TCP Rule |   TCP    |    6443     | 0.0.0.0/0              |  Inbound  |
| Custom UDP Rule |   UDP    |    8472     | sg-xxx (rancher-nodes) |  Inbound  |
| Custom TCP Rule |   TCP    | 10250-10252 | sg-xxx (rancher-nodes) |  Inbound  |
| Custom TCP Rule |   TCP    |    10256    | sg-xxx (rancher-nodes) |  Inbound  |
| Custom TCP Rule |   TCP    | 30000-32767 | 0.0.0.0/0              |  Inbound  |
| Custom UDP Rule |   UDP    | 30000-32767 | 0.0.0.0/0              |  Inbound  |
| All traffic     |   All    |     All     | 0.0.0.0/0              | Outbound  |

#### 集群托管在Kubernetes服务商的端口要求

如果您使用托管的Kubernetes服务商（如Google Kubernetes Engine、Amazon或Azure Kubernetes服务）启动集群，请参阅这些端口要求。

 accordion id="port-reqs-for-hosted-kubernetes" label="点击展开" 

下表描述了[托管的Kubernetes集群](/docs/cluster-provisioning/hosted-kubernetes-clusters)中节点的端口要求。

{{< ports-imported-hosted >}}

 /accordion 

#### 导入集群的端口要求

如果要导入现有集群，请参阅这些端口要求。

 accordion id="port-reqs-for-imported-clusters" label="点击展开" 

下表描述了[导入集群](/docs/cluster-provisioning/import-clusters/)的端口要求。

{{< ports-imported-hosted >}}

 /accordion 

#### 本地流量的端口要求

在端口要求中标记为`local traffic`（例如：`9099 TCP`）的端口用于Kubernetes运行状况检查`livenessProbe`和`readinessprobe`）。
这些运行状况检查在节点本身上执行。 在大多数云环境中，默认情况下允许此本地流量。

但是，在以下情况下，此流量可能会被阻止:

- 您已在节点上应用了严格的主机防火墙策略。
- 您正在使用具有多个接口（多宿主）的节点。

在这些情况下，您必须在您的主机防火墙中明确允许此流量，或者机器托管在公共/私有云（例如：AWS或OpenStack）的情况下，在您的安全组配置中明确允许此流量。 请记住，当使用安全组作为安全组中的源或目标时，显式开放端口仅适用于节点/实例的私有接口。

## 可选：安全注意事项

如果您想要配置符合CIS（Center for Internet Security）Kubernetes基准的Kubernetes集群，我们建议您在安装Kubernetes之前，遵循我们的强化指南来配置您的节点。

有关强化指南的更多信息以及指南的哪个版本与您的Rancher和Kubernetes版本对应的详细信息，请参阅[安全性部分。](/docs/security/#rancher-hardening-guide)

## 开放SUSE Linux端口

SUSE Linux可能具有默认情况下阻止所有端口的防火墙。 要开放将主机添加到自定义集群所需的主机端口,

1. SSH进入实例。
2. 编辑/`etc/sysconfig/SuSEfirewall2`并开放所需的端口。 在此示例中，还开放了端口9796和10250以进行监视：

```
FW_SERVICES_EXT_TCP="22 80 443 2376 2379 2380 6443 9099 9796 10250 10254 30000:32767"
FW_SERVICES_EXT_UDP="8472 30000:32767"
FW_ROUTE=yes
```

3. 使用新端口重启防火墙:

```
SuSEfirewall2
```

**结果:** 节点添加到自定义集群所需的节点端口已打开。
