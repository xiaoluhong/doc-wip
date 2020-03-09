---
title: 常见问题
---

#### 问题通常发生在哪里

大部分常见问题将在这3个名称空间中的对象上完成。

- `cattle-system` - `rancher` deployment and pods.
- `ingress-nginx` - Ingress controller pods and services.
- `kube-system` - `tiller` and `cert-manager` pods.

#### "default backend - 404"

多种原因可能导致ingress-controller无法将流量转发到您的rancher实例。 多数情况下，这是由于ssl配置错误所致。

检查事项

- [Rancher是否在运行](#is-rancher-running)
- [Cert CN is "Kubernetes Ingress Controller Fake Certificate"](#cert-cn-is-kubernetes-ingress-controller-fake-certificate)

#### Rancher是否在运行

使用`kubectl`来检查`cattle-system`系统名称空间，并查看Rancher容器是否处于Running状态。

```
kubectl -n cattle-system get pods

NAME                           READY     STATUS    RESTARTS   AGE
pod/rancher-784d94f59b-vgqzh   1/1       Running   0          10m
```

如果状态不是`Running`，则在容器上运行`describe`并检查事件。

```
kubectl -n cattle-system describe pod

...
Events:
  Type     Reason                 Age   From                Message
  ----     ------                 ----  ----                -------
  Normal   Scheduled              11m   default-scheduler   Successfully assigned rancher-784d94f59b-vgqzh to localhost
  Normal   SuccessfulMountVolume  11m   kubelet, localhost  MountVolume.SetUp succeeded for volume "rancher-token-dj4mt"
  Normal   Pulling                11m   kubelet, localhost  pulling image "rancher/rancher:v2.0.4"
  Normal   Pulled                 11m   kubelet, localhost  Successfully pulled image "rancher/rancher:v2.0.4"
  Normal   Created                11m   kubelet, localhost  Created container
  Normal   Started                11m   kubelet, localhost  Started container
```

#### 检查Rancher日志

使用`kubectl`查看pods列表

```
kubectl -n cattle-system get pods

NAME                           READY     STATUS    RESTARTS   AGE
pod/rancher-784d94f59b-vgqzh   1/1       Running   0          10m
```

使用`kubectl`和Pod名称来列出Pod中的日志。

```
kubectl -n cattle-system logs -f rancher-784d94f59b-vgqzh
```

#### Cert CN is "Kubernetes Ingress Controller Fake Certificate"

使用浏览器检查证书详细信息。 如果Common Name是`Kubernetes Ingress Controller伪证书`，则说明读取或颁发SSL证书时可能出了点问题。

> **注意事项:** 如果您使用LetsEncrypt颁发证书，则有时可能需要花一些时间来等待证书发布。

##### cert-manager issued certs (Rancher Generated or LetsEncrypt)

`cert-manager` 有3部分。

- `cert-manager` pod in the `kube-system` namespace.
- `Issuer` object in the `cattle-system` namespace.
- `Certificate` object in the `cattle-system` namespace.

对每个对象执行`kubectl describe`并检查事件。您可以追踪可能缺少的东西。

例如颁布证书存在问题:

```
kubectl -n cattle-system describe certificate
...
Events:
  Type     Reason          Age                 From          Message
  ----     ------          ----                ----          -------
  Warning  IssuerNotReady  18s (x23 over 19m)  cert-manager  Issuer rancher not ready
```

```
kubectl -n cattle-system describe issuer
...
Events:
  Type     Reason         Age                 From          Message
  ----     ------         ----                ----          -------
  Warning  ErrInitIssuer  19m (x12 over 19m)  cert-manager  Error initializing issuer: secret "tls-rancher" not found
  Warning  ErrGetKeyPair  9m (x16 over 19m)   cert-manager  Error getting keypair for CA issuer: secret "tls-rancher" not found
```

##### Bring Your Own SSL Certs


您的证书将直接应用于`cattle-system`名称空间中的Ingress对象。

检查Ingress对象的状态，并查看其是否准备就绪。

```
kubectl -n cattle-system describe ingress
```

如果其就绪并且SSL仍无法正常工作，则您的证书或密码可能格式错误。

检查nginx-ingress-controller日志。 由于nginx-ingress-controller的容器中有多个容器，因此您需要指定容器的名称。

```
kubectl -n ingress-nginx logs -f nginx-ingress-controller-rfjrq nginx-ingress-controller
...
W0705 23:04:58.240571       7 backend_ssl.go:49] error obtaining PEM from secret cattle-system/tls-rancher-ingress: error retrieving secret cattle-system/tls-rancher-ingress: secret cattle-system/tls-rancher-ingress was not found
```

#### no matches for kind "Issuer"

您选择的[SSL配置](/docs/installation/options/helm2/helm-rancher/#choose-your-ssl-configuration)要求在安装Rancher之前先安装[cert-manager](/docs/installation/options/helm2/helm-rancher/#optional-install-cert-manager) ，否则将显示以下错误:

```
Error: validation failed: unable to recognize "": no matches for kind "Issuer" in version "certmanager.k8s.io/v1alpha1"
```

安装[cert-manager](/docs/installation/options/helm2/helm-rancher/#optional-install-cert-manager) 并重新尝试安装Rancher。
