## Rancher 实战红宝书

Rancher 是非常优秀的容器管理平台，本书旨在沉淀和积累相关实战文档。本书通过 GitHub 和 GitBook 平台共同承载，并实现参与者的协作。本书使用开源社区的方式，由Rancher 用户社区合作维护。

参考源： http://docs.rancher.com

本书在线访问网址：http://rancher.hidocker.io   

本书的初始化版本，保持了和 Rancher 文档官网同步的目录结构（为了维护方便，RancherOS 被当做一个独立章节纳入了本书）；为了方便您的翻译，请建议直接下载 https://github.com/rancher/rancher.github.io 英文源站到本地（作为英文原文的离线参考）；然后再 Fork https://github.com/martinliu/rancher-docs 到自己的仓库，并从自己的仓库下载本书电子书源码到本地，您会看到本书的很多章节还都是空白文章；然后您就可以可以开始工作了。本书的第一个阶段性目标是：对官方英文文档做 1：1的翻译。然后是收集各位高手的原创好文。目标是打造出一份很实用和全面的参考资料库。

如果你没有 GitHub 的使用经验，请查看这个视频，否则忽略此段，继续仔细阅读本网页；视频会教会您纯网页版的操作和协作流程，其中示例了如何把 6.1 访问控制 做英文原文的初始化，和翻译提交。视频点这里。[http://v.youku.com/v_show/id_XMTU1NDk4Mzc2OA](http://v.youku.com/v_show/id_XMTU1NDk4Mzc2OA)

Github 是一个协作平台，pull request 是解决冲突的方式，你提交的 pull request 上来以后，如果有冲突，在这种情况下电子书源码的管理员需要解决这个冲突，从两个冲突的 pull request 中选择出一个更好的。为了快速响应所有文档参与者的贡献，现在征召电子书源码仓库管理员合作者（Collaborators）；如果您提交的 PR 被三次接受，且您有意参与源码管理工作；您就可以申请成为 Collaborators ；请发一个 Issue 做出申请，没有人反对的话，2天内就会成为 Collaborators。

为了避免两个人或者更多的人同时开始翻译同一篇文章，当您开始决定做某个文章的时候，请通过首个 PR 来标记一下您的目标文章。具体的操作说明：打开您的目标文章（空白文章，只有一行标题的那种），在首行写一句话“本文档首稿正在编辑中，请随后帮忙 review。”（初始化标记），提交一个更新 PR，等您的 PR 被确认之后，您即可进入了独占的编辑时段。您选择多篇文章的话，请在同一个 PR 中一次性提交初始化标记。

## 贡献者
本书源码开源托管在Github，欢迎参与维护：https://github.com/martinliu/rancher-docs， 贡献者名单见[ GitHub contributors](https://github.com/martinliu/rancher-docs/graphs/contributors) 页面。

欢迎参与线上讨论。

* QQ 群1 
![314700162](q1.png)

*  群名称：Rancher实战红宝书群1
*  群号：314700162

为了您能顺利领取到相关纪念品（首批为 Rancher logo 定制版 Tshirt），请加入以上 QQ 群，并用 Github ID 备注自己的姓名。 详情见 [http://hidocker.io/book/](http://hidocker.io/book/)

## 参与协作

为了提高参与的效率，并省去在 Gitbook 上的操作的步骤。请直接加入本书在 GitHub 上的源代码协作来参与到文档的维护中来。

参与步骤：注册 Github 账号，克隆本书源码，下载到本机，编辑目标文章，推送到自己的仓库，提交 pull request 到本书源码，PR 通过之后，您的贡献自动跟新到线上版本。

在网页上就可以完成所有文章编辑和提交工作，操作流程在这里 [http://v.youku.com/v_show/id_XMTU1NDk4Mzc2OA](http://v.youku.com/v_show/id_XMTU1NDk4Mzc2OA) 网页版直接操作，请随时保存，以免网络中断导致的信息丢失。

### 具体操作步骤（Github 命令行版）

在 GitHub 上 fork 到自己的仓库，如 docker_user/rancher-docs，然后 clone 到本地，并设置用户信息。

```
$ git clone git@github.com:martinliu/rancher-docs.git
$ cd rancher-docs
$ git config user.name "yourname"
$ git config user.email "your email"

```
修改代码后提交，并推送到自己的仓库。

```
$ #do some change on the content 编辑您参与的文章
$ git commit -am "Fix issue #1: change helo to hello"
$ git push

```
在 GitHub 网站上提交 pull request。

定期使用项目仓库内容更新自己仓库内容。

```
$ git remote add upstream https://github.com/martinliu/rancher-docs
$ git fetch upstream
$ git checkout master
$ git rebase upstream/master
$ git push -f origin master

```
或者使用 GitHub Desktop 图形化客户端完成以上所有操作，推荐使用 [Atom](http://atom.io) 作为本地 .mk 文件编辑器。


## 添砖加瓦
本书不仅限于对 Rancher 官方原文的翻译，更关注任何相关实战文档的汇聚。如果您有这个想法，请在 https://github.com/martinliu/rancher-docs/issues 页面点击 New Issue 按钮，在 Issue 中描述您想提交的“文章标题”和位置。Issue 被处理之后源码管理员会尽快初始化这篇文章，您同步更新到本地，就可以开始编辑上传这篇文章了。


## 主要版本历史

* v0.5.0 完成了第一版全目录，章节条目为 126 行，更新了 Readme
* v0.5.1 跟新了协作方法，用提交初始化标记 PR 替换了之前的认领方式，上传了网页版 github 操作视频。
* 


