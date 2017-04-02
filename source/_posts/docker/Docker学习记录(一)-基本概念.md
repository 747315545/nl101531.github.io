---
title: Docker学习记录(一)-基本概念
categories: docker
tags:
 - docker
date: 2017-03-10 17:21:00

---

# Docker学习记录(一)-基本概念

标签（空格分隔）： docker

---
因为做的项目用到了docker,所以开始学习下这方面的知识.

----------

1.基本概念
------
docker虚拟机:docker环境,docker的操作都要依赖此虚拟机,可以理解为JDK.
docker镜像:镜像可以用面向对象中的Model类来理解,就是一个已经建立好的模型.
docker容器:容器可以关联面向对象中的实例来理解,实例是依赖类来创建,所以容器就是依赖镜像创建,同样一个类可以有多个实例,那么一个镜像也可以对应多个容器.
docker仓库:仓库是镜像市场,里面有别人建立好的Model类,也就是镜像,可以直接拿来使用.

这样说应该很好理解了吧.

因此创建一个helloworld的流程就和清晰了.
启动docker虚拟机->创建docker镜像(或者从仓库拉取)->创建docker容器(运行helloworld)->结束

2.docker虚拟机
-----------
首先docker安装后自带的虚拟机配置下载镜像又要GFW的原因速度很慢,一般使用[阿里云加速器][1],登陆后找到加速器按照要求先创建一个新的docker主机,然后启动该主机.
这里要注意,阿里云给的命令是创建一个名字为default的主机,安装后自带了一个default,所以先运行`docker-machine rm default`删除默认主机.

2.1新建主机
![](http://ac-HSNl7zbI.clouddn.com/sAxM3IuAIRznxVzOKQUSSmnVuh4KGub9bNLDN9P3.jpg)

2.2为当前shell配置环境
![](http://ac-HSNl7zbI.clouddn.com/AK0TxhfaoaaJgUR6XLDAxWoiml1uNr5aEPyhOHkn.jpg)

2.3验证
![](http://ac-HSNl7zbI.clouddn.com/BgvivpB6bjf61IBPyswjjHCb5XfcYjvrpOS9sDNo.jpg)

到此docker虚拟机创建完毕,这里需要掌握一些基本增删改查基本命令.
```
docker-machine kill 停止某个Docker主机
docker-machine ls 列出所有管理的Docker主机
docker-machine regenerate-certs 为某个主机重新成功TLS认证信息
docker-machine restart 重启Docker主机
docker-machine rm 删除Docker主机
docker-machine scp 在Docker主机之间复制文件
docker-machine ssh SSH到主机上执行命令
docker-machine start 启动一个主机
docker-machine status 查看一个主机状态
docker-machine stop 停止一个主机
docker-machine upgrade 更新主机Docker版本为最新
docker-machine url 获取主机的URL
```

3.docker镜像
----------
使用`docker images`可以列出机器上所有的docker镜像.
![](http://ac-HSNl7zbI.clouddn.com/axr3cW667D3Awsul4QA0qnVlrx2OYsRz0QJel6yG.jpg)

其中:
REPOSTITORY：表示镜像的仓库源
TAG：镜像的标签
IMAGE ID：镜像ID
CREATED：镜像创建时间
SIZE：镜像大小

使用`docker search 镜像名`查找某一镜像,例如查找hello world,可以看到带有OFFICIAL的为官方提供的镜像.
![](http://ac-HSNl7zbI.clouddn.com/LGnffJHC3CQIrxAMdBqUr6YXQf4s4CRiMLkhzwzY.jpg)

使用`docker pull 镜像名`获取一个镜像,这里获取hello world,另外镜像后可以跟版本号,例如`docker pull redis:3.2`,就指定拉去redis3.2版本
![](http://ac-HSNl7zbI.clouddn.com/pGDCyoQUkK3vnLXFRasOUzpDyLFbprXFTghVbzLf.jpg)

使用`docker run 镜像名`从该镜像启动一个实例.

常见命令,另外对于docker镜像的创建和运行比较重要,后续文章单独学习分析.
```
docker inspect 查看镜像详情
docker rmi 删除镜像,带上-f参数则强制删除
docker save 导出镜像
docker load 导入镜像
docker push 上传镜像到仓库
docker tag 给镜像设置标签
```

4.docker容器
----------
容器是应用的实例,使用`docker create`创建一个容器,使用`docker start`启动一个容器,另一个简单方式就是`docker run`,等价于先创建再启动.

那么使用`docker run`的时候后台做了哪些操作?
1. 查找是否存在指定镜像,不存在则从公有仓库下载
2. 利用镜像创建并启动一个容器
3. 分配一个文件系统,在只读的镜像层外面挂载一层可读写层
4. 从宿主主机配置的网桥接口中桥接一个虚拟接到容器中去
5. 从地址池配置一个ip地址给容器
6. 执行用户指定应用程序
7. 执行完毕后容器被终止

使用`docker ps -a`查看最近启动的容器
![](http://ac-HSNl7zbI.clouddn.com/aYGJha5vP2SwSQUEHtlNmRBU67vXS8co5KTCMO75.jpg)

使用`docker rm`删除容器,清理完毕后再删除hello world镜像.

下面使用redis镜像实战整个流程,并学习容器常用命令.

5.创建redis镜像
-----------
有了helloworld经历,这里流程就很清晰了,搜索镜像->拉去镜像->创建实例->连接交互
![](http://ac-HSNl7zbI.clouddn.com/6184zD9Mp4SvaS1srJVGcXN4H2HqDj9QXa23l43H.jpg)

可以看到启动了redis,但是这里直接输出到当前控制台了,可以通过参数配置使其后台运行.
**docker run参数**
```
-a stdin: 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；
-d: 后台运行容器，并返回容器ID；
-i: 以交互模式运行容器，通常与 -t 同时使用；
-t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
--name="nginx-lb": 为容器指定一个名称；
--dns 8.8.8.8: 指定容器使用的DNS服务器，默认和宿主一致；
--dns-search example.com: 指定容器DNS搜索域名，默认和宿主一致；
-h "mars": 指定容器的hostname；
-e username="ritchie": 设置环境变量；
--env-file=[]: 从指定文件读入环境变量；
--cpuset="0-2" or --cpuset="0,1,2": 绑定容器到指定CPU运行；
-m :设置容器使用内存最大值；
--net="bridge": 指定容器的网络连接类型，支持 bridge/host/none/Container: 四种类型；
--link=[]: 添加链接到另一个容器；
--expose=[]: 开放一个端口或一组端口；
-p 指定容器端口映射,该参数可以使得容器端口和主机端口相互映射
```

首先使用-d -p参数,可以看到redis跑在了后台.
![](http://ac-HSNl7zbI.clouddn.com/spT76EzPOxiqmpHHvUfft1bCHwQPkeqVIjJAGtCt.jpg)

**外部连接:**
使用`docker port 容器id`查看映射出来的端口,该端口为**docker主机**的哈,所以要通过docker主机ip:端口才可以访问.
比如我的docker主机ip为:192.168.99.100(使用`docker-machine env查看`),docker分配映射端口为32768,那么访问就是192.168.99.100:32768,如果想用主机地址访问的话,就需要-p参数加上主机端口映射了

**进入容器**
使用`docker exec`命令可以进入容器内部,参数和run的参数作用相同.

![](http://ac-HSNl7zbI.clouddn.com/fUQQvk3ApsvI4UbNYxO6C7tHu7d31M6v04aEhWmX.jpg)

其他命令
```
docker stop 停止一个容器
docker rm 删除一个容器
docker import 导入一个容器
docker export 导出一个容器
```


  [1]: https://cr.console.aliyun.com