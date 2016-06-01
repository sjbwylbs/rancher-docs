---
title: quick-start-guide
layout: os-default

---

## 快速安装指南
---
如果你有特别的RancherOS主机要求，请浏览[运行RancherOS指南]({{site.baseurl}}/os/running-rancheros/)，
本指南的主要讲述使用[docker machine]({{site.baseurl}}/os/running-rancheros/workstation/docker-machine/)来启动RancherOS和介绍RancherOS能够做什么。

If you have a specific RanchersOS machine requirements, please check out our [guides on running RancherOS]({{site.baseurl}}/os/running-rancheros/).
With the rest of this guide, we'll start up a RancherOS using [docker machine]({{site.baseurl}}/os/running-rancheros/workstation/docker-machine/) and show you some of what RancherOS can do.

### 使用Docker Machine 启动RancherOS
### Launching RancherOS using Docker Machine

前进之前，你需要安装[Docker Machine](https://docs.docker.com/machine/)和[VirtualBox](https://www.virtualbox.org/wiki/Downloads)。
一旦你安装VirtualBox和Docker Machine，它只需要一个命令就可以运行RancherOS。

Before moving forward, you'll need to have [Docker Machine](https://docs.docker.com/machine/) and [VirtualBox](https://www.virtualbox.org/wiki/Downloads) installed.
Once you have VirtualBox and Docker Machine installed, it's just one command to get RancherOS running.

```bash
$ docker-machine create -d virtualbox --virtualbox-boot2docker-url https://releases.rancher.com/os/latest/rancheros.iso <实例机器名>
```

就是这么简单！你已经启动和运行了一个RancherOS实例。
That's it! You're up and running a RancherOS instance.

要登录到该实例，只需使用 `docker-machine` 命令。
To log into the instance, just use the docker-machine command.

```bash
$ docker-machine ssh <实例机器名>
```

### 初识RancherOS

### A First Look At RancherOS

在RancherOS中运行两个Docker守护进程。首先是所谓的**system-docker**，这就是RancherOS运行的系统服务，如NTPD和系统日志。您可以使用`system-docker`命令来控制**system-docker**守护进程。

另一个守护进程是**docker**，它可以通过使用正常的`docker`命令来访问。

There are two Docker daemons running in RancherOS. The first is called **system-docker**, which is where RancherOS runs system services like ntpd and syslog. You can use the `system-docker` command to control the **system-docker** daemon.

The other Docker daemon running on the system is **docker**, which can be accessed by using the normal `docker` command.

当你第一次启动RancherOS，还没有容器运行在'docker`守护进程。不过，如果你对运行`system-docker`的命令，你会看到一些RancherOS系统自己维护的服务。

When you first launch RancherOS, there are no containers running in the `docker` daemon. However, if you run the same command against the `system-docker` instance, you’ll see a number of system services that are shipped with RancherOS.

>**注：**`system-docker`只能由root用户使用，所以每当你需要使用`system-docker`时，交互使用`sudo`命令。

> **Note:** `system-docker` can only be used by root, so it is necessary to use the `sudo` command whenever you want to interact with `system-docker`.

```bash
$ sudo system-docker ps
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS               NAMES
4710a91a0729        rancher/os-docker:v0.4.4    "/usr/sbin/entry.sh /"   6 minutes ago       Up 6 minutes                            docker
7ef4fad1612c        rancher/os-console:v0.4.4   "/usr/sbin/entry.sh /"   6 minutes ago       Up 6 minutes                            console
1b436b6b7fdb        rancher/os-network:v0.4.4   "/usr/sbin/entry.sh /"   6 minutes ago       Up 6 minutes                            network
2ce47a55d1bd        rancher/os-ntp:v0.4.4       "/usr/sbin/entry.sh /"   6 minutes ago       Up 6 minutes                            ntp
c2237144ec41        rancher/os-udev:v0.4.4      "/usr/sbin/entry.sh /"   6 minutes ago       Up 6 minutes                            udev
5373e592dc51        rancher/os-acpid:v0.4.4     "/usr/sbin/entry.sh /"   6 minutes ago       Up 6 minutes                            acpid
c5d8cd81a94c        rancher/os-syslog:v0.4.4    "/usr/sbin/entry.sh /"   6 minutes ago       Up 6 minutes                            syslog
```


一些容器在启动时运行，和其余时间，如`console`，`docker`等。这些容器始终运行。

Some containers are run at boot time, and others, such as the `console`, `docker`, etc. containers are always running.

## 开始使用RancherOS
## Using RancherOS
---


### 部署一个Docker容器
### Deploying a Docker Container


让我们尝试使用`docker`守护进程部署一个正常的Docker容器。该RancherO S`docker`守护等同于其他任何Docker环境，使一切正常Docker命令的工作。
Let's try to deploy a normal Docker container on the `docker` daemon.  The RancherOS `docker` daemon is identical to any other Docker environment, so all normal Docker commands work.

```bash
$ docker run -d nginx
```
你可以看到，nginx镜像的容器启动并运行。
You can see that the nginx container is up and running:

```sh
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
e99c2c4b8b30        nginx               "nginx -g 'daemon off"   12 seconds ago      Up 11 seconds       80/tcp, 443/tcp     drunk_ptolemy
```

### 部署一个系统服务容器
### Deploying A System Service Container

下面是一个简单的Docker容器安装`Linux-dash`，它是用于监控Linux服务器最小的低开销网络信息中心。该Dockerfile将是这样的：
The following is a simple Docker container to set up Linux-dash, which is a minimal low-overhead web dashboard for monitoring Linux servers. The Dockerfile will be like this:

```
FROM hwestphal/nodebox
MAINTAINER hussein.galal.ahmed.11@gmail.com

RUN opkg-install unzip
RUN curl -k -L -o master.zip https://github.com/afaqurk/linux-dash/archive/master.zip
RUN unzip master.zip
WORKDIR linux-dash-master
RUN npm install

ENTRYPOINT ["node","server"]
```
使用`hwestphal/ nodebox`镜像，它采用了busybox的镜像，并安装`node.js`和`npm`。我们下载的`Linux-dash`的源代码，然后运行服务器。 `Linux-dash`将默认在端口80上运行。
Using the `hwestphal/nodebox` image, which uses a busybox image and installs `node.js` and `npm`. We downloaded the source code of Linux-dash, and then ran the server. Linux-dash will run on port 80 by default.

要使用`systemd-docker`运行这个容器使用下面的命令：

To run this container with system-docker use the following command:

```sh
$ sudo system-docker run -d --net=host --name busydash husseingalal/busydash
```

在这行命令中，我们使用`--net= host`告诉`system-docker`不使用容器化容器的网络，并使用主机的网络代替。 在运行容器后您可以通过访问`http://<IP_OF_MACHINE>`看到监控服务器。

In the commad, we used `--net=host` to tell `system-docker` not to containerize the container's networking, and use the host’s networking instead. After running the container, you can see the monitoring server by accessing `http://<IP_OF_MACHINE`.

![System Docker Container]({{site.baseurl}}/img/os/Rancher_busydash.png)

为了容器能在系统重启时存活，你可以创建``/opt/rancher/bin/start.sh`脚本，并添加Docker启动项以保证每次启动时自动启动。
To make the container survive during the reboots, you can create the `/opt/rancher/bin/start.sh` script, and add the docker start line to launch the docker at each startup.

```bash
$ sudo mkdir -p /opt/rancher/bin
$ echo “sudo system-docker start busydash” | sudo tee -a /opt/rancher/bin/start.sh
$ sudo chmod 755 /opt/rancher/bin/start.sh
```

### 使用[ROS工具包]({{site.baseurl}}/os/rancheros-tools/ros/)
### Using [ROS]({{site.baseurl}}/os/rancheros-tools/ros/)


可与RancherOS使用的另一种有用的命令是`ros`,它可用于控制和配置系统。
Another useful command that can be used with RancherOS is `ros` which can be used to control and configure the system.

```bash
$ ros -v
ros version 0.0.1
```
RancherOS状态由云配置文件控制。 `ros`用于编辑系统的配置，参见例如系统的DNS配置：
RancherOS state is controlled by a cloud config file. `ros` is used to edit the configuration of the system, to see for example the dns configuration of the system:

```sh
$ sudo ros config get rancher.dns
- 8.8.8.8
- 8.8.4.4
```

当使用本地控制台的Busybox，控制台的任何更改将重启后会丢失，只有改变为`/home`或`/opt`将是持久的。控制台始终在每次启动时执行**/opt/rancher/bin/start.sh**。
你可以用`ros`启用[持久控制台]({{site.baseurl}}/os/configuration/custom-console/#console-persistence)和替换本地Busybox的控制台。
为了使启用Ubuntu的控制台，使用以下命令：

When using the native Busybox console, any changes to the console will be lost after reboots, only changes to `/home` or `/opt` will be persistent. The console always executes **/opt/rancher/bin/start.sh** at each startup. You can use `ros` to enable a [persistent console]({{site.baseurl}}/os/configuration/custom-console/#console-persistence) and replace the native Busybox console. In order to enable the Ubuntu console, use the following command:

```sh
$ sudo ros service enable ubuntu-console
# 您必须在重启以切换到Ubuntu的控制台
# You must reboot in order to switch to the ubuntu console
$ sudo reboot
```
### 结论
### Conclusion

RancherOS是一个简单的Linux发行版非常适合运行Docker。通过采用容器化的系统服务和利用Docker进行管理，RancherOS希望能够为运行容器提供了非常可靠，易于管理的操作系统。

RancherOS is a simple Linux distribution ideal for running Docker.  By embracing containerization of system services and leveraging Docker for management, RancherOS hopes to provide a very reliable, and easy to manage OS for running containers.
