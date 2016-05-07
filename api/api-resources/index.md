##资源类型

<br>

[account]({{site.baseurl}}/account/)|
---|
Rancher上所有资源要么被一个账户拥有，要么由该账户创建。 |

<br><br>
<br>

[apiKey]({{site.baseurl}}/rancher/api/api-resources/apiKey/)|
---|
如果启动了访问控制，API密钥将会提供访问Rancher API的能力。由每个环境创建访问密钥以及秘密密钥对能够直接用来调用API或者和[rancher-compose]({{site.baseurl}}/rancher/rancher-compose)一起使用。 |

<br><br>
<br>

[certificate]({{site.baseurl}}/rancher/api/api-resources/certificate/)|
---|
证书用于在SSL终端添加到负载均衡器。 |

<br><br>
<br>


[container]({{site.baseurl}}/rancher/api/api-resources/container/)|
---|
container用来表示主机上的一个Docker容器。|

<br><br>
<br>

[dnsService]({{site.baseurl}}/rancher/api/api-resources/dnsService/)|
---|
API中的"dnsService"在UI和Rancher文档中被称为服务别名。我们在API文档中使用UI术语来称呼。一个服务别名有允许为你可被发现的服务添加DNS记录的能力。|

<br><br>
<br>

[environment]({{site.baseurl}}/rancher/api/api-resources/environment/)|
---|
API中的"environment"在UI和Rancher文档中被称为栈。我们在API文档中使用UI术语来称呼。Rancher栈和docker-compose工程反映了同样的概念。Rancher栈代表一组构建典型应用和负载程序的服务。|

<br><br>
<br>

[externalService]({{site.baseurl}}/rancher/api/api-resources/externalService/)|
---|
externalService允许添加任意IP或者主机作为服务的能力能够以服务的方式被发现。|

<br><br>
<br>

[host]({{site.baseurl}}/rancher/api/api-resources/host/)|
---|
host在Rancher中是资源的最小单元，满足以下最低要求的任何虚拟或物理Linux服务器都能表示自己是一台Rancher host。<br> <br> *任意支持Docker 1.10.3的Linux发型版本。<br> *必须能够经由http或者https的方式通过预先配置的端口(默认为8080)访问Rancher Server。<br> *必须对属于同一个环境中的主机可路由，来为Docker容器使用Rancher的跨主机网络功能。

<br><br>
<br>


[identity]({{site.baseurl}}/rancher/api/api-resources/identity/)|
---|
当Rancher启动了[访问控制]({{site.baseurl}}/rancher/configuration/access-control/)，identity是一个对象(例如 `ldap_group`,`github_user`)在Rancher的表示。identity中的`externalId`是在认证系统中用来表示对象的唯一识别符。除非identitty的角色被作为[projectMember]({{site.baseurl}}/rancher/api/api-resources/projectMember)返回，否则它总为null。|

<br><br>
<br>


[loadBalancerService]({{site.baseurl}}/rancher/api/api-resources/loadBalancerService/)|
---|
Rancher使用HAProxy实现了一个可管理的负载均衡器，能够手动地将规模扩大到其他主机上。通过将负载均衡器添加或者"连接"到基本服务，服务均衡器就能用来为单独的容器分化网络和应用流量。被"连接"的基本服务中所有基本的容器将自动被Rancher注册为服务均衡器目标。
|

<br><br>
<br>

[machine]({{site.baseurl}}/rancher/api/api-resources/machine/)|
---|
无论何时Rancher使用`docker-machine`在Rancher中创建主机，就会创建机器条目。在UI中添加任意类型的主机|

<br><br>
<br>

[mount]({{site.baseurl}}/rancher/api/api-resources/mount/)|
---|
mount是指在数据卷和容器中目录位置的关系。|

<br><br>
<br>

[project]({{site.baseurl}}/rancher/api/api-resources/project/)|
---|
A "project" in the API is referred to as an environment in the UI and Rancher documentation. In the API documentation, we'll use the UI terminology.  All hosts and any Rancher resources (i.e. containers, load balancers, etc.) are created and belong to an [environment]({{site.baseurl}}/rancher/configuration/environments/).  Access control to who can view and manage these resources are then defined by the owner of the environment.  Rancher currently supports the capability for each user to manage and invite other users to their environment and allows for the ability to create multiple environments for different workloads.  For example, you may want to create a "dev" environment and a separate "production" environment with its own set of resources and limited user access for your application deployment. |

<br><br>
<br>

[projectMember]({{site.baseurl}}/rancher/api/api-resources/projectMember/)|
---|
A "project member" in the API is referred to as an environment members in the UI and Rancher documentation. An environment member is a list of all of the members of the  environment. An environment member is an [identity]({{site.baseurl}}/rancher/api/api-resources/identity). UI和Rancher文档把API中的"projectMember"称为环境成员。环境成员是环境中所有成员的列表。环境成员是一个[identity]({{site.baseurl}}/rancher/api/api-resources/identity)。|

<br><br>
<br>

[registry]({{site.baseurl}}/rancher/api/api-resources/registry/)|
---|
registry是镜像仓库存储的地点。registry可以是DockerHub，Quay.io，或者是定制的私有registry。|

<br><br>
<br>

[registryCredential]({{site.baseurl}}/rancher/api/api-resources/registryCredential/)|
---|
registryCredential用来授权[registry]({{site.baseurl}}/rancher/api/api-resources/registry)。|

<br><br>
<br>

[schema]({{site.baseurl}}/rancher/api/api-resources/schema/)|
---|
这是结构定义资源|

<br><br>
<br>

[service]({{site.baseurl}}/rancher/api/api-resources/service/)|
---|
Rancher adopts the standard Docker Compose terminology for services and defines a basic service as one or more containers created from the same Docker image.  Once a service (consumer) is linked to another service (producer) within the same stack, a DNS record mapped to each container instance is automatically created and discoverable by containers from the "consuming" service. Other benefits of creating a service under Rancher include":" <br><br> * Service HA - the ability to have Rancher automatically monitor container states and maintain a service's desired scale. <br> * Health Monitoring - the ability to set basic monitoring thresholds for container health. Rancher对服务采用了标准Docker Compose术语，同时定义了一个基本服务为同一个Docker镜像创建的一个或多个容器。在同样的栈中一旦一个服务(消费者)被链接到另一个服务(生产者)，会自动创建一条映射到每一个容器实例的DNS记录，这条记录能够"正在消费"的服务发现。在Rancher中创建服务的其他好处包括":" <br><br> * 服务高可用 - 拥有Rancher自动监控容器状态和维护服务想要的规模的能力。<br> * 健康监控 - 拥有为容器健康设置基本监控阈值的能力。|

<br><br>
<br>

[storagePool]({{site.baseurl}}/rancher/api/api-resources/storagePool/)|
---|
storagePool是参与到共享存储的主机名单。|

<br><br>
<br>


[volume]({{site.baseurl}}/rancher/api/api-resources/volume/)|
---|
A volume can be associated to containers or storage pools. <br><br> * A container can have many volumes and containers are mapped to volumes the [mount]({{site.baseurl}}/rancher/api/api-resources/mount/) link on a container. <br> * A storage pool owns many volues. The volume is only available to containers deployed on hostst that are part of the storage pool. When a volume is being created, you do not directly associate it to a storage pool. You will only need to specify a driver and during allocation, Rancher will resolve it to a storage pool. 存储卷能够关联到容器或者存储池。<br><br> *一个容器能够有多个数据卷？？？<br> *存储池拥有多个存储卷。数据卷只对那些部署到作为存储池一部分的主机上的容器是有效的。当创建一个数据卷，你不能直接将它关联到存储池。你只需要在指定一个驱动，在分配过程中，Rancher将其解析到|

<br><br>

