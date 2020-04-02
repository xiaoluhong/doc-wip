---
title: Docker单容器安装备份
---

在完成Rancher的单节点安装后，或在升级Rancher到新版本之前，需要对Rancher进行数据备份。如果在Rancher数据损坏或者丢失，或者升级遇到问题时，可以通过最新的备份进行数据恢复。

## 备份准备

在创建备份期间，将输入很多命令。为了减少重复的参数输入，可以通过变量来定义某些固定的值。比如以下示例：

```bash
docker run \
--volumes-from rancher-data-${DATE} \
-v $PWD:/backup busybox \
tar pzcvf /backup/rancher-data-backup-${RANCHER_VERSION}-${DATE}.tar.gz /var/lib/rancher
```

<sup>终端输入`docker ps`命令，查看`RANCHER_CONTAINER_TAG`和`RANCHER_CONTAINER_NAME`</sup>
![Placeholder Reference](/img/rancher/placeholder-ref.png)

| Placeholder                  | 值          | Description                                               |
| ---------------------------- | ----------------- | --------------------------------------------------------- |
| `$RANCHER_CONTAINER_TAG` | `v2.0.5`          | 当前安装的Rancher server镜像 |
| `$RANCHER_CONTAINER_NAME` | `festive_mestorf` | 当前Rancher容器名称。                       |
| `$RANCHER_VERSION`     | `v2.0.5`          | 您正在为其创建备份的Rancher版本。|
| `$DATE`                | `9-27-18`         | 创建数据卷容器或备份的日期。  |

## 创建备份

为Rancher创建备份，如果Rancher遇到灾难场景，可通过此备份恢复。

1. 使用远程终端连接，登录到运行Rancher server的节点。

1. 停止当前正在运行的Rancher server容器，注意替换<RANCHER_CONTAINER_NAME>。

   ```bash
   docker stop ${RANCHER_CONTAINER_NAME}
   ```

1. <a id="backup"></a>使用以下命令，基于刚刚停止的Rancher容器创建一个数据卷容器，注意替换<RANCHER_CONTAINER_NAME>、<DATE>和<RANCHER_CONTAINER_TAG>。

   ```bash
   docker create \
   --volumes-from ${RANCHER_CONTAINER_NAME} \
   --name rancher-data-${DATE} \
   rancher/rancher:${RANCHER_CONTAINER_TAG}
   ```

1. <a id="tarball"></a>使用以下命令，为刚刚创建的数据容器`rancher-data-${DATE}`再创建一个压缩包备份`rancher-data-backup-${RANCHER_VERSION}-${DATE}.tar.gz`。因为在升级期间，新的容器需要链接到数据卷容器，并且会对数据卷容器中的数据进行`更新/更改`。因此，需要提前对数据卷容器进行备份，以防升级失败时用于数据回滚。

   ```bash
   docker run  \
   --volumes-from rancher-data-<DATE> \
   -v $PWD:/backup:z busybox \
   tar pzcvf /backup/rancher-data-backup-${RANCHER_VERSION}-${DATE}.tar.gz /var/lib/rancher
   ```

1. 输入`ls`命令以确认备份压缩包已经创建。它将有一个类似`rancher-data-backup-${RANCHER_VERSION}-${DATE}.tar.gz`的名字。

1. 建议将备份压缩包拷贝到Rancher server服务器以外的安全位置。

1. 启动停止的Rancher server容器

   ```bash
   docker start ${RANCHER_CONTAINER_NAME}
   ```

1. 如果需要恢复数据，请访问[Docker单容器安装恢复](/docs/backups/restorations/single-node-restoration)