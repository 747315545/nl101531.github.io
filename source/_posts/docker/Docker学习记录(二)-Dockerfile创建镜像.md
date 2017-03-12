---
title: Docker学习记录(二)-Dockerfile创建镜像
categories: docker
tags:
 - docker
date: 2017-03-10 21:21:00
---

# Docker学习记录(二)-Dockerfile创建镜像

标签（空格分隔）： docker

---

本文学习Dcokerfile的基本命令,并且创建一个支持ssh服务的镜像.


----------

1.Dockerfile
------------
### 1.1基本案例
dockerfile可以说是docker的描述符,该文件定义了docker镜像的所能拥有哪些东西.基本格式如下:
```
第一行指定该镜像基于的基础镜像(必须)
FROM java:8

维护者信息
MAINTAINER quding  niudear@foxmail.com

镜像操作指令
RUN echo $JAVA_HOME

启动时操作的命令

CMD ./usr/sbin/nginx
```
该文件说明从Java8这个基础镜像创建一个新的镜像,输出Java路径,启动成功则启动nginx服务,这也是一个Dockerfile需要包含的操作步骤.

### 1.2指令详解

1.FROM：格式为 `FROM <image>`或`FROM<image>:<tag>`第一条指令必须是FROM指令。并且，如果在同一个Dockerfile中创建多个镜像时，可以使用多个FROM指令（每个镜像一次）。

2.MAINTAINER：格式为MAINTAIER<name>，指定维护者信息。

3.RUN：格式为`RUN <command>`或者`RUN [“executable”，“param1”，“param2”]`。前者将在shell终端中运行的命令，即/bin/sh–c；后者则使用exec执行。指定使用其他终端可以通过第二种方式实现，例如`RUN[“/bin/bash”，“-c”，“echohello”]`。每条RUN指令将在当前镜像基础上执行指定命令，并提交为新的镜像。当命令较长时可以使用\来换行。这实际上就是在容器构建时需要执行哪些指令，例如容器构建时需要下拉代码，但是默认启动的容器中是没有Git指令的，就需要下载，可以执行：`RUN apt-get install -y git`，然后`RUN git clonexxxx`

4.CMD：指定容器启动后执行的命令，一般都是早就写好的脚本，例如：`CMD[“/run.sh”]`。注意：如果Dockerfile中指定了多条命令，只有最后一条会被执行。如果用户启动时候加了运行的命令，则会覆盖掉CMD指定的指令。

5.EXPOSE：告诉Docker服务端容器需要暴露的端口号，供互联系统使用。在启动容器时需要通过-P（注意是大写），Docker主机会自动分配一个端口转发到指定的端口；使用-p，则可以具体指定哪个本地端口映射过来。
例如：我在elasticsearch镜像的Dockerfile中指定了暴露出9200和9300端口，我可以在Dockerfile中写：`EXPOSE 9200 9300`

6.ENV：创建的时候给容器中加上个需要的环境变量。指定一个值，为后续的RUN指令服务

7.COPY：复制本地的文件或目录到容器中。目标路径不存在时，会自动创建。

8.ENTRYPOINT：配置容器启动后执行的命令，并且不可被docker run 提供的参数覆盖。
每个Dockerfile中只能有一个ENTRYPOINT，当指定多个ENTRYPOINT时，只有最后一个生效

9.VOLUME：创建一个挂在点，可以从本机或其他容器挂载的挂载点。意思就是从容器中暴露出一部分，和外界共享这块东西，一般放数据库的数据或者是代码。在容器启动运行的时候，如果需要将volume暴露的东西和本地的一个文件夹进行映射，想要通过本地文件直接访问容器中暴露的部分，可以在运行的时候进行映射：

10.USER：指定运行容器时的用户名或者UID，后续的RUN也会使用指定的用户。当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户。
要临时获取管理员权限的时候要使用gosu，不推荐使用sudo。如果不指定，容器默认是root运行。

11.WORKDIR：定义工作目录，如果容器中没有此目录，会自动创建

创建指令`docker build 路径`,该命令会读取路径下的Dockerfile文件和其他文件,然后发送给服务端,由服务端创建镜像.

----------

2.创建SSH服务镜像
-----------
### 2.1准备Java8环境
后续教程需要利用到Java8环境,因此先下载一个官方的Java8镜像作为基础镜像.直接执行如下命令.可以利用之前的教程,启动容器查看下java路径.
```
docker pull java:8
```
![](http://ac-HSNl7zbI.clouddn.com/sJUUIIhnu1bxyfWnYtf8VfN7W3z5NMMj7lARWGpw.jpg)

### 2.2编写Dockerfile
ssh服务主要是openssh-server来提供,因此需要在容器中安装该服务.
**Dockerfile:**
```
#显示该镜像是基于java8镜像
FROM java:8

#维护人信息
MAINTAINER quding niudear@foxmail.com
#更新源
RUN apt-get update
#安装软件
RUN apt-get install -y openssh-server

RUN mkdir -p /var/run/sshd
RUN mkdir -p /root/.ssh

#取消pam限制
RUN sed -ri 's/session  required   pam_loginuid.so/#session    required  pam_loginuid.so/g' /etc/pam.d/sshd

#复制配置文件到相应位置
COPY authorized_keys /root/.ssh/authorized_keys
COPY run.sh /run.sh

#赋予脚本权限
RUN chmod 755 /run.sh

#开放端口
EXPOSE 22

#设置启动命令

CMD ["/run.sh"]
```

**run.sh**
```
#!/bin/bash
/usr/sbin/sshd -D
```

**拷贝本机的id_ras**
```
cat ~/.ssh/id_rsa.pub >authorized_keys
//用来免密的
```

**执行构建**
```
docker build -t sshd:java .  
```

构建成功后使用`docker images`即可查看,然后像上篇一样启动容器,暴露出端口,再使用ssh连接,和一般linux系统就没什么差别了.


