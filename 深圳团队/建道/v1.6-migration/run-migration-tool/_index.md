---
标题: 2. 迁移您的服务
---

尽管默认情况下v1.6提供的服务将无法在Rancher v2.x中运行，但这并不意味着您必须重新开始在v2.x中手动重建应用程序。 为了帮助从v1.6迁移到v2.x，Rancher开发了一个迁移工具。 迁移工具CLI是一个实用程序，可帮助您在Rancher v2.x中重新创建应用程序。 该工具将您的Rancher v1.6服务导出为Compose文件，并将它们转换为Rancher v2.x可以使用的Kubernetes清单。

此外，对于Kubernetes无法使用的每个特定于Rancher v1.6的Compose指令，迁移工具CLI均提供了有关如何在Rancher v2.x中手动重新创建它们的说明。

此命令行界面工具将：

- 导出v1.6 Cattle环境中每个堆栈的Compose文件（即`docker-compose.yml`和`rancher-compose.yml`）。 对于每个堆栈，文件都将导出到一个唯一的文件夹： `<EXPORT_DIR>/<ENV_NAME>/<STACK_NAME>`.

- 解析从Rancher v1.6堆栈导出的Compose文件并将其转换为Rancher v2.x可以使用的Kubernetes清单。 该工具还会输出无法自动转换为Rancher v2.x的Compose文件中存在的指令列表。 这些是您必须使用Rancher v2.x UI手动配置的指令。

### 大纲

<!-- TOC -->

- [A. 下载迁移工具CLI](#a-download-the-migration-tools-cli)
- [B. 配置迁移工具CLI](#b-configure-the-migration-tools-cli)
- [C. 运行迁移工具CLI](#c-run-the-migration-tools-cli)
- [D. 使用Rancher CLI部署服务](#d-re-deploy-services-as-kubernetes-manifests)
- [现在该怎么做?](#what-now)

<!-- /TOC -->

### A. 下载迁移工具CLI

可以下载适用于您平台的迁移工具CLI从我们的[GitHub发布页面](https://github.com/rancher/migration-tools/releases). 这些工具可用于Linux，Mac和Windows平台

### B. 配置迁移工具CLI

下载迁移工具CLI后，将其重命名并使其可执行。

1. 打开一个终端窗口，然后进入到包含迁移工具文件的目录。

1. 将文件重命名为`migration-tools`，使其不再包含平台名称。

1. 输入以下命令以使 `migration-tools` 可执行：

   ```
   chmod +x migration-tools
   ```

### C. 运行迁移工具CLI

接下来，使用迁移工具CLI将所有Cattle环境中的所有堆栈导出到Compose文件中。 然后，对于要迁移到Rancher v2.x的堆栈，将Compose文件转换为Kubernetes清单。

> **前提条件:** 创建一个 [Account API Key]({{< baseurl >}}/rancher/v1.6/en/api/v2-beta/api-keys/#account-api-keys) 使用迁移工具CLI时使用Rancher v1.6进行身份验证。

1. 从Rancher v1.6导出适用于Cattle环境和堆栈的Docker Compose文件。

   在终端窗口中，执行以下命令，将每个占位符替换为您的值。

   ```
   migration-tools export --url http://<RANCHER_URL:PORT> --access-key <RANCHER_ACCESS_KEY> --secret-key <RANCHER_SECRET_KEY> --export-dir <EXPORT_DIR> --all
   ```

   **步骤结果:** 迁移工具导出`--export-dir`目录中每个堆栈的Compose文件（`docker-compose.yml`和`rancher-compose.yml`）。 如果省略此选项，则 Compose 文件将输出到当前目录。

   为每个环境和堆栈创建一个唯一的目录。 例如，如果我们从Rancher v1.6中导出每个[environment/stack](/docs/v1.6-migration/#migration-example-files)， 则创建以下目录结构：

   ```
   export/                            # migration-tools --export-dir
   |--<ENVIRONMENT>/                  # Rancher v1.6 ENVIRONMENT
       |--<STACK>/                    # Rancher v1.6 STACK
            |--docker-compose.yml     # STANDARD DOCKER DIRECTIVES FOR ALL STACK SERVICES
            |--rancher-compose.yml    # RANCHER-SPECIFIC DIRECTIVES FOR ALL STACK SERVICES
            |--README.md              # README OF CHANGES FROM v1.6 to v2.x
   ```

1) 将导出的Compose文件转换为Kubernetes清单。

   执行以下命令，将每个占位符替换为堆栈的Compose文件的绝对路径。 如果要迁移多个堆栈，则必须为导出的每对Compose文件重新运行该命令。

   ```
   migration-tools parse --docker-file <DOCKER_COMPOSE_ABSOLUTE_PATH> --rancher-file <RANCHER_COMPOSE_ABSOLUTE_PATH>
   ```

   > **注意:** 如果您从命令中省略了`--docker-file`和 `--rancher-file`选项，则迁移工具将使用当前工作目录来查找Compose文件。

> **想要迁移工具CLI的完整用法和选项?** 查看 [迁移工具CLI参考](/docs/v1.6-migration/run-migration-tool/migration-tools-ref/).

#### 迁移工具CLI的输出

运行migration-tools parse命令后，以下文件将输出到目标目录。

| 输出                    | 描述                                                                                                                                                                                                                     |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `output.txt`              | 该文件列出了如何在Kubernetes中重新创建特定于Rancher v1.6的功能。 每个清单都链接到有关如何在Rancher v2.x中实现它的相关博客文章。                                                |
| Kubernetes manifest specs | 迁移工具在内部调用 [Kompose](https://github.com/kubernetes/kompose) 为您要迁移到v2.x的每个服务生成Kubernetes清单。 每个YAML规范文件均以您要迁移的服务命名。                    |

##### 为什么会有单独的部署和服务清单?

为了使应用程序可以通过URL公开访问，需要Kubernetes服务来支持部署。 Kubernetes服务是一个REST对象，它抽象化了对工作负载中Pod的访问。 换句话说，服务通过将URL映射到一个或多个pod来为pod提供一个静态端点。因此，即使pod更改了IP地址，公共端点也保持不变。 服务对象使用选择器标签指向其相应的deployment（工作负载）。

当您从Rancher v1.6导出公共端口的服务时，迁移工具CLI会将这些端口解析为Kubernetes服务规范，该规范链接到deployment YAML规范。

##### 迁移示例文件输出

如果我们从[迁移示例文件]（/ docs / v1.6-migration /＃migration-example-files）中解析两个示例文件`docker-compose.yml`和`rancher-compose.yml`，则以下文件输出：

| 文件                       | 描述                                                                                                |
| -------------------------- | ---------------------------------------------------------------------------------------------------------- |
| `web-deployment.yaml`      | 包含用于Let's Chat部署的Kubernetes容器规范的文件。                                  |
| `web-service.yaml`         | 包含Let's Chat 服务的规范的文件。                                                        |
| `database-deployment.yaml` | 该文件包含用于支持Let's Chat的MongoDB部署的容器规范。                     |
| `webLB-deployment.yaml`    | 包含用于充当负载均衡器的HAProxy部署的容器规范的文件.<sup>1</sup> |
| `webLB-service.yaml`       | 包含HAProxy服务规范的文件。<sup>1</sup>                                               |

> <sup>1</sup> 由于Rancher v2.x使用Ingress进行负载均衡，因此我们不会将Rancher v1.6负载均衡器迁移到v2.x。

<!--
The following tabs display the contents of each parsed file. We've omitted `webLB-deployment.yaml` and `webLB-service.yaml` because we aren't migrating them to v2.x.

 tabs 
 tab "web-deployment.yaml" 

```YAML
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    io.rancher.container.pull_image: always
    io.rancher.scheduler.global: "true"
    kompose.cmd: ./migration-tools parse --docker-file docker-compose.yml --rancher-file
      rancher-compose.yml
    kompose.version: 1.16.0 ()
  creationTimestamp: null
  labels:
    io.kompose.service: web
  name: web
spec:
  replicas: 0
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        io.kompose.service: web
    spec:
      containers:
      - image: sdelements/lets-chat
        name: web
        ports:
        - containerPort: 8080
        resources: {}
        stdin: true
        tty: true
      restartPolicy: Always
status: {}
```

 /tab 
 tab "web-service.yaml" 

```YAML
apiVersion: v1
kind: Service
metadata:
  annotations:
    io.rancher.container.pull_image: always
    io.rancher.scheduler.global: "true"
    kompose.cmd: ./migration-tools parse --docker-file docker-compose.yml --rancher-file
      rancher-compose.yml
    kompose.version: 1.16.0 ()
  creationTimestamp: null
  labels:
    io.kompose.service: web
  name: web
spec:
  ports:
  - name: "9890"
    port: 9890
    targetPort: 8080
  selector:
    io.kompose.service: web
status:
  loadBalancer: {}
```

 /tab 
 tab "database-deployment.yaml" 

```YAML
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    io.rancher.container.pull_image: always
    io.rancher.scheduler.affinity:host_label_soft: db=true
    kompose.cmd: ./migration-tools parse --docker-file docker-compose.yml --rancher-file
      rancher-compose.yml
    kompose.version: 1.16.0 ()
  creationTimestamp: null
  labels:
    io.kompose.service: database
  name: database
spec:
  replicas: 0
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        io.kompose.service: database
    spec:
      containers:
      - image: mongo
        name: database
        resources: {}
        stdin: true
        tty: true
      restartPolicy: Always
status: {}

```

 /tab 


 /tabs 

-->

### D. 以Kubernetes清单的形式重新部署服务

> **注意:** 尽管这些说明将您的v1.6服务部署在Rancher v2.x中，但在您调整Kubernetes清单之前，它们将无法正常工作。

 tabs 
 tab "Rancher UI" 

您可以通过将迁移工具创建的Kubernetes清单导入Rancher v2.x来进行部署。

> **接收到 `ImportYaml Error`?**
>
> 删除错误消息中列出的YAML指令。 这些是Kubernetes无法读取的来自v1.6服务的YAML指令。

<figcaption>Deploy Services: Import Kubernetes Manifest</figcaption>

![部署服务](/img/rancher/deploy-service.gif)

 /tab 
 tab "Rancher CLI" 

> **前提条件:** 为 Rancher v2.x [安装Rancher CLI](/docs/cli/) 

使用以下Rancher CLI命令让Rancher v2.x部署应用程序。 对于迁移工具CLI输出的每个Kubernetes清单，请输入以下命令之一，以将其导入Rancherv2.x。

```
./rancher kubectl create -f <DEPLOYMENT_YAML_FILE> # DEPLOY THE DEPLOYMENT YAML

./rancher kubectl create -f <SERVICE_YAML_FILE> # DEPLOY THE SERVICE YAML
```

 /tab 
 /tabs 


导入后，您可以使用上下文菜单选择包含服务的`<CLUSTER>> <PROJECT>`，从而在Kubernetes清单中以v2.x UI形式查看v1.6服务。 导入的清单将显示在**资源 > 工作负载**以及**资源 > 工作负载 > 服务发现**的选项卡上。（在v2.3.0之前的Rancher v2.x中，这些文件在**工作负载**和顶部导航栏中的**服务发现**标签上。）

<figcaption>Imported Services</figcaption>

![导入服务](/img/rancher/imported-workloads.png)

### 现在怎么做?

尽管迁移工具CLI将您的Rancher v1.6 Compose文件解析为Kubernetes清单，但是您必须通过手动编辑已解析的[Kubernetes清单](#output) 来解决v1.6和v2.x之间的差异。 换句话说，您需要编辑导入到Rancher v2.x中的每个工作负载和服务，如下所示。

<figcaption>Edit Migrated Services</figcaption>

![编辑迁移的工作负载](/img/rancher/edit-migration-workload.gif)


如[迁移工具CLI输出]（＃migration-tools-cli-output）中所述，解析过程中生成的output.txt文件列出了每个部署必须执行的手动步骤。 查看即将发布的议题，以获取有关手动编辑Kubernetes规范的更多信息。


打开您的`output.txt`文件并查看其内容。 将Compose文件解析为Kubernetes清单时，迁移工具CLI会为其Kubernetes创建的每个工作负载输出清单。 例如，当我们将[迁移示例文件]（/ docs / v1.6-migration /＃migration-example-files）解析为Kubernetes清单时，`output.txt`列出了每个解析后的[Kubernetes清单文件]（＃ 迁移示例文件输出）（即工作负载）。 每个工作负载均具有一系列操作项，以还原v2.x中针对该工作负载的操作。

<figcaption>Output.txt Example</figcaption>

![output.txt](/img/rancher/output-dot-text.png)

下表列出了可能在`output.txt`中出现的指令和它们的含义以及有关如何解析它们的链接。

| Directive         | Instructions                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [ports][4]        | Rancher v1.6 _端口映射_无法迁移到v2.x。 相反，您必须手动声明与端口映射类似的HostPort或NodePort。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| [health_check][1] | Rancher v1.6健康检查微服务已被本机Kubernetes健康状况检查所取代，称为_探针_。 使用探针在v2.0中重新创建v1.6运行健康状况检查。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| [labels][2]       | Rancher v1.6使用标签来实现v1.6中的各种功能。 在v2.x中，Kubernetes使用不同的机制来实现这些功能。 点击此处的链接以获取有关如何处理每个标签的说明。<br/><br/> [io.rancher.container.pull_image] [7]：在v1.6中，该标签说明部署的容器拉取新版本的镜像然后重启。 在v2.x中，此功能由`imagePullPolicy`指令代替。<br/> <br/> [io.rancher.scheduler.global] [8]：在v1.6中，该标签在每个集群主机上调度了一个容器副本。 在v2.x中，此功能由[守护进程副本集](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/).取代。<br/> <br/> [io.rancher.scheduler.affinity[9]：在v2.x中，affinity 是以不同的方式应用的。 |
| [links][3]        | 在迁移期间，您必须在Kubernetes工作负载和服务之间创建链接，以使其在v2.x中正常运行。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| [scale][5]        | 在v1.6中，规模是指在单个节点上运行的容器副本的数量。 在v2.x中，此功能由副本集代替。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| start_on_create   | 没有等效的Kubernetes。 您无需采取任何措施。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |

[1]: /docs/v1.6-migration/monitor-apps/#configuring-probes-in-rancher-v2-x
[2]: /docs/v1.6-migration/schedule-workloads/#scheduling-using-labels
[3]: /docs/v1.6-migration/discover-services
[4]: /docs/v1.6-migration/expose-services
[5]: /docs/v1.6-migration/schedule-workloads/#scheduling-pods-to-a-specific-node

<!-- MB: oops, skipped 6 -->

[7]: /docs/v1.6-migration/schedule-workloads/#scheduling-using-labels
[8]: /docs/v1.6-migration/schedule-workloads/#scheduling-global-services
[9]: /docs/v1.6-migration/schedule-workloads/#label-affinity-antiaffinity

#### [下一步: 暴露您的服务](/docs/v1.6-migration/expose-services/)
