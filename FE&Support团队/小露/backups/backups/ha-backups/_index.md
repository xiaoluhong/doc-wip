---
title: HA安装备份
---

本节介绍如何创建高可用性Rancher安装的备份。

> **先决条件:**
>
> - Rancher Kubernetes Engine v0.1.7或更高版本
>
>   RKE v0.1.7以及更高版本才支持`etcd`快照备份功能
>
> - `rancher-cluster.yml`
>
>   需要使用到安装Rancher的RKE配置文件`rancher-cluster.yml`，将此文件需放在与RKE二进制文件同级目录中。
>
> - 在运行`etcd snapshot-restore`命令之前，您必须将*相同的*快照备份拷贝到每个`etcd`恢复节点。

## RKE Kubernetes集群数据架构

在RKE安装中，K8S集群与Rancher server的数据均保存在ETCD集群中，ETCD集群中的三个etcd实例相互同步数据，以在一个节点发生故障时，提供冗余和数据复制。

### ETCD集群容错表

建议在ETCD集群中使用奇数个成员，通过添加额外成员可以获得更高的失败容错

| 集群大小 | MAJORITY | 失败容错 |
| -------- | -------- | -------- |
| 1        | 1        | 0        |
| 2        | 2        | 0        |
| 3        | 2        | **1**    |
| 4        | 3        | 1        |
| 5        | 3        | **2**    |
| 6        | 4        | 2        |
| 7        | 4        | **3**    |
| 8        | 5        | 3        |
| 9        | 5        | 4        |

运行Rancher Management Server的RKE Kubernetes集群的体系结构图：

![运行Rancher管理服务器的RKE Kubernetes集群的体系结构](./images/rke-server-storage.svg)

## 备份大纲

1. [Take Snapshots of the `etcd` Database](#1-take-snapshots-of-the-etcd-database)

    使用RKE对当前的`etcd`数据库备份。

1. [Store Snapshot(s) Externally](#2-backup-snapshots-to-a-safe-location)

    在备份完成之后，将它们导出备份到一个安全的位置，如果集群遇到问题，这个位置不会受到影响。

## ETCD数据备份

有两种方案创建`etcd`快照: 定时自动创建快照和手动创建快照，每种方式对应特定的场景。

- [选项A: 自动的备份](#option-a-recurring-snapshots)

   在Rancher HA安装后，我们建议配置RKE以定时(默认5分钟)自动创建快照，以便始终拥有可用的安全恢复点。

- [选项B: 一次性备份](#option-b-one-time-snapshots)

   我们建议在升级或恢复其他快照等事件之前创建一次性快照。

### Option A: Recurring Snapshots

对于通过RKE高可用安装的Rancher，我们建议开启定时自动创建快照，以便始终拥有安全的恢复点。

定时自动创建快照服务是RKE附带的服务，默认没有开启。可以通过在`rancher-cluster.yml`中添加配置来启用etcd-snapshot(定时自动创建快照)服务。

#### 启用自动备份:

1. 编辑`rancher-cluster.yml`配置文件；

2. 编辑service `etcd`的配置以启用自动备份。

   从RKE v0.2.0开始，快照备份可以保存在S3存储。

   _RKE v0.2.0+_

   ```yaml
   services:
     etcd:
       backup_config:
         enabled: true     # 设置true启用ETCD自动备份，设置false禁用；
         interval_hours: 6 # 快照创建间隔时间，单位小时；
         retention: 60     # 快照保留天数(以天为单位)
         # Optional S3
         s3backupconfig:
           access_key: "myaccesskey"
           secret_key:  "myaccesssecret"
           bucket_name: "my-backup-bucket"
           folder: "folder-name" # Available as of v2.3.0
           endpoint: "s3.eu-west-1.amazonaws.com"
           region: "eu-west-1"
   ```

   _RKE v0.1.x_

   ```yaml
   services:
     etcd:
       snapshot: true # 设置true启用ETCD自动备份，设置false禁用
       creation: 6h0s # 快照创建间隔时间，单位小时；
       retention: 24h # 快照有效期，此时间后快照将被删除；
   ```

3. 保存并关闭 `rancher-cluster.yml`.

4. 打开**Terminal**并切换路径到RKE二进制文件所在目录.确保`rancher-cluster.yml`也在这个路径下；

5. 运行以下命令:

   ```bash
   rke up --config rancher-cluster.yml
   ```

**Result:** **结果:** RKE会在每个etcd节点上定时获取快照，并将快照将保存到每个etcd节点的:`/opt/rke/etcd-snapshots/`目录下。如果配置了S3存储配置，快照备份也会上传到S3兼容的存储后端。 

### Option B: One-Time Snapshots

> **警告** 1、在rke v0.2.0之前的版本，RKE将备份证书和配置文件到`pki.bundle.tar.gz`文件中，并保存在`/opt/rke/etcd-snapshots`目录中。通过RKE v0.2.0之前的版本恢复系统时，需要将快照和pki文件一并存放于`/opt/rke/etcd-snapshots`目录中。
> 2、从rke v0.2.0 开始，因为架构调整不再需要`pki.bundle.tar.gz`文件，当rke 创建集群后，会在RKE配置文件当前目录下生成`xxxx.rkestate`文件，此文件名称与RKE配置文件名称相同，文件中保存了集群的配置信息和各组件使用的证书信息，恢复集群时，此rkestate文件需要与RKE配置文件存放于同一级目录。

#### 一次性备份

1. 打开**Terminal**并切换路径到RKE二进制文件所在目录.确保`rancher-cluster.yml`也在该路径下。

2. 输入以下命令，注意替换快照名称，例如:<SNAPSHOT.db>

   ```bash
   rke etcd snapshot-save --name <SNAPSHOT.db> --config rancher-cluster.yml
   ```

**结果:** RKE会获取每个`etcd`实例的快照备份，并保存在每个etcd节点的`/opt/rke/etcd-snapshots`目录下。

**To Take a One-Time S3 Snapshot:**

_RKE v0.2.0版本可用_

1. 打开**Terminal**并切换路径到RKE二进制文件所在目录.确保`rancher-cluster.yml`也在该路径下。

2. 输入以下命令，注意替换快照名称，例如:<SNAPSHOT.db>

   ```shell
   rke etcd snapshot-save --config rancher-cluster.yml \
   --name snapshot-name  \
   --s3 --access-key S3_ACCESS_KEY \
   --secret-key S3_SECRET_KEY \
   --bucket-name s3-bucket-name  \
   --s3-endpoint  s3.amazonaws.com \
   --folder folder-name # 此参数在v2.3.0+可用
   ```

**结果:** RKE会获取每个`etcd`实例的快照备份，并保存在每个etcd节点的`/opt/rke/etcd-snapshots`目录下，并同时上传到S3后端存储。

## 将本地快照备份到安全位置

> **注意:** 如果您使用的是RKE v0.2.0，您可以直接将备份保存到S3兼容的存储后端，并跳过这一步。

在创建快照后，应该把它保存到安全的地方，以便在集群遇到灾难情况时快照不受影响，这个位置应该是持久的。

复制`/opt/rke/etcd-snapshots`目录下所有文件到安全位置。

- 在rke v0.2.0以前的版本，备份`/opt/rke/etcd-snapshots`目录中的快照文件和`pki.bundle.tar.gz`文件，以及rke 配置文件到安全位置，通过v0.2.0之前的版本恢复系统时，需要这些文件。
- 在rke v0.2.0以及以后的版本，备份`/opt/rke/etcd-snapshots`目录中的快照文件和rke配置文件，以及配置文件当前目录下的`xxxx.rkestate`文件，通过RKE v0.2.0之后版本恢复集群数据时，需要这些文件。

在本文档中，作为一个示例，我们使用Amazon S3作为我们的安全位置，并使用[S3cmd](http://s3tools.org/s3cmd)作为我们的工具来创建备份。

**例如:**

```bash
root@node:~# s3cmd mb s3://rke-etcd-snapshots
root@node:~# s3cmd put /opt/rke/etcd-snapshots/snapshot.db s3://rke-etcd-snapshots/
```
