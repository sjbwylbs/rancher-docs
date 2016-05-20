## 添加主机
---
在 Rancher 中，我们在网页上提供了关于如何添加公有云主机的简洁的说明，还包括了对于不支持的公有云如何做的说明。在 **Infrastructure** 页面点击  **Hosts** 菜单，在点击 **Add Host** 按钮。

### 主机最小化需求

* 任何能支持 Docker 1.10.3 的 Linux 发行版。其中 [RancherOS](http://docs.rancher.com/os/), Ubuntu, RHEL/CentOS 7 是经过了大量测试的。
* 1GB 内存
* 建议 CPU 具备 AES-NI 特性

### 工作机制?

当 Rancher Agent 容器在主机上被启动了之后，主机开始去连接 Rancher 服务器。在**Add Host** -> **Custom** 页面上显示的很长的添加主机命令中的注册秘钥被 Rancher Agent 用来做对服务器端的首次连接。基于这个连接，它在 Rancher 服务器中生成了一个 Agent 账户和 API 密钥对。这个秘钥对将被用于后续所有与服务器端的通讯中，这种认证方式和环境 API 秘钥的认证鉴权逻辑是完全一致的。

这种设计是假设运行在外部的主机和硬件是不可信的，由于这些资源具有潜在的风险。Agent 的账号仅仅具有它所需要访问的资源的权限，由 Agent 端发送来的事件实际上是被检查的。而反方向并没有Agent 去校验主机的机制，因此您还可以配置用于 Agent 和 Rancher 服务器通讯的 TLS 证书校验。 

不同环境的 IPSec 密钥是不同的，它被保存在数据库里，它在 Agent 用 API 密钥对注册的时候被发送给主机。主机直接的连接是点对点的，是 AES 加密的，这种加密是可以被大多数型号的 CPU 做加速的。

<a id="addhost"></a>

## 添加主机
---

在第一次做主机添加的时候，你需要配置一次  [Host Registration]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/settings/#host-registration/)。这个配置决定了主机用什么 DNS 名称或者 IP 地址和端口来连接 Rancher API。默认，我们已经选择了管理 IP 和端口`8080`。如果您确定需要改变 IP 地址，请确认这个地址和端口是被用来连接 RAncher API 的。在任何时候，您能更新[Host Registration]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/settings/#host-registration/)。在完成主机注册的配置后，点击 **Save** 按钮。

我们支持直接添加公有云主机或者云里已经创建好的主机。对于公有云，我们通过 `docker-machine`创建主机，并支持任何支持 `docker-machine` 的云主机镜像。

选择你想添加的公有云主机：

* [Adding Custom Hosts]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/custom/)
* [Adding Amazon EC2 Hosts]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/amazon/)
* [Adding Azure Hosts]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/azure/)
* [Adding DigitalOcean Hosts]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/digitalocean/)
* [Adding Exoscale Hosts]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/exoscale/)
* [Adding Packet Hosts]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/packet/)
* [Adding Rackspace Hosts]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/rackspace/)
* [Adding Hosts from Other Drivers]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/other/)

当一个主机被添加到 Rancher 中，一个 Rancher agent 容器在该主机上被启动。Ranher 将自动地下拉正确版本号的 `rancher/agent` 镜像，来运行所需版本的 Agent。Agent 版本的标签必须对应用相应的 Rancher 服务器版本。

<a id="labels"></a>

### 主机标签

对于每一个主机，你可以通过打标签的方式来组织它们。当启动 rancher/agent 容器的时候，标签被当做环境变量来添加了。在图形界面中添加主机标签是用键/值对格式的，其中见必须是唯一标识符。如果你添加了两个相同的键，并使用不了不同的值，我们将只使用后一个输入的键/值对。

通过给主机打标签，你可以在之后的服务调度/负载均衡/服务定义 ( [schedule services/load balancers/services]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/scheduling/))中用到，如给某个服务  [services]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/applications/stacks/adding-services/) 建立一个它运行主机的白名单或者黑名单。  

如果您打算使用一个外部的 DNS 服务  [external DNS service]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/dns-service/) 那么您需要让 DNS 记录使用一个 IP 而不是主机 IP  [to program the DNS records using an IP other than the host IP]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/dns-service/#using-a-specific-ip-for-external-dns)，接着您需要在主机上使用这个标签  `io.rancher.host.external_dns_ip=<IP_TO_BE_USED_FOR_EXTERNAL_DNS>` 。打此主机标签可以在注册这个主机的时候或者主机已经被添加到 Ranher 中以后，它应该在外部 DNS 服务开始使用到之前做。这个标签的值对于外部的 DNS 服务需要是符合某种可编程的逻辑。

当使用图形界面添加不同类型的云主机的时候，rancher/agent 命令会被自动地启动，并会使用网页图形界面中您所制定的标签。 

当您添加自动定义主机是，您可以在图形界面中添加标签，并且它会自动地添加环境变量 (`CATTLE_HOST_LABELS`) 标签对到命令中。

_示例_

```bash
# 在 rancher/agent 命令中添加一个主机标签
$  sudo docker run -e CATTLE_HOST_LABELS='foo=bar' -d --privileged \
    -v /var/run/docker.sock:/var/run/docker.sock rancher/agent:v0.8.2 \
    http://<rancher-server-ip>:8080/v1/projects/1a5/scripts/<registrationToken>

# 添加多个主机标签需要使用`&`来连接在一起
$  sudo docker run -e CATTLE_HOST_LABELS='foo=bar&hello=world' -d --privileged \
    -v /var/run/docker.sock:/var/run/docker.sock rancher/agent:v0.8.2 \
    http://<rancher-server-ip>:8080/v1/projects/1a5/scripts/<registrationToken>
```

<br>

> **注意:** `rancher/agent` 的版本是必须和 Rancher 服务器版本保持对应的。你需要在这里检查自定义命令中所使用的版本号标签是正确的。

#### 自动化添加的主机标签

Rancher 自动化生成的主机标签是关于主机的 Linux 内核版本和 Docker 引擎版本的。 

键 | 值 | 描述
----|----|----
`io.rancher.host.linux_kernel_version` | 主机的 Linux 内核版本 (例如： `3.19`) |  主机所运行的 Linux 内核版本号。
`io.rancher.host.docker_version` | 主机的 Docker 版本 (例如： `1.10.3`) | 主机所安装的 Docker 引擎版本。


### 主机在代理服务器后面
为了支持主机位于代理服务器之后，您将修改 Docker 引擎的配置来指向一个代理服务器。对此的描述请见  [adding custom host page]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/custom/#hosts-behind-a-proxy)页面。

<a id="machine-config"></a>

### 登录云主机 
在 Rancher 启动主机之后，您可以登录这些主机。我们提供在创建主机过程中生成的证书，并提供下载。点击主机上的**Machine Config** 下拉菜单。这将下载一个包含所有证书的 tar.gz 文件。

SSH 登录到您的主机，打开命令行工具，进入包含所有证书文件的目录，使用 `id_rsa` 证书，用如下命令连接。

```bash
$ ssh -i id_rsa root@<IP_OF_HOST>
```

## 克隆主机
---

由于需要使用一个访问密钥来启动云主机，您可能想通过不用每次都输入密钥的方式来更简单地创建另外其他的云主机。Rancher 提供了克隆的功能来创建新相似的云主机。从主机的下拉菜单中选择 **Clone** 。它会带您进入**Add Host**页面，这个页面上的密码部分已经为您填写好了。

## 编辑主机
---

这个菜单位于一个已经添加成功的主机的下拉菜单中。在  **Infrastructure** -> **Hosts**页面，该菜单当您的鼠标滑过主机的下拉菜单按钮的时候出现。如果您点击主机名来查看主机详情，此菜单在页面的右上角的下拉菜单图标中。它就位于主机状旁。

如果您点击 **Edit** 菜单，您可以更新名称，描述或者主机的标签。 

### 停用/激活主机

停用主机将把主机置于 _Inactive_ 状态。在这个状态里，不会有新容器被部署。该主机上任何已经存在的活动的容器请依然照旧运行，对容器还是可以执行动作（启动/停止/重启）。这个主机将会继续保持和Rancher 服务器的连接。

当一个主机处于_Inactive_ 状态，您还可以通过点击主机的下拉菜单中的 **Activate** 使其返回到 _Active_ 状态。

> **注意:** 如果一个主机在 Rancher 中已经宕机（例如：处于`reconnecting` 或 `inactive` 状态 ），您将需要执行一次健康检查来让您的服务所使用的容器在另外的主机上启动起来。

## 删除主机
---

为了从服务器上删除一个主机，您将需要在主机的下拉菜单中做这些操作。

选择 **Deactivate** 。当主机已经完全处于停用状态了，主机将进入 _Inactive_ 状态。选择 **Delete**。服务器将开始执行主机删除的操作。在完成了删除动作之后状态先会变为_Removed_。在彻底重图形界面上消失之前它还会进入  _Purged_ 状态。

如果主机是在 Ranher 中创建的云主机，主机会将被从云端删除。如果云主机是通过使用自定义命令添加的，主机将依然保留在云端。

> **注意:** 对于自定义主机，所有容器包括 Rancher agent 容器都还会保留在主机上。

## 在 Rancher 之外删除主机
---

如果主机不是在 Rancher 中删除的，Rancher 服务器还会持续连接它。最终，这个主机的状态会持续为 _Reconnecting_ 状态，即使它没有被重连成功。因此您需要在图形界面中通过  **Delete** 删除这些主机，并它们清除掉。

