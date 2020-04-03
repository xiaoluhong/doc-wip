---
title: '3. 使用RKE安装Kubernetes (只安装 Kubernetes)'
---

本节介绍如何创建Kubernetes集群，该集群用于离线环境下部署 Rancher sever。

由于 Kubernetes 安装需要 Kubernetes 集群， 因此我们将使用 [Rancher Kubernetes Engine]({{<baseurl>}}/rke/latest/en/) (RKE) 创建Kubernetes集群。 在启动Kubernetes集群之前，您需要 [install RKE]({{<baseurl>}}/rke/latest/en/installation/) 并创建一个RKE配置文件。

- [A. 创建一个 RKE 配置文件](#a-create-an-rke-config-file)
- [B. 运行 RKE](#b-run-rke)
- [C. 保存文件](#c-save-your-files)

#### A. 创建一个 RKE 配置文件

确保主机节点端口22 /tcp和端口6443 /tcp未被占用,使用下面的示例创建一个名为的 `rancher-cluster.yml` 新文件. 该文件是Rancher Kubernetes Engine配置文件（RKE配置文件），它是Kubernetes 集群的配置,您的 Rancher server 将要部署在该集群上。

根据 _RKE Options_ table 替换下面的代码示例中的值。使用您创建的 [3 nodes](/docs/installation/air-gap-high-availability/provision-hosts) 的IP地址或DNS名称。

> **提示:** 有关可用选项的更多详细信息，请参阅RKE [Config Options]({{<baseurl>}}/rke/latest/en/config-options/).

<figcaption>RKE Options</figcaption>

| Option             | Required             | Description                                                                             |
| ------------------ | -------------------- | --------------------------------------------------------------------------------------- |
| `address`          | ✓                    | The DNS or IP address for the node within the air gap network.                          |
| `user`             | ✓                    | A user that can run docker commands.                                                    |
| `role`             | ✓                    | List of Kubernetes roles assigned to the node.                                          |
| `internal_address` | optional<sup>1</sup> | The DNS or IP address used for internal cluster traffic.                                |
| `ssh_key_path`     |                      | Path to SSH private key used to authenticate to the node (defaults to `~/.ssh/id_rsa`). |

> <sup>1</sup> Some services like AWS EC2 require setting the `internal_address` if you want to use self-referencing security groups or firewalls.

```yaml
nodes:
  - address: 10.10.3.187 # node air gap network IP
    internal_address: 172.31.7.22 # node intra-cluster IP
    user: rancher
    role: ['controlplane', 'etcd', 'worker']
    ssh_key_path: /home/user/.ssh/id_rsa
  - address: 10.10.3.254 # node air gap network IP
    internal_address: 172.31.13.132 # node intra-cluster IP
    user: rancher
    role: ['controlplane', 'etcd', 'worker']
    ssh_key_path: /home/user/.ssh/id_rsa
  - address: 10.10.3.89 # node air gap network IP
    internal_address: 172.31.3.216 # node intra-cluster IP
    user: rancher
    role: ['controlplane', 'etcd', 'worker']
    ssh_key_path: /home/user/.ssh/id_rsa

private_registries:
  - url: <REGISTRY.YOURDOMAIN.COM:PORT> # private registry url
    user: rancher
    password: '*********'
    is_default: true
```

#### B. 运行 RKE

配置 `rancher-cluster.yml` 完之后，启动您的Kubernetes集群：

```
rke up --config ./rancher-cluster.yml
```

#### C. 保存文件

> **重要说明**
> 以下文件需要维护，故障排除和升级群集。

将以下文件的副本保存在安全的位置：

- `rancher-cluster.yml`: RKE 集群配置文件。
- `kube_config_rancher-cluster.yml`:集群的 [Kubeconfig file]({{<baseurl>}}/rke/latest/en/kubeconfig/) , 此文件包含用于访问集群的凭据。
- `rancher-cluster.rkestate`: [Kubernetes Cluster State file]({{<baseurl>}}/rke/latest/en/installation/#kubernetes-cluster-state)。<br/><br/>此文件包含用于访问群集的凭据。仅在使用RKE v0.2.0或更高版本时创建 Kubernetes 群集状态文件。

#### [Next: Install Rancher](/docs/installation/other-installation-methods/air-gap/install-rancher)
