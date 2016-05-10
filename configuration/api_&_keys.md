## API & Keys
------

点击 **API** 以便找到 API 端点。每当你创建了一个工作环境的 API key，提供的 URL 端点都会将你指向你当前正在操作的那个[环境]({{site.baseurl}}/configuration/environments).

如果[权限控制]({{site.baseurl}}/configuration/access-control/)未配置，任何知道IP地址的人都拥有访问你的 Rancher API的权限。高度建议启用权限控制管理。

一旦启用了权限控制并且未登录，对每个[环境]({{site.baseurl}}/configuration/environments/)均需要创建一个工作环境 API key，以便 API 能访问指定的工作环境。

在 Rancher 里，通过点击对象的下拉菜单中的 **View In API** ，所有的对象都能通过API来查看。在创建环境 API key 时提供的 URL 端点，同样提供了到 API 各个部分的全部链接。点击此处了解更多关于如何使用 [API]({{site.baseurl}}/api)

### 添加环境 API Keys

在添加任何环境 API key之前，请确认你在正确的[环境]({{site.baseurl}}/configuration/environments/)中。点击 **Add Environment API Key**。 提供 **Name** ，并根据需要，填写 **Description**。点击 **Create**。Rancher 将会生成并显示你的环境 API Key。一个 API Key 是一个用户名 (access key) 和一个密码  (secret key) 的组合 - 当执行 API 调用时，两个都需要用来进行认证。在你复制保存下相关信息后，点击 **Close**。

> **注意:** 一旦你关闭了窗口，你将无法重新获得你的 API Key的密码 (secret key) ，所以确保将其保存在了足够安全的地方。

### 使用环境 API Keys

当配置了权限控制且你没有登录，如果你试图打开 API 端点，你会被提示需要一个环境 API Key。用户名是access key ，密码是 secret key。

如果你使用 `cURL`，你能通过指定 `-u` 参数，按照格式 `username:password` 来使用环境 API Key，或者能通过在你的 `.netrc` 文件内加入一行。

### 编辑环境 API Keys

环境 API Key 的全部选项都可以通过位于 Key 列表右侧的下拉菜单进入。

对于任何 _Active_ 的 key，你能 **Deactivate** 这个 key，那将阻止这些证书的使用。Key将会标记为 _Inactive_ 状态。

对于任何 _Inactive_ 的 key，你有两种选项。 你能 **Activate** 这个 key，这将重新允许证书能访问 API。或者，你能 **Delete** 这个 key, 那将从环境中移除这些证书。

你能 **Edit** 任何 key，那允许你修改这个环境 API key 的名称和描述。你没法修改实际的access key 或 secret key。如果你想要新的键值对，你需要创建一个新的。
