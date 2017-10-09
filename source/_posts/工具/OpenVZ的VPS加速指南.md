---
title: OpenVZ的VPS加速指南
tags:
  - shadowsocks    
categories: vps
date: 2017-08-13 22:39:05
updated: 2017-08-13 22:39:05
---
### 实现理论
OpenVZ架构的VPS加速选择比较少,不然KVM方便,除去双边加速比如FinalSpeed等软件后可用选择并不多,其中比较好的方案是Google BBR加速,为了在OpenVZ架构上使用必须借助UML这一子linux系统.

所谓的UML全称为User Mode Linux,允许用户在Linux中以一个进程的方式再运行一个lInux,那么就很容易实现我们所需要的加速架构,原理是在UML中启动Shadowsocks,然后访问该Shadowsocks实现加速.

### 安装指南
拿来主义原则,给出原博主链接[OpenVZ的UML+BBR加速一键包](https://www.91yun.co/archives/5345),按照文章描述步骤配置即可,在这里做一些额外的补充.

**宿主机请求转发**
一键脚本配置完毕后其是运行在UML主机里面的程序,其内网相对宿主机ip为`10.0.0.2`,但我们只能访问到宿主机,需要如下命令转发,其中端口`8888`改成你的ss配置的端口即可.

```
iptables -t nat -A PREROUTING -i venet0 -p tcp --dport 8888 -j DNAT --to-destination 10.0.0.2
iptables -t nat -A PREROUTING -i venet0 -p udp --dport 8888 -j DNAT --to-destination 10.0.0.2
```

接下来像以往一样访问即可.

