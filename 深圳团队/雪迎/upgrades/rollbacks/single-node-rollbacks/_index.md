---
title: Docker 回滚
---

如果Rancher升级未成功完成，则必须回滚到 [Docker 升级](/docs/upgrades/upgrades/single-node-upgrade)之前使用的Rancher设置。回滚还原：

- 您之前的Rancher版本。
- 升级前已创建数据备份。

### 在你开始之前

回滚到较早版本的Rancher期间，您将输入一系列命令，用环境中的数据填充占位符。这些占位符用尖括号和所有大写字母（`<EXAMPLE>`）表示。这是带有占位符的命令示例：
```
docker pull rancher/rancher:<PRIOR_RANCHER_VERSION>
```

在此命令中， {`<PRIOR_RANCHER_VERSION>`} 是升级失败之前运行的Rancher的版本。例如`v2.0.5`。

请交叉参考下面的图像和参考表，以了解如何获取此占位符数据。在开始[以下步骤](#creating-a-backup)之前，写下或复制此信息。

<sup>终端 `docker ps` 命令，显示在何处找到{`<PRIOR_RANCHER_VERSION>`} 和 {`<RANCHER_CONTAINER_NAME>`}</sup>
![占位符参考](/img/rancher/placeholder-ref-2.png)

| 占位符                  | 例子        | 描述                                                |
| ---------------------------- | ----------------- | ------------------------------------------------------- |
| {`<PRIOR_RANCHER_VERSION>`}  | `v2.0.5`          | 升级之前使用的 rancher/rancher 镜像。      |
| {`<RANCHER_CONTAINER_NAME>`} | `festive_mestorf` | Rancher容器的名称。           |
| `<RANCHER_VERSION>`          | `v2.0.5`          | 备份所针对的Rancher版本。         |
| {`<DATE>`}                   | `9-27-18`         | 数据容器或备份的创建日期。 |

<br/>

您可以通过远程连接登录Rancher Server并输入命令以查看正在运行的容器：`docker ps`，从而获得 {`<PRIOR_RANCHER_VERSION>`} 和 {`<RANCHER_CONTAINER_NAME>`} 。您还可以查看使用其他命令停止的容器：`docker ps -a`。创建备份期间，随时可以使用这些命令获得帮助。

### 回滚 Rancher

如果您在升级Rancher时遇到问题，请通过拉起您使用的上一个版本，然后还原升级前所做的备份，将其恢复到最新的已知正常状态。


> **警告!** 升级到Rancher的先前版本会破坏对Rancher所做的所有更改。可能会发生不可恢复的数据丢失。



1. 使用远程终端连接，登录运行Rancher Server的节点。

1.提取升级前运行的Rancher版本。将{`<PRIOR_RANCHER_VERSION>`} 替换为[那个版本](#before-you-start).

   例如，如果升级前运行的是Rancher v2.0.5，请拉取v2.0.5。

   ```
   docker pull rancher/rancher:<PRIOR_RANCHER_VERSION>
   ```

1. 停止当前运行Rancher Server的容器。将{`<RANCHER_CONTAINER_NAME>`} 替换为Rancher容器的名称。

   ```
   docker stop <RANCHER_CONTAINER_NAME>
   ```

   您可以通过输入`docker ps`获得Rancher容器的名称。

1. 将在[Docker 升级](/docs/upgrades/upgrades/single-node-upgrade/) 压缩包 Server上。转到您将其移动到的目录。输入`dir`确认它在那里。



   如果您遵循我们在[Docker 升级](/docs/upgrades/upgrades/single-node-upgrade/)中建议的命名约定，则其命名类似于({`rancher-data-backup-<RANCHER_VERSION>-<DATE>.tar.gz`}).

1. 运行以下命令，将 `rancher-data`容器中的数据替换为备用压缩包中的数据，并替换[占位符](#before-you-start)，别忘了关闭引号。


   ```
   docker run  --volumes-from rancher-data \
   -v $PWD:/backup busybox sh -c "rm /var/lib/rancher/* -rf \
   && tar zxvf /backup/rancher-data-backup-<RANCHER_VERSION>-<DATE>.tar.gz"
   ```

1. 使用 {`<PRIOR_RANCHER_VERSION>`} 标记 [占位符](#before-you-start) 指向数据容器启动一个新的Rancher Server容器。

   ```
   docker run -d --volumes-from rancher-data \
   --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher:<PRIOR_RANCHER_VERSION>
   ```

   > **注意:** 不要在启动回滚后停止回滚，即使回滚过程似乎比预期的要长。停止回滚可能会在将来的升级期间导致数据库问题。

1.等待片刻，然后在Web浏览器中打开Rancher。确认回滚成功并且您的数据已还原。

**结果:** Rancher在升级之前会回滚到其版本和数据状态。
