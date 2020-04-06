---
title: Docker单容器安装恢复
---

如果遇到灾难场景，可以通过备份文件恢复Rancher Server数据。

## 恢复准备

在恢复备份期间，将输入很多命令。为了减少重复的参数输入，可以通过变量来定义某些固定的值。比如以下示例：

```bash
docker run \
--volumes-from ${RANCHER_CONTAINER_NAME} \
-v $PWD:/backup \
busybox \
sh -c "rm /var/lib/rancher/* -rf && \
tar pzxvf /backup/rancher-data-backup-${RANCHER_VERSION}-${DATE}"
```

在这个命令中, `${RANCHER_CONTAINER_NAME}`和`${RANCHER_VERSION>-<DATE>}` 是Rancher服务的环境变量。

终端输入`docker ps`命令，查看`RANCHER_CONTAINER_TAG`和`RANCHER_CONTAINER_NAME`
![Placeholder Reference](/img/rancher/placeholder-ref.png)

| 环境变量              | 值          | 说明                                             |
| ---------------------------- | ----------------- | --------------------------------------------------------- |
| `$RANCHER_CONTAINER_TAG` | `v2.0.5`          | 初始安装时获取的`rancher/rancher`镜像名。 |
| `$RANCHER_CONTAINER_NAME` | `festive_mestorf` | Rancher 容器名称.                 |
| `$RANCHER_VERSION`       | `v2.0.5`          | 备份的Rancher版本号。 |
| `$DATE`                  | `9-27-18`         | 创建数据容器或备份的日期。 |

## 恢复备份

使用您先前创建的[备份文件](/docs/backup /backup /single- nodes -backup /)，将Rancher恢复到已知的最后的健康状态。

1. 使用远程终端连接，登录到运行Rancher server的节点。

1. 停止当前正在运行的Rancher Server容器 `$RANCHER_CONTAINER_NAME`

   ```bash
   docker stop $RANCHER_CONTAINER_NAME
   ```

1. 将[Docker单容器安装备份](/docs/rancher/v2.x/en/backups/backups/single-node-backups/)保存的的压缩文件拷贝到rancher server所在主机上，通过`cd`命令切换到压缩文件所在的目录，并执行以下命令：

   > 如果您按照我们在[Docker单容器安装备份](/docs/rancher/v2.x/en/backups/backups/single-node-backups/)中建议的命名约定，它的名称将类似于`rancher-data-backup-<RANCHER_VERSION>-<DATE>.tar.gz`。

1. 输入以下命令删除当前集群数据，并将其替换为备份数据。

   > **警告! **此命令从Rancher server容器中删除所有的数据，上一次创建备份后保存的所有更改都将丢失。

   ```bash
   docker run  \
   --volumes-from <RANCHER_CONTAINER_NAME> \
   -v $PWD:/backup \
   busybox \
   sh -c "rm /var/lib/rancher/* -rf && \
   tar pzxvf /backup/rancher-data-backup-<RANCHER_VERSION>-<DATE>.tar.gz"
   ```

1. 重新启动Rancher Server容器。

   ```bash
   docker start $RANCHER_CONTAINER_NAME
   ```

1. 稍等片刻，然后在web浏览器中打开Rancher UI，确认是否成功恢复数据。
