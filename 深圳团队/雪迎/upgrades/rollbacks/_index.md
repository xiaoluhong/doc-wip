---
title: 回滚
---

本节包含有关如何将Rancher服务器回滚到以前版本的信息。



- [回滚使用Docker安装的Rancher](/docs/upgrades/rollbacks/single-node-rollbacks/)
- [回滚安装在Kubernetes集群上的Rancher](/docs/upgrades/rollbacks/ha-server-rollbacks/)

#### 有关回滚的特殊方案

如果要在这两种情况下都还原到版本，则必须遵循一些额外的说明才能使集群正常工作。

- 从v2.1.6 +回滚到v2.1.0-v2.1.5或v2.0.0-v2.0.10之间的任何版本。
- 从v2.0.11 +回滚到v2.0.0-v2.0.10之间的任何版本。

由于要解决[CVE-2018-20321](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-20321), 所需的更改，因此如果用户需要，则需要采取特殊步骤回滚到存在此漏洞的Rancher的早期版本。步骤如下：

1.记录每个集群的`serviceAccountToken`。为此，将以下脚本保存在具有对Rancher管理平面的`kubectl`访问权限的计算机上并执行它。您将需要在运行rancher容器的计算机上运行这些命令。在运行命令之前，请确保已安装JQ。这些命令将取决于您如何安装Rancher。


   **使用Docker安装的Rancher**
  
    ```
    docker exec <NAME OF RANCHER CONTAINER> kubectl get clusters -o json | jq '[.items[] | select(any(.status.conditions[]; .type == "ServiceAccountMigrated")) | {name: .metadata.name, token: .status.serviceAccountToken}]' > tokens.json
    ```

  **在Kubernetes集群上的Rancher**

    ```
    kubectl get clusters -o json | jq '[.items[] | select(any(.status.conditions[]; .type == "ServiceAccountMigrated")) | {name: .metadata.name, token: .status.serviceAccountToken}]' > tokens.json
    ```

2. 执行命令后，将创建一个 `tokens.json` 文件。重要！在安全的地方备份此文件。**回滚Rancher后，需要使用它来将功能恢复到集群。** 如果丢失此文件，则可能无法访问集群。\*\*

3. 按照[正常说明](/docs/upgrades/rollbacks/)回滚Rancher。

4. 一旦Rancher恢复正常，由Rancher管理的每个集群（导入集群除外）将处于`Unvailable`状态。


5. 根据您安装Rancher的方式应用备份的令牌。

   **使用Docker安装的Rancher**

   将以下脚本另存为`apply_tokens.sh`到运行Rancher docker容器的机器上。还将先前创建的`tokens.json`文件复制到脚本所在的目录。


   ```
   set -e

   tokens=$(jq .[] -c tokens.json)
   for token in $tokens; do
       name=$(echo $token | jq -r .name)
       value=$(echo $token | jq -r .token)

       docker exec $1 kubectl patch --type=merge clusters $name -p "{\"status\": {\"serviceAccountToken\": \"$value\"}}"
   done
   ```

   允许执行的脚本(`chmod +x apply_tokens.sh`) 并执行脚本，如下所示：

   ```
   ./apply_tokens.sh <DOCKER CONTAINER NAME>
   ```

   片刻之后，集群将从“Unavailable”回到“Available”。


   **Rancher安装在Kubernetes集群上**

   将以下脚本另存为`apply_tokens.sh`到具有对Rancher管理平面的kubectl访问权限的计算机。还将先前创建的 `tokens.json`文件复制到脚本所在的目录。

   ```
   set -e

   tokens=$(jq .[] -c tokens.json)
   for token in $tokens; do
       name=$(echo $token | jq -r .name)
       value=$(echo $token | jq -r .token)

      kubectl patch --type=merge clusters $name -p "{\"status\": {\"serviceAccountToken\": \"$value\"}}"
   done
   ```

   设置脚本以允许执行(`chmod +x apply_tokens.sh`) ，然后执行脚本，如下所示：
   ```
   ./apply_tokens.sh
   ```

片刻之后，集群将从`Unavailable`变回`Available`。

6.继续正常使用Rancher。
