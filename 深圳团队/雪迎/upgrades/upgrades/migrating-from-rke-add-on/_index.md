---
title: 从带有RKE add-on组件的Kubernetes安装迁移
---

> **重要提示：Rancher v2.0.8之前仅支持RKE add-on安装**
>
> 如果您当前正在使用RKE add-on安装方法，请按照以下说明迁移到Helm安装。

以下说明将帮助您从RKE add-on安装迁移到使用Helm软件包管理器管理Rancher。

您将需要安装[kubectl][kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl) 和由RKE生成的kubeconfig YAML文件(`kube_config_rancher-cluster.yml`)。

> **注意:** 本指南假定安装了标准的Rancher。如果您修改了任何对象名称或命名空间，请进行相应的调整。

> **注意:** 如果要从Rancher v2.0.13或更早版本或v2.1.8或更早版本升级，并且集群的证书已过期，则需要执行[其他步骤](/docs/cluster-admin/certificate-rotation/#rotating-expired-certificates-after-upgrading-older-rancher-versions) 以轮换证书。

#### 将kubectl指向Rancher集群

确保`kubectl`使用的是正确的kubeconfig YAML文件。将环境变量`KUBECONFIG`设置为指向`kube_config_rancher-cluster.yml`:

```bash
export KUBECONFIG=$(pwd)/kube_config_rancher-cluster.yml
```

设置环境变量 `KUBECONFIG` 之后，请验证其是否包含正确的 `server` 参数。它应直接指向端口 `6443`上的集群节点之一。

```bash
kubectl config view -o=jsonpath='{.clusters[*].cluster.server}'
https://NODE:6443
```

如果命令的输出显示后缀为 `/k8s/clusters`的Rancher主机名，则说明配置了错误的kubeconfig YAML文件。它应该是您使用RKE创建集群以运行Rancher时创建的文件。

#### 保存您的证书

如果您已在Rancher集群入口上终止了ssl，请恢复证书和密钥以在Helm安装中使用。

使用`kubectl`来获取密码，解码值并将输出定向到文件。

```bash
kubectl -n cattle-system get secret cattle-keys-ingress -o jsonpath --template='{ .data.tls\.crt }' | base64 -d > tls.crt
kubectl -n cattle-system get secret cattle-keys-ingress -o jsonpath --template='{ .data.tls\.key }' | base64 -d > tls.key
```

如果您指定了私有CA根证书

```bash
kubectl -n cattle-system get secret cattle-keys-server -o jsonpath --template='{ .data.cacerts\.pem }' | base64 -d > cacerts.pem
```

#### 删除以前的Kubernetes对象

删除由RKE安装创建的Kubernetes对象。

> **注意:** 删除这些Kubernetes组件不会影响Rancher的配置或数据库，但是在进行任何维护后，最好事先创建数据备份。有关详细信息，请参见[创建Backup-Kubernetes安装](/docs/backups/backups/ha-backups)。

```bash
kubectl -n cattle-system delete ingress cattle-ingress-http
kubectl -n cattle-system delete service cattle-service
kubectl -n cattle-system delete deployment cattle
kubectl -n cattle-system delete clusterrolebinding cattle-crb
kubectl -n cattle-system delete serviceaccount cattle-admin
```

#### 从`rancher-cluster.yml`中删除addons部分

来自rancher-cluster.yml的addons部分包含使用RKE部署Rancher所需的所有资源。通过切换到Helm，不再需要集群配置文件的这一部分。在您喜欢的文本编辑器中打开`rancher-cluster.yml`并删除addons部分：


> **重要:** 确保仅从集群配置文件中删除addons部分。

```yaml
nodes:
  - address: <IP> # hostname or IP to access nodes
    user: <USER> # root user (usually 'root')
    role: [controlplane,etcd,worker] # K8s roles for node
    ssh_key_path: <PEM_FILE> # path to PEM file
  - address: <IP>
    user: <USER>
    role: [controlplane,etcd,worker]
    ssh_key_path: <PEM_FILE>
  - address: <IP>
    user: <USER>
    role: [controlplane,etcd,worker]
    ssh_key_path: <PEM_FILE>

services:
  etcd:
    snapshot: true
    creation: 6h
    retention: 24h

## 从此处删除addons部分到文件结尾
addons: |-
  ---
  ...
## End of file
```

#### 遵循Helm和Rancher安装步骤

从这里开始执行标准安装步骤。
- [3 - 初始化 Helm](/docs/installation/options/helm2/helm-init/)
- [4 - 安装 Rancher](/docs/installation/options/helm2/helm-rancher/)
