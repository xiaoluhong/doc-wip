---
title: 升级
---


本节包含有关如何将Rancher服务器升级到较新版本的信息。无论您是否安装在内网的环境中，升级步骤都主要取决于您是否具有Rancher的单节点安装或高可用性安装。从以下选项中选择：

- [升级使用Docker安装的Rancher](/docs/upgrades/upgrades/single-node/)
- [升级安装在Kubernetes集群上的Rancher](/docs/upgrades/upgrades/ha/)

#### 已知的升级问题

下表列出了升级Rancher时要考虑的一些最值得注意的问题。可以在[GitHub](https://github.com/rancher/rancher/releases)和 [Rancher forums](https://forums.rancher.com/c/announcements/12)的发行说明中找到有关每个Rancher版本的已知问题的更完整列表。

| 升级场景                  | 问题                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 升级到 v2.3.0+              | 由于tolerations已添加到用于Kubernetes设置的镜像中，因此任何用户配置的集群都将在进行任何编辑时自动更新。                                                                                                                                                                                                                                                                                                                                                            |
| 升级到 v2.2.0-v2.2.x        | Rancher引入了 [system charts](https://github.com/rancher/system-charts) 存储库，其中包含监控，日志，告警和全局DNS等功能所需的所有目录项。为了能够在气隙安装中使用这些功能，您将需要在本地镜像“ system-charts”存储库，并将Rancher配置为使用该存储库。请按照说明 [配置 Rancher system charts](/docs/installation/options/local-system-charts/#setting-up-system-charts-for-rancher-prior-to-v2-3-0). |
| 从v2.0.13或更早版本升级 | 如果您集群的证书已过期，则需要执行[其他步骤](/docs/cluster-admin/certificate-rotation/#rotating-expired-certificates-after-upgrading-older-rancher-versions) 来轮换证书.                                                                                                                                                                                                                                                                                                                         |
| 从 v2.0.7 或更早版本升级  | Rancher 引入了 `system` 项目，这是一个自动创建的项目，用于存储 Kubernetes需要操作的重要命名空间。 在升级到v2.0.7 +的过程中，Rancher希望从所有项目中取消分配这些命名空间。在开始升级之前，请检查您的系统名称空间，以确保它们被取消分配以 [防止集群网络问题](/docs/upgrades/upgrades/namespace-migration/#preventing-cluster-networking-issues).                                                                              |

#### 警告

不支持升级到或从 [rancher-alpha 存储库](/docs/installation/options/server-tags/#helm-chart-repositories/) 中的任何图表。

#### RKE 附加安装

**重要提示：Rancher v2.0.8之前仅支持RKE附加安装**

请使用Rancher helm chart将Rancher安装在Kubernetes集群上。有关详细信息，请参见 [Kubernetes 安装 - 安装概述](/docs/installation/k8s-install/#installation-outline).

如果您当前正在使用RKE附加安装方法，请参阅 [从RKE附加安装迁移](/docs/upgrades/upgrades/migrating-from-rke-add-on/) 了解有关如何使用helm chart迁移的详细信息。
