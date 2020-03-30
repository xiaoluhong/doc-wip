---
title: Port 要求
---

为了保证运行，Rancher要求在Rancher节点和下游Kubernetes群集节点上开放许多端口。

### Rancher 节点

下表列出了使用[Docker安装](/docs/installation/single-node-install/)的Rancher server容器的节点与[Kubernetes上安装Rancher](/docs/installation/k8s-install/)的节点之间需要开放的端口。

{{< ports-rancher-nodes >}}

**注意** 对于已配置的外部身份验证提供程序（例如LDAP），Rancher节点可能还需要其他出站规则。

### 下游Kubernetes集群节点

群集节点需要开放的端口会根据群集的启动方式而变化。下面的每个选项卡都列出了需要为不同[集群创建选项](/docs/cluster-provisioning/#cluster-creation-options)开放的端口。

> **提示：**
>
> 如果安全不是一个大问题，并且可以打开一些其他端口，可以将[常用端口](#commonly-used-ports)中的表用作端口参考，而不是下面的完整表。

 tabs 

 tab "节点池" 

下表描述了在[基础设施提供商](/docs/cluster-provisioning/rke-clusters/node-pools/)中创建[Rancher Launched Kubernetes](/docs/cluster-provisioning/rke-clusters/)的节点的端口要求。

> **注意：**
> 在Amazon EC2或DigitalOcean等云提供商中创建集群时，Rancher会自动打开所需的端口。

{{< ports-iaas-nodes >}}

 /tab 

 tab "自定义节点" 

下表描述了带有[自定义节点](/docs/cluster-provisioning/rke-clusters/custom-nodes/)的[Rancher Launched Kubernetes](/docs/cluster-provisioning/rke-clusters/)的端口要求。

{{< ports-custom-nodes >}}

 /tab 

 tab "托管的集群" 

下表描述了[托管群集](/docs/cluster-provisioning/hosted-kubernetes-clusters)的端口要求。

{{< ports-imported-hosted >}}

 /tab 

 tab "导入的集群" 

下表描述了[导入群集](/docs/cluster-provisioning/imported-clusters/)的端口要求。

{{< ports-imported-hosted >}}

 /tab 

 /tabs 

### 其他端口注意事项

#### 常用端口

这些端口通常在Kubernetes节点上打开，无论它是哪种类型的群集。

| 协议 |    端口     | 描述                                         |
| :------: | :---------: | --------------------------------------------------- |
|   TCP    |     22      | Node driver SSH provisioning                        |
|   TCP    |    2376     | Node driver Docker daemon TLS port                  |
|   TCP    |    2379     | etcd client requests                                |
|   TCP    |    2380     | etcd peer communication                             |
|   UDP    |    8472     | Canal/Flannel VXLAN overlay networking              |
|   UDP    |    4789     | Flannel VXLAN overlay networking on Windows cluster |
|   TCP    |    9099     | Canal/Flannel livenessProbe/readinessProbe          |
|   TCP    |    6783     | Weave Port                                          |
|   UDP    |  6783-6784  | Weave UDP Ports                                     |
|   TCP    |    10250    | kubelet API                                         |
|   TCP    |    10254    | Ingress controller livenessProbe/readinessProbe     |
| TCP/UDP  | 30000-32767 | NodePort port range                                 |

---

#### 本地节点流量

标记为`local traffic`（即: 9099 TCP）的端口用于Kubernetes健康检查(`livenessProbe` 和`readinessProbe`)。这些健康检查在节点本身上执行。在大多数云环境中，默认情况下会允许此本地流量。

但是在以下情况下，此流量可能会被阻止：

- 您已在节点上应用了严格的主机防火墙策略。
- 您正在使用具有多个接口（多宿主）的节点。

在这些情况下，您必须在您的主机防火墙，或者在公有/私有云托管机器(AWS或OpenStack)安全组配置中显式地允许这些流量。请记住，在将安全组用作安全组中的源或目标时，显式打开端口只适用于节点/实例的私有接口。

#### Rancher AWS EC2 安全组

在使用[AWS EC2 node driver](/docs/cluster-provisioning/rke-clusters/node-pools/ec2/)在Rancher中配置群集节点时，您可以选择让Rancher创建一个名为rancher-nodes的安全组。以下规则将自动添加到此安全组。

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
