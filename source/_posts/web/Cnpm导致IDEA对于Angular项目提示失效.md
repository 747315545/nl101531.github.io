---
title: cnpm导致IDEA对于Angular项目提示失效.md
tags:
  - web
  - idea
categories: web
date: 2017-08-26 11:17:15
---
最近写Angular项目的时候,IDEA的提示时而有时而没有,找了好久的原因才发现是cnpm的锅.
对于`cnpm install`,安装的angular依赖时链接方式引入,如下图
![](http://oobu4m7ko.bkt.clouddn.com/1503717543.png?imageMogr2/thumbnail/!70p)

对于`npm install`,安装后的依赖时实在的文件,如下图
![](http://oobu4m7ko.bkt.clouddn.com/1503717577.png?imageMogr2/thumbnail/!70p)

解决方案老老实实的用npm命令,觉得慢的话可以使用http代理,mac下的shadowsocks支持直接导出http代理,复制命令后粘贴到终端,即可实现终端翻墙.
![](http://oobu4m7ko.bkt.clouddn.com/1503717675.png?imageMogr2/thumbnail/!70p)
