---
title: Docker学习记录(三)-构建非跨平台项目编译环境
categories: docker
tags:
 - docker
date: 2017-03-12 15:21:00

---

# Docker学习记录(三)-构建非跨平台项目编译环境

标签（空格分隔）： docker

---
>个人独立博客: http://mrdear.cn

因为毕业设计的问题所以去学了docker,本文描述这个问题解决的过程.

----------

1.问题
----
在毕业设计AUSTOJ中,判题端使用JNI方式调用C++来编译和执行代码,得到输出结果,Java端进行结果对比.然而该C++代码在mac下无法编译,总是会报错,JNI也会出问题.另外该子模块在mac下无法使用maven打包,所以打包也需要放在docker中.
因此docker需要环境 java maven gcc g++ make

2.构建编译环境
--------
编写dockerfile文件,该文件的maven包我是从本机复制进去的,同样你也可以从外网下载.
Dockerfile:
```
#构建judger端需要的环境,方便本地测试
#基于java8环境
FROM java:8

#维护人信息
MAINTAINER quding niudear@foxmail.com
#更新源
RUN apt-get update
#gcc g++ make安装
RUN apt-get install -y gcc-4.9
RUN apt-get install -y g++-4.9
RUN apt-get install -y build-essential

#配置mvn环境
ADD apache-maven-3.3.9.tar.gz /usr/local
ENV M2_HOME /usr/local/apache-maven-3.3.9
ENV PATH $PATH:$JAVA_HOME/bin:$M2_HOME/bin

#jni环境
RUN cp $JAVA_HOME/include/linux/jawt_md.h $JAVA_HOME/include/
RUN cp $JAVA_HOME/include/linux/jni_md.h $JAVA_HOME/include/

```

构建命令:
`docker build -t dev .`

3.挂载运行
------
运行时需要挂载本项目到docker中,该挂载是映射,因此本地和docker任意位置改变项目中文件都会反映在真实项目中,这也是想要的结果.
挂载命令:
```
docker run -ti -p 50013:50013  -v /Users/niuli/workspace/git/AUSTOJ2/:/AUSTOJ2 
-v /Users/niuli/workspace/git/testcase/:/austoj/testcase dev
```
该命令以交互模式启动一个docker容器,同时绑定docker的50013端口到此容器的50013,因为我的项目使用的是50013端口.另外我挂载了本项目目录AUSTOJ2和测试数据目录分别到docker的/AUSTOJ2目录和/austoj/testcase目录.

那么启动之后如下所示:
![](http://ac-HSNl7zbI.clouddn.com/sVRm9T6RaAgcL0tqAX7vGz0kaTVDT21kJbSSokIA.jpg)

ok,到此编译环境搞定,可以随心所欲的编译启动该子模块,并且还能实时反映到本机目录下

![](http://ac-HSNl7zbI.clouddn.com/y020GeCL2UrSuASyDaYbvWs0XF3LWRYqRbej5pAB.jpg)

![](http://ac-HSNl7zbI.clouddn.com/bOKHInF9SpgHTSmr361EhU2geUFRjKW1yPwHap6s.jpg)

![](http://ac-HSNl7zbI.clouddn.com/JXVXNAq7Q6JtPc9QhKzJAdu1h3HskLriYhruA1tY.jpg)