---
标题：配置Keycloak (SAML)
---

_v2.1.0版本可用_

如果您的组织使用Keycloak Identity Provider (IdP)进行用户身份验证，您可以配置Rancher来允许您的用户使用他们的IdP凭证登录。

### 先决条件

- 你必须有一个[Keycloak IdP服务器](https://www.keycloak.org/docs/latest/server_installation/)。
- 在Keycloak中，创建一个[新的SAML客户端](https://www.keycloak.org/docs/latest/server_admin/#saml-clients)，设置如下。参见[Keycloak文档](keycloak.org/docs/latest/server_admin/#saml-clients)获得帮助。 


  | 设置                        | 值                                                          |
  | --------------------------- | ----------------------------------------------------------- |
  | `Sign Documents`            | `ON` <sup>1</sup>                                           |
  | `Sign Assertions`           | `ON` <sup>1</sup>                                           |
  | 所有其它`ON/OFF`设置        | `OFF`                                                       |
  | `Client ID`                 | `https://yourRancherHostURL/v1-saml/keycloak/saml/metadata` |
  | `Client Name`               | <CLIENT_NAME> (e.g. `rancher`)                              |
  | `Client Protocol`           | `SAML`                                                      |
  | `Valid Redirect URI`        | `https://yourRancherHostURL/v1-saml/keycloak/saml/acs`      |

><sup>1</sup>: 您可以选择启用这些设置中的一个或两个。

- 从Keycloak客户端导出`metadata.xml`文件:
  在“安装”选项卡中，选择“SAML元数据IDPSSODescriptor”格式选项并下载文件。


### 在Rancher中配置Keycloak

1. 在**全局**视图中，从主菜单中选择**安全 > 认证**。
1. 选择**Keycloak**。

1. Complete the **Configure Keycloak Account** form. Keycloak IdP lets you specify what data store you want to use. You can either add a database or use an existing LDAP server. For example, if you select your Active Directory (AD) server, the examples below describe how you can map AD attributes to fields within Rancher.
1. 完成**配置Keycloak帐户**表单。Keycloak IdP允许您指定要使用的数据存储。您可以添加数据库，也可以使用现有的LDAP服务器。例如，如果您选择Active Directory (AD)服务器，下面的示例将描述如何将AD属性映射到Rancher中的字段。

    | 字段                      | 描述                                                                          |
    | ------------------------- | ----------------------------------------------------------------------------- |
    | Display Name Field        | 包含用户display name的AD属性。                                                |
    | User Name Field           | 包含用户user name/given name的AD属性                                          |
    | UID Field                 | 对每个用户唯一的AD属性。                                                      |
    | Groups Field              | 为管理组成员身份创建的条目。                                                  |
    | Rancher API Host          | Rancher服务器的URL地址                                                        |
    | Private Key / Certificate | 密钥/证书对，用于在Rancher和IdP之间创建安全shell。                            |
    | IDP-metadata              | 从你的IdP服务器导出的`metadata.xml`文件。                                     |

    >**提示:** 您可以使用openssl命令生成密钥/证书对。例如:
    >
    >        openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout myservice.key -out myservice.cert

1. After you complete the **Configure Keycloak Account** form, click **Authenticate with Keycloak**, which is at the bottom of the page.

   Rancher redirects you to the IdP login page. Enter credentials that authenticate with Keycloak IdP to validate your Rancher Keycloak configuration.

   > **Note:** You may have to disable your popup blocker to see the IdP login page.

**Result:** Rancher is configured to work with Keycloak. Your users can now sign into Rancher using their Keycloak logins.


1. 完成**配置Keycloak帐户**表单后，单击页面底部的**启用Keycloak认证**进行身份验证。

Rancher将重定向到IdP登录页面。输入Keycloak IdP系统的身份验证凭据，以验证您的Rancher Keycloak配置。

   > **注:** 您可能需要禁用弹出窗口拦截器以访问IdP登录页面。

**结果:** Rancher被配置为使用Keycloak认证。您的用户现在可以使用Keycloak账号登录Rancher。


{{< saml_caveats >}}

### 附录: 故障诊断

如果在测试到Keycloak服务器的连接时遇到了问题，首先需要再次检查SAML客户机的配置选项。您还可以检查Rancher日志，以帮助确定问题的原因。调试日志可能包含关于错误的更详细的信息。请参考本文档中的[如何启用调试日志](/docs/faq/technical/#how-can-i-enable-debug-logging)。

#### You are not redirected to Keycloak
#### 没有被重定向到Keycloak

When you click on **Authenticate with Keycloak**, your are not redirected to your IdP.
当您点击**启用Keycloak认证**时，没有被重定向到Keycloak IdP。

- 验证你的Keycloak客户端配置。
- 确保“Force Post Binding”设置为“OFF”。


#### IdP登录后显示禁止消息

您被正确地重定向到Keycloak IdP登录页面，可以输入凭据，但得到一个“禁止”的消息。

- 检查Rancher调试日志。
- 如果日志显示“ERROR: either the Response or Assertion must be signed”，确保`Sign Documents`或`Sign assertions` 在您的Keycloak客户端中被设置为“ON”。


#### Keycloak错误: "We're sorry, failed to process response"

- 检查Keycloak日志。
- 如果日志显示`failed: org.keycloak.common.VerificationException: Client does not have a public key`, 在Keycloak客户端中将`Encrypt Assertions` 设置为`OFF`。

#### Keycloak错误: "We're sorry, invalid requester"

- 检查Keycloak日志。
- 如果日志显示`request validation failed: org.keycloak.common.VerificationException: SigAlg was null`, 在Keycloak客户端中将`Client Signature Required`设置为`OFF`。

#### Keycloak 6.0.0+: 选项中没有IDPSSODescriptor设置

Keycloak versions 6.0.0 and up no longer provide the IDP metadata under the `Installation` tab.
You can still get the XML from the following url:
Keycloak 6.0.0及以上版本在“安装”选项卡下不再提供IDP元数据。
你仍然可以从以下网址获取XML:

`https://{KEYCLOAK-URL}/auth/realms/{REALM-NAME}/protocol/saml/descriptor`

从这个URL获得的XML包含`EntitiesDescriptor`作为根元素。Rancher期望的根元素是`EntityDescriptor`而不是`EntitiesDescriptor`。因此，在将此XML传递给Rancher之前，请按照以下步骤进行调整:

- 将所有标签从“EntitiesDescriptor”复制到“EntityDescriptor”
- 从文件开头删除`<EntitiesDescriptor>`标签。
- 从xml末尾删除`</EntitiesDescriptor>`。

您会得到类似下面的示例的文件:

```
<EntityDescriptor xmlns="urn:oasis:names:tc:SAML:2.0:metadata" xmlns:dsig="http://www.w3.org/2000/09/xmldsig#" entityID="https://{KEYCLOAK-URL}/auth/realms/{REALM-NAME}">
  ....

</EntityDescriptor>
```
