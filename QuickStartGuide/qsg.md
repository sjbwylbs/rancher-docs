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



