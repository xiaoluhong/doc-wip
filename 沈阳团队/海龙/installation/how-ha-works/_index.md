---
title: 关于高可用安装
---

我们建议使用[Helm](/docs/overview/architecture/concepts/#about-helm) (Kubernetes包管理器)在专用的Kubernetes集群上安装Rancher。这被称为高可用Kubernetes安装，因为通过在多个节点上运行Rancher可以提高可用性。

在标准安装中，首先将Kubernetes安装在基础设施提供商（例如Amazon的EC2或Google Compute Engine）中托管的三个节点上。

然后使用Helm在Kubernetes集群上安装Rancher。 Helm使用Rancher的Helm chart在Kubernetes集群的三个节点中的每个节点上安装Rancher的副本。我们建议使用负载均衡器将流量定向到集群中Rancher的每个副本，以提高Rancher的可用性。

Rancher server数据存储在etcd中。etcd数据库可以在所有三个节点上运行，并且需要奇数个节点，这样它就可以选举出拥有etcd集群大多数节点的leader。如果etcd数据库不能选出leader，则etcd可能会失败，从而需要从备份中还原集群。

有关Rancher如何工作的说明（与安装方法无关），请参阅[架构部分。](/docs/overview/architecture)

#### 推荐架构

- Rancher的DNS应该解析为4层负载均衡器
- 负载均衡器应将端口TCP/80和TCP/443转发到Kubernetes集群中的所有3个节点。
- Ingress控制器会将HTTP重定向到HTTPS，并在端口TCP/443上终止SSL/TLS。
- Ingress控制器会将流量转发到Rancher deployment中Pod上的端口TCP/80。

<figcaption>Kubernetes Rancher安装了4层负载均衡器，描述了ingress控制器的SSL终止</figcaption>
![High-availability Kubernetes Installation of Rancher](/img/rancher/ha/rancher2ha.svg)
<sup>Kubernetes Rancher安装了4层负载平衡器（TCP），描述了ingress控制器的SSL终止</sup>
