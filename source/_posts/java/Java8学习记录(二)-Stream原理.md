---
title: Java8学习记录(二)-Stream原理
tags:
  -  java8
categories:  java 
date: 2017-05-20 19:06:51
---
推荐一篇博文,很好的介绍了Stream的原理.本文对其进行一些补充更加详细的讲解.
> 作者: 李豪
> 地址: https://github.com/CarpenterLee/JavaLambdaInternals/blob/master/6-Stream%20Pipelines.md

### 操作如何记录?
操作的记录用的是双向链表,那么这个链表是怎么建立的呢?
类: `AbstractPipeline`是整个调用链建立的核心类,其含有成员变量`sourceStage(调用起始点)`,`previousStage(上一个调用)`,`nextStage(下一个调用)`,再看其构造函数
```java
   AbstractPipeline(AbstractPipeline<?, E_IN, ?> previousStage, int opFlags) {
        if (previousStage.linkedOrConsumed)
            throw new IllegalStateException(MSG_STREAM_LINKED);
       //previousStage为上一次调用,那么他的下一次调用则是当前的this
        previousStage.linkedOrConsumed = true;
        previousStage.nextStage = this;
        //当前this的上一次调用为previousStage
        this.previousStage = previousStage;
        this.sourceOrOpFlags = opFlags & StreamOpFlag.OP_MASK;
        this.combinedFlags = StreamOpFlag.combineOpFlags(opFlags, previousStage.combinedFlags);
        //调用起始点都是同一个
        this.sourceStage = previousStage.sourceStage;
        if (opIsStateful())
            sourceStage.sourceAnyStateful = true;
        this.depth = previousStage.depth + 1;
    }
```
每一次操作都会建立一个新的Stream,传入上一个Stream,调用上方构造函数初始化,进而形成双向链表.
```java
   StatelessOp(AbstractPipeline<?, E_IN, ?> upstream,
                    StreamShape inputShape,
                    int opFlags) {
            super(upstream, opFlags);
            assert upstream.getOutputShape() == inputShape;
        }
```
### 操作如何叠加
如原文所说,以filter方法为例,调用该方法会产生一个StatelessOp对象,也就是AbstractPipeline的子类,其方法`opWrapSink`很关键,产生一个`Sink`对象,在自身`accept(t)`方法处理自身逻辑,得到的结果传递给下游的stream`downstream.accept(u);`,也就是博主所说的**处理/转发模式**
```java
  @Override
    public final Stream<P_OUT> filter(Predicate<? super P_OUT> predicate) {
        Objects.requireNonNull(predicate);
        return new StatelessOp<P_OUT, P_OUT>(this, StreamShape.REFERENCE,
                                     StreamOpFlag.NOT_SIZED) {
            @Override
            Sink<P_OUT> opWrapSink(int flags, Sink<P_OUT> sink) {
                return new Sink.ChainedReference<P_OUT, P_OUT>(sink) {
                    @Override
                    public void begin(long size) {
                        downstream.begin(-1);
                    }
                    @Override
                    public void accept(P_OUT u) {
                        if (predicate.test(u))
                            downstream.accept(u);
                    }
                };
            }
        };
    }
```

### 叠加之后的操作如何执行
首先操作都在Sink中,操作的叠加已经没问题了,那问题就是怎么把这些操作连起来,形成一个单项的调用链.如下图所示,我们所期望的调用关系是从filter.sink开始的单链表.但是当操作到`sorted()`的时候我们只能得到`sort.sink`,无法得知上游信息,所以这个单链表的建立还需要其他方案.
![](http://oobu4m7ko.bkt.clouddn.com/1495593370.png?imageMogr2/thumbnail/!70p)
该调用单链表的建立是使用`wrapSink`方法.
```java
   final <P_IN> Sink<P_IN> wrapSink(Sink<E_OUT> sink) {
        Objects.requireNonNull(sink);
        //该循环体,从终端stage往前迭代,到最后形成的sink对象就是filter.sink
        for ( @SuppressWarnings("rawtypes") AbstractPipeline p=AbstractPipeline.this; p.depth > 0; p=p.previousStage) {
            sink = p.opWrapSink(p.previousStage.combinedFlags, sink);
        }
        return (Sink<P_IN>) sink;
    }
```
该方法从终端操作往前迭代,最终形成的sink就是如单链表结构,此时的sink对象就是filter.sink,并且其还指向map.sink.

现在拿到了调用链式,接下来就是想办法在一次循环遍历中执行全部操作.
```java
    @Override
    final <P_IN> void copyInto(Sink<P_IN> wrappedSink, Spliterator<P_IN> spliterator) {
        Objects.requireNonNull(wrappedSink);

        if (!StreamOpFlag.SHORT_CIRCUIT.isKnown(getStreamAndOpFlags())) {
            //调用begin方法准备容器
            wrappedSink.begin(spliterator.getExactSizeIfKnown());
            //对sink迭代
            spliterator.forEachRemaining(wrappedSink);
            //终端操作
            wrappedSink.end();
        }
        else {
            copyIntoWithCancel(wrappedSink, spliterator);
        }
    }
```
此时的wrappedSink对象是一个完整的调用链,那么对于` spliterator.forEachRemaining(wrappedSink);`的执行就是对每一个元素在调用链上走一遍的流程.这三行代码可以用下图表示
begin
![](http://oobu4m7ko.bkt.clouddn.com/1495594524.png?imageMogr2/thumbnail/!70p)
accept
![](http://oobu4m7ko.bkt.clouddn.com/1495594311.png?imageMogr2/thumbnail/!70p)
end
![](http://oobu4m7ko.bkt.clouddn.com/1495594563.png?imageMogr2/thumbnail/!70p)

### 执行后的结果在哪里
即终端操作所提供的容器中.







