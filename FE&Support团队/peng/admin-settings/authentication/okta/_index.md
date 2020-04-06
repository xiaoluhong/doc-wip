---
标题: 配置Okta (SAML)
---

_v2.2.0版本可用_

如果您的组织使用Okta Identity Provider (IdP)进行用户身份验证，您可以配置Rancher以允许您的用户使用他们的IdP凭据登录。

**注:** Okta集成仅支持服务提供商发起的登录。

### 先决条件

在Okta中，使用下面的设置创建一个SAML应用程序。参见[Okta文档](https://developer.okta.com/standards/SAML/setting_up_a_saml_application_in_okta)以获得帮助。

| 设置                          | 值                                                      |
| ----------------------------- | ------------------------------------------------------- |
| `Single Sign on URL`          | `https://yourRancherHostURL/v1-saml/okta/saml/acs`      |
| `Audience URI (SP Entity ID)` | `https://yourRancherHostURL/v1-saml/okta/saml/metadata` |

### 在Rancher中配置Okta

1.  在**全局**视图中，从主菜单中选择**安全 > 认证**。

1.  选择**Okta**。

1.  完成**配置Okta帐户**表单。下面的示例描述了如何将Okta属性映射到Rancher中的字段。


    | 字段                      | 描述                                                                          |
    | ------------------------- | ----------------------------------------------------------------------------- |
    | Display Name Field        | 包含用户display name的属性。                                                |
    | User Name Field           | 包含用户user name/given name属性                                          |
    | UID Field                 | 对每个用户唯一的属性。                                                      |
    | Groups Field              | 为管理组成员身份创建的条目。                                                  |
    | Rancher API Host          | Rancher服务器的URL地址                                                        |
    | Private Key / Certificate | 密钥/证书对，用于在Rancher和IdP之间创建安全shell。                            |
    | Metadata XML              | 在应用程序`登录`部分得到的`Identity Provider metadata`文件。                                     |
    >**提示:** 您可以使用openssl命令生成密钥/证书对。例如:
    >
    >        openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout myservice.key -out myservice.cert

1) 完成**配置Okta账户**表单后，点击页面底部的**启用Okta认证**。

   Rancher会将您重定向到IdP登录页面。输入使用Okta IdP进行身份验证的凭据，以验证您的Rancher Okta配置。

   >**注:** 如果什么都没有发生，很可能是因为您的浏览器阻塞了弹出窗口。请确保您禁用了rancher域的弹出窗口拦截器，并在您可能使用的任何其他扩展中将其列入白名单。

**结果:** Rancher被配置为使用Okta。您的用户现在可以使用他们的Okta账号登录到Rancher。

{{< saml_caveats >}}
