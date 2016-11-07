---
date: 2016-11-7  UTC
title: volatile 与 synchronized关键字详解
description: 在多线程并发编程中synchronized和volatile都扮演着重要的角色，volatile是轻量级的synchronized，它在多处理器开发中保证了共享变量的“可见性”。可见性的意思是当一个线程修改一个共享变量时，另一个线程就能读到这个线程修改的值。如果volatile变量修饰符使用恰当的话，它比synchronized的使用和执行成本更低，因为它不会引起线程上下文的切换和调度。
permalink: /posts/volatile/
key: 100006
labels: [Java基础]
encoding: UTF-8
---

在多线程并发编程中synchronized和volatile都扮演着重要的角色，volatile是轻量级的synchronized，它在多处理器开发中保证了共享变量的“可见性”。可见性的意思是当一个线程修改一个共享变量时，另一个线程就能读到这个线程修改的值。如果volatile变量修饰符使用恰当的话，它比synchronized的使用和执行成本更低，因为它不会引起线程上下文的切换和调度。

**volatile**
--

> **volatile定义**

Java语言规范第3版对volatile的定义如下：*Java编程语言允许线程访问共享变量，为了确保共享变量能被准确和一致的更新，线程应该确保排他锁单独获得这个变量。* Java语言提供了volatile，在某些情况下确实比锁要更加方便。如果一个字段被声明成volatile，Java线程内存模型确保所有线程看到这个变量的值是一致的。

volatile是如何保证可见性的呢？让我们在X86处理器下通过工具获取JIT编译期生成的汇编指令来查看对volatile进行写操作时，
Java代码如下：

```
instance = new Singleton();				//instance是volatile变量
```
转成汇编代码如下：

```
0x01a3deld: movb $0*0,0*1104800(%esi);
0*01a3de24: lock addl $0*0,(%esp);
```
有volatile变量修饰的共享变量进行写操作的时候会多出第二行汇编代码，通过查IA-32架构软件开发手册可知，Lock前缀的指令在多核处理器下会引发两件事情。
1）将当前处理器缓存的数据写回到系统内存。
2）这个写回内存的操作会使在其他CPU里缓存了该内存地址的数据无效。

为了提高处理速度，处理器不直接和内存进行通信，而是先将系统内存的数据读到内部缓存（L1，L2或其他）后进行操作，但操作完不知道何时会写到内存。如果对声明了volatile的变量进行写操作，JVM就会向处理器发送一条Lock前缀的指令，将这个变量所在的缓存行的数据写回到系统内存。但是，就算写回到内存，如果其他处理器缓存的值还是旧的，再执行计算操作就会有问题。所以，在多处理器下，为了保证各个处理器的缓存是一致的，就会实现缓存一致性协议，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。

> **volatile实现的两条原则**

1）**Lock前缀指令会引起处理器缓存写回到内存。**
2）**一个处理器的缓存写回到内存会导致其他处理器的缓存无效。**
处理器使用嗅探技术保证它的内部缓存、系统内存和其他处理器的缓存的数据在总线上保持一致。使用缓存一致性来确保修改的原子性，此操作被称为“缓存锁定”，缓存一致性机制会同时修改由两个以上处理器缓存的内存区域数据。

> **volatile的使用优化**

著名的Java并发编程大师Doug lea在JDK7的并发包里新增一个队列集合类LinkedTransferQueue，他在使用Volatile变量时，用一种追加字节的方式来优化队列出队和入队的性能。

追加字节能优化性能？这种方式看起来很神奇，但如果深入理解处理器架构就能理解其中的奥秘。让我们先来看看LinkedTransferQueue这个类，它使用一个内部类类型来定义队列的头队列（Head）和尾节点（tail），而这个内部类PaddedAtomicReference相对于父类AtomicReference只做了一件事情，就将共享变量追加到64字节。我们可以来计算下，一个对象的引用占4个字节，它追加了15个变量共占60个字节，再加上父类的Value变量，一共64个字节。

```
/** head of the queue */
private transient final PaddedAtomicReference < QNode > head;

/** tail of the queue */

private transient final PaddedAtomicReference < QNode > tail;


static final class PaddedAtomicReference < T > extends AtomicReference < T > {

    // enough padding for 64bytes with 4byte refs
    Object p0, p1, p2, p3, p4, p5, p6, p7, p8, p9, pa, pb, pc, pd, pe;

    PaddedAtomicReference(T r) {

        super(r);

    }

}

public class AtomicReference < V > implements java.io.Serializable {

    private volatile V value;

    //省略其他代码 ｝
```
为什么追加64字节能够提高并发编程的效率呢？ 因为对于英特尔酷睿i7，酷睿， Atom和NetBurst， Core Solo和Pentium M处理器的L1，L2或L3缓存的高速缓存行是64个字节宽，不支持部分填充缓存行，这意味着如果队列的头节点和尾节点都不足64字节的话，处理器会将它们都读到同一个高速缓存行中，在多处理器下每个处理器都会缓存同样的头尾节点，当一个处理器试图修改头接点时会将整个缓存行锁定，那么在缓存一致性机制的作用下，会导致其他处理器不能访问自己高速缓存中的尾节点，而队列的入队和出队操作是需要不停修改头接点和尾节点，所以在多处理器的情况下将会严重影响到队列的入队和出队效率。Doug lea使用追加到64字节的方式来填满高速缓冲区的缓存行，避免头接点和尾节点加载到同一个缓存行，使得头尾节点在修改时不会互相锁定。

那么是不是在使用Volatile变量时都应该追加到64字节呢？不是的。在两种场景下不应该使用这种方式。第一：缓存行非64字节宽的处理器，如P6系列和奔腾处理器，它们的L1和L2高速缓存行是32个字节宽。第二：共享变量不会被频繁的写。因为使用追加字节的方式需要处理器读取更多的字节到高速缓冲区，这本身就会带来一定的性能消耗，共享变量如果不被频繁写的话，锁的几率也非常小，就没必要通过追加字节的方式来避免相互锁定。

**synchronized**
--
synchronize实现同步的基础：Java中的每个对象都可以作为锁。具体表现为以下3种形式。
1）对于普通同步方法，锁是当前实例对象。
2）对于静态同步方法，锁是当前类的Class对象。
3）对于同步方法快，锁是synchronized括号里配置的对象。

从JVM规范中可以看到synchronized在JVM里的实现原理，JVM基于进入和退出Monitor对象来实现方法同步和代码块同步，但两者的实现细节不一样。代码块同步是使用monitorenter和monitorexit指令实现的，而方法同步是使用另一种方法实现的，细节在JVM规范中并没有详细说明。但是，方法的同步同样可以使用这个两个指令实现。



参考文献：Java并发编程的艺术  方腾飞著
