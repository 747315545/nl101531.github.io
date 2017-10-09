---
title: redis学习记录(三)-redis中的数据结构
tags:
  - redis
categories: redis
date: 2017-03-26 21:50:00
updated:  2017-03-26 21:50:00
---

# redis学习记录(三)-redis中的数据结构

标签（空格分隔）： redis

---

[Redis学习记录(一)--入门知识][1]o
[Redis学习记录(二)--使用Jedis连接][2]

redis有五种数据类型,string,list,set,hash,sort set,不同场景使用不同数据结构的前提是了解每一种数据结构.那么结构图片是最佳的了解方式,图片来自慕课网.

### string
string是的key->value类型的存储,可以存储字符串类型和数值类型,可对数值类型是可以增加减少,对string类型可以追加内容.

![](http://ac-HSNl7zbI.clouddn.com/FLiCD6qvq5cgURt9pK4GsmHAfikXMkTaTk0jH42D.jpg)


### list
list是列表,也就是一个key->多个value,可以支持双端队列,栈来操作,因此越靠近两端其查找速度越快,端点的复杂度查找为O(1),同时队列有阻塞操作,也就是可以当成阻塞队列使用.

![](http://ac-HSNl7zbI.clouddn.com/C66VGUQUQU36HG1VirdktiwGCsuch427TKeM9Mx8.jpg)


### set
set为无序,且不重复的集合,且提供O(1)复杂度度的快速查找.set集合支持集合的并,交,差操作,因为无序性,因此也提供迭代方法.

![](http://ac-HSNl7zbI.clouddn.com/eYnj2QzNgnS3IxzfR74ODXYREXCAwLrFAqmNCvDD.jpg)


### hash
hash类型适合存储对象,相比前面的string,所带来的优势是可以使用一个key查出该下面所有的键值对,并且可以单独对某一属性更改,如图所示:

![](http://ac-HSNl7zbI.clouddn.com/9EqPsIkU1iQeoMzRBYGGIEIPYvpQarWbgyADBehR.jpg)


### sort set
可排序的集合,如图所示存在score排名分数,隐藏属性rank排名,0为最小.注意对sort set来说value是唯一性的,而不是score,如果两个score相同,则按照value的字典序排序.

![](http://ac-HSNl7zbI.clouddn.com/3TBmsR4b8tGDoUvX3tlo1pHMuTTDnJOXAT5H0S9g.jpg)


### 应用场景

#### string
计数器:redis的incr操作是原子性的,因此可以应对高并发,如网站要求每个用户获取验证码后60秒内不得再次获取,那么第一次获取的时候用incr给该用户设置key,过期时间为60秒,如果结果等于1则为第一次请求,那么第二次获取时比较决定是否操作频繁.

#### List
list的优势的有序性,两端插入复杂度为O(1),那么对于**最近文章列表**等类似需求是最佳解决方案,维护一个定长的列表,每次插入后执行trim操作.

#### Set
set的优势是唯一性,O(1)的查找查找复杂度,并且支持差并集,那么二度好友问题就迎刃而解了.

#### hash
hash非常适合存储对象,不同的键为对象的特征,值为特征值,那么比string好的就是修改不需要每次都修改一个整串,而可以选择修改某一指定键值.

#### sort set
sort set可排序特性使其很容易解决排行榜类应用,但是要注意值需要存储不变的属性,因为值要求唯一性,score可不唯一.

  [1]: http://www.jianshu.com/p/da69edda2a43
  [2]: http://www.jianshu.com/p/c3b8180af944p/da69edda2a43