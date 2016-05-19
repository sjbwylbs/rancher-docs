# 多节点高可用(HA)

###### 基于 Rancher v1.0.1

#### 基本配置需求

- 多节点的HA配置请参照单节点 [需求]()
	- 节点需要开放的端口
		- 全局访问：TCP 端口`22`,`80`,`443`,`18080`（可选：用于在集群启动前 [查看并管理栈]() ）
		- 节点间连接：
			- UDP 端口：`500`,`4500`
			- TCP 端口：`2181`,`2376`,`2888`,`3888`,`6379`
- MySQL 数据库
	- 至少 1GB 内存
	- 每 Rancher 服务器节点 50 个连接（如：3 节点的高可用部署至少需要支持 150 个连接）
- 外部负载均衡器

#### 建议配置
- 每个 Rancher 服务器节点应该有 4GB 或 8GB 可用内存，意味着至少需要 8GB 或 16GB 的物理内存
- MySQL 数据库应该使用高速的磁盘
- 对于真正的高可用，建议使用主从的 MySQL，并做适当的备份。由于存在事物锁，可以选择 Galera 或强制写入单节点的方式。

#### 配置高可用的准备
1. 根据 [使用外部数据库启动单节点]() 说明部署一个至少拥有 1GB 内存的 MySQL 数据库，但是不要使用其中启动 Rancher 服务器的相关指令。因为默认情况下， 用户只能从本地访问数据库，你需要授权所有 Rancher 服务器节点对其的网络访问。
2. 配置一个外部负载均衡器并将端口 80 和 443 的流量指向运行 Rancher Server 的节点池。
3. 使用本文准备所有节点。这些节点都应该满足单节点部署 Rancher 服务器的[需求]()。（可选）您可以提前拉取 `rancher/server` 镜像到这些节点上。

	目前，我们的高可用集群支持 3 种配置。 1 节点：没有高可用； 3节点：任何一台主机可以宕机； 5 节点：任何两台主机可以宕机。
	> **注意：**
	>
	> 这些节点可以分布在同一个地区，并使用稳定的高速链路连接的多个数据中心，不建议分布在距离较远的区域中。如果你选择分布节点在同一个区域，Zookeeper 可以用来保证集群的高可用。如果你的节点分布在不同的数据中心，那么你只能保留问题最少的那个区域。

4. 在其中一个节点上，启动一个 Rancher 服务器用于生成配置脚本。下面这个脚本用于生成 Rancher 服务器同时连接到外部数据库并初始化数据。它将被引导高可用部署过程。最终，Rancher 服务器容器将使用此步骤替换为支持高可用的 Rancher 服务器容器。
	
	```
	$ sudo docker run -d -p 8080:8080 \
	-e CATTLE_DB_CATTLE_MYSQL_HOST=<hostname or IP of MySQL instance> \
	-e CATTLE_DB_CATTLE_MYSQL_PORT=<port> \
	-e CATTLE_DB_CATTLE_MYSQL_NAME=<Name of Database> \
	-e CATTLE_DB_CATTLE_USERNAME=<Username> \
	-e CATTLE_DB_CATTLE_PASSWORD=<Password> \
	-v /var/run/docker.sock:/var/run/docker.sock \
	rancher/server:v1.0.1
	```
	> **注意：**
	>
	> 请耐心等待，这个初始化步骤可能要15分钟才能完成
	
5. （可选）预先下载 `rancher/server` 镜像到 Rancher 节点。这里下载的镜像可用于配置脚本生成 Rancher 服务器。
	
	```
	# 这个版本需要是第四步中所使用的版本
	$ sudo docker pull rancher/server:v1.0.1
	```

#### 生成配置脚本
1. 访问 Rancher Server 地址 `http://<server_IP>:8080` 生成脚本。在 **Admin** -> **HA** 确认 Rancher Server 已经成功连接到外部数据库。如果没有正确配置，请重复上一节中的步骤 1 和 4 。
2. 选择集群大小，应该为您的 Rancher Server 节点数量，参照上一节中步骤 3 。
3. 在 **Host Registration URL** 中填写外部负载均衡器的 IPv4 地址或主机名。
4. 选择您想使用的证书类型。Rancher Server 可以为您生成一个自签名证书或者使用自己的有效证书。
5. 点击 **Generate Config Script** 。
6. 下载脚本并保存到本地。
7. 保存脚本后，停止用于生成脚本的 Rancher Server 容器。

#### 启动高可用的 Rancher
1. 为了使所有节点支持高可用，你需要在所有节点上使用配置脚本启动 Rancher Server 。脚本将启动一个 Rancher Server 容器并连接到之前创建的外部数据库。

	> **注意：**
	> 
	> 请确保您已经停止用于生成脚本 `rancher-ha.sh` 的 Rancher Server 容器后再运行配置脚本。否则，在你尝试在同一个节点运行配置脚本时，将会有一个端口冲突导致高可用节点无法启动。

2. 如果你之前生成配置脚本时提供了 **Host Registration URL** ，请导航到外部负载均衡器的 IP 或主机名。请注意，Rancher Server 的用户界面可能需要几分钟才可以使用。如果你的用户界面仍不可用，请参照 [查看并管理栈]() 。
3. 一旦用户界面可用，您将可以添加主机到 Rancher 高可用集群。在 **Admin** -> **HA** 标签可以查看高可用节点的数量。添加主机前，您需要保存证书 `/var/lib/rancher/etc/ssl/ca.crt` 并赋予 `400` 权限到您要添加的主机上。注册命令可以自动创建使用并管理证书。
4. 在您向环境中添加主机后，高可用设置已经完成，您可以开始通过用户界面 [添加服务]() ，从 Catalog [启动模板]() 或使用 [rancher-compose]() 启动服务。
	> **注意：**
	>
	> 如果您正在使用 AWS ，你需要为添加到 Rancher 的主机配置 IP 。如果你想添加 [自定义主机]() ，你需要在配置页面中填写公网 IP ，启动 Rancher agent 的命令会相应改变。已经通过页面添加的主机，必须 [ssh]() 登陆到机器重启 Rancher agent 使 IP 生效。

----
> 译者： XiaoBao Zhang
> 
> From： http://docs.rancher.com/rancher/installing-rancher/installing-server/multi-nodes/