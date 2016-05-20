## 无互联网访问
---

Rancher 服务器的运行可以不需要互联网，浏览器需要使用内网地址来访问 Rancher 的图形界面。Ranher 可以配置私有的镜像仓库或者 HTTP 代理。

在没有互联网访问的网络中运行 Rancher 服务器有下面几个功能将无法工作正常。

* 从网页界面上启动容器云主机 - 由于 Rancher 是通过调用 `docker-machine` 来在公有云里创建主机，因此这个功能会不能用。您依然可以使用[自定义主机]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/)来添加主机。
* GitHub 用户认证。 
* 来自 [Rancher Catalog](https://github.com/rancher/rancher-catalog) 和 [Community Catalog](https://github.com/rancher/community-catalog) 的模板 - 这些目录依赖从 Github 上做克隆，但是您将仍然可以使用 [添加内部目录]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/catalog/#creating-private-catalogs) 来把私有目录添加到 Rancher 中。


## 使用私有镜像仓库
---


这里假设您已经有了自己的内部私有镜像仓库或者相关方式来做 docker 镜像的分发。如果您在搭建私有镜像仓库方面需要帮助，请参考 Docker 官方文档  [Docker documentation for private registries](https://docs.docker.com/registry/). 

### Push 镜像到私有仓库 

在您开始安装或者升级 Rancher 服务器之前，确保所有镜像(例如： rancher/server, rancher/agent, rancher/agent-instance) 都已经分发到所有服务器是 **非常重要的** 如果您的私有镜像仓库中没有这些镜像，Rancher 服务器将可能变得不稳定。

对与 Rancher 服务器的每个版本，它所需要的相应的 Rancher agent 和 Rancher agent instance 镜像的版本将会在 Rancher 的发行说明文档中。

**推送镜像到私有仓库的命令 **

下面的例子适用于 v1.0.1 版本的 Rancher 服务器去访问 DockerHub 和您的私有仓库。我们建议在私有镜像仓库中的镜像打上相同的版本和标签。

```bash
# rancher/server 
$ docker pull rancher/server:v1.0.1
$ docker tag rancher/server:v1.0.1 localhost:5000/<NAME_OF_LOCAL_RANCHER_SERVER_IMAGE>:v1.0.1
$ docker push localhost:5000/<NAME_OF_LOCAL_RANCHER_SERVER_IMAGE>:v1.0.1 

# rancher/agent
$ docker pull rancher/agent:v1.0.1
$ docker tag rancher/agent:v1.0.1 localhost:5000/<NAME_OF_LOCAL_RANCHER_AGENT_IMAGE>:v1.0.1
$ docker push localhost:5000/<NAME_OF_LOCAL_RANCHER_AGENT_IMAGE>:v1.0.1

# rancher/agent-instance
$ docker pull rancher/agent-instance:v0.8.1
$ docker tag rancher/agent-instance:v0.8.1 localhost:5000/<NAME_OF_LOCAL_RANCHER_AGENT_INSTANCE_IMAGE>:v0.8.1
$ docker push localhost:5000/<NAME_OF_LOCAL_RANCHER_AGENT_INSTANCE_IMAGE>:v0.8.1
```

### 从私有镜像仓库启动 Rancher 服务器

在您的服务器上，使用您特定的 Rancher 镜像启动 Rancher 服务器。我们使用特定的版本号标签，而不是 `latest`标签，这样能确保您工作在正确的版本组合上。

使用 v1.0.1 版本示例：

```bash
$ sudo docker run -d --restart=always -p 8080:8080 \
    -e CATTLE_BOOTSTRAP_REQUIRED_IMAGE=<Private_Registry_Domain>:5000/<NAME_OF_LOCAL_RANCHER_AGENT_IMAGE>:v1.0.1 \
    -e CATTLE_AGENT_INSTANCE_IMAGE=<Private_Registry_Domain>:5000/<NAME_OF_LOCAL_RANCHER_AGENT_INSTANCE_IMAGE>:v0.8.1 \
    <Private_Registry_Domain>:5000/<NAME_OF_LOCAL_RANCHER_SERVER_IMAGE>:v1.0.1
```

#### Rancher 图形界面

图形界面和 API 的访问是通过暴露的 `8080` 端口。您可以在浏览器中访问这个 URL: `http://<SERVER_IP>:8080`.

### 添加主机

在进入图形界面之后，点击 **Add Host** 按钮。 然后 **Host Registration** 页面会立刻显示出来，点击 **Save**。

这里使用 `docker-machine` 添加公有云主机是不能工作的。点击 **Custom** 图标来获取添加主机的命令。 

网页中的命令将被配置为使用私有镜像仓库去拉取 Rancher agent 镜像。 

**添加自定义主机命令示例**

```bash
$ sudo docker run -d --privileged -v /var/run/docker.sock:/var/run/docker.sock <Private_Registry_Domain>:5000/<NAME_OF_LOCAL_RANCHER_AGENT_IMAGE>:v1.0.1 http://<SERVER_IP>:8080/v1/scripts/<security_credentials>
```

## 使用 HTTP 代理 
---

注意，在这个安装中浏览器是通过访问私有网络来使用 Rancher 的图形界面。

### 配置 Docker 使用 HTTP 代理

为了配置 HTTP 代理，Docker 后台进程在被修改配置之后可以使 Rancher 服务器和 Rancher 主机指向 HTTP 代理。在启动 Rancher 服务器和 Rancher Agent 之前，编辑配置文件 `/etc/default/docker` 添加你您的代理服务器，并重启 Docker 服务器。

```bash
$ sudo vi /etc/default/docker
```

在这个文件中，编辑 `#export http_proxy="http://127.0.0.1:3128/"` 这个参数去指向您的代理服务器。保持变更并重启 Docker 服务。重启 Docker 容器服务器在不同 Linux 发行版上可能命令不同。

> **注意:** 如果你使用 systemd 运行 Docker 服务，请参考 Docker 官方的关于配置 HTTP 代理服务器的文档 [instructions](https://docs.docker.com/articles/systemd/#http-proxy) 。

### 启动 Ranher 服务器

当使用代理服务器的时候，Rancher 服务器的启动时不需要加什么特别的环境变量参数的。因此，启动 Rancher 服务器容器的命令还是常规的格式。

```bash
sudo docker run -d --restart=always -p 8080:8080 rancher/server
```

#### Rancher 图形界面

图形界面和 API 的访问是通过暴露的 `8080` 端口。您可以在浏览器中访问这个 URL: `http://<SERVER_IP>:8080`。

### 添加主机

在进入图形界面之后，点击 **Add Host** 按钮。 然后 **Host Registration** 页面会立刻显示出来，点击 **Save**。

这里使用 `docker-machine` 添加公有云主机是不能工作的。点击 **Custom** 图标来获取添加主机的命令。

网页上所显示的命令可以用于添加任何一台已经配置好Docker 使用 HTTP 代理的主机。