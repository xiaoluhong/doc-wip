---
title: 在主机模板中开启磁盘UUIDs
---

从Rancher v2.0.4开始，在vSphere主机模板中默认启用了磁盘UUIDs。

对于v2.0.4之前版本的Rancher，我们建议在配置vSphere主机模板时自动启用磁盘UUIDs，因为Rancher在操作vSphere资源时需要磁盘UUIDs。

为所有创建的VM启动磁盘UUIDs，

1. 在Rancher UI中以管理员登录，然后跳转到 **主机模板** 页面。

2. 添加或编辑现有的vSphere主机模板。

3. 在 **实例选项** 中点击 **添加参数**。

4. 输入 `disk.enableUUID` 作为key，填写value为 **TRUE**.

   ![vsphere-nodedriver-enable-uuid](/img/rke/vsphere-nodedriver-enable-uuid.png")

5. 点击 **创建** 或 **保存**。

**结果:** vSphere主机模板已启用磁盘UUID。
