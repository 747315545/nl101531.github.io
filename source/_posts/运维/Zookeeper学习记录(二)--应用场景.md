---
title: Zookeeper学习记录(二)--应用场景
tags:
  - zookeeper    
categories: 运维
date: 2017-08-19 10:54:00
---
本次学习的重点就是如下的应用场景.
### Name Service
参考博文[关于命名服务的知识点都在这里了](http://www.hollischuang.com/archives/1595),文章讲解的很详细,下面画下重点.
zk实现命名服务有两个很重要的前提
1. 其目录结构类似于文件管理形式
2. 其可以插入顺序节点
以实现自增id为例,简单代码如下
```
  private static final ZkClient ZK_CLIENT = new ZkClient(ZkClientConf.zkHost, ZkClientConf.zkTimeOut);


  private static final String nameServicePath = "/quding/nameService/name";

  /**
   * 每次拿到id都需要创建相应的节点,再删除,适合读多写少的环境下
   * @return id
   */
  public Long getUUID() {
    final String sequential = ZK_CLIENT.createPersistentSequential(nameServicePath, "id");
    ZK_CLIENT.delete(sequential);
    return Long.valueOf(sequential.substring(nameServicePath.length()));
  }
```
zk其内部维护着一个顺序,每次创建返回会自动为其加上序号如下图,那么这个顺序能保证分布式下全局的唯一性,类似于单表的数据库自增主键,但是每次创建都要连接zk,如果写>读则不会有很高的性能,读>写则适合使用zk作为id生成器.
![](http://oobu4m7ko.bkt.clouddn.com/1503113342.png?imageMogr2/thumbnail/!100p)

### 分布式锁
// todo 待更新


