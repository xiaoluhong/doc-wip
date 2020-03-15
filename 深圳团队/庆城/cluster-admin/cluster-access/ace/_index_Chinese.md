---
标题: 授权的集群终端是如何工作的
---

本节介绍kubectl CLI、kubeconfig文件和授权的集群终端如何协同工作，从而允许您直接访问下游的Kubernetes集群，而无需通过Rancher服务器进行身份验证。它的目的是为[如何设置kubectl来直接访问集群](../kubectl/#authenticating-directly-with-a-downstream-cluster)提供背景信息和上下文的指示。

#### 关于kubeconfig文件

_kubeconfig文件_ 是一个当与kubectl命令行工具(或其他客户端)一起使用时，用于配置对Kubernetes的访问的文件。

这个kubeconfig文件及其内容是特定于您正在查看的集群的。您可以从Rancher中的Cluster视图下载它。对于Rancher中可以访问的每个集群，您都需要一个单独的kubeconfig文件。

下载kubeconfig文件后，您将能够使用kubeconfig文件及其Kubernetes[上下文](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#kubectl-context-and-configuration)访问下游集群。

#### RKE集群的两种身份验证方法

如果集群不是 [RKE 集群,](/docs/cluster-provisioning/rke-clusters/) kubeconfig文件只允许您以一种方式访问集群:它允许您通过Rancher服务器进行身份验证，然后Rancher允许您在集群上运行kubectl命令。

对于RKE集群，kubeconfig文件允许您以两种方式进行身份验证:

- **通过Rancher服务器身份验证代理:** Rancher's authentication proxy validates your identity, then connects you to the downstream cluster that you want to access.
- **直接使用下游集群的API Server:** 默认情况下，RKE集群具有启用的授权集群端点。这个端点允许您使用kubectl CLI和kubeconfig文件访问下游的Kubernetes集群，RKE集群默认启用了该端点。在这个场景中，下游集群的Kubernetes API Server通过调用Rancher设置的webhook ( `kube-api-auth` 微服务) 对您进行身份验证。

第二种方法是能够直接连接到集群的Kubernetes API Server，这很重要，因为如果不能连接到Rancher，它允许您访问下游集群。

要使用授权的集群端点，您需要配置kubectl来使用Rancher在创建RKE集群时为您生成的kubeconfig文件中的额外kubectl上下文。这个文件可以从Rancher UI的cluster视图中下载，配置kubectl的说明在 [此页](../kubectl/#authenticating-directly-with-a-downstream-cluster)中

这些与下游Kubernetes集群通信的方法也在 [架构页面](/docs/overview/architecture/#communicating-with-downstream-user-clusters) 上范围更广的上下文中解释了Rancher是如何工作的，以及Rancher是如何与下游集群通信的。

#### 关于kube-api-auth认证Webhook

部署 `kube-api-auth` 微服务是为了为 [已授权的集群端点](/docs/overview/architecture/#4-authorized-cluster-endpoint) 提供用户身份验证功能，该功能仅对 [RKE 集群](/docs/cluster-provisioning/rke-clusters/) 可用。当您使用 `kubectl`, 访问用户集群时，集群的Kubernetes API服务器将使用 `kube-api-auth` 务作为webhook对您进行身份验证。

在集群创建过程中, `/etc/kubernetes/kube-api-authn-webhook.yaml` 文件被部署，而且 `kube-apiserver` 配置了 `--authentication-token-webhook-config-file=/etc/kubernetes/kube-api-authn-webhook.yaml`. 这将 `kube-apiserver` 配置为查询 `http://127.0.0.1:6440/v1/authenticate` 以确定bearer tokens的身份验证。

`kube-api-auth` 的调度规则如下:

_适用于 v2.3.0 及以上版本_

| Component     | nodeAffinity nodeSelectorTerms                                                             | nodeSelector | Tolerations       |
| ------------- | ------------------------------------------------------------------------------------------ | ------------ | ----------------- |
| kube-api-auth | `beta.kubernetes.io/os:NotIn:windows`<br/>`node-role.kubernetes.io/controlplane:In:"true"` | none         | `operator:Exists` |
