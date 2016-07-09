# 公有云配置参考

---
title: Cloud-Config
layout: os-default

---

## 使用Cloud-config配置RancherOS
## Configure RancherOS with Cloud-Config
---

Cloud-config是许多Linux发行版都支持的声明性配置文件格式，是RancherOS的主要配置机制。

Cloud-config is a declarative configuration file format supported by many Linux distributions and is the primary configuration mechanism for RancherOS.


Linux操作系统`cloud-config`支持将启动时调用`cloud-init`进程来解析`cloud-config`文件和配置操作系统。 RancherOS运行在系统容器自身的`cloud-init`进程。`cloud-init`进程将试图检索各种数据源的云配置文件。一旦`cloud-init`获得一个`cloud-config`文件，它根据`cloud-config`文件的内容配置Linux操作系统。

A Linux OS supporting cloud-config will invoke a cloud-init process during startup to parse the cloud-config file and configure the operating system. RancherOS runs its own cloud-init process in a system container. The cloud-init process will attempt to retrieve a cloud-config file from a variety of data sources. Once cloud-init obtains a cloud-config file, it configures the Linux OS according to the content of the cloud-config file.

当你在AWS创建一个RancherOS实例，例如，您可以选择性地提供云配置中的`user-data`配置项。在RancherOS实例中，`cloud-init`进程将会通过其AWS云配置数据源，它只是提取由虚拟机实例接收到的`user-data`的`cloud-config`内容。如果文件用“`#cloud-config`”开始，cloud-init将解释该文件作为一个`cloud-config`文件。如果该文件以'#!<解释器>`（如`#!/bin/sh`）开头，`cloud-init`将直接执行该文件。您可以作为脚本文件一样写入任何配置命令。

When you create a RancherOS instance on AWS, for example, you can optionally provide cloud-config passed in the `user-data` field. Inside the RancherOS instance, cloud-init process will retrieve the cloud-config content through its AWS cloud-config data source, which simply extracts the content of user-data received by the VM instance. If the file starts with "`#cloud-config`", cloud-init will interpret that file as a cloud-config file. If the file starts with `#!<interpreter>` (e.g., `#!/bin/sh`), cloud-init will simply execute that file. You can place any configuration commands in the file as scripts.

`cloud-config`文件使用YAML格式。 YAML是很容易理解和容易解析。有关YAML的更多信息，请阅读[YAML网站（http://www.yaml.org/start.html）。最重要的格式化原理是缩进或空格。缩进指明项之间彼此的关系。如果某项的缩进比前一行多，则它是上一项的子项。

A cloud-config file uses the YAML format. YAML is easy to understand and easy to parse. For more information on YAML, please read more at the [YAML site](http://www.yaml.org/start.html). The most important formatting principle is indentation or whitespace. This indentation indicates relationships of the items to one another. If something is indented more than the previous line, it is a sub-item of the top item that is less indented.

例如：注意为何两者都是在`SSH-authorized_keys`下方缩进。

Example: Notice how both are indented underneath `ssh-authorized-keys`.

```yaml
#cloud-config
ssh_authorized_keys:
  - ssh-rsa AAA...ZZZ example1@rancher
  - ssh-rsa BBB...ZZZ example2@rancher
```

在上面的例子中，我们发现有`#cloud-config`这一行，表明它是一个`cloud-config`文件。我们有1个的顶级属性`ssh_authorized_keys`。它在`ssh_authorized_keys:`项下面列出了以虚线开头的公钥列表。
In our example above, we have our `#cloud-config` line to indicate it's a cloud-config file. We have 1 top-level property, `ssh_authorized_keys`. Its value is a list of public keys that are represented as a dashed list under `ssh_authorized_keys:`.

##  RancherOS 如何应用 Cloud-Config
## How RancherOS Applies Cloud-Config

RancherOS自带的默认配置也是使用cloud-config格式.cloud-config 文件将会被cloud-init继承并覆盖默认的配置，然后以`boot.yml`的文件名保存在`/var/lib/rancher/conf/cloud-config.d`目录.
最终.`cloud-config.yml`文件又继承并覆盖`boot.yml`文件的配置。您不应该直接编辑`cloud-config.yml`文件。应该使用`ros config`命令来修改`cloud-config.yml`文件。

RancherOS comes with a default configuration, which is also in cloud-config format. The cloud-config file processed by cloud-init will extend and override the default configuration and be saved as `boot.yml` in `/var/lib/rancher/conf/cloud-config.d`. Finally, the `cloud-config.yml` file will extend and override the `boot.yml` file. You should not edit `cloud-config.yml` file directly. The `ros config` command allows you to change the content of the `cloud-config.yml` file.

通常情况下，当你第一次启动服务器，您在`cloud-config`文件的修改会传递到配置服务器的初始化。第一次开机后，如果您需要对配置进行任何更改，建议您使用`ROS config`设置必要的配置属性。任何更改都将被保存在'cloud-config.yml`文件，并需要重新启动才能生效。

Typically, when you first boot the server, you pass in the cloud-config file to configure the initialization of the server. After the first boot, if you have any changes for the configuration, it's recommended that you use `ros config` to set the necessary configuration properties. Any changes will be saved in the `cloud-config.yml` file and need to be re-booted in order to take effect.

## RancherOS 启动后更新Cloud-Config
## Updating Cloud-Config After RancherOS has booted

`ros config` is the main command to interact with RancherOS configuration, here's the link to the [full ros config command docs]({{site.baseurl}}/os/rancheros-tools/ros/config/). With these commands, you can get and set values in the `cloud-config.yml` file as well as import/export configurations.

You can view the content of `cloud-config.yml` file by issuing the `ros config export` command. Another command `ros config export --full` prints the current effective configuration of RancherOS, taking into account the initial default configuration and the impact of `cloud-config.yml`.


## 已经支持的 Cloud-Config 指令
## Supported Cloud-Config Directives

RancherOS暂时支持一小部分的`cloud-config`指令
RancherOS currently supports a small number of cloud-config directives.

> **注意:** RancherOS现在不支持为RancherOS添加其他用户
> **Note:** Currently, RancherOS doesn't support adding other users to RancherOS.

### SSH 密钥
### SSH Keys

您可以添加 SSH 密钥给`rancher`这个默认用户
You can add SSH keys to the default `rancher` user.

```yaml
#cloud-config
# 添加 SSH keys 给rancher用户
# Adds SSH keys to the rancher user
ssh_authorized_keys:
  - ssh-rsa AAA... darren@rancher
```

### Write Files to Disk

You can write files to the disk using the `write_files` directive.

```yaml
#cloud-config
write_files:
  - path: /etc/rc.local
    permissions: "0755"
    owner: root
    content: |
      #!/bin/bash
      echo "I'm doing things on start"
```

#### Logging into DockerHub or Private Repositories

By using the `write_files` directive, you can add your login credentials so that when RancherOS boots, you will already by logged in.

```
#cloud-config
write_files:
  - path: /home/rancher/.docker/config.json
    permissions: "0755"
    owner: rancher
    content: |
      {
        "auths": {
          "https://index.docker.io/v1/": {
            "auth": "asdf=",
            "email": "not@val.id"
          }  
        }    
      }
```

> **Note:** Currently, you will not be able to log into a repository and launch the service in the same cloud-config file, which means you will not be able to launch a container using cloud-config from a private repository.

### Set Hostname

You can set the hostname of the host using cloud-config. The example below shows how to configure it.

```yaml
hostname: myhost
```

### Launching Containers in RancherOS

To start containers in RancherOS when booting it up, you can just add the services to the cloud-config file.

```
#cloud-config
rancher:
  services:
    nginxapp:
      image: nginx
      restart: always
```  

#### System-Docker vs. User Docker

RancherOS uses labels to determine if the container should be deployed in system-docker. By default without the label, the container will be deployed in user docker.

```yaml
labels:
- io.rancher.os.scope=system
```

#### Labels

We use labels to determine how to handle the service containers.

Key | Value |Description
----|-----|---
`io.rancher.os.detach` | Default: `true` | Equivalent of `docker run -d`. If set to `false`, equivalent of `docker run --detach=false`
`io.rancher.os.scope` | `system` | Use this label to have the container deployed in system-docker instead of docker.
`io.rancher.os.before`/`io.rancher.os.after` | Service Names (Comma separated list is accepted) | Used to determine order of when containers should be started
`io.rancher.os.createonly` | Default: `false` | When set to `true`, only a `docker create` will be performed and not a `docker start`.
`io.rancher.os.reloadconfig` | Default: `false`| When set to `true`, it reloads the configuration.


Read more about [system-services]({{site.baseurl}}/os/configuration/system-services) in RancherOS.

### RancherOS Specific Configuration

To configure other parts of RancherOS, the cloud-config information must be within the `rancher` key in the cloud-config file.

#### Network Configuration

Network configuration section must start with the `rancher` key. The example below shows network interface and DNS configuration. For each interface, you can configure DHCP, IP Address, Gateway, and MTU. There are three ways to specify network interfaces:

1. Wild card (e.g., `eth*`). This is the least specific and the lowest priority. An `eth1` specification will take precedence over `eth*` specification.
2. Exact interface name (e.g., `eth0`). Exact interface names take precedence over wild card specifications.
3. MAC address (e.g., `"mac=ea:34:71:66:90:12:01"`). Mac addresses take precedence over exact interface names.

```yaml
rancher:
  network:
    interfaces:
      eth*:
        dhcp: false
      eth0:
        address: 192.168.100.100/24
        gateway: 192.168.100.1
        mtu: 1500
      # If this MAC address happens to match eth0, eth0 will be programmed to use DHCP.
      "mac=ea:34:71:66:90:12:01":
        dhcp: true
    dns:
      nameservers:
        - 8.8.8.8
        - 8.8.4.4
```

**DNS**

In the DNS section, you can set the `nameservers`, and `search`, which directly map to the fields of the same name in `/etc/resolv.conf`.

**Interfaces**

In the `interfaces` section, the keys are used to match the desired interface to configure.  Wildcard globbing is supported so `eth*` will match `eth1` and `eth2`.  Specific MAC address can be used to pick the NIC interface using `"mac=XXX"` as a key. The available options you can set are `address`, `gateway`, `mtu`, and `dhcp`.

#### Cloud Init Data Sources

You can configure which data sources to use for cloud-init.  Multiple data sources can be set, but the data source that is available the fastest will be used.  This value is usually pre-populated with the current setting for your environment.  Valid value are:

1. `configdrive:PATH` - Look for an OpenStack compatible config drive mounted at `PATH`
1. `file:PATH` - Read the `FILE` as the user data.
1. `ec2` - Look for EC2 style meta data at 169.254.169.254
1. `ec2:IP_ADDRESS` - Look for EC2 style meta data at the `IP_ADDRESS`
1. `url:URL` - Download `URL` and use that as the user data
1. `cmdline:URL` - Look for `cloud-config-url=URL` in `/proc/cmdline` and download `URL` as user data

Within the cloud-init key, you can define the data sources.

```yaml
rancher:
  cloud_init:
    datasources:
      - configdrive:/media/config-2
```

#### Persistent State Partition

RancherOS will store its state in a single partition specified by the `dev` field.  The field can be a device such as `/dev/sda1` or a logical name such `LABEL=state` or `UUID=123124`.  The default value is `LABEL=RANCHER_STATE`.  The file system type of that partition can be set to `auto` or a specific file system type such as `ext4`.

```yaml
rancher:
  state:
    fstype: auto
    dev: LABEL=RANCHER_STATE
    autoformat:
    - /dev/sda
    - /dev/vda
```

**Auto formatting**

You can specify a list of devices to check to format on boot.  If the state partition is already found, RancherOS will not try to auto format a partition. By default, auto-formatting is off.

RancherOS will autoformat the partition to ext4 if the device specified in `autoformat`:

* Contains a boot2docker magic string
* Starts with 1 megabyte of zeros and `rancher.state.formatzero` is true

#### Upgrades

In the `upgrade` key, the `url` is used to find the list of available and current versions of RancherOS.

```yaml
rancher:
  upgrade:
    url: https://releases.rancher.com/os/releases.yml
    image: rancher/os
```

#### Docker Configuration

The `docker` key configures the docker arguments and TLS settings.

```yaml
rancher:
  docker:
    tls_args: [--tlsverify, --tlscacert=ca.pem, --tlscert=server-cert.pem, --tlskey=server-key.pem,
      '-H=0.0.0.0:2376']
    args: [daemon, --log-opt, max-size=25m, --log-opt, max-file=2, -s, overlay, -G, docker, -H, 'unix:///var/run/docker.sock', --userland-proxy=false]
```

#### System-Docker Configuration

The `system_docker` key configures the system-docker arguments.

```yaml
rancher:
  system_docker:
    args: [daemon, --log-opt, max-size=25m, --log-opt, max-file=2, -s, overlay, -b, docker-sys,
      --fixed-cidr, 172.18.42.1/16, --restart=false, -g, /var/lib/system-docker, -G, root,
      -H, 'unix:///var/run/system-docker.sock', --userland-proxy=false]
```
