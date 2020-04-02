---
title: Chart 选项
---

#### 通用选项

| Option                    | Default Value | Description                                                                        |
| ------------------------- | ------------- | ---------------------------------------------------------------------------------- |
| `hostname`                | " "           | `string` - the Fully Qualified Domain Name for your Rancher Server                 |
| `ingress.tls.source`      | "rancher"     | `string` - Where to get the cert for the ingress. - "rancher, letsEncrypt, secret" |
| `letsEncrypt.email`       | " "           | `string` - Your email address                                                      |
| `letsEncrypt.environment` | "production"  | `string` - Valid options: "staging, production"                                    |
| `privateCA`               | false         | `bool` - Set to true if your cert is signed by a private CA                        |

<br/>

#### 高级选项

| Option                         | Default Value                                         | Description                                                                                                                                       |
| ------------------------------ | ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `additionalTrustedCAs`         | false                                                 | `bool` - See [Additional Trusted CAs](#additional-trusted-cas)                                                                                    |
| `addLocal`                     | "auto"                                                | `string` - Have Rancher detect and import the "local" Rancher server cluster [Import "local Cluster](#import-local-cluster)                       |
| `antiAffinity`                 | "preferred"                                           | `string` - AntiAffinity rule for Rancher pods - "preferred, required"                                                                             |
| `auditLog.destination`         | "sidecar"                                             | `string` - Stream to sidecar container console or hostPath volume - "sidecar, hostPath"                                                           |
| `auditLog.hostPath`            | "/var/log/rancher/audit"                              | `string` - log file destination on host (only applies when `auditLog.destination` is set to `hostPath`)                                           |
| `auditLog.level`               | 0                                                     | `int` - set the [API Audit Log](/docs/installation/api-auditing) level. 0 is off. [0-3]                                                           |
| `auditLog.maxAge`              | 1                                                     | `int` - maximum number of days to retain old audit log files (only applies when `auditLog.destination` is set to `hostPath`)                      |
| `auditLog.maxBackups`          | 1                                                     | `int` - maximum number of audit log files to retain (only applies when `auditLog.destination` is set to `hostPath`)                               |
| `auditLog.maxSize`             | 100                                                   | `int` - maximum size in megabytes of the audit log file before it gets rotated (only applies when `auditLog.destination` is set to `hostPath`)    |
| `busyboxImage`                 | "busybox"                                             | `string` - Image location for busybox image used to collect audit logs _Note: Available as of v2.2.0_                                             |
| `debug`                        | false                                                 | `bool` - set debug flag on rancher server                                                                                                         |
| `extraEnv`                     | []                                                    | `list` - set additional environment variables for Rancher _Note: Available as of v2.2.0_                                                          |
| `imagePullSecrets`             | []                                                    | `list` - list of names of Secret resource containing private registry credentials                                                                 |
| `ingress.extraAnnotations`     | {}                                                    | `map` - additional annotations to customize the ingress                                                                                           |
| `ingress.configurationSnippet` | ""                                                    | `string` - Add additional Nginx configuration. Can be used for proxy configuration. _Note: Available as of v2.0.15, v2.1.10 and v2.2.4_           |
| `proxy`                        | ""                                                    | `string` - HTTP[S] proxy server for Rancher                                                                                                       |
| `noProxy`                      | "127.0.0.0/8,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16" | `string` - comma separated list of hostnames or ip address not to use the proxy                                                                   |
| `resources`                    | {}                                                    | `map` - rancher pod resource requests & limits                                                                                                    |
| `rancherImage`                 | "rancher/rancher"                                     | `string` - rancher image source                                                                                                                   |
| `rancherImageTag`              | same as chart version                                 | `string` - rancher/rancher image tag                                                                                                              |
| `tls`                          | "ingress"                                             | `string` - See [External TLS Termination](#external-tls-termination) for details. - "ingress, external"                                           |
| `systemDefaultRegistry`        | ""                                                    | `string` - private registry to be used for all system Docker images, e.g., http://registry.example.com/ _Available as of v2.3.0_                  |
| `useBundledSystemChart`        | `false`                                               | `bool` - select to use the system-charts packaged with Rancher server. This option is used for air gapped installations. _Available as of v2.3.0_ |

<br/>

#### API审计日志

开启 [API审计日志](/docs/installation/api-auditing/)。

您可以像收集任何容器日志一样收集此日志。 为Rancher Server集群上的`System`项目启用 [Logging service under Rancher Tools](/docs/tools/logging/)。

```plain
--set auditLog.level=1
```

默认情况下，启用`审计日志`将在Rancher pod中创建一个sidecar容器。 这个容器（`rancher-audit-log`）会将日志流传输到`stdout`。 您可以像收集任何容器日志一样收集此日志。 将sidecar用作审计日志时，`hostPath`，`maxAge`，`maxBackups`和`maxSize`选项不适用。 建议使用您的操作系统或Docker守护进程的日志轮换功能来控制磁盘空间的使用。 为Rancher Server集群或系统项目启用[Rancher Tools下的日志记录服务](/docs/tools/logging/)。

将`auditLog.destination`设置为`hostPath`的值，以将日志转发至与主机系统共享的卷，而不是流至Sidecar容器。 将目标设置为`hostPath`时，您可能需要调整其他auditLog参数以进行日志轮换。

#### 设置扩展环境变量

_自v2.2.0起可用_

您可以使用`extraEnv`为Rancher Server设置额外的环境变量。 该列表使用与容器清单定义相同的`name`和`value`键。 记住需要给值加上双引号。

```plain
--set 'extraEnv[0].name=CATTLE_TLS_MIN_VERSION'
--set 'extraEnv[0].value=1.0'
```

#### TLS设置

_自v2.2.0起可用_

要设置不同的TLS配置，可以使用`CATTLE_TLS_MIN_VERSION`和`CATTLE_TLS_CIPHERS`环境变量。 例如，将TLS 1.0配置为接受的最低TLS版本:

```plain
--set 'extraEnv[0].name=CATTLE_TLS_MIN_VERSION'
--set 'extraEnv[0].value=1.0'
```

参阅 [TLS 配置](/docs/admin-settings/tls-settings) 获取更多信息和选项。

#### 导入`local`集群

默认情况下，Rancher服务器将检测并导入正在运行的`local`集群。有权访问`local`集群的用户实际上将具有对Rancher Server管理的所有集群的`root`访问权。

如果您的环境中存在此问题，则可以在初次安装时将此选项设置为`false`。

> 注意事项: 此选项仅对Rancher的初始安装有效。 参阅 [Issue 16522](https://github.com/rancher/rancher/issues/16522) 获取更多信息。

```plain
--set addLocal="false"
```

#### 自定义您的Ingress

要使用Rancher Server自定义或使用其他Ingress，您可以设置自己的Ingress注释。

设置自定义证书颁发者的示例:

```plain
--set ingress.extraAnnotations.'certmanager\.k8s\.io/cluster-issuer'=ca-key-pair
```

_v2.0.15, v2.1.10 和 v2.2.4可用_

使用`ingress.configurationSnippet`设置静态代理头的示例。 该值像模板一样进行解析，因此可以使用变量。

```plain
--set ingress.configurationSnippet='more_set_input_headers X-Forwarded-Host {{ .Values.hostname }};'
```

#### HTTP代理

Rancher需要Internet访问才能使用某些功能 (helm charts)。 使用`proxy`设置您的代理服务器。

在`noProxy`添加例外的IP。确保添加了Service cluster IP(默认: 10.43.0.1/16)和任何worker集群`controlplane`节点。Rancher在此列表中支持CIDR范围表示法。

```plain
--set proxy="http://<username>:<password>@<proxy_url>:<proxy_port>/"
--set noProxy="127.0.0.0/8\,10.0.0.0/8\,172.16.0.0/12\,192.168.0.0/16"
```

#### 附加授信CAs

如果您有私有registries，catalogs或拦截证书的代理，则可能需要向Rancher添加额外的受信任的CA。

```plain
--set additionalTrustedCAs=true
```

创建完Rancher deployment后，将pem格式的CA证书复制到一个名为`ca-additional.pem`的文件中，并使用`kubectl`在`cattle-system`命名空间中创建`tls-ca-additional` secret。

```plain
kubectl -n cattle-system create secret generic tls-ca-additional --from-file=ca-additional.pem
```

#### 私有镜像仓库(Registry)和离线安装

有关使用私有registry安装Rancher的详细信息，请参阅:

- [离线环境: 安装Docker](/docs/installation/air-gap-single-node/)
- [离线环境: 安装Kubernetes](/docs/installation/air-gap-high-availability/)

#### 外部 TLS Termination

我们建议将负载平衡器配置为4层平衡器，将普通80/tcp和443/tcp转发到Rancher管理集群节点。 集群上的Ingress Controller会将端口80上的http通信重定向到端口443上的https。

您可以在Rancher集群（ingress）外部的L7负载平衡器上终止SSL/TLS。 使用`--set tls=external`选项，将负载均衡器指向所有Rancher集群节点上的端口http 80。 这将在http端口80上公开Rancher接口。 请注意，允许直接连接到Rancher集群的客户端将不会被加密。 如果您选择这样做，我们建议您将网络级别上的直接访问限制为仅用于您的负载均衡器。

> **注意事项:** 如果您使用的是专用CA签名的证书，请添加`--set privateCA=true`并参阅[添加 TLS Secrets - 使用私有的CA签名证书](/docs/installation/options/helm2/helm-rancher/tls-secrets/#using-a-private-ca-signed-certificate)来完成给Rancher添加CA证书。

您的负载均衡器必须支持长期存在的Websocket连接，并且需要插入代理标头，以便Rancher可以正确路由链接。

##### 使用NGINX v0.25为外部TLS配置Ingress

在NGINX v0.25中，关于转发头和外部TLS Termination，NGINX的行为已[更改](https://github.com/kubernetes/ingress-nginx/blob/master/Changelog.md#0220)。因此，在将外部TLS Termination配置与NGINX v0.25结合使用的情况下，必须编辑`cluster.yml`来启用用于ingress的`use-forwarded-headers`选项:

```yaml
ingress:
  provider: nginx
  options:
    use-forwarded-headers: 'true'
```

##### 必须的头部

- `Host`
- `X-Forwarded-Proto`
- `X-Forwarded-Port`
- `X-Forwarded-For`

##### 建议的超时时间

- Read Timeout: `1800 seconds`
- Write Timeout: `1800 seconds`
- Connect Timeout: `30 seconds`

##### 健康检查

Rancher将对`/healthz`端点上的健康检查响应`200`。

##### NGINX配置示例

此NGINX配置已在NGINX 1.14上进行了测试。

> **注意事项:** 此NGINX配置只是一个示例，可能不适合您的环境。 完整文档请参阅[NGINX负载平衡 - HTTP负载平衡](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)。

- 将`IP_NODE1`，`IP_NODE2`和`IP_NODE3`替换为集群中节点的IP地址。
- 将两个出现的`FQDN`替换为Rancher的DNS名称。
- 将`/certs/fullchain.pem`和`/certs/privkey.pem`分别替换为服务器证书和服务器证书密钥的位置。

```
worker_processes 4;
worker_rlimit_nofile 40000;

events {
    worker_connections 8192;
}

http {
    upstream rancher {
        server IP_NODE_1:80;
        server IP_NODE_2:80;
        server IP_NODE_3:80;
    }

    map $http_upgrade $connection_upgrade {
        default Upgrade;
        ''      close;
    }

    server {
        listen 443 ssl http2;
        server_name FQDN;
        ssl_certificate /certs/fullchain.pem;
        ssl_certificate_key /certs/privkey.pem;

        location / {
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-Port $server_port;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://rancher;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection $connection_upgrade;
            # This allows the ability for the execute shell window to remain open for up to 15 minutes. Without this parameter, the default is 1 minute and will automatically close.
            proxy_read_timeout 900s;
            proxy_buffering off;
        }
    }

    server {
        listen 80;
        server_name FQDN;
        return 301 https://$server_name$request_uri;
    }
}
```
