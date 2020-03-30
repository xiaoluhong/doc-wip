---
title: 设置NGINX负载均衡器
---

我们将使用NGINX作为4层负载均衡器(TCP)，它将连接转发到您的某一台Rancher节点。

> **注意：**
> 在此配置中，负载均衡器位于节点的前面。负载平衡器可以是任何能够运行NGINX的主机。
>
> 一个警告：不要使用任意一个Rancher节点作为负载均衡器节点，这会出现端口冲突。

### 安装 NGINX

首先在负载均衡器主机上安装NGINX，NGINX具有适用于所有已知操作系统的软件包。我们测试了`1.14`和`1.15`版本。有关安装NGINX的帮助，请参阅[安装文档。](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)

`stream`模块是必需的，在安装官方NGINX软件包时提供。请参阅您的操作系统文档来了解如何在操作系统上安装和启用NGINX `stream`模块。

### 创建NGINX配置

安装NGINX之后，您需要使用节点的IP地址更新NGINX配置文件`nginx.conf`。

1. 将下面的配置示例复制并粘贴到您喜欢的文本编辑器中，保存为`nginx.conf`。

2. 在nginx.conf配置中，用[节点](/docs/installation/k8s-install/create-nodes-lb/)的IP替换 `<IP_NODE_1>`，`<IP_NODE_2>`和`<IP_NODE_3>`。

    > **注意:** 有关所有配置选项，请参见[NGINX文档：TCP和UDP负载均衡。](https://docs.nginx.com/nginx/admin-guide/load-balancer/tcp-udp-load-balancer/)

    <figcaption>NGINX配置示例</figcaption>
    ```
    worker_processes 4;
    worker_rlimit_nofile 40000;

    events {
    worker_connections 8192;
    }

    stream {
    upstream rancher_servers_http {
    least_conn;
    server <IP_NODE_1>:80 max_fails=3 fail_timeout=5s;
    server <IP_NODE_2>:80 max_fails=3 fail_timeout=5s;
    server <IP_NODE_3>:80 max_fails=3 fail_timeout=5s;
    }
    server {
    listen 80;
    proxy_pass rancher_servers_http;
    }

        upstream rancher_servers_https {
            least_conn;
            server <IP_NODE_1>:443 max_fails=3 fail_timeout=5s;
            server <IP_NODE_2>:443 max_fails=3 fail_timeout=5s;
            server <IP_NODE_3>:443 max_fails=3 fail_timeout=5s;
        }
        server {
            listen     443;
            proxy_pass rancher_servers_https;
        }

    }

    ```

    ```

3. 将`nginx.conf`保存到`/etc/nginx/nginx.conf`。

4. 运行以下命令重新加载NGINX配置：

    ```
    # nginx -s reload
    ```

### 可选 - 将NGINX作为Docker容器运行

与其将NGINX作为软件包安装在操作系统上，还不如将其作为Docker容器运行。将已编辑的**示例NGINX配置**另存为`/etc/nginx.conf`并运行以下命令启动NGINX容器：

```
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v /etc/nginx.conf:/etc/nginx/nginx.conf \
  nginx:1.14
```
