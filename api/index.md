## API实践
---

API有可以从web浏览器访问的用户接口。这是查看资源，执行动作，以及查看等价于cURL命令执行的HTTP请求&回应。点击**API**来获得URL endpoint来访问API。

![User menu](/images/apikeys.png)

点击端点链接：

![Endpoint link](/images/api_endpoint.png)

## 术语
---

API中一些资源类型命名和当前UI上所用的术语，并不一致。尤其是：

| UI | API | Description |
|----|-----|-------------|
| [Environment]({{site.baseurl}}/configuration/environments/) | [project]({{site.baseurl}}/api/api-resources/project) | 一组物理资源， 例如 [主机]({{site.baseurl}}/api-resources/host) |
| [Stack]({{site.baseurl}}/rancher-ui/applications/stacks/) | [environment]({{site.baseurl}}/api/api-resources/environment) | 一个(API) 环境是一组服务以及rancher-compose所操作的级别。 |

在这文档中，我们使用UI所用的术语进行描述，在不同点也会提供额外的声明。这些混乱将会在未来的`/v2`版本API清除。

## 验证
---

如果[Access Control]({{site.baseurl}}/configuration/access-control/)启动的话，API请求必须包含验证信息。验证操作通过带有[API keys]({{site.baseurl}}/api/api-resources/apikey)的HTTP基本验证来完成。API密钥或者属于一个(UI) Environment/ (API) [Project]({{site.baseurl}}/api/api-resources/project})，只能访问该Environment；或者属于一个[Account]({{site.baseurl}}/api/api-resource/account)，能够访问该账户所有的Environment，也能够创建新的Api密钥。在UI上有单独的JSON Web Token接口。

### 一个环境的API密钥


环境的API密钥能够在UI中创建，查看[API & Kes[({{site.baseurl}}/configuration/api-keys/)。 密钥所属于一个环境，而且完全有权管理该环境，但无权管理其他环境。[Membership roles]({{site.baseurl}}/configuration/environments/#membership-roles)并不支持密钥。

### 账户的API密钥

账户的API密钥当前并没有在UI中显示。你可以点击浏览器的API页面来创建一个:

  - 点击UI上的Endpoint链接
  - 导航到`/v1/apikeys`
  - 点击`Create`
  - 选择你要创建密钥的`accountId`
  - 可选择设置`name`和`description`
  - 点击`Show Request`， 然后`Send Request`
  - 保存回应中的`publicValue`和`secretValue`

Account密钥能够创建新的Environments，也能够用来通过`/v1/projects`来访问多个Environments。这些密钥所采用的[Membership roles]({{site.baseurl}}/configuration/environments/$membership-roles)限制该Account哪些Environments以及哪些动作能够使用。

## 作用域
---

- 大部分的资源被一个Environment所拥有，并通过`/v1/projects/<project_id>/<resources>`地址进行访问。
- 因为环境证书只能够存取一个Environment(project)，你可以选择跳过`/project/<project_id>`部分。
-例如，如果你在project `1a5`使用了一个Environment API密钥，那么`/v1/projects/1a5`等同于`v1`，`/v1/projects/1a5/hosts`等同于`/v1/hosts`。
- 这篇文档一般只涉及更短的`/v1/<type>`版本.如果使用了Account密钥，则将适当的environment添加到路径中。

## 发送请求
---

这些API一般都是RESTful API，但拥有一些特性使得客户端能够查看任何资源的定义，因而可以编写通用的客户端，而不是每种资源类型编写特定代码的客户端。对那些想费力深入通用API规范细节的人，[查看这里](https://github.com/rancher/api-spec/blob/master/specification.md)

- 每种类型有一个[Schema]({{site.baseurl}}/api/api/resources/schema/)，负责描述：
  - 到达该资源类型的collection的URL
  - 该资源拥有的所有域，连同域的类型，域的基本验证规则，它们是必须还是可选等.
  - 该类型资源能够执行每个动作，以及输入和输出(和schemas一样)
  - 每个域允许的过滤方式
  - collection中哪些HTTP动词方法对其或者其中独立的资源可用

- 因此理论上你能够只需要加载schemas列表就知道API的一切。这就是实际上UI如何工作的，它并没有包含和Rancher特定的代码。获取Schemas的URL，作为一个`X-Api-Schemas`头部，在每一个HTTP回应被发送。从该URL，你可以通过访问每一个schema的`collection`链接，来知道哪里显示资源；通过返回资源的其他`links`来获取其他信息。

- 实际上，你可能只想构建URL字符串。我们强烈建议限制构建的URL字符串只在顶级显示collection(`/v1/<type>`)或者获取一个特定的资源(`/v1/<type>/<id>`)时使用。任何更底层的动作都将在将来版本改变。(译者注：这里指用户应尽通过可能返回的HTTP回应信息中提取想要资源的URL，不要根据当前版本资源的URL结构自己构建。)

- 资源间的关系称为links。每个资源都包含一个`links`表，其中包含link的名字以及获取link信息的URL. 再次强调，你应该通过发送`GET`请求来获取资源以及使用`links`表中的URL，而不是自己构建这些字符串。

- 大多数资源都有些动作，用来处理某事或者改变资源的状态. 通过发送`POST`请求到`action`表中的URL来执行你想要的动作. 一些动作会需要输入或者会产生输出，请查看每种类型资源或者schemas的独立文档来获取特定信息.

- 编辑资源， 则发送一个带有资源的域数据的`PUT`HTTP请求到你想要改变的资源的`links.self`链接.未知域或者不可编辑的域将被忽略.

- 删除资源， 则发送一个`DELETE`HTTP请求到资源的`links.self`链接. 注意一些资源必须要在删除之前停止;被删除的资源仍然通过API根据指定资源ID来恢复.

- 创建资源，发送一个`POST`HTTP请求schema中的collection URL(也即是`/v1/<type>`).

## 过滤
---

大多数的collections能够在服务端通过HTTP的检查参数按照通用域进行过滤.`filter`表显示哪些域能够被过滤， 以及哪些值是你的请求过滤的. API UI控制了安装过滤的过程，为你显示正确的请求。"equals"和`field=value`相匹配。修饰符可以添加到域名，例如，`field_gt=42`意思就是`field大于42`。更多细节查看[API spec](https://github.com/rancher/api-spec/blob/master/specification.md#filtering)。

## 排序
---

大多数的collections在服务端通过HTTP的查询参数按照通用域进行排序.`sortLinks`表将显示哪些排序是可用的，连同显示已经按照该方式排序的collection的URL.如果指定的话，它也将包含当前回应是以何种方式排序的信息。

## 分页
---

API回应默认以每页100资源的限制进行分页.可以通过添加`limit=1000`查询参数来将上限提升到1000. collection回应中的`pagination`会告诉你是否收到全部结果；如果没有收到全部结果，将会有一个链接指向下一页。

## WebSockets
---

一些Rancher特性，比如容器日志，shell访问，统计都使用websockets来传输信息流。通过API来使用：

  - 点击适当的链接或者执行适当的动作
  - 回应将包含一个URL(以`ws://`或者`wss://`开头)以及一段长的`token`字符串.
  - 打开一个指向返回URL的WebSocket客户端
  - token是由Rancher server签名的，允许所在的主机容器授权这次请求，所以它作为一个HTTP的头部，如`Authorization: Bearer <token_string>`，发送给Rancher server。
  - 如果你通过web浏览器发送请求，你不能发送包容任意HTTP头部的请求，因此你可以选择的是将token添加到URL中作为URL查询参数来代替，如`token=<token_string>`。
