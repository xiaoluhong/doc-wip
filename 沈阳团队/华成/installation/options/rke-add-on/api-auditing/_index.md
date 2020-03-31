---
标题: 启用API Auditing
---

> **重要说明: Rancher v2.0.8之前仅支持RKE附加安装**
>
> 请使用Rancher Helm chart在Kubernetes集群上安装Rancher。有关详细信息，请参见[Kubernetes安装-安装概述](/docs/installation/k8s-install/#installation-outline)。
>
> 如果您当前正在使用RKE附加组件安装方法，请参阅[使用RKE附加组件从Kubernetes安装迁移](/docs/upgrades/upgrades/migrating-from-rke-add-on/)以获取有关如何使用Helm chart的详细信息。

如果使用RKE安装Rancher，则可以使用指令为Rancher安装启用API Auditing。您可以知道发生了什么，何时发生，由谁发起的以及它影响了哪些集群。API Auditing记录了与Rancher API之间的所有请求和响应，包括对Rancher UI的使用以及通过程序使用对Rancher API的任何其他使用。

### 内联参数

通过向Rancher容器添加参数来使用RKE启用API Auditing。

要启用API Auditing：

- 将API Auditing参数 (`args`) 添加到Rancher容器中。
- 在容器的 `volumeMounts` 指令中声明一个 `mountPath`。
- 在 `volumes` 指令中声明一个 `path`。

有关每个参数，其语法以及如何查看API Audit日志的更多信息，请参阅[Rancher v2.0文档：API Auditing](/docs/installation/api-auditing)。

```yaml
...
containers:
        - image: rancher/rancher:latest
          imagePullPolicy: Always
          name: cattle-server
          args: ["--audit-log-path", "/var/log/auditlog/rancher-api-audit.log", "--audit-log-maxbackup", "5", "--audit-log-maxsize", "50", "--audit-level", "2"]
          ports:
          - containerPort: 80
            protocol: TCP
          - containerPort: 443
            protocol: TCP
          volumeMounts:
          - mountPath: /etc/rancher/ssl
            name: cattle-keys-volume
            readOnly: true
          - mountPath: /var/log/auditlog
            name: audit-log-dir
        volumes:
        - name: cattle-keys-volume
          secret:
            defaultMode: 420
            secretName: cattle-keys-server
        - name: audit-log-dir
          hostPath:
            path: /var/log/rancher/auditlog
            type: Directory
```
