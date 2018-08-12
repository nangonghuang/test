---
title: 《java并发编程的艺术》outline
date: 2018-08-07 11:30:58
tags:
categories: Java
---

最近几天把618在京东买的《java并发编程的艺术》看完了，感觉挺不错的，很多东西都是第一次看到这么详细的内容，可以和《深入理解Java虚拟机(第2版)》配合看，推荐先看后面这一本<!--more-->


## Java并发编程的限制
1. 上下文切换
2. 死锁
3. 资源限制

## 底层实现原理

### volatile
1. 插入了汇编的lock指令，由处理器实现

### synchronized
1. 锁对象： 普通同步方法的锁是当前实例，静态同步方法的锁是当前类的class对象
2. jvm使用Monitor对象来实现锁，包括 monitorenter 和 monitorexit 指令
3. 锁的升级和对比：
    * 无锁状态
    * 偏向锁：java对象头和栈帧简单记录线程id，对比确定线程锁状态
    * 轻量级锁 
    * 重量级锁

### 原子操作的实现原理
* 处理器实现原子操作:
    * 总线锁定
    * 缓存锁定
* Java实现原子操作: CAS，CAS的底层实现是处理器提供的指令

## Java内存模型
(这部分很多地方都有了)
Java中，实例域，静态域和数组都存在堆内存中，堆内存在线程之间共享。局部变量不会共享，不存在内存可见性问题。

内存模型抽象图：
线程，线程本地内存，主存 结构图

Java涉及的重排序：
1. 编译器优化的重排序
2. 指令级并行的重排序
3. 内存系统的重排序：主要是内存的加载和存储的顺序

内存屏障：  
```
load/store  
code 
load/store
```
确保在执行code的前后，先读取主存中的最新值/存储最新值到主存

volatile内存语义：
* 可见性 ： 对于一个volatile变量的读，总是能看到任意线程堆这个volatile变量的最后的写入。通过插入内存屏障来实现，从结局上看是限制了指令的重排序。
    * 写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值刷新到主存
    * 读一个volatile变量时，JMM会把该线程对应的本地内存置为无效，线程将从主存中读取变量的值
* 原子性 ： 对于一个voloatile变量的读/写具有原子性，但是对于volatile++ 这种复合操作不保证原子性

CAS ： CAS操作同时具有volatile读和写的内存语义，这个是通过java底层代码获取的编译器的功能

final 变量的内存语义：
* 构造函数内对final域写入，和随后把这个对象的引用赋值给变量，这两个操作不能重排序
* 读一个包含final域的对象的引用，和随后初次读这个final域的值，这两个操作不能重排序

happens-before 的定义和规则:

双重检查锁使用volatile变量的原因: 为对象分配内存和引用与对象的初始化之间可能会有重排序，导致判断空值出错，加volatile来防止这种错误

## Java并发基础
终止线程：
1. 中断操作
2. 使用布尔变量

线程通信：
* volatile和synchronized关键字
* wait/notify
* pipe 输入输出流
* Thread.join()

## Java锁
AQS ：同步队列器的原理和使用：
* 维护一个同步队列
* 独占式同步状态获取和释放
* 共享式同步状态获取和释放

重入锁的实现:

读写锁的实现：

Condition接口：

## Java并发容器和框架
* ConcurrentHashmap的实现:
* ConcurrtntLinkedQueue的实现:
* 7种阻塞队列：  ， 阻塞队列的实现： 基于Condition的等待通知模式
* Fork/Join 框架

## 原子操作类
共有13个：
```
AtomicXXX:
```

## Java并发工具类
* CountDownLatch
* CyclicBarrier: 一个线程到达屏障时被阻塞，等最后一个线程到达才打开屏障，继续运行
* Semaphore : 
* Exchanger :

## 线程池 

## Executor框架和 FutureTask
Runnable和Callable：

FutureTask: 目前Future接口的唯一实现类，可用于异步任务的取消，获取返回值等