# 使用 Docker 原生命令行
Rancher与Docker原生命令行的集成，使它能够轻易地与其他DevOps、Docker工具结合使用。在更高一个层面来说，在Rancher的外部去启动，停止或者销毁容器，Rancher也将会检测到这些容器的变化，并将其更新到对应的状态

### Docker 事件监控
Rancher通过监听Docker events来更新所有主机的容器状态。当在Rancher平台之外启动、停止或者销毁容器（比如说，直接在主机上执行`docker stop sad_einstein`），Rancher将会检测到这些容器的所发生的变化并将其更新到相应的状态。


> **注意：** 当前的一个限制是：Rancher会等待容器已经启动了（不是创建）才会将其导入到Rancher，执行`docker create ubuntu`会导致这个容器在Rancher UI上消失，但是执行`docker start ubuntu`或者`docker run ubuntu`则不会。

Rancher通过`docker events`命令去监控一个主机上容器的变化，你也可以试试使用这个命令去观察Docker的事件流。

### 将已启动的容器加入到Rancher中
你可以不通过Rancher去启动容器，并能让Rancher去控制这些容器。这意味着容器可以运行在跨主机的网络中。在创建容器的时候添加一个值为`true`的`io.rancher.container.network`标签来启用这个特性。

```bash
$ docker run -l io.rancher.container.network=true -itd ubuntu bash
```

如需了解更多关于Rancher管理网络和跨主机网络，请阅读Rancher[基本概念]({{site.baseurl}}/concepts/).

### 导入已存在的容器
Rancher还支持直接导入主机上已经存在的容器。你可以参考[添加自己的主机]({{site.baseurl}}/rancher-ui/infrastructure/custom.html)添加一个自己的主机，添加之后会发现主机上所有的容器都会被检测到并导入Rancher中。


### 定期同步状态
除了实时监听Docker事件外，Rancher还定期的去同步主机的状态。Rancher每隔5秒钟会去收集主机所有的容器信息，以确保平台上所展示的容器状态和实际的主机上的状态相符。防止由于网络中断或者服务器重启导致Rancher没接受到Docker事件。使用这种方式去同步状态，Rancher上的容器状态将会和主机上的容器状态保持一致。比如说，Rancher觉得一个容器正在运行中，但它在主机上实际上是被停止了，Rancher将会把该容器的状态更新成停止状态，而不是去尝试重启这个容器。

