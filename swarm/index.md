## Swarm

若想在Rancher中使用Swarm，首先需要创建一个集群管理方式配置为**Swarm**的[环境]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/environments/)。

### 创建Swarm集群管理的环境
在右上角的环境下拉菜单中，单击**管理环境**。要创建一个新环境，单击**新建环境**，选择**Swarm**作为集群管理方式，提供**名称**，**说明**（可选）。如果打开了[访问控制]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/access-control/)功能，你可以[添加成员]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/environments/#editing-members)并赋予相应的[角色]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/environments/#membership-roles)。所有位于成员列表的用户都可以访问此新增环境。

创建完由Swarm管理的环境后，你可以通过在右上角的环境下拉框中根据环境名字切换到对应的环境。

> **注意**
> Rancher目前支持多种集群管理方式，但是不支持在已有服务运行的环境中切换集群管理方式。

### 启用Swarm
创建完由Swarm集群管理的环境后，现在可以添加至少一台主机来启用Swarm集群。对于不同的集群管理方式，[添加主机]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts)的过程都是一样的。一旦添加完一台主机，Rancher就会自动部署Swarm需要的组件（比如swarm和swarm-agent）在主机上。你可以在Swarm标签页看到部署的进度。
> **注意**
> swarm不需要安装在所有的主机上。

### 使用Swarm
swarm组件部署完成后，你就可以通过以下方式来创建或者管理Swarm程序：
#### RANCHER界面
用户可以在Rancher界面上完成创建、获取、更新、销毁项目的所有操作。在Swarm标签页，选择**项目**并单击**创建项目**。创建项目时，你可以在界面上通过上传文件或者粘贴文件内容的方式来提供docker-compose.yml。如果你的项目装配模板中包含环境变量，你需要通过添加**变量替换**的方式来声明变量，最后单击**创建**。
#### RANCHER目录
Rancher支持加载整套Swarm模板。如果使用模板，可以单击**目录**标签页。选择你想加载的模板并单击**查看详情**。审核并编辑工作站名称，工作站说明和相关的配置，然后单击**加载**。
如果你想添加模板到Swarm，你可以先添加到[Rancher目录]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/catalog/)中，并放置在swarm-templates目录。
#### 命令行界面
为了配置你的工作机器用来管理swarm，你可以依次单击**Swarm**->**CLI**->**生成配置**来生成必要的API秘钥并将配置文件放置在docker-cli.zip压缩文件中。遵循界面上的说明去配置TLS并连接到Docker。
#### SHELL脚本
Rancher提供了一套方便的SHELL接口来执行docker或者docker-compose命令。