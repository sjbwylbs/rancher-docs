## rancher中的标签和调度
---

当通过[服务(service)]({{site.baseurl}}/rancher/rancher-ui/applications/stacks/adding-servies/)，或者通过[负载均衡器(loadbalancer)]({{site.baseurl}}/rancher/rancher-ui/applications/stacks/adding-balancers/)，或者通过[容器页面(container page)]({{site.baseurl}}/rancher/rancher-ui/infrastructure/containers/)启动容器的时候，我们提供为容器创建标签的选项以及提供调度容器到你想要放置容器的主机上的能力。在这章节剩下的部分，我们将使用术语容器/服务(container/service),但这些标签也适用于负载均衡器(也就是一种特殊类型的服务)。

调度规则为你想要Rancher如何挑选使用的主机提供了灵活性。在Rancher中，我们使用标签来帮助定义调度规则。你能为一个容器创建你想要的数量的标签。有了多种规则，你完全控制哪种主机上创建容器。你可以请求具有特定主机标签，容器标签或名字，或者特定服务的容器在一台主机启动。这些调度规则能帮助创建你容器托管关系的黑名单和白名单。

### 标签

对于[服务]({{site.baseurl}}/rancher/rancher-ui/applications/stacks/adding-services/)/[容器]({{site.baseurl}}/rancher/rancher-ui/infrastructure/containers/), 标签可以在**Advanced Options** -> **Labels**中找到。对于负载均衡器，标签可以在**Labels**选项卡中找到。


通过添加标签给[负载均衡器]({{site.baseurl}}/rancher/rancher-ui/applications/stacks/adding-balancers/)，每个负载均衡器容器都会收到该标签，这标签是一个键值对。Rancher中我们使用这些容器标签来帮助定义调度规则。你能在负载均衡器上创建你想要的数量的标签。默认地Rancher已经为每个容器添加了系统相关标签。

### 调度选项

对于[服务]({{site.baseurl}}/rancher/rancher-ui/application/stacks/adding-services/)/[容器]({{site.baseurl}}/rancher/rancher-ui/infrastructure/containers/),标签能在**Advanced Options** -> **Scheduling**中找到。对于负载均衡器，标签能在**Scheduling**选项卡中找到。

对于[服务]({{site.baseurl}}/rancher/rancher-ui/applications/stacks/adding-services/)/[容器]({{site.baseurl}}/rancher/rancher-ui/infrastructure/containers/), 提供了两个选项来决定在哪里启动你的容器。

**选项 1: 在指定主机上运行_所有_容器**
通过选择该选项运行所有容器在指定的主机，容器/服务将会在指定的主机上启动。如果你的主机停止运行，那么主机上的容器也将停止运行。如果你在容器页面创建一个容器，即使存在端口冲突，容器也能被启动。如果你创建一个规模(scale)大于1的服务，存在端口冲突，你的服务可能陷入未激活状态(inActivating)直到你编辑服务的规模值。

**选项 2: 自动选择匹配调度规则的主机**
通过选择该选项自动选择匹配调度规则的主机，你可以灵活地选择你的调度规则。任何遵守所有规则的主机都是容器能在上面启动的主机。你能通过点击**+**按钮来添加规则。

对于负载均衡器({{site.baseurl}})/rancher/rancher-ui/applications/stacks/adding-balancers/), 因为端口冲突问题只有选项2是有效的。你只能选择添加调度规则。点击**Scheduling**选项卡。你能通过**Add Scheudling Rule**按钮添加你想要的调度规则。

对于每个规则，你要选择规则的**条件**。有四个不同的条件，分别定义规则必须遵守的规则有多严格。**字段(field)**决定由你希望规则应用于哪些字段。**键**和**值**是那些你想要检查的字段的值。如果你启动一个服务或者负载均衡器，rancher将根据每台主机上的负载来传播容器的分布到合适的主机上。决定哪些主机合适取决于条件的选择。

> **注意：**对于[服务]({{site.baseurl}}/rancher/rancher-ui/applications/stacks/adding-services/)/[负载均衡器]({{site.baseurl}}/rancher/rancher-ui/applications/stacks/adding-balancers/)，如果你选择了规模为**总是在每台主机上运行这个容器的一个实例**选项，那么只有主机标签作为一个可能的字段出现。

_条件_

* **必须**或**必须不**：Rancher只使用匹配或者不匹配字段和值的主机。如果Rancher找不到一台主机符合带有这些条件的所有规则，你的服务将一直陷入正在激活状态(Activating)。服务将不停为容器查找可用的主机。你能编辑服务的规模活着添加/编辑另一台满足这些规则的主机。
* **应该**或**不应该**。Rancher试图使用匹配这些字段和值的主机。在没有主机匹配所有规则的条件下，Rancher将一个接一个移除软限制直到有主机满足剩下的限制。

_字段_

* **主机标签**：当为容器/服务选择主机时，Rancher将会检测主机上的标签来查看是否这些标签匹配所提供的键/值对。因为每台主机能有一个或多个标签，Rancher将把键/值对和主机上所有标签做对比。当添加一台主机到Ranche时，你就能为主机添加标签。你也能通过主机下拉菜单中的**Edit**选项来编辑主机上的标签。激活主机上的标签列表可从下拉菜单的键值字段获得。
* **带标签的容器**：当选择这个字段时，Rancher将查找主机上存在已经有标签匹配键值对的容器的主机。因为每个容器能有一个或多个标签，Rancher将把键值对和主机上的每个容器的所有标签进行比较。容器标签在容器的**Advanced Options** -> **Lables**选项卡中。你不能在容器启动后编辑容器的标签。为了能创建拥有同样设定的新容器，你能在容器启动前**Clone**容器/服务以及增加新的标签。正在运行的容器的用户标签列表看从下拉菜单的键值字段获得
**带名字的服务**：Rancher将检测一台主机上是否有指定服务的容器在运行。如果在稍后的时间，服务改变了名字或者取消处于未激活或者已移除状态，这规则就不再有效。如果你选择了这个字段，输入的值必须为`栈名/服务名(stack_name/service_name)`的格式。正在运行的服务的列表可以从下拉菜单中的值(value)字段获得。
* **带名字的容器**：Rancher将检测一台主机上是否指定的名字的容器正咋运行。如果稍后的时间，容器改变了名字或者处于未激活/已移除状态，整条规则变不再有效。运行中的容器列表可以从下拉菜单的值（value)字段获得。




