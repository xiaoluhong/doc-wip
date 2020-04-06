---
title: 设置容器的技巧
---

运行构建良好的容器会极大地提高环境的整体性能和安全性。

下面是设置容器的一些技巧。

关于容器安全性的更详细的文档，您也可以参考Rancher的[容器安全性指南](https://rancher.com/compleguide-contain-security)。

## 使用通用容器操作系统

如果可能，您应该尝试在通用的容器基础操作系统上进行标准化。

较小的发布版本，如Alpine和BusyBox，减少了容器镜像的大小，通常具有较小的攻击/漏洞。

Ubuntu、Fedora和CentOS等流行的发行版经过了更多的实地测试，提供了更多的功能。

## 从scratch容器开始

如果您的微服务是一个独立的静态二进制文件，那么应该使用从scratch容器开始。

[docker scratch](https://hub.docker.com/_/scratch)镜像它是空的，因此您可以使用它来设计最小的镜像。

这将有最小的攻击面和最小的镜像大小。

## 非特权运行容器

如果可能，在容器中运行进程时使用非特权用户。虽然容器运行时提供隔离，但是仍然可能存在漏洞和攻击。如果容器作为根运行，无意或意外的主机挂载也会受到影响。有关为pod或容器配置安全上下文的详细信息，请参考[Kubernetes docs](https://kubernetes.io/docs/tasks/configu-po-container/secur-context/)。

## 资源限制

对您的Pods应用配置CPU和内存限制，这可以帮助管理工作节点上的资源，并避免异常的微服务（比如内存溢出）影响其他微服务。

在标准Kubernetes中，可以在命名空间级别设置资源限制。在Rancher中，您可以在项目级别设置资源限制，它们将同步到项目中的所有命名空间，具体配置请查阅Rancher文档。

在设置资源配额时，如果您在一个项目或名称空间上设置任何与CPU或内存相关的内容(即限制或保留)，那么所有容器都需要在创建的过程中设置相应的CPU或内存字段。为了避免在工作负载创建期间对每个容器设置这些限制，可以在名称空间上指定缺省容器资源限制。

Kubernetes文档提供了更多关于如何在[容器级别](https://kubernetes.io/docs/concepts/configuration/manage-comput-resources-container/#资源请求-限制-点-容器)和名称空间级别上设置资源限制的说明。

## 资源预留

您应该将CPU和内存需求配置到您的Pod上。这对于通知调度器需要将pod放置在哪种类型的计算节点上，并确保它不会过度调度到该节点非常重要。在Kubernetes中，您可以在Pods容器的`spec`的`resources.requests` 请求字段中配置资源请求。有关详细信息，请参考[Kubernetes docs](https://kubernetes.io/docs/concepts/configuration/manage-comput-resources-container/#资源请求-限制-点和容器)。

> **注意:** 如果您为部署Pod的命名空间设置了资源限制，而Pod没有设置资源预留和资源限制，那么将不允许启动pod。为了避免在工作负载创建期间在每个容器上设置这些字段，可以在命名空间上指定缺省资源预留和资源限制。

建议为`容器/Pod`定义资源需求，这样可以相对精确的镜像调度。否则，可能会出现调度不均匀导致某些节点资源使用率过高，从而无法充分利用机器资源。

## 配置健康检查（存活和就绪）

为您的Pod配置健康检查（存活和就绪）。如果不配置健康检查，除非您的Pod完全崩溃，否则Kubernetes不会知道它是不健康的。

有关如何[配置活性和就绪探测器](https://kubernetes.io/docs/tasks/configurepo-container/configu-livenreadinprobes/)更详细说明，请参考Kubernetes文档。
