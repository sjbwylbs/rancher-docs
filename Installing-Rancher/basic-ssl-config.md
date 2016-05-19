## SSL基本配置

为了可以正常使用 `https` 地址访问 Rancher 服务器，您需要设置代理的证书卸载功能。同时，我们提供了一些配置 NGINX 或 Apache 代理的例子，当然，您也可以使用其他工具。

#### 需求
除了典型的 Rancher 服务器 [需求]() 您还需要：

- 有效的 SSL 证书：如果您的证书不包含在 Ubuntu CA 中，请参照 [自签名证书说明]() 。
- 配置 DNS 记录。

#### 启动 Rancher 服务器
在我们的示例配置方案中，所有流量将通过代理被发送到 Rancher 服务器的 Docker 容器中。也有替代的配置方案可以实现，但这个示例相对比较简单。
启动 Rancher 服务器，这里我们添加了选项 `--name=rancher-server` 用来连接代理容器和 Rancher 服务器容器。

```
$ sudo docker run -d --restart=always --name=rancher-server rancher/server
```
> **注意：**
> 
> 在我们的示例中，我们假设代理运行在单独的容器中。如果您的代理计划由主机提供服务，需要将 Rancher 容器的 `8080` 端口映射至本地主机，可以向 `docker run` 命令添加参数 `-p 127.0.0.1:8080:8080` 实现。

如果您想使用现有的 Rancher 实例，您可以基于现有的 Rancher 实例创建一个新的 Rancher 实例。

- 若 Rancher 实例的 MySQL 数据库运行在容器内，在启动新的 Rancher 实例时，请参照 [升级说明]() 创建数据容器并且在启动时添加 `--volumes-from=<data_container>` 。
- 若 Rancher 实例使用了 [绑定挂载数据库]() 的方式，请参照 [绑定挂载实例升级说明]()。
- 若 Rancher 实例使用外部数据库，则直接停止并删除现有的 Rancher 实例容器，并参照 [外部数据库连接说明]() 启动新的容器。

> **注意：**
> 
> 在新的 Rancher 实例容器运行后，请确认已经 **删除** 了旧的 Rancher 实例容器，否则，在您的主机重启后，因为我们在 `docker run` 命令时加入了 `--restart=always` 参数，会导致旧容器启动。

### NGINX 配置示例

这里的配置是 NGINX 运行的最低配置，您应该定制配置来满足您的需求。

#### 节点设置
- `rancher-server` 是您的 Rancher 服务器容器名。您必须在启动 Rancher 服务器容器的时候加入 `--name=rancher-server` 选项。并且在启动 NGINX 容器的时候加入 `--link=rancher-server` 选项才能使这些配置正常工作。
- `<server>` 可以任意命名，但是必须保证 http 和 https 配置中的名称相同。

```
upstream rancher {
    server rancher-server:8080;
}

server {
    listen 443 ssl;
    server_name <server>;
    ssl_certificate <cert_file>;
    ssl_certificate_key <key_file>;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Port $server_port;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://rancher;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        # This allows the ability for the execute shell window to remain open for up to 15 minutes. Without this parameter, the default is 1 minute and will automatically close.
        proxy_read_timeout 900s;
    }
}

server {
    listen 80;
    server_name <server>;
    return 301 https://$server_name$request_uri;
}
```

# APACHE 配置示例
这是一个 Apache 的配置。

#### 节点设置
- `<server_name>` 是您 Rancher 服务器容器的名字，所以在启动 Apache 容器时需要加入 `--link=<server_name>` 才能使配置正常工作。
- 在代理配置中，您需要替换 `rancher` 为您的信息。

```
<VirtualHost *:80>
  ServerName <server_name>
  Redirect / https://<server_name>/
</VirtualHost>

<VirtualHost *:443>
  ServerName <server_name>

  SSLEngine on
  SSLCertificateFile </path/to/ssl/cert_file>
  SSLCertificateKeyFile </path/to/ssl/key_file>

  ProxyRequests Off
  ProxyPreserveHost On

  RewriteEngine On
  RewriteCond %{HTTP:Connection} Upgrade [NC]
  RewriteCond %{HTTP:Upgrade} websocket [NC]
  RewriteRule /(.*) ws://rancher:8080/$1 [P,L]

  RequestHeader set X-Forwarded-Proto "https"
  RequestHeader set X-Forwarded-Port "443"

  <Location />
    ProxyPass "http://rancher:8080/"
    ProxyPassReverse "http://rancher:8080/"
  </Location>

</VirtualHost>
```

## 更新主机注册
在 Rancher 配置完成后，用户界面将使用 `https://<your domain>/` 访问。在 [添加主机]() 之前，您需要正确配置 SSL 方式的 [主机注册]() 地址。

## 在 AWS ELB 后使用 SSL 运行 Rancher
默认情况下，ELB 不支持在 HTTP/HTTPS 模式下启用 websockets 。由于 Rancher 使用了 websockets ，则 ELB 必须修改配置以便 Rancher 的 websockets 可以正常工作。

#### Rancher 的 ELB 配置需求
- 启用 [代理协议](http://docs.aws.amazon.com/ElasticLoadBalancing/latest/DeveloperGuide/enable-proxy-protocol.html) 模式
- 配置 TLS/SSL 方式的前端以及 TCP 方式的后端

## 使用自签名证书（测试）
#### 免责声明
这个配置可以工作在单节点 Rancher 服务器（非HA安装）模式的核心服务上，没有人验证是否支持 [Rancher 目录]() 是否被支持。

Rancher Compose 命令行默认需要将 CA 证书存储在操作系统，请参照 [Golang](https://golang.org/src/crypto/x509/)。

#### 服务器必备条件
- CA 证书 PEM 格式的文件
- 已经签署证书的 Rancher 服务器
- NGINX 和 Apache 配置证书卸载，并反向代理到 Rancher 服务器

#### Rancher 服务器
1. 使用修改后的 Docker 命令启动 Rancher 服务器容器。证书 **必须** 在容器中被命名为 `ca.crt`。
	
	```
	$ sudo docker run -d --restart=always -p 8080:8080 -v /some/dir/cert.crt:/ca.crt rancher/server
	```
	> **注意：**
	> 
	> 如果你正在运行 NGINX 或 Apache 容器，您可以直接访问这些实例，不需要通过 Rancher 服务器的 8080 端口访问用户界面。
	这条命令将会配置 Rancher 服务器的证书链，所以 Rancher 的服务，如机器配置，目录和 compose 均可以和 Rancher 服务器通讯。

2. 如果您正在使用一个 NGINX 或 Apache 容器卸载证书，您需要在启动容器的命令增加 `--link=` 选项。
3. 通过 `https` 访问 Rancher 用户界面，即 `https://rancher.server.domain` 。
4. 更新 [主机注册]() 为 SSL 方式。
> **注意：**
> 
> 除非您的机器信任了用于签署 Rancher 服务器的 CA 证书，否则当你访问页面时，浏览器会给出一个不可信网站的警告信息。

#### 添加主机
1. 如果您想添加主机到 Rancher ，您必须在主机上以 pem 格式将 CA 证书保存到 `/var/lib/rancher/etc/ssl` 目录并命名为 `ca.crt` 。
2. 添加 [自定义主机]() ，也就是从用户界面粘贴命令。命令已经加入了 `-v /var/lib/rancher:/var/lib/rancher` 选项，所以文件会被自动复制到您的主机上。


----
> 译者： XiaoBao Zhang
> 
> From： http://docs.rancher.com/rancher/latest/en/installing-rancher/installing-server/basic-ssl-config/