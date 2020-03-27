---
title: 增加TLS密文
---

当我们在 `cattle-system` 命名空间，将自签名证书和密钥配置到 `tls-rancher-ingress` 的密文中，Kubernetes才会为Rancher创建所有的对象和服务。

将服务器证书和任何所需的中间证书合并到名为 `tls.crt` 的文件中，将您的证书密钥拷贝到名称为 `tls.key` 的文件中。

使用 `kubectl` 来创建 `tls` 类型的密文。

```
kubectl -n cattle-system create secret tls tls-rancher-ingress \
  --cert=tls.crt \
  --key=tls.key
```

> **提示:** 如果您想要更换证书，您可以使用 `kubectl -n cattle-system delete secret tls-rancher-ingress` 来删除 `tls-rancher-ingress` 密文，之后使用上面的命令创建一个新的密文。如果您使用的是私有CA签发的证书，仅当新证书与当前证书是由同一个CA签发的，才可以替换。

#### 使用私有CA签发证书

如果您使用的是私有CA，Rancher需要您提供CA证书的副本，用来校验Rancher Agent与Server的连接。

拷贝CA证书到名为 `cacerts.pem` 的文件，使用 `kubectl` 命令在 `cattle-system` 命名空间中创建名为 `tls-ca` 的密文。

> **重要:** 请确保文件名称为 `cacerts.pem`，因为Rancher使用该文件名来配置CA证书。

```
kubectl -n cattle-system create secret generic tls-ca \
  --from-file=cacerts.pem
```
