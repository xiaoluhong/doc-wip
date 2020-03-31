---
title: 创建vSphere集群
---

使用Rancher和vSphere，您可以在本地体验云环境的操作。

Rancher可以在vSphere中创建主机并在其上安装Kubernetes。 在vSphere中创建Kubernetes集群时，Rancher首先通过与vCenter API通信来创建指定数量的虚拟机。 然后将Kubernetes安装在主机上。

vSphere集群可能由具有不同属性（例如内存或vCPU数量）的多组虚拟机组成。 该分组允许对每个Kubernetes角色的主机大小进行细粒度的控制。

## vSphere增强

更新后的vSphere主机模板，使您可以通过以下增强功能在本地体验云环境的部署操作：

#### 自我修复的主机池

_从Rancher v2.3.0开始可用_

使用Rancher设置vSphere节点的最大优势之一是，它允许您在本地集群中利用Rancher的自我修复主机池，也称为[主机自动替换功能](/docs/cluster-provisioning/rke-clusters/node-pools/#node-auto-replace)。 自我修复主机池旨在帮助您替换无状态应用程序的工作主机。 当Rancher通过主机模板创建主机时，Rancher可以自动替换无法访问的主机。

> **重要：** 不建议在主主机或具有持久卷连接的主机的主机池上启用主机自动替换。 当主机池中的主机失去与集群的连接时，其持久卷将被破坏，从而导致有状态应用程序丢失数据。

#### 实例配置选项的动态填充

_从Rancher v2.3.3开始可用_

更新后的vSphere的主机模板在使用vSphere凭证创建主机模板时，模板选项会自动填充与在vSphere控制台中配置VM相同的内容。

对于要自动填充的字段，您的设置需要满足一些[前提条件](/docs/cluster-provisioning/rke-clusters/node-pools/vsphere/provisioning-vsphere-clusters/#prerequisites)。

#### 更多操作系统的支持

在Rancher v2.3.3+中，您可以在创建虚拟机时选择任意支持`cloud-init`的操作系统。目前[cloud config](https://cloudinit.readthedocs.io/en/latest/topics/examples.html)只支持YAML格式。

在v2.3.3之前版本的Rancher中包含的vSphere主机驱动程序仅支持以[RancherOS]({{<baseurl>}}/os/v1.x/en/)作为VM的操作系统。

## v2.3.3主机模板功能的视频介绍

在这段YouTube视频中，我们演示了如何使用新的主机模板来帮助您在本地环境体验云环境一样的操作。

{{< youtube id="dPIwg6x1AlU">}}
