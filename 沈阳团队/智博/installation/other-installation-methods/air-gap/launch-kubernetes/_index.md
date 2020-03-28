---
title: '3. Install Kubernetes with RKE (Kubernetes Installs Only)'
---
使用RKE安装Kubernetes（仅限Kubernetes安装Rancher）

This section is about how to prepare to launch a Kubernetes cluster which is used to deploy Rancher server for your air gapped environment.
本节介绍如何准备启动Kubernetes集群，该集群用于为私有环境部署Rancher server。

Since a Kubernetes Installation requires a Kubernetes cluster, we will create a Kubernetes cluster using [Rancher Kubernetes Engine]({{<baseurl>}}/rke/latest/en/) (RKE). Before being able to start your Kubernetes cluster, you'll need to [install RKE]({{<baseurl>}}/rke/latest/en/installation/) and create a RKE config file.
我们需要使用RKE来部署Kubernetes集群。

- [A. Create an RKE Config File](#a-create-an-rke-config-file)
- [B. Run RKE](#b-run-rke)
- [C. Save Your Files](#c-save-your-files)

#### A. Create an RKE Config File
创建RKE配置文件

From a system that can access ports 22/tcp and 6443/tcp on your host nodes, use the sample below to create a new file named `rancher-cluster.yml`. This file is a Rancher Kubernetes Engine configuration file (RKE config file), which is a configuration for the cluster you're deploying Rancher to.
在可以访问主机节点上的端口22 / tcp和6443 / tcp的系统上，使用以下示例创建一个名为`rancher-cluster.yml'的新文件。 该文件是Rancher Kubernetes Engine配置文件（RKE配置文件）。

Replace values in the code sample below with help of the _RKE Options_ table. Use the IP address or DNS names of the [3 nodes](/docs/installation/air-gap-high-availability/provision-hosts) you created.
替换下面的代码示例中的值， 使用您创建的[3个节点](/docs/installation/air-gap-high-availability/provision-hosts)的IP地址或DNS名称。

> **Tip:** For more details on the options available, see the RKE [Config Options]({{<baseurl>}}/rke/latest/en/config-options/).
> **提示：**有关可用选项的更多详细信息，请参见RKE [Config Options]（{{<baseurl>}}/rke/latest/en/config-options/)。

<figcaption>RKE Options</figcaption>

| Option             | Required             | Description                                                                             |
| ------------------ | -------------------- | --------------------------------------------------------------------------------------- |
| `address`          | ✓                    | The DNS or IP address for the node within the air gap network.                          |
| `user`             | ✓                    | A user that can run docker commands.                                                    |
| `role`             | ✓                    | List of Kubernetes roles assigned to the node.                                          |
| `internal_address` | optional<sup>1</sup> | The DNS or IP address used for internal cluster traffic.                                |
| `ssh_key_path`     |                      | Path to SSH private key used to authenticate to the node (defaults to `~/.ssh/id_rsa`). |

> <sup>1</sup> Some services like AWS EC2 require setting the `internal_address` if you want to use self-referencing security groups or firewalls.
如果您想使用自引用安全组或防火墙，则某些服务（如AWS EC2）需要设置`internal_address`。

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

#### B. Run RKE

After configuring `rancher-cluster.yml`, bring up your Kubernetes cluster:
配置完`rancher-cluster.yml`之后，启动您的Kubernetes集群：

```
rke up --config ./rancher-cluster.yml
```

#### C. Save Your Files

> **Important**
> The files mentioned below are needed to maintain, troubleshoot and upgrade your cluster.
> **重要**
> 以下文件需要被维护，用来故障排除和升级集群。

Save a copy of the following files in a secure location:
将以下文件的副本保存在安全的位置：

- `rancher-cluster.yml`: The RKE cluster configuration file.
- `kube_config_rancher-cluster.yml`: The [Kubeconfig file]({{<baseurl>}}/rke/latest/en/kubeconfig/) for the cluster, this file contains credentials for full access to the cluster.
- `rancher-cluster.rkestate`: The [Kubernetes Cluster State file]({{<baseurl>}}/rke/latest/en/installation/#kubernetes-cluster-state), this file contains the current state of the cluster including the RKE configuration and the certificates.<br/><br/>_The Kubernetes Cluster State file is only created when using RKE v0.2.0 or higher._

- `rancher-cluster.yml`: RKE配置文件
- `kube_config_rancher-cluster.yml`: [Kubeconfig 文件]({{<baseurl>}}/rke/latest/en/kubeconfig/)
- `rancher-cluster.rkestate`：集群状态文件[Kubernetes Cluster State file]({{<baseurl>}}/rke/latest/en/installation/#kubernetes-cluster-state)


#### Issues or errors?

See the [Troubleshooting](/docs/installation/options/troubleshooting/) page.

#### [Next: Install Rancher](/docs/installation/other-installation-methods/air-gap/install-rancher)
