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

如果你点击 **_first_container_**的下拉菜单，你可以执行各种动作，例如：停止容器，查看日志，或者进入容器的 CLI 控制台。

### Create a Container through Native Docker CLI

Rancher will display any containers on the host even if the container is created outside of the UI. Create a container in the host's shell terminal.

```bash
$ docker run -it --name=second_container ubuntu:14.04.2
```

In the UI, you will see **_second_container_** pop up on your host! If you terminate the container by exiting the shell, the Rancher UI will immediately show the stopped state of the container.

Rancher reacts to events that happen out of the band and just does the right thing to reconcile its view of the world with reality. You can read more about using Rancher with the [native docker CLI]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/native-docker/).

If you look at the IP address of the **_second_container_**, you will notice that it is not in `10.42.*.*` range. It instead has the usual IP address assigned by the Docker daemon. This is the expected behavior of creating a Docker container through the CLI. 

What if we want to create a Docker container through CLI and still give it an IP address from Rancher’s overlay network? All we need to do is add a label in the command. 

```bash
$ docker run -it --label io.rancher.container.network=true ubuntu:14.04.2
```
<br>
The label `io.rancher.container.network` enables us to pass a hint through the Docker command line so Rancher will set up the container to connect to the overlay network.

### Create a Multi-Container Application

We have shown you how to create individual containers and connect them to a cross-host network. Most real-world applications, however, are made out of multiple services, with each service made up of multiple containers. A WordPress application, for example, could consist of the following services:

1. A load balancer. The load balancer redirects Internet traffic to the WordPress application.
2. A WordPress service consisting of two WordPress containers.
3. A database service consisting of one MySQL container.

The load balancer targets the WordPress service, and the WordPress service links to the MySQL service.

In this section, we will walk through how to create and deploy the WordPress application in Rancher.

From the Rancher UI, click the **Applications** -> **Stacks**, and click on the **Add Service** button to add a service. 

First, we'll create a database service called _database_ and use the mysql image. In the **Command** tab, add the environment variable `MYSQL_ROOT_PASSWORD=pass1`. Click **Create**. You will be immediately brought to a stack page, which will contain all the services.

Next, click on  **Add Service** again to add another service. We'll add a WordPress service and link to the mysql service. Let's use the name, _mywordpress_, and use the wordpress image. We'll move the slider to have the scale of the service be 2 containers. In the **Service Links**, add the _database_ service and provide the name _mysql_. Just like in Docker, Rancher will link the necessary environment variables in the WordPress image from the linked database when you select the name as _mysql_. Click **Create**.

Finally, we'll create our load balancer. Click on the dropdown menu icon next to the **Add Service** button. Select **Add Load Balancer**. Provide a name like _wordpresslb_ and select a source port and target port on the host that you'll use to access the wordpress application. In this case, we'll use `80` for both ports.  The target service will be _mywordpress_ service. Click **Save**. 

Our multi-service application is now complete! On the **Applications** -> **Stack** page, you'll be able to find the exposed port of the load balancer as a link. Click on that link and a new browser will open, which will display the wordpress application.

### Create a Multi-Container Application using Rancher Compose

In this section, we will show you how to create and deploy the same WordPress application we created in the previous section using a command-line tool called `rancher-compose`. 

The `rancher-compose` tool works just like the popular `docker-compose` tool. It takes in the same `docker-compose.yml` file and deploys the application on Rancher. You can specify additional attributes in a `rancher-compose.yml` file which extends and overwrites the `docker-compose.yml` file.

In the previous section, we created a Wordpress application with a load balancer. If you had created it in Rancher, you can download the files directly from our UI by selecting **Export Config** from the stack's dropdown menu. The `docker-compose.yml` and `rancher-compose.yml` files would look like this:

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

Download the `rancher-compose` binary from the Rancher UI by clicking on `Download CLI`, which is located on the right side of the footer. We provide the ability to download binaries for Windows, Mac, and Linux.

In order for services to be launched in Rancher using `rancher-compose`, you will need to set some variables in `rancher-compose`. You will need to create an [environment API Key]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/api-keys/) in the Rancher UI. Click on **API** and click on **Add API Key**. Save the username (access key) and password (secret key). Set up the environment variables needed for rancher-compose: `RANCHER_URL`, `RANCHER_ACCESS_KEY`, and `RANCHER_SECRET_KEY`.

```bash
# Set the url that Rancher is on
$ export RANCHER_URL=http://server_ip:8080/
# Set the access key, i.e. username
$ export RANCHER_ACCESS_KEY=<username_of_key>
# Set the secret key, i.e. password
$ export RANCHER_SECRET_KEY=<password_of_key>
```

Now, navigate to the directory where you saved `docker-compose.yml` and `rancher-compose.yml` and run the command.

```bash
$ rancher-compose -p NewWordpress up
```

In Rancher, a new stack will be created called **NewWordPress** with all of the services launched.

