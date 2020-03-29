---
title: Other Installation Methods
---
其他安装方式

#### Docker Installations
基于Docker安装

The [single-node Docker installation](/docs/installation/other-installation-methods/single-node-docker) is for Rancher users that are wanting to test out Rancher. Instead of running on a Kubernetes cluster using Helm, you install the Rancher server component on a single node using a `docker run` command.
[单节点Docker安装](/docs/installation/other-installation-methods/single-node-docker)适用于想要测试Rancher的Rancher用户。 无需使用Helm在Kubernetes集群上运行，而是使用docker run命令在单个节点上安装Rancher server组件。

Since there is only one node and a single Docker container, if the node goes down, there is no copy of the etcd data available on other nodes and you will lose all the data of your Rancher server.
由于只有一个节点和一个Docker容器，因此，如果该节点发生故障，则其他节点上没有可用的etcd数据副本，您将丢失Rancher服务器的所有数据。

#### Air Gapped Installations
私有环境安装

Follow [these steps](/docs/installation/other-installation-methods/air-gap) to install the Rancher server in an air gapped environment.
请按照[这些步骤](/docs/installation/other-installation-methods/air-gap)在私有环境安装Rancher server。

An air gapped environment could be where Rancher server will be installed offline, behind a firewall, or behind a proxy.
私有安装可能是离线安装，也可能是在防火墙或者代理之后安装。
