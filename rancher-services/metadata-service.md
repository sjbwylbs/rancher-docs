# 元数据服务
初始化版本翻译中，欢迎随后 review

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

_势力_

In Rancher, there is a stack called `foostack` and it contains a service called `barservice` with 3 containers. 

```bash
# Using V1 returns only container names of the service
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-07-25/services/barservice/containers'
["foostack_barservice_1", "foostack_barservice_2", "foostack_barservice_1"]

# Using V2 returns container objects of the service
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-12-19/services/barservice/containers'
[{"create_index":1, "health_state":null,"host_uuid":...
...
# Lists all metadata for all containers of the service
...
...}]

# Using V2, you can drill down to a specific container's object
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-12-19/services/barservice/containers/foostack_barservice_1'
[{"create_index":1, "health_state":null,"host_uuid":...
...
# Lists all metadata for all containers of the service
...
...}]

# Using /stacks/<service-name>, you can drill down to services and containers

# Using V1 returns only container names of the service
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-07-25/stacks/foostack/services/barservice/containers'
["foostack_barservice_1", "foostack_barservice_2", "foostack_barservice_1"]

# Using V2 returns container objects of the service
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-12-19/stacks/foostack/services/barservice/containers'
[{"create_index":1, "health_state":null,"host_uuid":...
...
# Lists all metadata for all containers of the service
...
...}]
```

### Plaintext vs JSON 

The metadata can be returned in either plaintext or JSON format. Depending on how you want to use your metadata, you can pick either format. 

#### Plaintext

When executing the curl command, you'll receive plaintext for the path that was requested. You can start at the top level of the path and continue to update the path based on available keys until your metadata service provides the data you were looking for. 

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
# Note: Curl will not provide a new line, so single values will be on same line as the command prompt
Default_Example_1$root@<container_id>
$ curl 'http://rancher-metadata/2015-12-19/self/container/label/io.rancher.stack.name'
Default$root@<container_id>
# Arrays can use either the index or name to go get the values
$ curl 'http://rancher-metadata/2015-12-19/services'
0=Example
# You can either user the index or name as a path
$ curl 'http://rancher-metadata/2015-12-19/services/0'
$ curl 'http://rancher-metadata/2015-12-19/services/Example'
```

#### JSON

The JSON format can be retrieved by adding an `Accept: application/json` header to your curl command. 

```bash
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-12-19/self/container' 
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-12-19/self/stack' 
# Retrieve the metadata for another service in the stack
$ curl --header 'Accept: application/json' 'http://rancher-metadata/2015-12-19/services/<service-name>' 
```

## Metadata Fields 
---

### Container

| Fields | Description |
| ----| ----|
| `create_index` | The order number of which the container was launched in the service, i.e. 2 means it was the second container launched in the service. Note: Create_index is never reused. If you had a service with 2 containers and deleted the 2nd container, the next container that gets launched for the service would have a `create_index` of 3 even though there are only 2 containers in the service.
| `health_state` | The state of health for the container if a [health check]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/health-checks/) was enabled.
| `host_uuid` | Unique host identifier that Rancher server assigns to hosts
| `hostname` | The hostname of the container.
| `ips` | When multiple NICs are supported, it will be the list of IPs.
| `labels` | List of [Labels on Container]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/scheduling/#labels). Format for labels is `key`:`value`.
| `name` | Name of Container 
| `ports` | List of [Ports used in the container]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/containers/#port-mapping). Format for ports is `hostIP:publicIP:privateIP[/protocol]`.
| `primary_ip` | IP of container
| `service_index` | The last number in the container name of the service
| `service_name` | Name of service (if applicable)
| `stack_name` | Name of stack that the service is in (if applicable)
| `start_count` | The number of times the container was started.
| `uuid` | Unique container identifier that Rancher assigns to containers

### Service

 Fields | Description
----|----
`containers` | List of container names in the service
`create_index` | Create_index of the last container created of the service. Note: Create_index is never reused. If you had a service with 2 containers and deleted the 2nd container, the create_index will be 2. The next container that gets launched for the service would update the create_index to 3 even though there are only 2 containers.
`expose` | 
`external_ips` | List of External IPs for [External Services]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/applications/stacks/adding-external-services/)
`fqdn` | Fqdn of the service 
`hostname` | CNAME for [External Services]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/applications/stacks/adding-external-services/)
`kind` | Type of Rancher Service 
`labels` | List of [Labels on Service]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/scheduling/#labels). Format for labels is `key:value`.
`links` | List of linked services. Format for links is `stack_name/service_name:service_alias`. The `links` would show all the keys (i.e. `stack_name/service_name` for all links) and to retrieve the `service_alias`, you would need to drill down to the specific key.
`metadata` | [User added metadata]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-services/metadata-service/#adding-user-metadata-to-a-service) 
`name` | Name of Service
`ports` | List of [Ports used in the Service]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/applications/stacks/adding-services/#port-mapping). Format for ports is `hostIP:publicIP:privateIP[/protocol]`.
`scale` | Scale of Service
`sidekicks` | List of service names that are [sidekicks]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-compose/#sidekicks)
`stack_name` | Name of stack the service is part of
`uuid` | Unique service identifier that Rancher assigns to services

### Stack

Fields | Description
----|----
`environment_name` | Name of [Environment]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/configuration/environments/) that the Stack is in
`name` | Name of [Stack]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/applications/stacks/)
`services` | List of Services in the Stack
`uuid` | Unique stack identifier that Rancher assigns to stacks

### Host

Fields | Description
----|----
`agent_ip` | IP of the Rancher Agent, i.e. the value of the `CATTLE_AGENT_IP` environment variable.
`hostId` | Identifier of the host in the specific environment
`labels` | List of [Host Labels]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/#host-labels). Format for labels is `key:value`.
`name` | Name of [Host]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-ui/infrastructure/hosts/)
`uuid` | Unique host identifier that Rancher server assigns to hosts

## Adding User Metadata To a Service
---

Rancher allows users to add in their own metadata to a service. Currently, this is only supported through [rancher-compose]({{site.baseurl}}/rancher/{{page.version}}/{{page.lang}}/rancher-compose/) and the metadata is part of the `rancher-compose.yml` file. In the `metadata` key, the yaml will be parsed into JSON format to be used by the metadata-service.

Example `rancher-compose.yml` 

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

After the service is up, you can find metadata using the metadata service in `.../self/service/metadata` or in `.../services/<service_id>/metadata`. 


### JSON Query

```bash
$ curl --header 'Accept: application/json' 'http://rancher-metadata/latest/self/service/metadata'
{"example":{"name":"hello","value":"world"},"example2":{"foo":"bar"}}

```

### Plaintext Queries

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

     