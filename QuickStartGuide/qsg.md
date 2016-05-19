## 快速安装指南
本文档首稿正在编辑中，请随后帮忙 review。

在本文中，我们将做一个 Rancher 系统的快速安装，即在一台 Linux 机器上安装并运行所有几乎所有的必要功能。

### 准备 Linux 主机
先安装一个64位的 Ubuntu 14.04 Linux 主机，其内核必须高于 3.10 。或者其它同等的 Linux 发行版。你可以使用一台笔记本、一个虚拟机或者一台物理的服务器。请确保目标安装 Linux 主机的内存至少1GB。

然后安装 Docker 在这个 Linux 主机上， 可以参考 [Docker](https://docs.docker.com/installation/ubuntulinux/) 网站的安装说明。

### 启动 Rancher 服务器
启动 Rancher 服务器所需要做的动作就只有一条命令。在启动了这个容器之后，我们将能查看到这个运行中的服务器的日志。

```
$ sudo docker run -d --restart=always -p 8080:8080 rancher/server
# 显示 Rancher 服务器的容器 ID，替换containerid
$ sudo docker ps
# 显示并查看 Rancher 服务器的日志
$ sudo docker logs -f containerid
```
启动 Rancher 服务器可能需要花几分钟时间。这取决于您下载 Rancher Server镜像的速度。当日志中显示 “.... Startup Succeeded, Listening on port...” 以后，Rancher UI 图形界面现在就能正常访问了。

Rancher 服务器的图形界面访问端口是 8080 ，通过在浏览器中访问这个网址 http://linux_host_ip:8080 , 您就可以打开 Rancher 服务器的图形界面。如果您的浏览器和 Rancher 服务器都运行在同一台服务器上，你需要使用主机的真实 Ip 地址，如： http://192.168.1.100:8080 ， 而不是 http://localhost:8080 或者 http://127.0.0.1:8080

> **注意:** Rancher 的访问控制在初始安装时并没有配置，你的 Rancher 服务器图形界面和 API 能在任何能访问到您的 IP 地址的地方被访问到。我们建议配置访问控制参考 [访问控制]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/access-control/).

### 添加主机

为简化操作，我们将添加运行 Rancher 服务器容器的主机。而在实际的生产环境中，我们建议使用专用的主机来运行 Rancher 服务器。 

通过点击图形界面的 **Infrastructure** 标签来添加主机，然后您将会看到 **Hosts** 页面。Rancher 会提示您选择一个 IP 地址。这个 IP 地址必须可以被所有即将添加的主机访问到。把 Rancher 服务器的端口通过防火墙的 NAT 或者负载均衡器暴露出来，或者暴露到 Internet上在有些情况下是很有用的。如果你的主机有一个私有或者本地 IP 地址，例如： `192.168.*.*`；Rancher 将打印一个提示信息，告诉您是否确认这个 IP 地址可以被正常访问到。

现在我们添加 Rancher 服务器主机自身，因此我们可以忽略这个提示信息。点击  **Save** ；您将进入默认的 **Custom** 选项页面，您在这可以得到运行 rancher/agent 容器的命令。这里还有其它的公有云的选项，使用这个选项可以实现通过 `docker-machine` 去启动主机节点。在 Web 界面上，Rancher 提供的用于添加主机的命令如下：

```bash
$ sudo docker run -d --privileged -v /var/run/docker.sock:/var/run/docker.sock rancher/agent:v0.7.9 http://172.17.0.3:8080/v1/scripts/DB121CFBA836F9493653:1434085200000:2ZOwUMd6fIzz44efikGhBP1veo
```

由于我们正在添加 Rancher 服务器的主机，我们需要添加这个主机所使用的共有 IP。Rancher agent 命令中如果没有这个参数，这个主机的 IP 很可能会是个错误的配置。您可以添加这个 IP 地址在**Step 4**，这将会修改命令，并加入一个环境变量。

```bash
$ sudo docker run -e CATTLE_AGENT_IP=172.17.0.3 -d --privileged -v /var/run/docker.sock:/var/run/docker.sock rancher/agent:v0.7.9 http://172.17.0.3:8080/v1/scripts/DB121CFBA836F9493653:1434085200000:2ZOwUMd6fIzz44efikGhBP1veo
```

在运行 Rancher 服务器的主机上运行这个命令。 

当您在 Rancher 的页面中点击 **Close** 按钮后，您会被返回到 **Infrastructure** -> **Hosts** 页面。在一两分钟后，这个主机将自动出现在这里。

### 使用图形界面创建一个容器

进入 **Applications** -> **Stacks** 页面，如果这里还没有服务，你可以点击 "Add Service" 按钮。你可以输入一个类似  “first_container” 的名字。您现在使用默认配置并点击 **Create** 。Rancher 将开始在这个主机上启动两个容器。一个容器是您所创建的名为**_first_container_** ；另外一个容器是**_Network Agent_**，这是个由 Rancher 创建的系统容器，它用来处理扩主机联网和健康检查等任务。

不管你的主机是什么 IP 地址，**_first_container_** 和 **_Network Agent_** 将会的到 `10.42.*.*` 网段的 IP 地址。Rancher 已经创建了能在不同主机之上的让所有容器可以相互通信的覆盖网络。

如果你点击 **_first_container_**的下拉菜单，你可以执行各种动作，例如：停止容器，查看日志，或者进入容器的 控制台。

### 使用 Docker 原生命令创建一个容器

Rancher 会显示所有在主机上的容器，即使有些容器是在图形界面之外创建的。在主机的 shell 命令行里创建一个容器。

```bash
$ docker run -it --name=second_container ubuntu:14.04.2
```

在图形界面中，你将看到 **_second_container_** 在你的主机上出现！如果你通过退出命令行来退出用命令方式创建的容器，在 Rancher 图形界面中将立刻显示这个容器的状态为停止。

Rancher 可以对带外发生的事件作出反应，并把当前的显示状况如实地整合在它的视图中。你能了解更多信息在 [使用 Docker 原生命令行]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/native-docker/) 这一章。

如果你查看容器 **_second_container_** 的 IP 地址，你会注意到他不在 `10.42.*.*` 网段中。它的 IP 地址是通过 Docker 后台服务获得的。这是通过命令行方式创建容器的正常的结果。

如果我希望通过命令行创建的容器依然具有 Ranger 覆盖网络的网络地址呢？我们所需要做的就仅仅是在命令中加一个标签。

```bash
$ docker run -it --label io.rancher.container.network=true ubuntu:14.04.2
```
<br>
标签 `io.rancher.container.network` 让我们通过命令行传递了一个通知，这样 Rancher 会为把容器配置为连接到覆盖网络。


### 创建一个多容器应用

我们已经展示了如何创建当个容器，并把他们连接到跨主机的网络。然而大多数真实世界中的应用都是由多种服务构成的。例如一个 WordPress 应用是由下列服务组成的：

1. 一个负载均衡服务。负载均衡器把 Internet 流量转发给 WordPress 应用。
2. 一个由两个 WordPress 容器组成的 WordPress 服务。
3. 一个由一个 MySQL 容器组成的数据库服务。

负载均衡器的目标地是 WordPress 服务，WordPress 服务连接到 MySQL 服务。

在这一章里，我们将在 Rancher 中逐步地创建和部署 WordPress 应用堆栈。

在 Rancher 的图形界面中，点击  **Applications** -> **Stacks**，点击 **Add Service** 按钮来添加一个服务。

首先，我们将使用 mysql 镜像来创建一个名为 _database_ 的数据库服务。在 **Command** 标签里，添加环境变量 `MYSQL_ROOT_PASSWORD=pass1` ，点击  **Create** 按钮。然后我们会立刻进入堆栈页面，这里包含了所有服务。

然后，再次点击 **Add Service** 按钮来添加另外一个服务。我们将添加一个 WordPress 服务并连接到 mysql 服务。让我们使用 _mywordpress_ 做名称，使用 wordpress 镜像。我们将拉动滚动条增加此服务的容器数量到2。在 **Service Links** 标签中，添加 _database_  服务，并提供 _mysql_ 名称。就想在 Docker 中一样，当你选择了 _mysql_ 后，Rancher 将从所连接的数据库服务中连接必要的环境变量到 WordPress 镜像中。然后点击**Create**。

最后，我们来创建负载均衡器。点击 **Add Service**按钮傍边的下拉菜单。选择 **Add Load Balancer**。提供像  _wordpresslb_  这样的名称，并选择一个源端口和你将用来访问 wordpress 应用的主机上的目标端口。在这个例子中，我们把这两个端口都使用 `80` 。目标服务将是  _mywordpress_ 。点击 **Save**。 

至此我们的多服务应用堆栈创建完毕！在 **Applications** -> **Stack**页面，我们将能够在负载均衡器的开放端口处找到一个连接。点击这个连接，浏览器将新打开一个窗口，它将显示 wordpress 应用。

### 使用 Rancher Compose 创建一个多容器应用

在这一个部分，我们将使用名为 `rancher-compose` 的命令行工具来创建上一节里已经创建和部署的相同的 WordPress 应用。

命令行工具 `rancher-compose` 的功能和流行的 `docker-compose`命令行工具类似。它使用相同的`docker-compose.yml`文件来在 Rancher 上部署应用。你能在 `rancher-compose.yml` 文件中制定更多的属性，它会扩展和覆盖 `docker-compose.yml` 文件。

在上一节里，我们创建了一个具有一个负载均衡器的 WordPress 应用。如果你已经在 Ranher 中创建了它，你能直接在图形界面的堆栈的下拉菜单中选择  **Export Config** 来直接下载文件。`docker-compose.yml` 和 `rancher-compose.yml` 文件内容将与下面实例类似：

**docker-compose.yml**

```yaml
mywordpress:
  tty: true
  image: wordpress
  links:
    database: mysql
  stdin_open: true
wordpresslb:
  ports:
  - 80:80
  tty: true
  image: rancher/load-balancer-service
  links:
    mywordpress: mywordpress
  stdin_open: true
database:
  environment:
    MYSQL_ROOT_PASSWORD: pass1
  tty: true
  image: mysql
  stdin_open: true
```

**rancher-compose.yml**

```yaml
mywordpress:
  scale: 2
wordpresslb:
  scale: 1
  load_balancer_config:
    haproxy_config: {}
  health_check:
    port: 42
    interval: 2000
    unhealthy_threshold: 3
    healthy_threshold: 2
    response_timeout: 2000
database:
  scale: 1
```

从 Ranher 图形界面中点击 `Download CLI` 来下载 `rancher-compose` 可执行文件，这个链接位于页面的页脚。我们提供了 Windows，Mac 和 Linux 的不同的版本。

为了使用`rancher-compose`在 Rancher 中启动服务，你需要设置一些必须的变量。你需要在 Rancher 图形界面中创建一个 [environment API Key]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/api-keys/) 。点击  **API** ，在点击  **Add API Key**。保存用户名(access key)和密码(secret key)。设置 rancher-compose 所需要的环境变量：`RANCHER_URL`, `RANCHER_ACCESS_KEY`, 和 `RANCHER_SECRET_KEY`。 

```bash
# Set the url that Rancher is on
$ export RANCHER_URL=http://server_ip:8080/
# Set the access key, i.e. username
$ export RANCHER_ACCESS_KEY=<username_of_key>
# Set the secret key, i.e. password
$ export RANCHER_SECRET_KEY=<password_of_key>
```

现在进入保存 `docker-compose.yml` 和 `rancher-compose.yml`  文件的目录中，并运行下面的命令。

```bash
$ rancher-compose -p NewWordpress up
```

在 Rancher 中，一个名为 **NewWordPress** 的新堆栈将被创建，并且具有了所以的服务。

