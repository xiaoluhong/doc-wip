---
标题: '4. 配置运行状况健康检查'
---

Rancher v1.6使用其自身的运行状况检查微服务在你的节点和服务上提供TCP和HTTP运行状况健康检查。 这些运行状况检查监控你的容器，以确认它们是否按预期运行。 如果一个容器没有通过健康检查，Rancher将销毁不健康的容器，然后复制一个健康的容器来替换它。

对于Rancher v2.x，我们已取代了运行状况检查微服务，而是利用Kubernetes的本机运行状况健康检查进行支持。

使用本文档修改Rancher v2.x的工作负载和服务并在 `output.txt`中列出`health_check`. 你可以通过配置活性探针（即健康检查）来修正它们。

例如，对于下面的图像，我们将为 `web` 和 `weblb` 工作负载配置活性探针（即迁移工具CLI输出的Kubernetes清单）。

<figcaption>Resolve <code>health_check</code> for the <code>web</code> and <code>webLB</code> Workloads</figcaption>

![解析 health_check](/img/rancher/resolve-health-checks.png)

### In This Document

<!-- TOC -->

- [Rancher v1.6 运行状况健康检查](#rancher-v1-6-health-checks)
- [Rancher v2.x 运行状况健康检查](#rancher-v2-x-health-checks)
- [在 Rancher v2.x 中配置探针](#configuring-probes-in-rancher-v2-x)

<!-- /TOC -->

### Rancher v1.6 运行状况健康检查

在Rancher v1.6中，你可以添加运行状况健康检查来监控特定服务的操作。 这些检查由Rancher运行状况检查微服务执行，该服务在与托管受监控服务的节点不同的节点容器中启动（但是，Rancher v1.6.20和更高版本也运行本地健康状况检查容器，作为另一个节点上主运行状况检查容器的冗余）。 健康检查设置存储在你堆栈的`rancher-compose.yml`文件中。

运行状况检查微服务具有两种类型的运行状况检查，它们具有超时，检查间隔等各种选项：

- **TCP 健康状况检查**:

这些运行状况健康检查将检查是否在指定端口为受监控服务打开了TCP连接。 有关详细信息，请参见 [Rancher v1.6 文档]({{< baseurl >}}/rancher/v1.6/en/cattle/health-checks/).

- **HTTP 健康状况检查**:

这些运行状况检查会监控对指定路径的HTTP请求，并检查响应是否为预期响应（与运行状况检查一起配置）。

下图显示了运行状况检查微服务，该服务评估运行Nginx的容器。 请注意，微服务正在跨节点进行检查。

![Rancher v1.6 健康状况检查](/img/rancher/healthcheck.svg)

### Rancher v2.x 运行状况健康检查

在Rancher v2.x中，运行状况检查微服务已被Kubernetes的本机运行状况健康检查机制_探针_取代。 这些探针类似于Rancher v1.6运行状况检查微服务，可监控Pod上TCP和HTTP的运行状况。

但是，Rancher v2.x中的探针有一些重要的区别，如下所述。 有关探头的完整详细信息，请参见 [Kubernetes文档](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#configure-probes).

#### 本地健康检查

与Rancher v1.6跨主机执行的运行状况健康检查不同，Rancher v2.x中的探针发生在由kubelet执行的_相同_主机上。

#### 多种探针类型

Kubernetes包含两种不同的探针类型：活性检查和就绪检查。

- **活性检查**:

检查受监控的容器是否正在运行。 如果探针报告失败，则Kubernetes将杀死Pod，然后根据部署重新启动它 [重新启动策略](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy).

- **就绪检查**:

检查容器是否准备好接受和服务请求。 如果探针报告失败，则从公众中隔离该pod，直到其自愈为止。

下图显示了kubelet在它们正在监控的容器上运行探针 ([kubelets](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) 是在每个节点上运行的主要 "agent"). 左侧的节点正在运行活性探针，而右侧的节点正在运行就绪检查。 请注意，kubelet正在扫描其主机节点上的容器，而不是像Rancher v1.6中那样跨节点扫描容器。

![Rancher v2.x 探针](/img/rancher/probes.svg)

### 在Rancher v2.x中配置探针

[迁移工具CLI](/docs/v1.6-migration/run-migration-tool/) 无法将运行状况检查从Compose文件解析为Kubernetes清单。 因此，如果要向Rancher v2.x工作负载添加运行状况健康检查，则必须手动添加它们。

使用Rancher v2.x UI可以向Kubernetes工作负载添加TCP或HTTP运行状况健康检查。 默认情况下，Rancher要求你为工作负载配置就绪检查，并使用相同的配置应用活性检查。可选，你可以定义单独的活性检查。

如果探针报告失败，那么将根据工作负载规范中定义的重新启动策略重新启动容器。 此设置等效于Rancher v1.6中的运行状况检查的策略参数。

编辑`output.txt`中调用的deployments时，使用**运行状况健康检查**部分配置探针。

<figcaption>Edit Deployment: Health Check Section</figcaption>

![状况健康检查部分](/img/rancher/health-check-section.png)

#### 配置检查

使用Rancher v2.x创建工作负载时，建议你配置检查以监控部署的Pod的运行状况。

 tabs 

 tab "TCP Check" 

TCP检查通过尝试指定的端口打开并与Pod的连接来监控deployment的运行状况。 如果探针可以打开端口，则认为它是健康的。 未能打开它被认为是不健康的，这会通知Kubernetes应该杀死该pod，然后更换它根据[重新启动策略](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy). (这适用于活性探针，对于就绪探针，它将标记pod为未就绪).

您可以通过选择**运行状况健康检查**部分中的**TCP连接成功打开**选项来配置探针以及指定对应行为的值。 有关更多信息，请参阅 [部署工作负载](/docs/k8s-in-rancher/workloads/deploy-workloads/).  有关设置探针超时和阈值的帮助，请参见 [运行状况健康检查参数映射](#health-check-parameter-mappings).

![TCP 检查](/img/rancher/readiness-check-tcp.png)

使用Rancher v2.x配置就绪检查时，会将`readinessProbe`指令和你设置的值添加到部署的Kubernetes清单中。 配置就绪检查还会自动向部署中添加活性检查（`livenessProbe`）。

<!--

```YAML
...
    - image: nginx
      imagePullPolicy: Always
      readinessProbe:           # ADDED DIRECTIVE
        failureThreshold: 3
        initialDelaySeconds: 10
        periodSeconds: 2
        successThreshold: 1
        tcpSocket:
          port: 80
        timeoutSeconds: 2
      livenessProbe:            # ADDED DIRECTIVE
        failureThreshold: 3
        initialDelaySeconds: 10
        periodSeconds: 2
        successThreshold: 1
        tcpSocket:
          port: 80
        timeoutSeconds: 2
 ```

-->

 /tab 

 tab "HTTP Check" 

HTTP检查通过将HTTP GET请求发送到你定义的特定URL路径来监控deployment的运行状况。 如果pod响应的消息范围为`200`-`400`，则认为健康检查成功。 如果Pod回复了其他任何值，则认为检查不成功，因此Kubernetes将终止并替换Pod根据Pod的 [重新启动策略](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy). (这适用于活性探针，对于就绪探针，它将标记容器为未就绪).

你可以通过选择**HTTP返回成功状态**或**HTTPS返回成功状态**来配置探针以及用于指定对应行为的值。 有关更多信息，请参见 [部署工作负载](/docs/k8s-in-rancher/workloads/deploy-workloads/). 有关设置探针超时和阈值的帮助，请参阅 [运行状况健康检查参数映射](#healthcheck-parameter-mappings).


![HTTP 检查](/img/rancher/readiness-check-http.png)

使用Rancher v2.x配置就绪检查时，会将`readinessProbe`指令和你设置的值添加到deployment的Kubernetes清单中。 配置就绪检查还会自动向部署中添加活性检查（`livenessProbe`）。

 /tab 

 /tabs 

#### 配置单独的活性检查

在为TCP或HTTP协议配置就绪检查时，你可以通过单击**定义单独的活性检查**来配置单独的活动检查。 有关设置探针超时和阈值的帮助，请参阅 [运行状况健康检查参数映射](#health-check-parameter-mappings).


![单独的活性检查](/img/rancher/separate-check.png)

#### 其他探针选项

与v1.6一样，Rancher v2.x允许你使用TCP和HTTP协议执行运行状况健康检查。 但是，Rancher v2.x还允许你通过在Pod内运行命令来检查其状态。 如果在运行该命令后容器以代码`0`退出，则该容器被认为是健康的。

你可以配置活性检查或就绪检查，以执行指定的命令，方法是从 **运行状况健康检查**中选择`Command run inside the container exits with status 0` [部署工作负载](/docs/k8s-in-rancher/workloads/deploy-workloads/).


![运行状况健康检查执行命令](/img/rancher/healthcheck-cmd-exec.png)

##### 运行状况健康检查参数映射

在配置就绪检查和活性检查时，Rancher会提示你填写各种超时和阈值，这些值和值确定探针是成功还是失败。 下表中的参考表显示了Rancher v1.6中的等效运行状况检查值。

| Rancher v1.6 构成参数 | Rancher v2.x Kubernetes 参数 |
| ------------------------------ | --------------------------------- |
| `port`                         | `tcpSocket.port`                  |
| `response_timeout`             | `timeoutSeconds`                  |
| `healthy_threshold`            | `failureThreshold`                |
| `unhealthy_threshold`          | `successThreshold`                |
| `interval`                     | `periodSeconds`                   |
| `initializing_timeout`         | `initialDelaySeconds`             |
| `strategy`                     | `restartPolicy`                   |

#### [下一步: 调度你的服务](/docs/v1.6-migration/schedule-workloads/)
