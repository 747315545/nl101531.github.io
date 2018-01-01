---
title: 如何搭建属于自己的shadowsocks
subtitle: 如何搭建属于自己的shadowsocks
cover: http://oobu4m7ko.bkt.clouddn.com/1514806095.png
author: 
  nick: 屈定
tags:
  - shadowsocks    
categories: vps
date: 2017-08-07 23:19:01
updated: 2017-08-07 23:19:01
---

### shadowsocks是什么?
能看到这篇文章的人大概对这个都有些了解,可以理解为一个代理隧道,通过其可以代理到指定的vps服务器,然后服务器去获取到你所访问的内容再返回给你.
如果你把服务器当做跳板机的话,那么shadowsocks就是你与跳板机之间的关联.

### vps服务器的选择
vps服务器有很多,这里推荐下搬瓦工KVM架构的机器,推荐理由便宜,可靠,支持支付宝.
> 不介意可以使用我的邀请链接: [https://bandwagonhost.com/aff.php?aff=17639](https://bandwagonhost.com/aff.php?aff=17639)

注册后选择最便宜款的KVM架构,重要的事情说三遍,KVM架构,KVM架构,KVM架构.至于好处是可以使用锐速,能让你的shadowsocks更加快.
![](http://oobu4m7ko.bkt.clouddn.com/1502120173.png?imageMogr2/thumbnail/!70p)
接下来是选择付款方案,一般选择$19.9的年付,三四个小伙伴一起用,平摊这个费用的话,就相当划算了.
![](http://oobu4m7ko.bkt.clouddn.com/1502120257.png?imageMogr2/thumbnail/!70p)
买完后会进去类似的管理后台,选择一键安装即可.
![](http://oobu4m7ko.bkt.clouddn.com/1502120362.png?imageMogr2/thumbnail/!70p)
到这里,vps服务端的shadowsocks就部署完毕了.接下来是客户端连接.

### shadowsocks客户端
在github上有各个平台的客户端[https://github.com/shadowsocks](https://github.com/shadowsocks),windows一般用`shadowsocks-windows`,mac用`ShadowsocksX-NG`,linux则用`shadowsocks-qt5`,下载对应客户端,配置好vps生成的shadowsocks端口和密码,启用即可,其主要作为一个本地服务器,其他应用软件通过其余vps服务器通信.

**如何访问?**
浏览器安装插件[switchyomega](https://switchyomega.com/),该插件会代理浏览器的请求链接,根据规则列表决定该链接是否要使用shadowsocks代理.
首先新建一个代理模式,该模式下所有请求都会走shadowsocks.
![](http://oobu4m7ko.bkt.clouddn.com/1502120910.png?imageMogr2/thumbnail/!70p)
其次建立一个自动情景切换模式,该模式会根据配置的规则自动选择对应的情景模式来处理.该模式主要分为三部分,第一部分是用户自定义,比如图片中我指定匹配`*.github.com`的连接走的是ss情景模式.
第二部分是规则列表,我配置的为`https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt`,该文件中是被墙的一些地址,这些地址都走ss情景模式,其余的都是直接连接.
![](http://oobu4m7ko.bkt.clouddn.com/1502121104.png?imageMogr2/thumbnail/!70p)

接下来就可以访问google了.

### 参考连接
锐速安装教程: [https://github.com/91yun/serverspeeder](https://github.com/91yun/serverspeeder)
Google BBR : [https://teddysun.com/489.html](https://teddysun.com/489.html)




