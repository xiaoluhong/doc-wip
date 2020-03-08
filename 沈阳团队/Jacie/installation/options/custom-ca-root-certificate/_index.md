---
title: 自定义CA证书
---

如果您在内部生产环境使用Rancher，而不需要在公网暴露应用，可以使用私有证书颁发机构（CA）的证书。

如果Rancher访问的服务使用的是自定义/内部CA根证书（也称为自签名证书）进行配置的，如果Rancher无法验证服务中提供的证书，会显示如下的错误信息：`x509: certificate signed by unknown authority`.

要正常验证应用证书，需要将CA根证书添加到Rancher。由于Rancher是用Go编写的，我们可以使用环境变量`SSL_CERT_DIR`指向容器中CA根证书所在的目录，并且在启动Rancher容器时，可以使用`-v host-source-directory:container-destination-directory`挂载主机上CA根证书目录到容器中。

Rancher需要访问的服务如下:

- 自定义应用商店
- 第三方认证平台
- 使用节点驱动访问托管/云API

### 使用自定义CA证书安装

有关使用私有CA证书启动Rancher容器的详细信息，请参阅安装文档：

- [Docker 安装方式](/docs/installation/other-installation-methods/single-node-docker/#custom-ca-certificate)

- [Kubernetes 安装方式](/docs/installation/options/chart-options/#additional-trusted-cas)
