---
title: 备份ETCD
---

_从v2.2.0版本开始支持_

在Rancher UI中，可以轻松执行[Rancher启动的Kubernetes集群](/docs/cluster-provisioning/rke-clusters/)的etcd备份和恢复。 进行etcd数据库的快照并保存在[本地到etcd节点上](#本地磁盘备份)或[S3备份](#s3-备份)中。 配置S3的优点是，如果所有etcd节点都丢失了，则快照将远程保存，并可用于还原集群。

Rancher建议为所有生产集群配置循环的`etcd`快照。 此外，还可以轻松制作一次快照。

> **Note:** 如果您有任何在v2.2.0之前创建的Rancher启动的Kubernetes集群，则在升级Rancher之后，必须[编辑集群](/docs/cluster-admin/editing-clusters/)并对其进行 _save_ 以便启用 更新了快照功能。 即使您已经在v2.2.0之前创建快照，也必须执行此步骤，因为较早的快照将无法用于[通过UI备份和还原etcd](/docs/cluster-admin/restoring-etcd/)。

## 快照创建时间和保留计数

选择您希望重复制作快照的频率以及保留多少快照。 时间量以小时为单位。 使用带时间戳的快照，用户可以进行时间点恢复。

#### 为集群配置周期性快照

默认情况下，[Rancher launched Kubernetes clusters](/docs/cluster-provisioning/rke-clusters/)配置为拍摄定期快照（保存到本地磁盘）。为了防止本地磁盘故障，建议使用[S3备份](#s3-备份)或在磁盘上复制路径。

在集群配置或编辑集群期间，可以在**集群选项**的高级部分找到快照的配置。 点击**显示高级选项**。

在**高级集群选项**部分中，有几个选项可以配置：

| 选项                                                                                   | 描述                                                                        | 默认值 |
| ---------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- | ------------- |
| [etcd快照备份目标](#etcd快照备份目标)                                  | 选择您需要备份的存储目标，local（本地备份）或者S3（AWS S3备份） | local（本地备份）         |
| 启用定期etcd快照                                                         | 启用/停用etcd定期快照功能                                                 | Yes（启用）           |
| etcd重复快照周期 | 重复快照之间的时间间隔（小时）                                          | 12 小时      |
| 重复的etcd快照保留数量 | 要保留的快照数量                                                      | 6             |

#### 一次性快照

除了重复快照外，您可能还需要创建“一次性”快照。 例如，在升级集群的Kubernetes版本之前，最好备份集群的状态以防止升级失败。

1.在**全局**视图中，导航到要创建一次性快照的集群。

2.单击**垂直椭圆（...）> 立即快照**。

**结果：**根据您的[etcd快照备份目标](#etcd快照备份目标)，将拍摄一次快照并将其保存在选定的备份目标中。

## etcd快照备份目标

Rancher支持两种不同的备份目标：

- [本地磁盘备份](#本地磁盘备份)
- [S3 备份](#s3-备份)

#### 本地磁盘备份

默认情况下，选择 `local(本地)` 备份目标。 此选项的好处是没有额外的配置。 快照会自动保存到[Rancher launched Kubernetes clusters](/docs/cluster-provisioning/rke-clusters/)集群，etcd节点中的 `/opt/rke/etcd-snapshots` 路径。 所有重复快照均以配置的时间间隔拍摄。 使用 `local(本地)` 备份目标的不利之处在于，如果发生灾难，并且 _所有的_ etcd节点丢失，则无法恢复集群。

##### 安全时间戳

_从v2.3.0开始支持_

从v2.2.6版本开始，快照文件已打上时间戳，以简化使用外部工具和脚本处理文件的步骤，但是在某些与S3兼容的后端中，这些时间戳不可用。 从Rancher v2.3.0开始，添加了 `safe_timestamp` 选项以支持兼容的文件名。 当此标志设置为 `true` 时，快照文件名时间戳中的所有特殊字符将被替换。

> > **Note:** 此选项在用户界面中不直接可用，只能通过 `编辑为Yaml` 界面使用。

#### S3 备份

`S3` 备份目标允许用户配置兼容S3的后端来存储快照。 此选项的主要好处是，如果集群丢失所有etcd节点，则由于快照存储在外部，因此仍可以还原该集群。 Rancher建议使用诸如 `S3` 备份之类的外部目标，但是其配置要求确实需要付出额外的努力，应予以考虑。

| 选项                  | 描述                                                                      | 是否必须 |
| --------------------- | -------------------------------------------------------------------------------- | -------- |
| S3 Bucket名称         | 存储备份的S3 bucket名称                                      | \*       |
| S3 区域             | 备份S3 Bucket的区域                                                 |          |
| S3 区域终结点         | 备份S3 Bucket区域终结点                                        | \*       |
| S3 Access Key         | 有权限访问S3 Bucket的access key                       | \*       |
| S3 Secret Key         | 有权限访问S3 Bucket的secret key                       | \*       |
| Custom CA Certificate | 访问私有S3服务的自定义CA证书 _从v2.2.5版本开始支持_ |          |

##### 使用一个自定义CA证书访问S3

_从v2.2.5版本开始支持_

The backup snapshot can be stored on a custom `S3` backup like [minio](https://min.io/). If the S3 back end uses a self-signed or custom certificate, provide a custom certificate using the `Custom CA Certificate` option to connect to the S3 backend.

备份快照可以存储在自定义的 `S3` 备份中，例如[minio](https://min.io/)。 如果S3后端使用自签名或自定义证书，请使用 `自定义CA证书` 选项提供自定义证书以连接到S3后端。

## S3快照存储中使用IAM支持

除了使用API凭证外，`S3` 备份目标还支持对AWS API使用IAM身份验证。 IAM角色授予应用程序在对S3存储进行API调用时可以使用的临时权限。 要使用IAM身份验证，必须满足以下要求：

- 集群etcd节点必须具有实例角色，该角色具有对指定备份存储桶的读/写访问权限。
- 集群etcd节点必须对指定的S3端点具有网络访问权限。
- Rancher Server工作程序节点必须具有实例角色，该实例角色已对指定的备份存储桶进行读/写。
- Rancher Server工作程序节点必须具有对指定S3端点的网络访问权限。

要授予应用程序对S3的访问权限，请参阅[使用IAM角色向在Amazon EC2实例上运行的应用程序授予权限](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html)

## 查看可用的快照

集群的所有可用快照的列表均可用。

1.在**全局**视图中，导航到要查看快照的集群。

2.在导航栏中单击**工具 > 快照**，以查看已保存快照的列表。 这些快照包括创建时间的时间戳。
