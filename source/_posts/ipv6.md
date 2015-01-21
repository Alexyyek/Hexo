title: 教育网配置IPV6回避收费及轻松翻墙
date: 2014-12-27 17:06:18
categories: 黑科技
tags: skill
description: IPV6谁用谁知道
---

明光村职业技术学院终于还是没能逃离收费的命运，虽然还有半年就卷铺盖跑路了，但面对每个月只给10G外网流量，100块都不给我的抠门政策，废话不多说，直接上干货。

##目录
###设置IPV6

* 首先安装ipv6，这个是基础，win7默认安装，xp运行命令`ipv6 install`。访问网站 [http://6rank.edu.cn](http://6rank.edu.cn/) 看看IPv6配置是否正常。更详细介绍请自行百度。
* 打开网络和共享中心 → 更改适配器设置 → 本地连接 → 属性 → 双击Internet协议版本6（IPV6），设置首选DNS服务器为：**2001:778::37**

<center> ![](http://alexyoung.qiniudn.com/QQ图片20141227164226.jpg) <center/>

[**原理**](http://ipv6.lt/nat64_en.php)：NAT64服务器，能将所有ipv4的网站都转化对应的ipv6地址，也相当于一种代理。晚上速度比较受影响，日间科研完全可以满足。

<center> ![](http://alexyoung.qiniudn.com/graph_relay.png) <center/>

* 将Internet协议版本4（IPV4）前的对勾去掉，确定后退出，即可只通过IPV6访问网站，不用登陆 [http://10.3.8.211/](http://10.3.8.211/)，此时流量不计入已使用流量


###设置hosts翻墙

如果你是通过代理访问 [youtube](https://www.youtube.com/)，往往速度很不给力。如果是教育网有ipv6的话，那就好办了，看1080P流畅的一比，而且是教育网使用ipv6是不计费的。方法很简单，就是修改hosts.

* 访问 [__网站__](http://serve.netsh.org/pub/ipv6-hosts/)，勾选你所要的服务，如下图，点击立刻获取。
<center> ![](http://alexyoung.qiniudn.com/QQ图片20141227165359.jpg) <center/>

* 将生成的hosts文件内容贴到你的hosts文件里并保存：`C:\Windows\System32\drivers\etc`
* 最后总是以**HTTPS**的方式打开Youtube、facebook，如：[https://www.youtube.com/](https://www.youtube.com/)，[https://www.facebook.com/](https://www.facebook.com/)
如果嫌麻烦，可以 [**点这里**](http://blog.netsh.org/posts/chrome-http-redirect-https_369.netsh.html) 设置chrome强制http定向到https


PS.如果某天发生访问不了例如youtube的情况，是因为ipv6地址会经常失效，也可能是地址变更，或者河蟹出现，记得更新hosts。
访问youtube看1080P无压力

![](http://alexyoung.qiniudn.com/QQ图片20141227161806.jpg)

https://code.google.com/p/ipv6-hosts/

**PS**：本方法只是临时解决方案，目的是培养大家合理利用搜索引擎，解决实际问题的意识，欢迎大家提出不同的改进意见和更完善的解决方案。