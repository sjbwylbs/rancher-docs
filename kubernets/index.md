# Kubernetes
# 英文原版与中文翻译版初稿 by Jordan
To deploy Kubernetes in Rancher, you’ll first need to create a new environment that has specified the cluster management to be Kubernetes.
为了在Rancher上部署Kubernetes，首先你需要创建一个指定用Kubernetes管理的集群环境。

CREATING A KUBERNETES ENVIRONMENT
创建一个KUBERNETES环境

In the dropdown of environments, click on the Manage Environments. To create a new environment, click on Add Environment, select Kubernetes as the cluster management, provide a Name, Description (Optional). If access control is turned on, you can add members and select their membership role. Anyone added to the membership list would have access to your environment.
在下拉式环境中，点击管理环境。 为了创建一个新的环境，点击“Add Environment”，选择Kubernetes做为集群管理者，起一个名字，Description (可选项)。 如果访问控制权限是开着的，你可以增加成员，选择membership角色。 在membership列表中的任何成员都可以访问你的环境。

After a Kubernetes environment has been created, you can navigate to the environment by either selecting the name of the environment in the environment’s dropdown in the upper right hand corner or by selecting Switch to this Environment in the specific environment’s drop down.
在Kubernetes环境创建后，你既可以通过选择环境名字又可以通过切换环境在下拉菜单中，浏览你的环境。

NOTE:
As Rancher adds support for multiple cluster management frameworks, Rancher currently does not support the ability to switch between environments that already have services running in it.
注意：
Rancher支持多种集群管理框架，目前不支持已经运行服务的环境之间切换。

STARTING KUBERNETES
启动KUBERNETES
After a Kubernetes environment has been created, you can start the Kubernetes cluster by adding at least one host to your environment. The process of adding hosts is the same steps for all cluster management types. Once the first host has been added, Rancher will automatically start the deployment of the required Kubernetes components (i.e. master, kubelet, etcd, proxy, etc.). You can see the progress of the deployment by accessing the Kubernetes tab.
在Kubernetes环境创建后，你可以启动Kubernetes集群通过至少增加一个主机到你的环境。在所以集群管理类型中添加主机的流程是一样的步骤。 一旦第一个主机被添加后，Rancher将自动开始部署Kubernetes组件（如，master, kubelet, etcd, proxy等）。你可以通过Kubernetes选项卡看到部署过程。

USING KUBERNETES
使用KUBERNETES
Once the setup has completed, you can begin to create or manage your own Kubernetes applications via the following ways:
一旦配置完成，你就可以通过下面方法开始创建或管理你自己的Kubernetes应用。

RANCHER UI
RANCHER 用户界面
Rancher provides full CRUD capability of creating services, replication controllers (RCs), and pods. In the Kubernetes tab, click on the one of these items and click Add. A kubernetes template will be shown in the UI and is editable. After you have made changes to the configuration file, click on Create.
Rancher对创建services, replication controllers (RCs)和pods提供了增删改查的能力。 在Kubernetes tab中点击这些项目和点击添加，一个可编辑的kubernetes模板将会被展现出来，然后你在配置文件修做一些改，点击创建。

RANCHER CATALOG
RANCHER 服务目录
Rancher supports the capability of hosting a catalog of Kubernetes templates. To use a template, click on the Catalog tab. Select the template that you want to launch and click View Details. Review and edit the stack name, stack description, and configuration options and click on Launch.
Rancher支持Kubernetes模板的服务目录的能力。要使用模板，请单击“服务目录”选项卡。选择你要启动的模板，然后单击“View Details”。审查和编辑堆栈名称、堆栈描述和配置选项，然后单击“启动”

If you want to add your own templates to Kubernetes, you add them to the Rancher catalog and place your templates in a kubernetes-templates folder.
如果你想添加自己的模板到Kubernetes，你将它们添加到Rancher的服务目录，把你的模板放在Kubernetes模板文件夹里。

KUBECTL
KUBECTL
To configure your own kubectl to talk to your newly created Kubernetes cluster, go to Kubernetes -> kubectl -> Generate Config to generate the necessary kube/config_file that you can download and add to your local directory.
为了配置你自己的kubectl与新创建的Kubernetes集群关联，通过Kubernetes -> kubectl -> Generate Config 生成所需的kube/config_file，你可以下载并添加到你的本地目录。 

KUBECTL VIA SHELL
KUBECTL VIA SHELL
Rancher provides a convenient shell access to a managed kubectl instance that can be used to manage Kubernetes clusters and applications.
Rancher提供了一个方便的Shell进入到托管kubectl实例来管理Kubernetes集群和应用。

KUBERNETES NAMESPACES
KUBERNETES命名空间
Rancher supports the ability to manage different Kubernetes namespaces. In the upper right hand corner, you will be able to see which Namespace that you are working in. After the first host is added, Rancher creates the default namespace.
Rancher提供了支持管理不同的Kubernetes命名空间的能力。在右上角，你可以看到你在工作中的那个名字空间，在添加第一主机后，Rancher创建默认的命名空间。

ADDING NAMESPACES
添加NAMESPACES
To add an additional namespace into Kubernetes, click on the current namespace and a dropdown of available namespaces and Manage Namespaces will appear. Click on Manage Namespaces.
为了添加一个额外的命名空间到Kubernetes，点击当前命名空间和下拉可用的命名空间，“Manage Namespaces将会出现。单击”Manage Namespaces“。 

In the Namespaces page, click on Add Namespace. Update the configuration file and click Create.
在命名空间中的页面中点击“Add Namespace”，更新配置文件，然后单击“创建”。 

EDITING NAMESPACES
编辑NAMESPACES
For existing namespaces, in the Namespaces page, click on Edit in the namespace’s dropdown to update it. Click on Save.
对于现有的命名空间，在命名空间的页面中点击编辑命名空间的下拉菜单，然后更新，点击保存。 

SWITCHING NAMESPACES
切换NAMESPACES
In the dropdown of namespaces, you can select the namespace that you want to launch services in to switch between the namespaces.
在命名空间的下拉菜单，你可以选择你想启动服务的命名空间，在名称空间之间进行切换。
