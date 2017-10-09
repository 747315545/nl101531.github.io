---
title: cnpm导致IDEA对于Angular项目提示失效
tags:
  - angular
  - idea
categories: web
date: 2017-08-26 11:17:15
updated: 2017-08-26 11:17:15
---
最近写Angular项目的时候,IDEA的提示时而有时而没有,找了好久的原因才发现是cnpm的锅.
对于`cnpm install`,安装的angular依赖时链接方式引入,如下图
![](http://oobu4m7ko.bkt.clouddn.com/1503717543.png?imageMogr2/thumbnail/!70p)

对于`npm install`,安装后的依赖时实在的文件,如下图
![](http://oobu4m7ko.bkt.clouddn.com/1503717577.png?imageMogr2/thumbnail/!70p)

解决方案老老实实的用npm命令,觉得慢的话可以使用http代理,mac下的shadowsocks支持直接导出http代理,复制命令后粘贴到终端,即可实现终端翻墙.
![](http://oobu4m7ko.bkt.clouddn.com/1503717675.png?imageMogr2/thumbnail/!70p)

如果你不会翻墙,可以参考我之前写的教程  [如何搭建属于自己的shadowsocks](http://mrdear.cn/2017/08/07/%E5%B7%A5%E5%85%B7/%E5%A6%82%E4%BD%95%E6%90%AD%E5%BB%BA%E5%B1%9E%E4%BA%8E%E8%87%AA%E5%B7%B1%E7%9A%84shadowsocks/)
