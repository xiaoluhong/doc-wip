---
title: Installing Rancher in an Air Gapped Environment
---
在私有环境中安装Rancher

This section is about installations of Rancher server in an air gapped environment. An air gapped environment could be where Rancher server will be installed offline, behind a firewall, or behind a proxy.

本节介绍在私有环境中安装Rancher server，我们将采取离线安装方式，同时要考虑防火墙和代理设置的因素。

Throughout the installations instructions, there will be _tabs_ for either a high availability Kubernetes installation or a single-node Docker installation.

以下为您准备了基于Kubernetes的高可用安装方式和基于Docker的单节点安装方式。

#### Air Gapped Kubernetes Installations
私有环境基于Kubernetes安装

This section covers how to install Rancher on a Kubernetes cluster in an air gapped environment.

本节介绍如何在私有环境中的Kubernetes集群上安装Rancher。

A Kubernetes install is composed of three nodes running the Rancher server components on a Kubernetes cluster. The persistence layer (etcd) is also replicated on these three nodes, providing redundancy and data duplication in case one of the nodes fails.

Kubernetes组件本身和Rancher server都通过三个节点组成，持久层（etcd）也可以在这三个节点上复制，以便节点之一发生故障时提供冗余和数据复制。

#### Air Gapped Docker Installations
私有环境基于Docker安装

These instructions also cover how to install Rancher on a single node in an air gapped environment.

这些说明介绍了如何在私有环境中的单个节点上安装Rancher。

The Docker installation is for Rancher users that are wanting to test out Rancher. Instead of running on a Kubernetes cluster, you install the Rancher server component on a single node using a `docker run` command. Since there is only one node and a single Docker container, if the node goes down, there is no copy of the etcd data available on other nodes and you will lose all the data of your Rancher server.

Docker安装适用于想要测试Rancher的用户。 您可以使用docker run命令在单个节点上安装Rancher服务器组件，而不是在Kubernetes集群上运行。 由于只有一个节点和一个Docker容器，因此，如果该节点发生故障，则其他节点上没有可用的etcd数据副本，您将丢失Rancher服务器的所有数据。

> **Important:** If you install Rancher following the Docker installation guide, there is no upgrade path to transition your Docker Installation to a Kubernetes Installation.

> **重要：**如果您按照Docker安装指南安装Rancher，则没有升级路径可以将Docker安装过渡到Kubernetes安装。

Instead of running the Docker installation, you have the option to follow the Kubernetes Install guide, but only use one node to install Rancher. Afterwards, you can scale up the etcd nodes in your Kubernetes cluster to make it a Kubernetes Installation.

您可以选择遵循Kubernetes安装指南，而不必运行Docker安装，但是仅使用一个节点来安装Rancher。 之后，您可以扩展Kubernetes集群中的etcd节点，使其成为Kubernetes安装。

## Installation Outline
安装概要

- [1. 准备节点](/docs/installation/other-installation-methods/air-gap/prepare-nodes/)
- [2. 整理镜像到私有镜像库](/docs/installation/other-installation-methods/air-gap/populate-private-registry/)
- [3. 使用RKE部署Kubernetes集群](/docs/installation/other-installation-methods/air-gap/launch-kubernetes/)
- [4. 安装Rancher](/docs/installation/other-installation-methods/air-gap/install-rancher/)

#### [下一步: 准备节点](/docs/installation/other-installation-methods/air-gap/prepare-nodes/)
