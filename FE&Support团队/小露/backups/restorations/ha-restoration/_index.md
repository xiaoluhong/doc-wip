---
title: HA安装恢复
---

本章节将说明如何使用RKE来恢复Rancher-Kubernetes集群备份。Rancher-Kubernetes集群备份包含了Kubernetes配置和Rancher数据。

另外，`rke v0.2.0及以后版本``pki.bundle.tar.gz`文件已不再需要，由`xxx.rkestate`替代。具体请查看[Kubernetes集群状态存储]({{< baseurl >}}/rke/latest/en/installation/#kubernetes-cluster-state)。

## 1. 准备恢复工具

恢复集群时将需要使用[RKE](https://docs.rancher.cn/rancher2x/install-prepare/download/rke.html)和[kubectl](https://docs.rancher.cn/rancher2x/install-prepare/download/kubernetes.html)CLI工具，请提前下载。

你可以在原K8S集群中选择一台主机用于集群恢复的目标主机，然后通过[节点清理](https://docs.rancher.cn/rancher2x/install-prepare/remove-node.html)的方法初始化节点。或者使用一台全新的节点用于集群恢复的目标主机。

> **重要:** 在开始恢复之前，请确保已停止旧集群所有节点上的所有kubernetes服务。

## 2. 准备RKE配置文件

1. 将原集群的`rancher-cluster.yml`复制一份，命名为`rancher-cluster-restore.yml`

    ```bash
    cp rancher-cluster.yml rancher-cluster-restore.yml
    ```

    > 名称根据实际修改

2. 注释掉`rancher-cluster-restore.yml`文件中除了`nodes`以外的所有配置参数，并且只保留用于集群恢复的目标主机。

_例如_ `rancher-cluster-restore.yml`

```yaml
nodes:
  - address: 52.15.238.179 # 新的目标主机
    user: ubuntu
    role: [etcd, controlplane, worker]
## - address: 52.15.23.24
##   user: ubuntu
##   role: [ etcd, controlplane, worker ]
## - address: 52.15.238.133
##   user: ubuntu
##   role: [ etcd, controlplane, worker ]

## addons: |-
##   ---
##   kind: Namespace
##   apiVersion: v1
##   metadata:
##     name: cattle-system
##   ---
```

## 3. 准备备份数据和PKI证书

用于恢复集群的备份文件，根据`RKE版本`不同，需要使用不同的文件。

### RKE v0.1.x 备份恢复

在使用`RKE v0.1.x`备份数据时，RKE会保存K8S集群证书到`pki.bundle.tar.gz`文件中，并与ETCD备份一起存放在每个节点的`/opt/rke/etcd-snapshots`目录下。

恢复K8S集群时需要将以下两个文件一并存放到`/opt/rke/etcd-snapshots/`目录中。

- ETCD备份 - `<snapshot>.db`
- PKI证书 - `pki.bundle.tar.gz`

### RKE v0.2.0+ 备份恢复

从`RKE v0.2.0`开始，因为RKE软件架构调整，不再需要`pki.bundle.tar.gz`文件。

当`rke up`创建集群后，会在与`RKE配置文件`相同目录下生成`.rkestate`文件，此`.rkestate`文件名称与RKE配置文件名称相同，比如: `rancher-cluster.rkestate`，`.rkestate`文件中保存了K8S集群的`配置信息和各组件的证书文件`。

如果使用本地的ETCD备份文件恢复集群，您需要将ETCD备份文件拷贝到目标主机的`/opt/rke/etcd-snapshots/`目录中。拷贝一份原集群的`rancher-cluster.rkestate`文件，然后以`新RKE配置文件`重命名拷贝的`.rkestate`文件，例如：`rancher-cluster-restore.rkestate`。

> **注意:** 从RKE v0.2.0开始，ETCD备份支持存储在S3，恢复集群时从S3获取ETCD备份数据来恢复集群。如果从S3获取备份数据来恢复集群，则可以跳过此步骤，然后在第四步中配置S3相关的参数。

## 4. 恢复ETCD数据

RKE通过新的`rancher-cluster-restore.yml`配置文件恢复数据到目标主机上，并且RKE将在`目标主机`上创建包含已还原数据的ETCD容器。此ETCD容器将保持运行状态，但无法完成ETCD初始化。

### 从本地备份恢复

```bash
rke etcd snapshot-restore --name <snapshot>.db --config ./rancher-cluster-restore.yml
```

### 从S3备份恢复

_RKE v0.2.0 +可用_

```bash
rke etcd snapshot-restore \
--config rancher-cluster-restore.yml \
--name snapshot-name \
--s3 --access-key S3_ACCESS_KEY \
--secret-key S3_SECRET_KEY \
--bucket-name s3-bucket-name \
--s3-endpoint s3.amazonaws.com \
--folder folder-name # v2.3.0 + 可用
```

#### `rke etcd snapshot-restore`选项

> RKE v0.2.0+ 可用

| 选项                    | 描述                                                                                             | S3特有 |
| ------------------------- | ------------------------------------------------------------------------------------------------------- | ----------- |
| `--name` value            | 指定备份文件名称                                                                                 |             |
| `--config` value          | 指定RKE cluster YAML file (default: "cluster.yml") [$RKE_CONFIG]                           |             |
| `--s3`                    | 启用s3备份恢复                                                                               | \*          |
| `--s3-endpoint` value     | 指定s3访问地址(默认: `s3.amazonaws.com`)                                                   | \*          |
| `--access-key` value      | 指定 s3 accessKey                                                                                    | \*          |
| `--secret-key` value      | 指定 s3 secretKey                                                                                    | \*          |
| `--bucket-name` value     | 指定 s3 bucket name                                                                                  | \*          |
| `--folder` value          | 指定 s3 folder in the bucket name # v2.3.0+ 可用                                        | \*          |
| `--region` value          | 指定 s3 bucket location (可选)                                                               | \*          |
| `--ssh-agent-auth`        | [Use SSH Agent Auth defined by SSH_AUTH_SOCK]({{< baseurl >}}/rke/latest/en/config-options/#ssh-agent)  |             |
| `--ignore-docker-version` | [Disable Docker version check]({{< baseurl >}}/rke/latest/en/config-options/#supported-docker-versions) |

## 5. Bring Up the Cluster

使用RKE在单个`目标主机`上拉起K8S集群。

```bash
rke up --config ./rancher-cluster-restore.yml
```

### 测试 Cluster

`rke up --config ./rancher-cluster-restore.yml`执行完成后，将在RKE配置文件相同目录中创建一个`kube_config_rancher-cluster-restore.yml`文件。配置`kubectl`使用`kube_config_rancher-cluster-restore.yml`去检查集群状态，具体请查看[安装和配置kubectl](https://docs.rancher.cn/rancher2x/install-prepare/kubectl.html)。

```bash
kubectl get nodes

NAME            STATUS    ROLES                      AGE       VERSION
52.15.238.179   Ready     controlplane,etcd,worker    1m       v1.10.5
18.217.82.189   NotReady  controlplane,etcd,worker   16d       v1.10.5
18.222.22.56    NotReady  controlplane,etcd,worker   16d       v1.10.5
18.191.222.99   NotReady  controlplane,etcd,worker   16d       v1.10.5
```

### 清理旧节点

通过kubectl从集群中删除旧节点

```bash
kubectl --kubeconfig=kube_config_rancher-cluster-restore.yml delete node 18.217.82.189 18.222.22.56 18.191.222.99
```

### 重启目标主机

在继续之前，重新启动`目标主机`以确保集群网络和服务重新初始化。

### 检查Kubernetes Pods

等待`kube-system`、`ingress-nginx`和`cattle-system`中的Pod处于`Running`状态。

> **注意:** 在`rancher server`启动成功并且`DNS/负载均衡器`指向新集群之前，`cattle-cluster-agent`和`cattle-node-agentpods`将处于`Error`或者`CrashLoopBackOff`状态。

```bash
kubectl get pods --all-namespaces

NAMESPACE       NAME                                    READY     STATUS    RESTARTS   AGE
cattle-system   cattle-cluster-agent-766585f6b-kj88m    0/1       Error     6          4m
cattle-system   cattle-node-agent-wvhqm                 0/1       Error     8          8m
cattle-system   rancher-78947c8548-jzlsr                0/1       Running   1          4m
ingress-nginx   default-http-backend-797c5bc547-f5ztd   1/1       Running   1          4m
ingress-nginx   nginx-ingress-controller-ljvkf          1/1       Running   1          8m
kube-system     canal-4pf9v                             3/3       Running   3          8m
kube-system     cert-manager-6b47fc5fc-jnrl5            1/1       Running   1          4m
kube-system     kube-dns-7588d5b5f5-kgskt               3/3       Running   3          4m
kube-system     kube-dns-autoscaler-5db9bbb766-s698d    1/1       Running   1          4m
kube-system     metrics-server-97bc649d5-6w7zc          1/1       Running   1          4m
kube-system     tiller-deploy-56c4cf647b-j4whh          1/1       Running   1          4m
```

### 添加其他主机

编辑`rancher-cluster-restore.yml`RKE配置文件，添加其他主机。

_例如_`rancher-cluster-restore.yml`

```yaml
nodes:
  - address: 52.15.238.179 # 新的目标主机
    user: ubuntu
    role: [etcd, controlplane, worker]
  - address: 52.15.23.24
    user: ubuntu
    role: [etcd, controlplane, worker]
  - address: 52.15.238.133
    user: ubuntu
    role: [etcd, controlplane, worker]
## addons: |-
##   ---
##   kind: Namespace
```

运行RKE命令将新节点添加到新的集群中。

```bash
rke up --config ./rancher-cluster-restore.yml
```

### 完成恢复

Rancher现在应该可以运行和管理您的Kubernetes集群。如果您是根据[推荐架构](https://docs.rancher.cn/rancher2x/installation/helm-ha-install/#_1-%E6%8E%A8%E8%8D%90%E7%9A%84%E6%9E%B6%E6%9E%84)安装的Kubernetes集群，那么需要修改外部负载均衡器代理的后端主机IP。一旦更新了端点，托管集群上的代理应该会自动重新连接，这可能需要5-15分钟。

> **重要:** 请将您的新RKE配置(`rancher-cluster-restore.yml`)和`kubectl`凭证(`kube_config_rancher-cluster-restore.yml`)保存在一个安全的地方，以便将来进行维护。
