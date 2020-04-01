---
title: 安装要求
---

这个页面描述了安装Rancher server节点的软件，硬件和网络要求。 Rancher server可以安装在单个节点或高可用的Kubernetes群集上。

> 请务必注意，如果在Kubernetes群集上安装Rancher，则要求与[用户群集的节点要求](/docs/cluster-provisioning/node-requirements/)不同，后者将运行您的应用程序和服务。

请确保Rancher server的节点满足以下要求:

- [操作系统和Docker要求](#operating-systems-and-docker-requirements)
- [硬件要求](#hardware-requirements)
  - [CPU 和 Memory](#cpu-and-memory)
  - [磁盘](#disks)
- [网络要求](#networking-requirements)
  - [节点IP地址](#node-ip-addresses)
  - [端口要求](#port-requirements)

有关在生产环境中运行Rancher server的最佳实践列表，请参阅[最佳实践](/docs/best-practices/deployment-types/)部分。

Rancher UI 最适合Firefox 或 Chrome.

## 操作系统和Docker要求

Rancher可以兼容当前任何Linux发行版和Docker版本。

Rancher已经过测试，支持Ubuntu，CentOS，Oracle Linux，RancherOS和RedHat Enterprise Linux

有关每个Rancher版本测试了哪些OS和Docker版本的详细信息，请参阅[支持维护条款](https://rancher.com/support-maintenance-terms/)。

所有受支持的操作系统都是 64-bit x86。

应该安装 `ntp` (Network Time Protocol)，这样可以防止在客户端和服务器之间因为时钟不同步而发生证书验证错误。

一些源自RHEL的Linux发行版，包括Oracle Linux，可能有默认的防火墙规则来阻止与Helm的通信。本[操作指南](/docs/installation/options/firewall)展示了如何检查默认的防火墙规则，以及在必要时如何使用`firewalld`开放端口。

如果计划在ARM64上运行Rancher，请参阅[在ARM64上运行（实验性）](/docs/installation/options/arm64-platform/)。

#### 安装 Docker

可以按照[Docker官方文档](https://docs.docker.com/)中的步骤安装Docker。 Rancher还提供了使用命令来安装Docker的[脚本](/docs/installation/requirements/installing-docker)。

## 硬件要求

本节描述安装Rancher server的节点的CPU、内存和磁盘要求。

#### CPU 和 内存

硬件要求根据您的Rancher部署规模而定。根据要求配置每个单独的节点要求是不同的，具体取决于您是将Rancher与Docker一起安装还是在Kubernetes集群上安装。

 tabs 
 tab "在Kubernetes中安装Rancher" 

这些要求适用于[在Kubernetes集群上安装Rancher](/docs/installation/k8s-install/)。

| 部署规模 | 集群  | 节点      | vCPUs                                           | 内存                                             |
| --------------- | --------- | ---------- | ----------------------------------------------- | ----------------------------------------------- |
| Small           | 最多5个   | 最多50个   | 2                                               | 8 GB                                            |
| Medium          | 最多15个  | 最多200个  | 4                                               | 16 GB                                           |
| Large           | 最多50个  | 最多500个  | 8                                               | 32 GB                                           |
| X-Large         | 最多100个 | 最多1000个 | 32                                              | 128 GB                                          |
| XX-Large        | 100+      | 1000+      | [联系 Rancher](https://rancher.com/contact/) | [联系 Rancher](https://rancher.com/contact/) |

 /tab 
 tab "在Docker中安装Rancher" 

这些要求适用于Rancher的[单节点](/docs/installation/other-installation-methods/single-node-docker)安装。

| 部署规模 | 集群 | 节点     | vCPUs | 内存  |
| --------------- | -------- | --------- | ----- | ---- |
| Small           | 最多5个  | 最多50个  | 1     | 4 GB |
| Medium          | 最多15个 | 最多200个 | 2     | 8 GB |

 /tab 
 /tabs 

#### 磁盘

Rancher的性能取决于etcd在集群中的性能。为了确保最佳速度，我们建议使用SSD磁盘来支持Rancher管理Kubernetes集群。在云提供商上，您还需要使用允许最大IOPS的最小大小。在较大的集群中，请考虑使用专用存储设备存储etcd数据和wal目录。

## 网络要求

本节描述了安装Rancher server的节点的网络要求。

#### 节点IP地址

无论您是在单个节点上还是在HA群集上安装Rancher，每个节点都应配置一个静态IP。如果使用DHCP，则每个节点应具有DHCP预留，以确保该节点分配的相同IP地址。

#### 端口要求

本节描述运行`rancher/rancher`容器的节点的端口要求。

端口要求会有所不同，这取决于您是在单个节点上还是在高可用性Kubernetes群集上安装Rancher。

- **对于Docker安装，** 您只需要开放使Rancher能够与下游用户集群通信所需的端口即可。
- **对于高可用安装,** 需要开放相同的端口，以及建立Rancher所安装的Kubernetes集群所需的其他端口。

 tabs 
 tab "Kubernetes安装的端口要求" 

#### 用于与下游集群通信的端口

为了与下游群集通信，Rancher要求开放不同的端口，具体取决于您使用的基础架构。

例如，如果您在基础设施提供商托管的节点上部署Rancher，则必须为SSH开放`22`端口。

下图描述了为每种[群集类型](/docs/cluster-provisioning)开放的端口。

<figcaption>Rancher管理平台的端口要求</figcaption>

![Basic Port Requirements](/img/rancher/port-communications.svg)

下表细分了入站和出站流量的端口要求：

<figcaption>Rancher节点的入站规则</figcaption>

| 协议 | 端口 | 源                                                                                                                                                                                | 描述                                          |
| -------- | ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| TCP      | 80   | 进行外部SSL终止的负载均衡器/代理                                                                                                                                | 使用外部SSL终止时的Rancher UI/API |
| TCP      | 443  | <ul><li>etcd 节点</li><li>controlplane 节点</li><li>worker 节点</li><li>托管/导入 Kubernetes</li><li>任何需要使用Rancher UI或API的资源</li></ul> | Rancher agent, Rancher UI/API, kubectl               |

<figcaption>Rancher节点的出站规则</figcaption>

| 协议 | 端口 | 目的                                              | 描述                                   |
| -------- | ---- | -------------------------------------------------------- | --------------------------------------------- |
| TCP      | 22   | 使用Node Driver创建的节点中的任何节点IP      | 使用Node Driver通过SSH进行节点配置   |
| TCP      | 443  | `35.160.43.145/32`, `35.167.242.46/32`, `52.33.59.17/32` | git.rancher.io (catalogs)                     |
| TCP      | 2376 | 使用Node Driver创建的节点中的任何节点IP      | 使用Node Driver通过SSH进行节点配置        | Docker Machine使用的Docker守护程序TLS端口 |
| TCP      | 6443 | 托管/导入 Kubernetes API                           | Kubernetes API server                         |

**注意** 对于所有已经配置的外部[身份验证程序](/docs/admin-settings/authentication/)（例如LDAP），Rancher节点可能还需要其他出站规则。

#### HA/Kubernetes集群中节点的其他端口需求

启动Kubernetes集群你还需要开放其他的端口，这是Rancher高可用安装所必须的。

如果您按照Rancher安装文档来使用RKE设置Kubernetes集群，您将设置一个集群，其中所有三个节点都具有所有三个角色:etcd、controlplane和worker。在这种情况下，您可以参考具有所有三个角色的每个节点的需求列表:

<figcaption>具有所有三个角色的节点的入站规则:etcd、Controlplane和Worker</figcaption>

| 协议 | 端口        | 源                                                                                                 | 描述                                                                                  |
| -------- | ----------- | ------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------- |
| TCP      | 22          | 仅Linux worker节点，以及您希望从远程访问这个节点的任何网络。  | 通过SSH进行远程访问                                                                       |
| TCP      | 80          | 任何使用Ingress服务的资源                                                              | Ingress controller (HTTP)                                                                    |
| TCP      | 443         | 任何使用Ingress服务的资源                                                              | Ingress controller (HTTPS)                                                                   |
| TCP      | 2376        | Rancher 节点                                                                                          | Docker Machine使用的Docker守护进程TLS的端口（仅在使用Node Driver/Templates时需要) |
| TCP      | 2379        | etcd 节点 和 controlplane 节点                                                                      | etcd 客户端请求                                                                         |
| TCP      | 2380        | etcd nodes 和 controlplane nodes                                                                      | etcd 节点通信                                                                      |
| TCP      | 3389        | 仅Windows worker节点，以及您希望能够从远程访问这个节点的任何网络。 | 通过RDP远程访问                                                                       |
| TCP      | 6443        | etcd节点, controlplane节点, 和worker节点                                                       | Kubernetes apiserver                                                                         |
| UDP      | 8472        | etcd节点, controlplane节点, 和worker节点                                                       | Canal/Flannel VXLAN overlay 网络                                                       |
| TCP      | 9099        | 节点本身 (本地流量， 不跨节点)                                                      | Canal/Flannel livenessProbe/readinessProbe                                                   |
| TCP      | 10250       | controlplane 节点                                                                                     | kubelet                                                                                      |
| TCP      | 10254       | 节点本身 (本地流量， 不跨节点)                                                      | Ingress controller livenessProbe/readinessProbe                                              |
| TCP/UDP  | 30000-32767 | 任何使用NodePort服务的资源                                                             | NodePort 端口范围                                                                          |

<figcaption>具有所有三个角色的节点的出站规则:etcd、Controlplane和Worker</figcaption>

| 协议 | 端口  | 源                                            | 目的                                       | 描述                     |
| -------- | ----- | ------------------------------------------------- | ------------------------------------------------- | ------------------------------- |
| TCP      | 22    | RKE 节点                                          | 集群配置文件中配置的任何节点 | RKE通过SSH进行节点的配置 |
| TCP      | 443   | Rancher 节点                                     | Rancher agent                                     |
| TCP      | 2379  | etcd 节点                                        | etcd 客户端请求                              |
| TCP      | 2380  | etcd 节点                                        | etcd 节点通信                           |
| TCP      | 6443  | RKE 节点                                          | controlplane 节点                                | Kubernetes API server           |
| TCP      | 6443  | controlplane 节点                                | Kubernetes API server                             |
| UDP      | 8472  | etcd节点, controlplane节点, 和worker节点  | Canal/Flannel VXLAN overlay 网络            |
| TCP      | 9099  | 节点本身（本地流量，不跨节点） | Canal/Flannel livenessProbe/readinessProbe        |
| TCP      | 10250 | etcd节点, controlplane节点和worker节点  | kubelet                                           |
| TCP      | 10254 | 节点本身（本地流量，不跨节点 | Ingress controller livenessProbe/readinessProbe   |

每个节点需要开放的端口取决于节点的Kubernetes角色：etcd、controlplane或worker。如果您将Rancher安装在Kubernetes集群上，而该集群在每个节点上没有这三个角色，请参阅[Rancher Kubernetes引擎（RKE）的端口要求]({{<baseurl>}}/rke/latest/en/os/#ports)。RKE文档显示了每个角色的端口需求。

 /tab 
 tab "单节点端口需求" 

#### 与下游集群通信的端口

为了与下游群集通信，Rancher要求开放不同的端口，具体端口取决于使用的基础架构。

例如，如果要在基础设施提供商托管的节点上部署Rancher，则必须为SSH开放`22`端口。

下图描述了为每种[群集类型](/docs/cluster-provisioning)开放的端口 .

<figcaption>Rancher管理平台的端口要求</figcaption>

![Basic Port Requirements](/img/rancher/port-communications.svg)

下表细分了入站和出站流量的端口要求：

**注意** 对于所有已经配置的外部[身份验证程序](/docs/admin-settings/authentication/)（例如LDAP），Rancher节点可能还需要其他出站规则。

<figcaption>Rancher节点的入站规则</figcaption>

| 协议 | 端口 | 源                                                                                                                                                                                | 描述                                          |
| -------- | ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| TCP      | 80   | 进行外部SSL终止的负载均衡器/代理                                                                                                                                | 使用外部SSL终止时的Rancher UI/API |
| TCP      | 443  | <ul><li>etcd 节点</li><li>controlplane 节点</li><li>worker 节点</li><li>管/导入 Kubernetes</li><li>任何需要使用Rancher UI或API的资源</li></ul> | Rancher agent, Rancher UI/API, kubectl               |

<figcaption>Rancher节点的出站规则</figcaption>

| 协议 | 端口 | 源                                                   | Description                                   |
| -------- | ---- | -------------------------------------------------------- | --------------------------------------------- |
| TCP      | 22   | 使用Node Driver创建的节点中的任何节点IP        | 使用Node Driver通过SSH进行节点配置   |
| TCP      | 443  | `35.160.43.145/32`, `35.167.242.46/32`, `52.33.59.17/32` | git.rancher.io (catalogs)                     |
| TCP      | 2376 | 使用Node Driver创建的节点中的任何节点IP        | 使用Node Driver创建的节点中的任何节点IP |
| TCP      | 6443 | 托管/导入 Kubernetes API                           | Kubernetes API server                         |

**注意** 对于所有已经配置的外部[身份验证程序](/docs/admin-settings/authentication/)（例如LDAP），Rancher节点可能还需要其他出站规则。
 /tab 
 /tabs 
