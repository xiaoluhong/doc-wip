---
title: 证书轮换
---

> **Warning:** 轮换Kubernetes证书可能会导致群集在重新启动组件时暂时不可用。 对于生产环境，建议在维护时段内执行此操作。

默认情况下，Kubernetes集群需要证书，并且Rancher启动的Kubernetes集群会自动为Kubernetes组件生成证书。 在证书过期之前以及证书被泄露之前，轮换这些证书非常重要。 轮换证书后，Kubernetes组件将自动重新启动。

可以为以下服务轮换证书：

- etcd
- kubelet
- kube-apiserver
- kube-proxy
- kube-scheduler
- kube-controller-manager

#### Rancher v2.2.x中的证书轮换

Rancher启动的Kubernetes集群能够通过UI轮换自动生成的证书。

1. 在**全局**视图中，导航到要轮换证书的集群。

2. 选择**省略号（...）>轮换证书**。

3. 选择要轮换的证书。

   - 轮换所有服务证书（保持相同的CA）
   - 轮换单个服务，然后从下拉菜单中选择一项服务

4. 单击**保存**。

**结果：**所选证书将被轮换，相关服务将重新启动以开始使用新证书。

> **Note:** 即使RKE CLI可以为Kubernetes群集组件使用自定义证书，Rancher当前不允许在Rancher Launched Kubernetes群集中上传这些证书。

#### Rancher v2.1.x和v2.0.x中的证书轮换

_在版本v2.0.14以及v2.1.9中支持_

Rancher启动的Kubernetes集群能够通过API轮换自动生成的证书。

1.在**全局**视图中，导航到要轮换证书的集群。

2.选择**省略号（...）>在API中查看**。

3.单击**RotateCertificates**。

4.单击**显示请求**。

5.单击**发送请求**。

**结果：**所有Kubernetes证书将被轮换。

#### 升级较旧的Rancher版本后轮换过期的证书

如果要从Rancher v2.0.13或更早版本或v2.1.8或更早版本升级，并且您的群集已过期证书，则需要一些手动步骤来完成证书轮换。

1. 对于 `controlplane` 和 `etcd` 节点，登录到每个对应的主机，并检查证书 `kube-apiserver-requestheader-ca.pem` 是否在以下目录中：

   ```
   cd /etc/kubernetes/.tmp
   ```

   如果证书不在目录中，请执行以下命令：

   ```
   cp kube-ca.pem kube-apiserver-requestheader-ca.pem
   cp kube-ca-key.pem kube-apiserver-requestheader-ca-key.pem
   cp kube-apiserver.pem kube-apiserver-proxy-client.pem
   cp kube-apiserver-key.pem kube-apiserver-代理客户端-key.pem
   ```

   如果`.tmp`目录不存在，则可以将整个SSL证书复制到`.tmp`中：

   ```
   cp -r / etc / kubernetes / ssl /etc/kubernetes/.tmp
   ```

1. 轮换证书。对于Rancher v2.0.x和v2.1.x，请使用[Rancher API.](#certificate-rotation-in-rancher-v2-1-x-and-v2-0-x)，对于Rancher 2.2.x请使用[Rancher UI。](#certificate-rotation-in-rancher-v2-2-x)

1. 命令完成后，检查 `worker` 节点是否处于活动状态。如果不是，请登录到每个 `worker` 节点，然后重新启动kubelet和代理。
