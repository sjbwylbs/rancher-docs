# 元数据服务
 Martin 初始化版本翻译中，欢迎随后 review

使用 Rancher 的元数据服务，你能在任意 Rancher 所管理的何容器内部来查询到关于容器所管理的网络和的相关信息。 元数据可以涉及到容器自身、所在服务或者堆栈，以及容器所允许的主机，等等。元数据默认格式是 JSON 。 

容器可以通过以下几种方式在 Rancher 管理的网络上启动。

* 通过图形界面 [UI]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/applications/stacks/adding-services/), 一个服务/容器 被启动时使用了 _Managed_ 网络参数。默认情况下，服务网络已经被设置为 _Managed_ 。
* 通过 [Rancher-Compose]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-compose/) 命令行，一个服务/容器，当没有被设置为网络模式 (`net`)，在Rancher 管理的网络上被启动。
* 当用 [ docker 原生命令]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/native-docker/#joining-natively-started-containers-to-the-rancher-network) 启动容器, 如果你在 `docker run` 命令中加入标签 `io.rancher.container.network=true` ，这样容器就加入了 Rancher 的管理网络。

> **注意:** 元数据服务对于Rancher 的系统容器是不可用的，如：Network Agent 和 LB Agent 容器。

## 如何获取到元数据 
---

在 Rancher 的图形界面上，您能从容器的下拉菜单 **Execute Shell** 上进入到容器的命令行。下拉菜单在鼠标滑过容器的时候可以出现。 

你可以使用 curl 命令来过去元数据。

```bash
# 如果 curl 没有安装，安装它
$ apt-get install curl
# curl 命令可以返回纯文本的相应结果
$ curl http://rancher-metadata/<version>/<path>
```
访问路径依赖于你想访问什么样元数据和想得到的相应结果的格式。

元数据 | 路径  | 描述
----|---- | ---
容器 | `self/container` | 提供的元数据是您执行元数据查询命令的容器自身的信息
容器所在的服务  | `self/service` | 提供的元数据是您执行元数据查询命令的容器所属服务的信息
容器所在的堆栈  | `self/stack` | 提供的元数据是您执行元数据查询命令的容器所属堆栈的信息 
容器所运行的主机  | `self/host` | 提供的元数据是您执行元数据查询命令的容器所运行于主机的信息
其它容器  | `containers` | 提供所有其它容器的元数据。用纯文本格式，它可返回了所有容器索引后的清单。用 JSON 格式，它可返回所有容器的所有元数据信息。使用索引或者容器名称在路径中，你可以获取特定容器的元数据。
其它服务  | `services` | 提供所有其它服务的元数据。用纯文本格式，能返回所有服务的索引编号。用 JSON 格式，它能返回所有服务的元数据信息。在路径中使用索引编号或者名称，可以获得一个特定服务的元数据信息。如果去查看容器相关信息，在V1 (`2015-07-25`)中只能返回容器的名称，而在V2 (`2015-12-19`)中则可以容器对象清单。 
其它堆栈  | `stacks/<stack-name>` | 提供所有其它堆栈的元数据。用纯文本格式，能返回所有堆栈的索引编号。用 JSON 格式，它能返回所有堆栈的元数据信息。在路径中使用索引编号或者名称，可以获得一个特定堆栈的元数据信息。如果去查看容器相关信息，在V1 (`2015-07-25`)中只能返回容器的名称，而在V2 (`2015-12-19`)中则可以容器对象清单。

### 元数据的版本

在 `curl` 命令里，我们强烈建议使用一个的定的版本，而不总是使用 `latest`。

> **注意:** 我们对 `latest` 版本会持续更改，不同版本返回的数据可能不同，因此可能会影响到您的代码的兼容性。

元数据服务的版本是基于日期的。

版本参考 | 版本|
---- | ----
V2 | 2015-12-19 |
V1 | 2015-07-25 |

#### 版本的差异

**V1 vs. V2**

当使用路径  `/services/<service-name>/containers` 或 `</stacks/<stack-name>/services/<service-name>/containers` 查看容器元数据信息时，V1 返回容器的名称，而 V2 返回所有容器对象。元数据服务的 V2 版本可以提供更多信息。 

_实例_

在 Rancher 中，有一个堆栈名为 `foostack` ，它包含一个由三个容器组成的名为 `barservice` 的服务。 

```bash
# 使用 V1 返回的只是服务的容器名称
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-07-25/services/barservice/containers'
["foostack_barservice_1", "foostack_barservice_2", "foostack_barservice_1"]

# 使用 V2 返回的是服务中所有容器对象
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-12-19/services/barservice/containers'
[{"create_index":1, "health_state":null,"host_uuid":...
...
# 服务里所有容器的元数据清单
...
...}]

# 使用 V2, 你能详细查看某个特定容器对象
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-12-19/services/barservice/containers/foostack_barservice_1'
[{"create_index":1, "health_state":null,"host_uuid":...
...
# 服务里所有容器的详细信息清单
...
...}]

# 使用 /stacks/<服务-名称>, 你能详细查看服务和容器

# 使用 V1 只能返回服务中容器的名称 
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-07-25/stacks/foostack/services/barservice/containers'
["foostack_barservice_1", "foostack_barservice_2", "foostack_barservice_1"]

# 使用 V2 能返回服务中所有容器对象详情清单
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-12-19/stacks/foostack/services/barservice/containers'
[{"create_index":1, "health_state":null,"host_uuid":...
...
# 服务里所有容器的详细信息清单
...
...}]
```

### 纯文本 vs JSON 格式

元数据信息能返回成纯文本合作和 JSON 格式。基于你怎样使用元数据的需求，你能选择任意格式。 

#### 纯文本

在执行 curl 命令时，您会从所请求的路径中获得纯文本。你可以从路径的最顶级开始查询，根据返回的可用的键值信息继续向下层访问，直到元数据服务返回了您想查询的数据。

```bash
$ curl 'http://rancher-metadata/2015-12-19/self/container'
create_index
health_state
host_uuid
hostname
ips/
labels/
name
ports/
primary_ip
service_name
stack_name
start_count
uuid
$ curl 'http://rancher-metadata/2015-12-19/self/container/name'
# 注意: Curl 不能换行，因此最后一行的返回值和命令提示符出现在同一行行。
Default_Example_1$root@<container_id>
$ curl 'http://rancher-metadata/2015-12-19/self/container/label/io.rancher.stack.name'
Default$root@<container_id>
# 数组可以通过索引号和名称来返回值
$ curl 'http://rancher-metadata/2015-12-19/services'
0=Example
# 你可以在路径中使用索引号或者名称
$ curl 'http://rancher-metadata/2015-12-19/services/0'
$ curl 'http://rancher-metadata/2015-12-19/services/Example'
```

#### JSON

把 `Accept: application/json` 添加在 curl 命令的请求标记中，可以返回 JSON 格式的数据。

```bash
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-12-19/self/container' 
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-12-19/self/stack' 
# 返回某个堆栈总其它服务的元数据
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-12-19/services/<service-name>' 
```

## 元数据字段 
---

### 容器

| 字段 | 描述 |
| ----| ----|
| `create_index` | 在服务里容器被启动的顺序编号，例如：2意味着它应该是在服务中第二个被启动的。注意：Crete_index 是不会被重复使用的。如果您的服务有两个容器，当你删除了第二个容器后，下一个被启动的容器得到的 create_index 编号是3，尽管这个服务中总共只有2个容器。
| `health_state` | 如果 [健康检查]({{site.baseurl}}/rancher-services/health-checks/) 被启用用了，这是容器健康状态信息。
| `host_uuid` | Rancher 服务器分配给主机的唯一识别标识。
| `hostname` | 容器的主机名。
| `ips` | 当多网卡被支持了，它将显示 IP 地址清单。
| `labels` | [容器上的标签]({{site.baseurl}}/rancher-ui/scheduling/#labels) 清单。标签的格式是 `key`:`value`。
| `name` | 容器的名称 
| `ports` |  [容器内使用的端口]({{site.baseurl}}/rancher-ui/infrastructure/containers/#port-mapping) 清单. 端口的格式 `hostIP:publicIP:privateIP[/protocol]`。
| `primary_ip` | 容器 IP 地址
| `service_index` | 容器所在的服务的编号
| `service_name` | 服务的名字 (如果存在)
| `stack_name` | 服务所在的堆栈的名字 (如果存在)
| `start_count` | 容器被自动的次数。
| `uuid` | Rancher 分配给容器的唯一标识。

### 服务

 字段 | 描述
----|----
`containers` | 服务中容器的名称清单。
`create_index` | Create_index of the last container created of the service. Note: Create_index is never reused. If you had a service with 2 containers and deleted the 2nd container, the create_index will be 2. The next container that gets launched for the service would update the create_index to 3 even though there are only 2 containers.
`expose` | 
`external_ips` |  [外部服务]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/applications/stacks/adding-external-services/) 的外部 IP 地址清单
`fqdn` | 服务的 FQDN 全域名 
`hostname` | [外部服务]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/applications/stacks/adding-external-services/) 的CNAME
`kind` | Rancher 服务的类型
`labels` | [服务上的标签]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/scheduling/#labels) 清单。标签的格式是 `key:value`.
`links` | 所连接的服务。 连接的格式是 `stack_name/service_name:service_alias`。`links` 会显示所有的键值  (例如： `stack_name/service_name` 所有连接) 和返回 `service_alias`, 你会需要继续向下查看来找到特定的键值。
`metadata` | [用户添加的元数据]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/metadata-service/#adding-user-metadata-to-a-service) 
`name` | 服务的名称
`ports` |  [在服务中所使用的端口]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/applications/stacks/adding-services/#port-mapping). 端口的格式是 `hostIP:publicIP:privateIP[/protocol]`。
`scale` | 服务的数量
`sidekicks` |  [sidekicks]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-compose/#sidekicks) 的服务名清单
`stack_name` | 服务所在的堆栈的名称
`uuid` | Rancher 分配给服务的唯一标识

### 堆栈

字段 | 描述
----|----
`environment_name` | 堆栈所在 [环境]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/environments/) 的名称
`name` |  [堆栈]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/applications/stacks/) 的名称
`services` | 堆栈中的服务的清单
`uuid` | Rancher 分配给堆栈的唯一标识。

### 主机

字段 | 描述
----|----
`agent_ip` | Rancher Agent 的 IP 地址，例如：`CATTLE_AGENT_IP`环境变量的值。
`hostId` | 在某个环境中主机的唯一标识 
`labels` | [主机标签]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/#host-labels) 清单。标签格式是 `key:value`
`name` |  [主机]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/) 名称
`uuid` |  Rancher 分配给主机的唯一标识

## 给服务添加用户元数据
---

Rancher 可以让用户给服务增加自己的元数据。目前，这个功能只能通过 [rancher-compose]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-compose/) 来实现，并且元数据需要是 `rancher-compose.yml` 文件的一部分。在 `metadata` 键里， yaml 将被解析为 JSON 格式用于元数据服务。

 `rancher-compose.yml` 示例

```yaml
service:
  # Scale of service
  scale: 3
  # User added metadata
  metadata:
    example:
      name: hello
      value: world
    example2:
      foo: bar

```

在这个服务启动之后，你可以通过元数据服务找到这个信息在  `.../self/service/metadata` 或者 `.../services/<service_id>/metadata` 中。


### JSON 查询

```bash
$ curl --header 'Accept: application/json' 'http://rancher-metadata/latest/self/service/metadata'
{"example":{"name":"hello","value":"world"},"example2":{"foo":"bar"}}

```

### Plaintext 查询

```bash
$ curl 'http://rancher-metadata/latest/self/service/metadata'
example/
$ curl 'http://rancher-metadata/latest/self/service/metadata/example'
name
value
$ curl 'http://rancher-metadata/latest/self/service/metadata/example/name'
# Note: Curl will not provide a new line, so single values will be on same line as the command prompt
hello$root@<container_id>
```

     