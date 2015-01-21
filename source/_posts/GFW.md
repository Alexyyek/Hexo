title: 免费且使用便捷的翻墙方案
date: 2014-12-31 17:55:42
categories: 黑科技
tags: skill
description: 不要问我为什么翻墙，因为墙就在那里。
---

##目录
    
翻墙方法多如牛毛，但都因为自身的因素而各有利弊。Freegate使用简单，适合初学者，但开启后访问国内网站将出现问题；VPN有的收费有的免费，免费的VPN由于使用者众多网速low的一比，收费的能够提供更好的服务，但需要为之买单。而且从技术角度分析VPN加密流程会发现：你的机子->(加密)->VPN->(解密后的)->访问目标机子，就算你的机子和VPN之间怎么加密，在最终发给目标机子都会还原成没加密的。这并不是吓唬大家，只是万事有利有弊，自行取舍罢了。而上一篇[改Hosts](http://alexyyek.github.io/2014/12/27/ipv6/)的办法虽然简单，但能翻的网站有限，而且需要不断更新。

下面推荐的是一种基于GAE（Google App Engine）+ goagent + switchysharp + chrome的翻墙之法，相对刚才介绍的方案，此方法上手较易，使用体验也非常不错（当你配置完后便能充分的感受到），并且完全免费。所以请不要害怕这过长的篇幅，按部就班10分钟左右搞定。

那么让我们一起迎接2015这新世界吧。
<center> ![](http://alexyoung.qiniudn.com/464564.jpg) <center/>
<br/>
***
###入门介绍
1. GAE（Google App Engine）是一个开发、托管网络应用程序的平台；
2. goagent是一个使用Python和Google Appengine SDK编写的代理软件；
3. switch sharp是一个chrome插件，具体功能类似于Firefox的AutoProxy.

###申请Google App Engine并创建app
* 首先去注册一个google账户，没有绑定手机号码的建议[绑定](https://security.google.com/settings/security/signinoptions/rescuephone)。
* 创建属于你的[**Application**](https://appengine.google.com/start/createapp)，填写你的APP ID，

**错误体位**：
<center> ![](http://alexyoung.qiniudn.com/GAE1.png) <center/>

**正确体位**
<center> ![](http://alexyoung.qiniudn.com/GAE2.png) <center/>

**填写方式**
<center> ![](http://alexyoung.qiniudn.com/GAE3.png) <center/>

* 一切正常的话就会看到下面的界面（告诉你App已经申请成功了）
<center> ![](http://alexyoung.qiniudn.com/GAE4.png) <center/>

<br/>
**注意**：GAE的check不一定可靠，可能出现提示available但是commit后提示`This application ID or version is already in use`的情况，所以请另换ID。现在每个用户可以创建25个APP，每个应用每天有1GB的免费流量。如果你经常下载或者观看视频建议多创建几个Google App Engine应用。
<br/>
***
###下载 goagent 并上传至 Google App Engine
* 去Github下载 [**goagent客户端**](https://github.com/goagent/goagent) 并解压，目前是3.2.3版
* 修改`\local\proxy.ini`，把其中 appid = goagent 中的 goagent 改成你之前申请的应用的 appid 后保存。
<center> ![](http://alexyoung.qiniudn.com/GFW9.jpg) <center/>

如果要使用多个appid，appid之间用`|`隔开，如：`appid1|appid2|appid3`，每个 appid 必须确认上传成功才能使用
> [gae]
appid = appid1|appid2|appid3

* 双击 server 文件夹下的 uploader.bat， 依次输入你的Application ID，邮箱地址和密码（如果设置了Gmail的两步验证请输入两步验证中生成的16位密码，即为独立应用单独生成的随机16位字符，而不是你的邮箱密码，Gmail两步验证在这里[详细介绍](http://www.cooear.com/archives/168.htm)）
<center> ![](http://alexyoung.qiniudn.com/GFW10.jpg) <center/>

<br/>
* 看到下面的提示便上传成功。
<center> ![](http://alexyoung.qiniudn.com/GFW6.jpg) <center/>

**注意**：如果输完账户密码后出现：`AttributeError: can’t set attribute` 的问题，可尝试进入 [**Google 安全性**](https://www.google.com/settings/security) 页面里看到了『帐户所授权限』 中的『不够安全的应用的访问权限』，进入设置之后，点击『启用』即可。
<br/>
***

###Chrome下安装Switchysharp
* 去chrome的[应用商店](https://chrome.google.com/webstore/category/home)安装Switchysharp
* 下载好后Switchysharp图标会出现在chrome的右上角，点击选项
<center> ![](http://alexyoung.qiniudn.com/GFW11.jpg) <center/>

* 在导入/导出界面，选择从文件恢复，选中`\local\SwitchyOptions.bak`确认导入
<center> ![](http://alexyoung.qiniudn.com/GFW8.jpg) <center/>

* 成功导入后，想去外网时选择自动切换模式即可，不访问时便选择直接连接
<center> ![](http://alexyoung.qiniudn.com/GFW12.jpg) <center/>

**注意**：使用Firefox浏览器和opera浏览器的同学去[**这里**](https://github.com/goagent/goagent/blob/wiki/InstallGuide.md)看配置插件教程。
<br/>
***
###最后
恭喜你已经成功配置好了插件，想去外网时记得先打开`local\goagent.exe`，最好管理员权限打开，如果嫌每次都打开麻烦，设置开机自动启动也OK，浏览器会根据你浏览的具体界面选择代理方式，一键都不用便可翻墙，这便是此方案便捷的地方。

有兴趣的同学可以研究研究[ShadowSocks](http://shadowsocks.cn/)，ShadowSocks服务器端提供了各种版本，如Python、Nodejs、Go、C libev等等，安装配置过程极其简单。而用户端则可以在windows、mac、iOS和android上轻松运行。首先，当然你需要有一个shadowsocks账号。(￣∇￣;)
