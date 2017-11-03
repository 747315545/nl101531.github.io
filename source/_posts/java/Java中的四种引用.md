---
title: Java中的四种引用
tags:
  - java    
categories: java
date: 2017-10-30 22:51:00
updated: 2017-10-30 22:51:01
---

Java中存在四种引用,StrongReference(强引用) 、SoftReferenc(软引用) 、WeakReferenc(弱引用)、PhantomReference(虚引用).虽然不常用,但是对于理解Java的回收等级还是很有帮助的,一句话来说这些引用只是不同回收等级的一种表现形式.
![](http://oobu4m7ko.bkt.clouddn.com/1509454563.png)
- - - - -
### StrongReference(强引用)
强引用是最经常使用的一种引用,如new操作创建的对象就属于强引用.如下代码,对于强引用要记住**无论如何JVM都不会去回收其内存**.
```java
Object obj = new Object();
```
### SoftReferenc(软引用)
软引用是由`java.lang.ref.SoftReference`所提供的功能,被其所关联的对象不存在强引用并且此时JVM内存不足才会去回收该对象.
个人不知道其用处,做缓存的话,现在的企业项目基本不是单体架构所以用处不大,倒是可以做内存警告,当对象被回收时则说明系统所需要的内存不足,那么就可以发邮件通知相关人员.

### WeakReferenc(弱引用)
弱引用是java.lang.ref包下的WeakReferenc类所提供的包装功能,对于弱引用**JVM会回收仅被弱引用所关联的对象**.也就是说弱引用对象会在一次gc之后被回收,如下代码,其中`obj1`没被回收,因为其的引用是强引用,但是`weakObj1`与其关联是弱引用,因此不属于被收回对象.`weakObj2`所关联的`new Object()`只有一个弱引用关联,因此会被回收.
```java
    Object obj1 = new Object();
    WeakReference<Object> weakObj1 = new WeakReference<Object>(obj1);
    WeakReference<Object> weakObj2 = new WeakReference<Object>(new Object());
    //主动回收
    System.gc();

    System.out.println(weakObj1.get()); // 非null
    System.out.println(weakObj2.get()); // null
```
Java中提供了一个很棒的工具类`WeakHashMap`,按照注释所说,该类是一个键为弱引用类型的Map,与传统Map不同的是其键会自动删除释放掉,因为gc()时会自动释放,因此很适合做缓存这一类的需求,下面代码是Tomcat所实现的LRU(最少使用策略)缓存算法的实现,关键点在注释中给出.
```java
import java.util.Map;
import java.util.WeakHashMap;
import java.util.concurrent.ConcurrentHashMap;

public final class ConcurrentCache<K,V> {
    //LRU所允许的最大缓存量    
    private final int size;
    private final Map<K,V> eden;
    private final Map<K,V> longterm;
    
    public ConcurrentCache(int size) {
        this.size = size;
        //eden是主要缓存
        this.eden = new ConcurrentHashMap<>(size);
        //longterm是实现LRU算法的关键点.
        this.longterm = new WeakHashMap<>(size);
    }
    
    //get是先从eden中取出缓存,当不存在时则去longterm中获取缓存,并且此时获取到的缓存说明还在使用,因此会put到eden中(LRU算法)
    public V get(K k) {
        V v = this.eden.get(k);
        if (v == null) {
            synchronized (longterm) {
                v = this.longterm.get(k);
            }
            if (v != null) {
                this.eden.put(k, v);
            }
        }
        return v;
    }
    //put操作当size大于LRU最大容量时,则把缓存都放入到longterm,当this.eden.clear()后使其成为弱引用,那么LRU的实现则在get方法中体现了出来.
    public void put(K k, V v) {
        if (this.eden.size() >= size) {
            synchronized (longterm) {
                this.longterm.putAll(this.eden);
            }
            this.eden.clear();
        }
        this.eden.put(k, v);
    }
}
```
此方法如果操作时刚好遇到了一次gc,那么longterm的引用就会丢失,那么缓存就gg了.

### PhantomReference(虚引用)
虚引用是由`java.lang.ref.PhantomReference`所提供的关联功能,**虚引用对其原对象的生命周期毫无影响**,其可以算是一种标记,当其所引用对象被回收时其会自动加入到引用队列中.也就是说你可以通过虚引用得到哪些对象已被回收.具体用法可以分析`common.io`中的`org.apache.commons.io.FileCleaningTracker`
该类中有一内部类`class Tracker extends PhantomReference<Object>`,也就是其包裹着虚引用对象,分析其构造函数,`marker`参数是该具体的虚引用,当marker被回收时,该对应的Track会被加入到引用队列`queue`中.
```java
        Tracker(String path, FileDeleteStrategy deleteStrategy, Object marker, ReferenceQueue<? super Object> queue) {
            //marker是具体的虚引用对象
            super(marker, queue);
            this.path = path;
            this.deleteStrategy = deleteStrategy == null ? FileDeleteStrategy.NORMAL : deleteStrategy;
        }
```
文件删除则是该类维护的一个线程来进行的操作,既然对象回收后会加入到引用队列`queue`,那么该线程要做的功能自然是从引用队列中获取到对应的`Track`,然后执行其删除策略.
在这个流程中虚引用起到的是跟踪所包裹对象作用,当包裹的的对象被回收时,这边会得到一个通知(将其加入到引用队列).
```java
@Override
        public void run() {
            // thread exits when exitWhenFinished is true and there are no more tracked objects
            while (exitWhenFinished == false || trackers.size() > 0) {
                try {
                    // Wait for a tracker to remove.
                    Tracker tracker = (Tracker) q.remove(); // cannot return null
                    trackers.remove(tracker);
                    if (!tracker.delete()) {
                        deleteFailures.add(tracker.getPath());
                    }
                    tracker.clear();
                } catch (InterruptedException e) {
                    continue;
                }
            }
        }
```

### 参考文章
[理解Java中的弱引用](http://droidyue.com/blog/2014/10/12/understanding-weakreference-in-java/index.html)