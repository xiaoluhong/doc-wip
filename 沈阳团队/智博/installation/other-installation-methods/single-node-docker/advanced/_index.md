---
title: Advanced Options for Docker Installs
---
基于Docker安装的高级选项

When installing Rancher, there are several [advanced options](/docs/installation/options/) that can be enabled:
安装Rancher时，可以启用几个[高级选项](/docs/installation/options/)：

- [Custom CA Certificate](#custom-ca-certificate)
- [自定义CA证书](#custom-ca-certificate)
- [API Audit Log](#api-audit-log)
- [API审计](#api-audit-log)
- [TLS Settings](#tls-settings)
- [TLS配置](#tls-settings)
- [Air Gap](#air-gap)
- [私有环境](#air-gap)
- [Persistent Data](#persistent-data)
- [持久化数据](#persistent-data)
- [Running `rancher/rancher` and `rancher/rancher-agent` on the Same Node](#running-rancher-rancher-and-rancher-rancher-agent-on-the-same-node)
- [在相同节点运行`rancher/rancher` 和 `rancher/rancher-agent` ](#running-rancher-rancher-and-rancher-rancher-agent-on-the-same-node)

#### Custom CA Certificate

If you want to configure Rancher to use a CA root certificate to be used when validating services, you would start the Rancher container sharing the directory that contains the CA root certificate.
如果要配置Rancher使用的CA根证书，则应启动Rancher容器，共享包含CA根证书的目录。

Use the command example to start a Rancher container with your private CA certificates mounted.
使用命令示例来启动安装了私有CA证书的Rancher容器。

- The volume flag (`-v`) should specify the host directory containing the CA root certificates.
- The environment variable flag (`-e`) in combination with `SSL_CERT_DIR` and directory declares an environment variable that specifies the mounted CA root certificates directory location inside the container.
- Passing environment variables to the Rancher container can be done using `-e KEY=VALUE` or `--env KEY=VALUE`.
- Mounting a host directory inside the container can be done using `-v host-source-directory:container-destination-directory` or `--volume host-source-directory:container-destination-directory`.

- 卷标志（-v）应该指定包含CA根证书的主机目录。
- 环境变量标记（-e）与SSL_CERT_DIR和目录结合使用，声明一个环境变量，该变量指定容器内已安装的CA根证书目录的位置。
- 可以使用-e KEY = VALUE或--env KEY = VALUE将环境变量传递到Rancher容器。
- 使用`-v host-source-directory:container-destination-directory`或`--volume host-source-directory:container-destination-directory`，可以在容器内安装主机目录。

The example below is based on having the CA root certificates in the `/host/certs` directory on the host and mounting this directory on `/container/certs` inside the Rancher container.
以下示例基于将CA根证书放在主机上的“ / host / certs”目录中，并将此目录挂载到Rancher容器中的“ / container / certs”上。

```
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v /host/certs:/container/certs \
  -e SSL_CERT_DIR="/container/certs" \
  rancher/rancher:latest
```

#### API Audit Log
API审计日志

The API Audit Log records all the user and system transactions made through Rancher server.
API审计日志记录通过Rancher服务器进行的所有用户和系统事务。

The API Audit Log writes to `/var/log/auditlog` inside the rancher container by default. Share that directory as a volume and set your `AUDIT_LEVEL` to enable the log.
默认情况下，API审计日志会写入rancher容器内的`/ var / log / auditlog`中。 将该目录作为卷共享，并设置您的“ AUDIT_LEVEL”以启用日志。

See [API Audit Log](/docs/installation/api-auditing) for more information and options.
参考[API审计日志](/docs/installation/api-auditing)获取更多信息。

```
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v /var/log/rancher/auditlog:/var/log/auditlog \
  -e AUDIT_LEVEL=1 \
  rancher/rancher:latest
```

#### TLS settings
TLS设置

_Available as of v2.1.7_
v2.1.7以上可用

To set a different TLS configuration, you can use the `CATTLE_TLS_MIN_VERSION` and `CATTLE_TLS_CIPHERS` environment variables. For example, to configure TLS 1.0 as minimum accepted TLS version:
要设置其他TLS配置，您可以使用`CATTLE_TLS_MIN_VERSION`和`CATTLE_TLS_CIPHERS`环境变量。 例如，要将TLS 1.0配置为可接受的最低TLS版本：

```
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -e CATTLE_TLS_MIN_VERSION="1.0" \
  rancher/rancher:latest
```

See [TLS settings](/docs/admin-settings/tls-settings) for more information and options.

#### Air Gap

If you are visiting this page to complete an air gap installation, you must prepend your private registry URL to the server tag when running the installation command in the option that you choose. Add `<REGISTRY.DOMAIN.COM:PORT>` with your private registry URL in front of `rancher/rancher:latest`.
如果要访问此页面以完成私有安装，则在您选运行安装命令时，必须在server镜像之前添加私有镜像库的URL。 在您的rancher/rancher:latest前面添加带有您的私有镜像库URL的<REGISTRY.DOMAIN.COM:PORT>。

**Example:**

     <REGISTRY.DOMAIN.COM:PORT>/rancher/rancher:latest

#### Persistent Data
持久化数据

{{< persistentdata >}}

#### Running `rancher/rancher` and `rancher/rancher-agent` on the Same Node

In the situation where you want to use a single node to run Rancher and to be able to add the same node to a cluster, you have to adjust the host ports mapped for the `rancher/rancher` container.
在要使用单个节点运行Rancher并将同一个节点添加到集群的情况下，必须调整为“ rancher / rancher”容器映射的主机端口。

If a node is added to a cluster, it deploys the nginx ingress controller which will use port 80 and 443. This will conflict with the default ports we advise to expose for the `rancher/rancher` container.
如果将节点添加到集群中，它将部署使用端口80和443的nginx入口控制器。这将与我们建议为“ rancher / rancher”容器公开的默认端口冲突。

Please note that this setup is not recommended for production use, but can be convenient for development/demo purposes.
请注意，不建议将此设置用于生产环境，但可以方便地进行开发/演示。

To change the host ports mapping, replace the following part `-p 80:80 -p 443:443` with `-p 8080:80 -p 8443:443`:
要更改主机端口映射，请将以下部分`-p 80:80 -p 443:443`替换为`-p 8080:80 -p 8443:443`：

```
docker run -d --restart=unless-stopped \
  -p 8080:80 -p 8443:443 \
  rancher/rancher:latest
```
