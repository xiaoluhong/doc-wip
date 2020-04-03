---
title: API
---

## 如何使用API

API具有独立的用户界面，可从Web浏览器访问。这是查看资源、执行操作以及查看等效cURL或HTTP请求和响应的最简便方法。要访问它，请单击右上角用户头像，在**API和Keys**下，您可以找到API访问入口以及创建 [API keys](/docs/user-settings/api-keys/).

## 认证

API请求必须包含身份验证信息。身份验证是通过使用带有[API Keys](/docs/user-settings/api-keys/)的HTTP基本身份验证完成的。API KEY可以创建新的集群，并可以通过`/v3/clusters/`访问多个集群。 [集群和项目角色](/docs/admin-settings/rbac/cluster-project-roles/) 绑定这些KEY，并限制用户可以看到哪些集群和项目以及它们可以执行哪些操作。

默认情况下，一些集群级别的API KEY是永不过期的，因为他们使用`ttl=0`生成。如不再需要这些API KEY，可以删除这些KEY使其失效，有关操作说明请阅读[API tokens page](/docs/api/api-tokens)。

## 请求

API通常是RESTful，但有几个功能可以定义客户端可发现的所有内容，以便可以编写通用客户端，而不必为每种类型的资源编写特定代码。有关通用API规范的详细信息，请参阅[文档](https://github.com/rancher/api-spec/blob/master/specification.md)。

- 每种类型都有一个Schema描述:
  - 获取此类型资源集合的URL。
  - 资源中的每个字段，以及它们的类型、基本验证规则、是必需的还是可选的等等。
  - 对这种类型的资源及其输入和输出(也作为schemas)进行的所有操作。
  - 允许过滤的每个字段
  - 哪些HTTP verb方法可用于集合本身，或集合中的各个资源。

* So the theory is that you can load just the list of schemas and know everything about the API. This is in fact how the UI for the API works, it contains no code specific to Rancher itself. The URL to get Schemas is sent in every HTTP response as a `X-Api-Schemas` header. From there you can follow the `collection` link on each schema to know where to list resources, and other `links` inside of the returned resources to get any other information.

* In practice, you will probably just want to construct URL strings. We highly suggest limiting this to the top-level to list a collection (`/v3/<type>`) or get a specific resource (`/v3/<type>/<id>`). Anything deeper than that is subject to change in future releases.

* Resources have relationships between each other called links. Each resource includes a map of `links` with the name of the link and the URL to retrieve that information. Again you should `GET` the resource and then follow the URL in the `links` map, not construct these strings yourself.

* Most resources have actions, which do something or change the state of the resource. To use these, send a HTTP `POST` to the URL in the `actions` map for the action you want. Some actions require input or produce output, see the individual documentation for each type or the schemas for specific information.

* To edit a resource, send a HTTP `PUT` to the `links.update` link on the resource with the fields that you want to change. If the link is missing then you don't have permission to update the resource. Unknown fields and ones that are not editable are ignored.

* 要删除资源，发送HTTP `DELETE` 到资源的 `links.remove` 链接。如果链接丢失，则您没有更新资源的权限。

* 要创建一个新的资源，通过 HTTP `POST` 方法访问集合URL (比如： `/v3/<type>`).

## 过滤

大多数集合可以使用HTTP查询参数在服务端通过公共字段进行过滤。The `filters` map shows you what fields can be filtered on and what the filtered values were for the request you made. The API UI has controls to setup filtering and show you the appropriate request. For simple "equals" matches it's just `field=value`. Modifiers can be added to the field name, e.g. `field_gt=42` for "field is greater than 42". See the [API spec](https://github.com/rancher/api-spec/blob/master/specification.md#filtering) for full details.

## 排序

大多数集合可以使用HTTP查询参数在公共字段的服务器端进行排序。 `sortLinks`映射显示了可用的排序种类以及URL，可以根据这些URL对集合进行排序。它还包括有关当前响应排序的信息（如果已指定）。

## 分页

默认情况下，API响应的页面分页限制为每页100个资源。可以通过`limit`查询参数进行更改，最多可以为`1000`，例如: `/v3/pods?limit=1000`。集合响应中的分页映射告诉您是否拥有完整的查询结果，如果没有完整结果，则提供下一页的链接。
