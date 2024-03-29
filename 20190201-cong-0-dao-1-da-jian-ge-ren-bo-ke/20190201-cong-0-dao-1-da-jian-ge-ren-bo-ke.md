# 在阿里云上搭建个人博客步骤如下： \#

* 1、购买阿里云云服务器ECS\([传送门](https://promotion.aliyun.com/ntms/act/enterprise-discount.html?userCode=p7a3gxbs)\)
* 2、安装个人博客平台
* 3、购买域名\([传送门](https://tm.aliyun.com/?userCode=p7a3gxbs)\)
* 4、申请备案\([传送门](https://beian.aliyun.com)\)
* 5、设置域名解析\([传送门](https://help.aliyun.com/document_detail/29716.html)\)

经过1,2骤其实就可以通过ECS实例的IP进行博客访问，如果想要通过域名进行访问，我们还要进行3,4,5。

## 下面我就介绍上我们会遇到的一些坑点：\#\#

* 1、如果你一开始购买的是window系统，那么我们如果改成Linux系统：

  > 1.1 我们先找到ECS实例，然后选中ECS实例，进行停用 ![](https://i.imgur.com/Ev2BIwB.png)

> 1.2 更换系统盘\(我没有停用，所以是灰色不可点击状态，停用的话是可以点击的\) ![](https://i.imgur.com/ztVsGlb.png)
>
> 1.3 进入镜像市场、选择操作系统 1.4 选择免费的宝塔控制面板，并设置宝塔的登录密码 1.5 需要配置安全组规则【安全组：允许开放的端口号列表】（如何配置请自行Google） ![](https://i.imgur.com/Mua3Pm3.png) 1.6 其他可以参考下述\([参考博客](https://www.cnblogs.com/venom95/p/9760445.html)\)

* 2、使用宝塔面板快速搭建\([参考博客](https://2012.pro/index.php/20180806/cid=40.html)\)

  > 2.1 在搭建的时候，发现宝塔面板不是最新的，这是无法忍受的，于是对宝塔面板进行升级，但是默认有不允许直接进行更新，于是Google Allo到下述方法\([从5.9平滑升级到6.x](https://www.bt.cn/bbs/thread-19552-1-1.html)\)
  >
  > 2.2 个人博客平台选择：目前比较常见的是WordPress，Typecho，Hexo，这个自行选择，笔者是选择Typecho。\([ Typecho和Hexo](https://zhuanlan.zhihu.com/p/35764312)\)\([Typecho和WordPress](https://www.imydl.com/wzjs/6684.html)\)
  >
  > 2.3 安装typecho时报错：对不起，无法连接数据库，请先检查数据库配置再继续进行安装:这个我们需要查看，宝塔面板数据库是否创建，没有就进行创建。 创建后需要查看我们设置的数据库名称，数据库用户名和密码，是否和宝塔面板数据库上述三项相同。

> 3、域名解析 3.1 当时遇到gocpplua.com可以访问，但是www.gocpplua.com无法访问，此时需要进入宝塔面板在网站出，进行站点设置，加上域名：www.gocpplua.com ![](https://i.imgur.com/Lm30aAc.png)

其他参考博客： [Linux服务器+宝塔面板搭建Typecho博客等网站](https://waxxh.me/archives/linux-bt-typecho.html)

