---
title: 运行Rancher的提示
---

在生产或者一些很重要的环境中部署Rancher，应该使用至少有三个节点的高可用Kubernetes集群上安装Rancher。运行在多个节点上的多个Rancher实例确保了单节点环境无法实现的高可用性。

## 在单独的集群上运行Rancher

不要在安装Rancher的Kubernetes集群中运行其他工作负载或微服务。

## 不要在托管的Kubernetes环境中运行Rancher

当Rancher server安装在Kubernetes集群上时，它不应该在托管的Kubernetes环境中运行，比如谷歌的GKE、Amazon的EKS或Microsoft的AKS。这些托管的Kubernetes解决方案没有将etcd公开到Rancher可以管理的程度，并且它们的自定义可能会干扰Rancher的操作。

强烈建议使用托管基础设施，如Amazon的EC2或谷歌的GCE。在基础设施提供者上使用RKE创建集群时，可以将集群配置为创建etcd快照作为备份。然后，您可以[使用RKE]({{<baseurl>}}/ RKE /latest/en/etcd-snapshot /)或[Rancher](/docs/backup /restorations/)从这些快照之一恢复您的集群。在托管的Kubernetes环境中，不支持这种备份和恢复功能。

## 确保Kubernetes的节点配置正确

当你部署节点时需要遵循k8和etcd最佳实践，比如：禁用互换、反复检查你有完整的集群中的所有机器之间的网络连接、使用唯一的主机名、MAC地址、每个节点product_uuids、检查所有正确的端口被打开，和部署与ssd etcd支持。更多的细节可以在[kubernetes docs](https://kubernetes.io/docs/setup/producenvironment/tools/kubeadm/install-kubeadm/#before-you-begin)和[etcd的性能操作指南](https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/performance.md)中找到。

## 使用RKE备份状态文件

对于`RKE v0.2`之前的版本，ETCD备份会自动将`/etc/kubernetes/ssl/`目录下的所有证书打包为`pki.bundle.tar.gz`文件，然后保存在`/opt/rke/etcd-snapshot`目录中。

对于`RKE v0.2`之后的版本，RKE将集群状态记录在一个名为`cluster.rkestate`的文件中，这个文件存放于RKE 配置文件相同目录。这个文件保存了集群的SSL证书信息，对于通过RKE恢复集群`和/或`集群的后期维护非常重要。由于该文件包含证书信息，我们强烈建议在备份之前对该文件进行加密，并且每次运行`rke up`之后，您应该备份此状态文件。

## 所有节点应为同一个数据中心

为了获得最佳性能，请在相同的数据中心中运行所有local集群节点。

如果您正在运行云中的节点，例如：AWS，请在单独的可用区域中运行每个节点。例如，启动us-west-2a中的节点，us-west-2b中的节点2，us-west-2c中的节点3。

## 开发和生产环境应该类似

强烈建议使用Rancher创建`staging`或`pre-production`环境的Kubernetes集群，这个环境应该在软件和硬件配置方面尽可能的与生产环境相同。

## 监视集群以计划容量

Rancher server的 Local Kubernetes集群应该尽可能符合[系统和硬件需求](/docs/installation/requirements/)。您越偏离系统和硬件需求，您承担的风险就越大。

但是，metrics-driven的容量规划分析应该是扩展Rancher的最终指导，因为发布的需求考虑了各种工作负载类型。

使用Rancher，您可以通过与领先的开源监控解决方案Prometheus和Grafana的集成来监视集群节点、Kubernetes组件和软件部署的状态和过程，Grafana可以可视化来自Prometheus的指标。

在集群中[启用监控](/docs/cluster-admin/tools/monitoring/)之后，您可以设置[通知通道](/docs/cluster-admin/tools/notifiers/)和[集群警报](/docs/cluster-admin/tools/alerts/)，让您知道您的集群是否接近其容量。您还可以使用Prometheus和Grafana监控框架来建立关键指标的基准。

