---
title: 主机网关（L2bridge）的网络要求
---

本节介绍如何配置使用 _主机网关（L2bridge）_ 模式的自定义Windows集群.

#### 禁用私有IP地址检查

如果你使用 _主机网关（L2bridge）_ 模式 并将您的节点托管在下面列出的任何云服务上, 那么您必须在启动时禁用Linux或Windows主机的私有IP地址检查. 要禁用每个节点的此检查，请按照下面每个服务提供的说明操作.

| 服务    | 禁用私有IP地址检查的说明                                                                                                                         |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Amazon EC2 | [禁用源/目标检查](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html#EIP_Disable_SrcDestCheck)                                  |
| Google GCE | [为实例启用 IP 转发](https://cloud.google.com/vpc/docs/using-routes#canipforward) (默认情况下，虚拟机无法转发由另一个虚拟机发出的数据包。) |
| Azure VM   | [启用或禁用IP转发](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-network-interface#enable-or-disable-ip-forwarding)             |

#### 云托管的VM路由配置

如果您使用的后端是Flannel的[**主机网关（L2bridge）**](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#host-gw), 则同一节点上的所有容器都属于一个私有子网, 和通信路由从一个节点上的子网通过主机网络路由到另一个节点上的子网.

- 当工作节点配置在AWS、虚拟化集群或裸机服务器上时, 确保它们属于相同的第2层子网. 如果节点不属于同一第2层子网，则`host-gw`网络将不起作用.

- 当工作节点配置在GCE或Azure上时, 它们不在相同的第2层子网中. GCE和Azure上的节点属于可路由的第3层网络. 按照下面的说明配置GCE和Azure, 以便云网络知道如何在每个节点上路由主机子网.

要在GCE或Azure上配置主机子网路由,首先运行以下命令来查找每个工作节点上的主机子网:

```bash
kubectl get nodes -o custom-columns=nodeName:.metadata.name,nodeIP:status.addresses[0].address,routeDestination:.spec.podCIDR
```

然后按照每个云提供商的说明为每个节点配置路由规则:

| 服务    | 说明                                                                                                                                                         |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Google GCE | 对于GCE, 为每个节点添加一个静态路由:[添加一个静态路由](https://cloud.google.com/vpc/docs/using-routes#addingroute).                                      |
| Azure VM   | 对于Azure,创建一个路由表:[自定义路由:用户定义](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#user-defined). |
