# 单节点服务器安装

Rancher 使用基于 Docker 容器的部署方式。仅需简单地启动两个容器即可运行。一个容器用于管理 Rancher 服务，其他容器使用代理的方式管理主机或节点。

## 需求
- 任何支持 Docker 1.10.3 的 Linux 发行版。其中 [RancherOS](http://docs.rancher.com/os/)，Ubuntu，RHEL/CentOS 7 经过了更多的测试
- 1GB 内存
- MySQL 服务，并且设置 max_connections > 150

## 启动 Rancher 服务器
在安装了 Docker 的服务器上可以很简单的使用命令启动 Rancher 服务器。

```
$ sudo docker run -d --restart=always -p 8080:8080 rancher/server
```

### Rancher UI
Rancher 的 UI 和 API 默认工作在 8080 端口。Docker 镜像下载后，还需要 1-2 分钟才可以成功启动并显示页面。

访问以下地址：`http://<SERVER_IP>:8080`，这里`<SERVER_IP>`是指运行 Rancher 服务器的主机用于网络访问的IP地址。

一旦 UI 处于启动和运行状态，就可以开始 [添加主机]({{site.baseurl}}/rancher/installing-rancher/add-host.html) 。主机添加到 Rancher 服务器中后，您可以开始添加 [服务]({{site.baseurl}}/rancher-ui/applications/stacks/add-service.html) 或 使用 [Rancher catalog]({{site.baseurl}}/rancher/installing-rancher/add-host.html) 启动模板。

### 开启活动目录或 OpenLDAP 支持(TLS)
为了开启 Rancher 活动目录或 OpenLDAP 支持(TLS)，Rancher Server 容器在启动时需要加载证书。您需要在运行 Rancher Server 的 Linux 主机上保存证书。

启动 Rancher 服务器并使用绑定挂载卷的方式加载证书，证书在容器中 **必须** 使用`ca.crt`命名。

```
$ sudo docker run -d --restart=always -p 8080:8080 \
  -v /dir_that_contains_the_cert/cert.crt:/ca.crt rancher/server
```

您可以通过检查 Rancher 服务器容器日志确认 `ca.crt` 是否已经成功的传递给了 Rancher 服务器。

```
$ docker logs <server_container_id>
```

在日志的开始，会提示证书 `ca.crt` 已经添加。

```
DEFAULT_CATTLE_RANCHER_COMPOSE_WINDOWS_URL=https://releases.rancher.com/compose/beta/latest/rancher-compose-windows-386.zip
Adding ca.crt to Certs.
Updating certificates in /etc/ssl/certs... 1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d....
done.
done.
[BOOTSTRAP] Starting Cattle
```

### 绑定挂载 MYSQL 卷
如果你想将容器内数据库持久化在主机上，可以在启动 Rancher 服务容器时，绑定并挂载 MySQL 数据卷。

```
$ sudo docker run -d -v <host_vol>:/var/lib/mysql --restart=always -p 8080:8080 rancher/server
```

使用这个命令，数据库将持久化保存在主机上。如果您已有一个 Rancher 服务器容器，请参照我们的 [升级文档]() 。

### 使用外部数据库
如果您更希望使用外部的数据库运行 Rancher 服务器，请参照如下操作连接 Rancher 到数据库。您需要一个已经创建好的数据库，但是不需要创建任何数据库对象，Rancher 将会自动创建所有相关的数据库对象。

Rancher 服务器需要使用以下环境变量，并通过 `docker run` 命令启动 Rancher 服务器来连接外部数据库。

- CATTLE\_DB\_CATTLE\_MYSQL\_HOST:  数据库实例的主机名或IP地址
- CATTLE\_DB\_CATTLE\_MYSQL\_PORT:  3306
- CATTLE\_DB\_CATTLE\_MYSQL\_NAME:  数据库名
- CATTLE\_DB\_CATTLE\_USERNAME:  用户名
- CATTLE\_DB\_CATTLE\_PASSWORD:  密码

> **注意：**
> 
> 数据库名和用户名必须已经存在，Rancher 不会去创建数据库。 

这是一个用于创建数据库和用户的 SQL 命令。

```
sql CREATE DATABASE IF NOT EXISTS cattle COLLATE = 'utf8_general_ci' CHARACTER SET = 'utf8'; GRANT ALL ON cattle.* TO 'cattle'@'%' IDENTIFIED BY 'cattle'; GRANT ALL ON cattle.* TO 'cattle'@'localhost' IDENTIFIED BY 'cattle';
```
以上命令创建了名为 ‘cattle’ 的数据库，创建了用户名为 ‘cattle’ 的用户。

创建了数据库和用户后，使用环境变量参数，并启动 Rancher 服务器。

```
$ sudo docker run -d --restart=always -p 8080:8080 \
    -e CATTLE_DB_CATTLE_MYSQL_HOST=<hostname or IP of MySQL instance> \
    -e CATTLE_DB_CATTLE_MYSQL_PORT=<port> \
    -e CATTLE_DB_CATTLE_MYSQL_NAME=<Name of Database> \
    -e CATTLE_DB_CATTLE_USERNAME=<Username> \
    -e CATTLE_DB_CATTLE_PASSWORD=<Password> \
    rancher/server
```

### 使用 HTTP 代理环境变量
为了使用 HTTP 代理，需要修改 Docker 服务支持代理。在 Rancher 服务器启动前，编辑`/etc/defalut/docker`文件指定您的代理并重新启动 Docker 服务。

```
$ sudo vi /etc/default/docker
```

在这个文件中编辑 `#export http_proxy="http://127.0.0.1:3128/"` 指向您的代理。保存修改并重新启动Docker 服务，每个操作系统重启 Docker 服务的方式是不同的。

> **注意：**
> 
> 如果您使用 systemd 运行 Docker ，请参照 Docker [文档](https://docs.docker.com/articles/systemd/#http-proxy) 配置 HTTP 代理。

为了加载 [Rancher catalog]() ，需要在启动 Rancher 服务器时加载 HTTP 代理的环境变量信息。

```
$ sudo docker run -d \
    -e http_proxy=<proxyURL> \
    -e https_proxy=<proxyURL> \
    -e no_proxy="localhost,127.0.0.1" \
    --restart=always -p 8080:8080 rancher/server
```

如果不使用 [Rancher catalog]() ，则使用正常方式运行启动 Rancher Server 命令。

在 Rancher 中 [添加主机]({{site.baseurl}}/rancher/installing-rancher/add-host/) ，无需使用 HTTP 代理。

----
> 译者： XiaoBao Zhang
> 
> From： http://docs.rancher.com/rancher/installing-rancher/installing-server/