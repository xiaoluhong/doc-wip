---
title: HTTP Proxy Configuration
---
HTTP代理配置

If you operate Rancher behind a proxy and you want to access services through the proxy (such as retrieving catalogs), you must provide Rancher information about your proxy. As Rancher is written in Go, it uses the common proxy environment variables as shown below.
如果您在代理后面操作Rancher，并且想要通过代理访问服务（例如检索catalogs），则必须提供有关代理的Rancher信息。 由于Rancher是用Go编写的，因此它使用了常见的代理环境变量，如下所示。

Make sure `NO_PROXY` contains the network addresses, network address ranges and domains that should be excluded from using the proxy.
确保“ NO_PROXY”包含不应使用代理的网络地址，网络地址范围和域。

| Environment variable | Purpose                                                                                                                 |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| HTTP_PROXY           | Proxy address to use when initiating HTTP connection(s)                                                                 |
| HTTPS_PROXY          | Proxy address to use when initiating HTTPS connection(s)                                                                |
| NO_PROXY             | Network address(es), network address range(s) and domains to exclude from using the proxy when initiating connection(s) |

| HTTP_PROXY           | HTTP连接的代理地址                                                                |
| HTTPS_PROXY          | HTTPS连接的代理地址                                                               |
| NO_PROXY             | 不使用代理的网络地址，网络地址范围和域 |
> **Note** NO_PROXY must be in uppercase to use network range (CIDR) notation.
> **注意** NO_PROXY必须为大写才能使用网络范围（CIDR）表示法。

### Docker Installation
基于Docker安装

Passing environment variables to the Rancher container can be done using `-e KEY=VALUE` or `--env KEY=VALUE`. Required values for `NO_PROXY` in a [Docker Installation](/docs/installation/single-node-install/) are:
可以使用-e KEY = VALUE或--env KEY = VALUE将环境变量传递到Rancher容器。 [Docker安装](/docs/installation/single-node-install/)中的NO_PROXY的必需值为：

- `localhost`
- `127.0.0.1`
- `0.0.0.0`
- `10.0.0.0/8`

The example below is based on a proxy server accessible at `http://192.168.0.1:3128`, and excluding usage the proxy when accessing network range `192.168.10.0/24` and every hostname under the domain `example.com`.
以下示例基于可从“ http://192.168.0.1:3128”访问的代理服务器，并排除了访问网络范围“ 192.168.10.0/24”以及域“ example.com”下的每个主机名时代理的使用情况 。

```
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -e HTTP_PROXY="http://192.168.10.1:3128" \
  -e HTTPS_PROXY="http://192.168.10.1:3128" \
  -e NO_PROXY="localhost,127.0.0.1,0.0.0.0,10.0.0.0/8,192.168.10.0/24,example.com" \
  rancher/rancher:latest
```
