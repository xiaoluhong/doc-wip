---
title: 安装 Docker
---

任何运行Rancher server的节点上都需要安装Docker。

有两种安装Docker的选项。一种选择是参考[官方Docker文档](https://docs.docker.com/install/)来了解如何在Linux上安装Docker。这些安装步骤将根据Linux发行版而有所不同。

另一种选择是使用Rancher提供的Docker安装脚本，该脚本可用于Docker的最新版本。

例如，此命令可用于在Ubuntu上安装Docker 18.09：

```
curl https://releases.rancher.com/install-docker/18.09.sh | sh
```

要了解某个Docker版本是否有可用的安装脚本，请参考这个[GitHub存储库，](https://github.com/rancher/install-docker)这里包含了Rancher的所有Docker安装脚本。
