---
title: Zookeeper学习记录(一)--环境部署
tags:
  - zookeeper    
categories: 运维
date: 2017-08-19 09:34:10
updated: 2017-08-19 09:34:10
---
zk在公司系统中承担着一个很重要的角色,因此作为开发有必要了解关于zk的一些知识,推荐文档资料[Zookeeper文档目录](http://www.majunwei.com/category/201612011952003333/).
- - - - -
### 单机部署
zk的安装很简单,只需要下载修改配置启动即可,本文主要是用Docker方式安装,直接上Dockerfile文件
```conf
# Version 0.0.1
FROM java:8
MAINTAINER quding mrdear.cn 

#修改源信息
RUN echo "deb http://mirrors.163.com/ubuntu/ wily main restricted universe multiverse" > /etc/apt/sources.list
RUN echo "deb http://mirrors.163.com/ubuntu/ wily-security main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "deb http://mirrors.163.com/ubuntu/ wily-updates main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "deb http://mirrors.163.com/ubuntu/ wily-proposed main restricted universe multiverse">> /etc/apt/sources.list
RUN echo "deb http://mirrors.163.com/ubuntu/ wily-backports main restricted universe multiverse">> /etc/apt/sources.list
RUN echo "deb-src http://mirrors.163.com/ubuntu/ wily main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "deb-src http://mirrors.163.com/ubuntu/ wily-security main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "deb-src http://mirrors.163.com/ubuntu/ wily-updates main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "deb-src http://mirrors.163.com/ubuntu/ wily-proposed main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "deb-src http://mirrors.163.com/ubuntu/ wily-backports main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "APT::Get::AllowUnauthenticated 1 ;" >> /etc/apt/apt.conf

#更新源,并安装vim
RUN apt-get update
RUN apt install vim -y

ENV ZOOKEEPER_VERSION 3.4.9

# 下载zk
RUN wget -q http://mirror.vorboss.net/apache/zookeeper/zookeeper-${ZOOKEEPER_VERSION}/zookeeper-${ZOOKEEPER_VERSION}.tar.gz

# 安装
RUN tar -xvf zookeeper-${ZOOKEEPER_VERSION}.tar.gz -C /opt

# 配置zoo.cfg
RUN mv /opt/zookeeper-${ZOOKEEPER_VERSION}/conf/zoo_sample.cfg /opt/zookeeper-${ZOOKEEPER_VERSION}/conf/zoo.cfg
ENV ZK_HOME /opt/zookeeper-${ZOOKEEPER_VERSION}

RUN sed  -i "s|/tmp/zookeeper|$ZK_HOME/data|g" $ZK_HOME/conf/zoo.cfg; mkdir $ZK_HOME/data

EXPOSE 2181 2888 3888

WORKDIR /opt/zookeeper-${ZOOKEEPER_VERSION}
VOLUME ["/opt/zookeeper-${ZOOKEEPER_VERSION}/conf", "/opt/zookeeper-${ZOOKEEPER_VERSION}/data"]

CMD bash /opt/zookeeper-${ZOOKEEPER_VERSION}/bin/zkServer.sh start-foreground

```
build之后,启动该docker image,暴露出2181端口,然后使用zkClient.sh连接即可操作.

### 多机部署
多机部署主要参考文章[分布式服务框架 Zookeeper -- 管理分布式环境中的数据](https://www.ibm.com/developerworks/cn/opensource/os-cn-zookeeper/index.html).下面说下注意事项.
#### 注意端口配置
**initLimit**：这个配置项是用来配置 Zookeeper 接受客户端（这里所说的客户端不是用户连接 Zookeeper 服务器的客户端，而是 Zookeeper 服务器集群中连接到 Leader 的 Follower 服务器）初始化连接时最长能忍受多少个心跳时间间隔数。当已经超过 10 个心跳的时间（也就是 tickTime）长度后 Zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是 5*2000=10 秒
**syncLimit**：这个配置项标识 Leader 与 Follower 之间发送消息，请求和应答时间长度，最长不能超过多少个 tickTime 的时间长度，总的时间长度就是 2*2000=4 秒
**server.A=B：C：D**：其中 A 是一个数字，表示这个是第几号服务器；B 是这个服务器的 ip 地址；C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口；D 表示的是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。如果是伪集群的配置方式，由于 B 都是一样，所以不同的 Zookeeper 实例通信端口号不能一样，所以要给它们分配不同的端口号。
#### 如何验证
zk是主从复制形式,那么只要在leader节点中插入一个node,然后去salve中查看该node是否存在即可.

### 如何连接?
zk的安装目录下bin中有zkCli.sh,该工具格式为`zkCli.sh -server host:port`,使用其可以对zk进行增删改查,具体命令使用参数`-?`即可翻阅.

