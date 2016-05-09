##资源类型

<br>

[账号]({{site.baseurl}}/account/)|
---|
Rancher上所有资源要么被一个账户(account)拥有，要么由该账户(account)创建。 |

<br><br>
<br>

[api密钥]({{site.baseurl}}/rancher/api/api-resources/apiKey/)|
---|
如果启动了访问控制(access control)，API密钥(apiKey)将会提供访问Rancher API的能力。由每个环境(environment)创建访问密钥(accesskey)以及秘密密钥(secretkey)对能够直接用来调用API或者和[rancher-compose]({{site.baseurl}}/rancher/rancher-compose)一起使用。 |

<br><br>
<br>

[证书]({{site.baseurl}}/rancher/api/api-resources/certificate/)|
---|
证书(certificate)用于在SSL终端添加到负载均衡器(loadbalancer)。 |

<br><br>
<br>


[容器]({{site.baseurl}}/rancher/api/api-resources/container/)|
---|
容器(container)用来表示主机上的一个Docker容器。|

<br><br>
<br>

[dns服务]({{site.baseurl}}/rancher/api/api-resources/dnsService/)|
---|
API中的"dns服务"(dnsService)在用户界面和Rancher文档中被称为服务别名。我们在API文档中使用用户界面术语来称呼。一个服务别名有允许为你可被发现的服务添加DNS记录的能力。|

<br><br>
<br>

[环境]({{site.baseurl}}/rancher/api/api-resources/environment/)|
---|
API中的"环境"(environment)在用户界面和Rancher文档中被称为栈(stack)。我们在API文档中使用用户界面术语来称呼。Rancher栈(stack)和docker-compose工程反映了同样的概念。Rancher栈(stack)代表一组构建典型应用和负载程序的服务。|

<br><br>
<br>

[外部服务]({{site.baseurl}}/rancher/api/api-resources/externalService/)|
---|
外部服务(externalService)允许添加任意IP或者主机作为服务的能力能够以服务的方式被发现。|

<br><br>
<br>

[主机]({{site.baseurl}}/rancher/api/api-resources/host/)|
---|
主机(host)在Rancher中是资源的最小单元，满足以下最低要求的任何虚拟或物理Linux服务器都能表示自己是一台Rancher主机。<br> <br> *任意支持Docker 1.10.3的Linux发型版本。<br> *必须能够经由http或者https的方式通过预先配置的端口(默认为8080)访问Rancher Server。<br> *必须对属于同一个环境中的主机可路由，来为Docker容器使用Rancher的跨主机网络功能。

<br><br>
<br>


[身份证明]({{site.baseurl}}/rancher/api/api-resources/identity/)|
---|
当Rancher启动了[访问控制]({{site.baseurl}}/rancher/configuration/access-control/)，身份证明(identity)是一个对象(例如 `ldap_group`,`github_user`)在Rancher的表示。身份证明(identity)中的`externalId`是在认证系统中用来表示对象的唯一识别符。除非身份证明(identity)的角色被作为[项目成员(projectMember)]({{site.baseurl}}/rancher/api/api-resources/projectMember)返回，否则它总为null。|

<br><br>
<br>


[负载均衡服务]({{site.baseurl}}/rancher/api/api-resources/loadBalancerService/)|
---|
Rancher使用HAProxy实现了一个可管理的负载均衡器(loadbalancer)，能够手动地将规模扩大到其他主机上。通过将负载均衡器添加或者"连接"到基本服务，服务均衡器就能用来为单独的容器分化网络和应用流量。被"连接"的基本服务中所有基本的容器将自动被Rancher注册为服务均衡器(loadbalancer)目标。
|

<br><br>
<br>

[机器]({{site.baseurl}}/rancher/api/api-resources/machine/)|
---|
无论何时Rancher使用`docker-machine`在Rancher中创建主机，就会创建机器(machine)条目。在用户界面上添加任意类型的主机|

<br><br>
<br>

[挂载]({{site.baseurl}}/rancher/api/api-resources/mount/)|
---|
挂载(mount)是指在数据卷和容器中目录位置的关系。|

<br><br>
<br>

[项目]({{site.baseurl}}/rancher/api/api-resources/project/)|
---|
用户界面和Rancher文档把API中的"项目(project)"称为环境(environment)。在API文档中,我们使用用户界面上的术语来描述。所有主机和任何Rancher资源(比如，容器,负载均衡器等)在被创建后都属于一个[环境(environment)]({{site.baseurl}}/rancher/configuration/environments/).环境(environment)的拥有者决定谁能查看和管理这些资源。Rancher当前支持每个用户能管理和邀请其他用户到他所属的环境，也允许为不同的工作量创建多个环境。例如，你可能想创建一个"开发"和一个分离的"产品"环境，这些环境各自带有自己的一系列资源，以及应用程序部署限制的用户访问。|

<br><br>
<br>

[项目成员]({{site.baseurl}}/rancher/api/api-resources/projectMember/)|
---|
用户界面和Rancher文档把API中的"项目成员(projectMember)"称为环境成员(environmentMember)。环境成员(environmentMember)是环境中所有成员的列表。环境成员(environmentMember)是一个[身份证明(identity)]({{site.baseurl}}/rancher/api/api-resources/identity)。|

<br><br>
<br>

[镜像仓库]({{site.baseurl}}/rancher/api/api-resources/registry/)|
---|
镜像仓库(registry)是镜像存储处的地点。镜像仓库(registry)可以是DockerHub，Quay.io，或者是定制的私有镜像仓库。|

<br><br>
<br>

[镜像仓库证书]({{site.baseurl}}/rancher/api/api-resources/registryCredential/)|
---|
镜像仓库证书(registryCredential)用来授权[镜像仓库(registry)]({{site.baseurl}}/rancher/api/api-resources/registry)。|

<br><br>
<br>

[结构定义]({{site.baseurl}}/rancher/api/api-resources/schema/)|
---|
这是结构定义(schema)资源|

<br><br>
<br>

[服务]({{site.baseurl}}/rancher/api/api-resources/service/)|
---|
Rancher对服务(service)采用了标准Docker Compose术语，同时定义一个基本服务为同一个Docker镜像创建的一个或多个容器。在同样的栈中一旦一个服务(消费者)被链接到另一个服务(生产者)，会自动创建一条映射到每一个容器实例的DNS记录，这条记录能够"正在消费"的服务发现。在Rancher中创建服务的其他好处包括":" <br><br> * 服务高可用 - 拥有Rancher自动监控容器状态和维护服务想要的规模的能力。<br> * 健康监控 - 拥有为容器健康设置基本监控阈值的能力。|

<br><br>
<br>

[存储池]({{site.baseurl}}/rancher/api/api-resources/storagePool/)|
---|
存储池(storagePool)是参与到共享存储的主机名单。|

<br><br>
<br>


[volume]({{site.baseurl}}/rancher/api/api-resources/volume/)|
---|
数据卷能够关联到容器或者存储池。<br><br> *一个容器能够有多个数据卷，容器能被映射到数据卷所在的容器上的[挂载]({{site.baseurl}}/rancher/api/api-resources/mount/)连接。<br> *存储池拥有多个存储卷。数据卷只对那些部署到作为存储池一部分的主机上的容器是有效的。当创建一个数据卷，你不能直接将它关联到存储池。你只需要在指定一个驱动，在分配过程中，Rancher将其解析给一个存储池。|

<br><br>

