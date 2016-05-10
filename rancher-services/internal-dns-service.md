## Rancher 内部 DNS 服务
---
在 Rancher 里，我们有自己的内部 DNS 服务，允许同一个 [cattle 环境]({{site.baseurl}}/configuration/environments) 里的服务能解析到另外任意一个服务。

环境里的所有服务都通过 `<service_name>` 被解析，并且不需要在服务间建立连接。对于处于不同栈的其他任何服务，你需要通过 `<service_name>.<stack_name>` 而不是 `<service_name>`。如果你想要使用不同名字来解析服务，你能设置一个连接，以便服务能够通过服务别名解析。

### 通过连接设置服务别名

在 UI 里，当 [添加服务]({{site.baseurl}}/rancher/ui/applications/stacks/adding-services/) 时, 展开 **Service Links** 部分，选择服务，提供别名。

如果你是使用 `rancher-compose` 来 [添加服务]({{site.baseurl}}/rancher-compose/)， `docker-compose.yml` 可以使用 `links` 或 `external_links` 指令。

```yaml
service1:
  image: wordpress
  # If the other service is in the same stack
  links:
    # <service_name>:<service_alias>
    - service2:mysql
  # If the other service is in a different stack
  external_links:
    # <service_name>:<service_alias>
    - service3:mysql
```

### 从容器和链接

当启动一个服务，你可能需要服务一起一直启动在同一主机上。特殊用例是当试图使用另一个服务的 `volumes_from` 或 `net`。当无论是通过[UI]({{site.baseurl}}/rancher-ui/applications/stacks/adding-services/#sidekick-services) 还是使用 [rancher-compose]({{site.baseurl}}/rancher-compose/#sidekicks)创建一个伙伴关系，服务都自动通过他们的名字互相解析到。我们当前不支持在伙伴服务内通过链接/外部链接创建服务别名。

当创建伙伴关系，总是有一个主服务和若干伙伴服务。合在一起，他们被认为是一个单独的运行配置。运行配置会作为一组容器被部署在一台主机上，主服务一个，另外每个伙伴服务一个。在运行配置内的任何一个服务，你都能通过他们的名称解析到主服务或者伙伴服务。对于运行配置外的任何服务，主服务能够通过名字解析到，但伙伴服务仅能通过 `<sidekick_name>.<primary_service_name>` 解析到。


### 容器名

所有的容器都能通过他们的名字被全局解析到，因为每个服务的容器名在每个环境内都是独一无二的。不需要在后面加上服务名或栈名。

## 例子
---

### Pinging 同一栈内的服务

如果你 exec 进入容器的 shell，你能通过服务名 ping 同一栈内的其他服务。

在我们的例子中，栈名为 `stackA`，带有两个服务 `foo` 和 `bar`。

在 exec 进入 `foo` 服务的一个容器中后，你能 ping `bar` 服务。

```bash
$ ping bar
PING bar.stacka.rancher.internal (10.42.x.x) 58(84) bytes of data.
64 bytes from 10.42.x.x: icmp_seq=1 ttl=62 time=1.63 ms
64 bytes from 10.42.x.x: icmp_seq=2 ttl=62 time=1.13 ms
64 bytes from 10.42.x.x: icmp_seq=3 ttl=62 time=1.07 ms
```

### Pinging 不同栈内的服务

对于不同栈内的服务，你能通过使用 `<service_name>.<stack_name>` ping 不同栈内的服务。

在我们的例子中，我们有一个栈为 `stackA`，带有服务 `foo` ，然后我们还有一个栈为 `stackB`，带有服务 `bar`。

如果我们 exec 进入服务 `foo` 的一个容器中，你能通过 `foo.stackB` ping `bar` 服务。

```bash
$ ping bar.stackb
PING bar.stackb (10.42.x.x) 56(84) bytes of data.
64 bytes from 10.42.x.x: icmp_seq=1 ttl=62 time=1.43 ms
64 bytes from 10.42.x.x: icmp_seq=2 ttl=62 time=1.15 ms
64 bytes from 10.42.x.x: icmp_seq=3 ttl=62 time=1.27 ms
```

### Pinging 伙伴服务

取决于你从哪个服务发起 ping，你能通过 `<sidekick_name>` 或 `<sidekick_name>.<primary_service_name>` 到达伙伴服务。

在我们的例子中，我们有一个栈为 `stackA`，包含服务 `foo`，带有伙伴服务 `bar`,还有个服务为 `hello`。我们还有个栈为 `stackB`，包含服务 `world`。

如果我们 exec 进入 `foo` 服务的一个容器，你能通过他的名字直接 ping `bar` 服务。

```bash
# Inside  one of the containers in the `foo` service, which `bar` is a sidekick to.
$ ping bar
PING bar.foo.stacka.rancher.internal (10.42.x.x) 56(84) bytes of data.
64 bytes from 10.42.x.x: icmp_seq=1 ttl=64 time=0.060 ms
64 bytes from 10.42.x.x: icmp_seq=2 ttl=64 time=0.111 ms
64 bytes from 10.42.x.x: icmp_seq=3 ttl=64 time=0.114 ms
```

如果我们 exec 进入 `hello` 服务的一个容器，在同一个栈内，你能通过 `foo` ping `foo` 服务，通过 `bar.foo` ping `bar` 伙伴服务。

```bash
# Inside one of the containers in the `hello` service, which is not part of the service/sidekick service
# Ping the primary service (i.e. foo)
$ ping foo
PING foo.stacka.rancher.internal (10.42.x.x) 56(84) bytes of data.
64 bytes from 10.42.x.x: icmp_seq=1 ttl=62 time=1.04 ms
64 bytes from 10.42.x.x: icmp_seq=2 ttl=62 time=1.40 ms
64 bytes from 10.42.x.x: icmp_seq=3 ttl=62 time=1.07 ms
# Ping the sidekick service (i.e. bar)
$ ping bar.foo
PING bar.foo (10.42.x.x) 56(84) bytes of data.
64 bytes from 10.42.x.x: icmp_seq=1 ttl=62 time=1.01 ms
64 bytes from 10.42.x.x: icmp_seq=2 ttl=62 time=1.12 ms
64 bytes from 10.42.x.x: icmp_seq=3 ttl=62 time=1.05 ms
```

如果我们 exec 进入 `world` 服务的一个容器，在不同栈内，你能通过 `foo.stacka` ping `foo` 服务，通过 `bar.foo.stacka` ping `bar` 伙伴服务。

```bash
# Inside one of the containers in the `world` service, which is in a different stack
# Ping the primary service (i.e. foo)
$ ping foo.stacka
PING foo.stacka (10.42.x.x) 56(84) bytes of data.
64 bytes from 10.42.x.x: icmp_seq=1 ttl=62 time=1.13 ms
64 bytes from 10.42.x.x: icmp_seq=2 ttl=62 time=1.05 ms
64 bytes from 10.42.x.x: icmp_seq=3 ttl=62 time=1.29 ms
# Ping the sidekick service (i.e. bar)
$ ping bar.foo.stacka
PING bar.foo.stacka (10.42.x.x) 56(84) bytes of data.
64 bytes from 10.42.x.x: icmp_seq=1 ttl=62 time=1.23 ms
64 bytes from 10.42.x.x: icmp_seq=2 ttl=62 time=1.00 ms
64 bytes from 10.42.x.x: icmp_seq=3 ttl=62 time=0.994 ms
```

### Pinging 容器名

从任何容器，你能通过他们的名字 ping 环境内的其他容器，而不用管他们是否在不同栈或服务内。

在我们的例子中，我们有一个栈为 `stackA`，包含服务 `foo`。我们还有个栈为 `stackB`，包含服务 `bar`。
容器的名字是 `<environment_name>_<service_name>_<number>` 。

如果我们 exec 进入 `foo` 服务的一个容器，你能 ping `bar` 服务内的容器。

```bash
$ ping stackB_bar_1
PING stackB_bar_1.rancher.internal (10.42.x.x): 56 data bytes
64 bytes from 10.42.x.x: icmp_seq=1 ttl=62 time=1.994 ms
64 bytes from 10.42.x.x: icmp_seq=2 ttl=62 time=1.090 ms
64 bytes from 10.42.x.x: icmp_seq=3 ttl=62 time=1.100 ms
```
