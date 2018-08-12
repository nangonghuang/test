---
title: RxJava学习记录1
date: 2017-12-11 13:50:48
tags:
categories: Android_基础
---
之前就已经听说过这个了，一直没有引入到项目中去，然后看过就忘了，所以现在准备重新捡起来。

首先是gradle设置
```
 implementation 'io.reactivex.rxjava2:rxjava:2.1.0'
 implementation 'io.reactivex.rxjava2:rxandroid:2.0.1'
```
然后是<!--more-->
## 基本操作
```

        Observable<String> observable = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> e) throws Exception {
                e.onNext("123");
                e.onNext("456");
                e.onComplete();
            }
        });

        Observer<String> observer = new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.i(TAG, "onSubscribe: ");
            }

            @Override
            public void onNext(String s) {
                Log.i(TAG, "onNext: " + s);
            }

            @Override
            public void onError(Throwable e) {
                Log.i(TAG, "onError: ");
            }

            @Override
            public void onComplete() {
                Log.i(TAG, "onComplete: ");
            }
        };
        observable.subscribe(observer);
```
这里面涉及到的东西也就两个，一个是创建被观察者，重写subscribe ，在里面写一些自定义的动作，里面的参数是发给观察者的。
然后是创建观察者，被观察者调用订阅方法拿到观察者的引用，之后会按照被观察者的流程调用相应的方法
输出是
```
12-11 06:52:14.454 3517-3517/com.example.alan.rxpractice I/Activity1: onSubscribe: 
12-11 06:52:14.454 3517-3517/com.example.alan.rxpractice I/Activity1: onNext: 123
12-11 06:52:14.454 3517-3517/com.example.alan.rxpractice I/Activity1: onNext: 456
12-11 06:52:14.454 3517-3517/com.example.alan.rxpractice I/Activity1: onComplete: 
```


Observable    
Observable的其他几种创建方法：
1. 使用Observable.just()创建被观察者
```
Observable<String> just = Observable.just("123", "456");
```
2. 使用Observable.from()创建被观察者
```
String [] words = {"123", "456"};
Observable observable3 = Observable.from(words);
```
或者
```
List<String> list = new ArrayList<String>();
list.add("123");
list.add("456");
Observable observable4 = Observable.from(list);
```
效果相同

## 线程控制 —— Scheduler
在RxJava 中，Scheduler ——调度器，相当于线程控制器，RxJava 通过它来指定每一段代码应该运行在什么样的线程。RxJava 已经内置了几个 Scheduler ，它们已经适合大多数的使用场景：

    Schedulers.immediate(): 直接在当前线程运行，相当于不指定线程。这是默认的 Scheduler。
    Schedulers.newThread(): 总是启用新线程，并在新线程执行操作。
    Schedulers.io(): I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 Scheduler。行为模式和 newThread() 差不多，区别在于 io() 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 io() 比 newThread() 更有效率。不要把计算工作放在 io() 中，可以避免创建不必要的线程。
    Schedulers.computation(): 计算所使用的 Scheduler。这个计算指的是 CPU 密集型计算，即不会被 I/O 等操作限制性能的操作，例如图形的计算。这个 Scheduler 使用的固定的线程池，大小为 CPU 核数。不要把 I/O 操作放在 computation() 中，否则 I/O 操作的等待时间会浪费 CPU。
    另外， Android 还有一个专用的 AndroidSchedulers.mainThread()，它指定的操作将在 Android 主线程运行。

有了这几个 Scheduler ，就可以使用 subscribeOn() 和 observeOn() 两个方法来对线程进行控制了。 * subscribeOn(): 指定 subscribe() 所发生的线程，即 Observable.OnSubscribe 被激活时所处的线程。或者叫做事件产生的线程。 * observeOn(): 指定 Subscriber 所运行在的线程。或者叫做事件消费的线程。

## 操作符分类

按照官方(http://reactivex.io/)的分类，操作符大致分为以下几种：

    Creating Observables(Observable的创建操作符)，比如：Observable.create()、Observable.just()、Observable.from()等等；

    Transforming Observables(Observable的转换操作符)，比如：observable.map()、observable.flatMap()、
    observable.buffer()等等；

    Filtering Observables(Observable的过滤操作符)，比如：observable.filter()、observable.sample()、observable.take()等等；

    Combining Observables(Observable的组合操作符)，比如：observable.join()、observable.merge()、observable.combineLatest()等等；

    Error Handling Operators(Observable的错误处理操作符)，比如:observable.onErrorResumeNext()、observable.retry()等等；

    Observable Utility Operators(Observable的功能性操作符)，比如：observable.subscribeOn()、observable.observeOn()、observable.delay()等等；

    Conditional and Boolean Operators(Observable的条件操作符)，比如：observable.amb()、observable.contains()、observable.skipUntil()等等；

    Mathematical and Aggregate Operators(Observable数学运算及聚合操作符)，比如：observable.count()、observable.reduce()、observable.concat()等等；
    
    其他如observable.toList()、observable.connect()、observable.publish()等等；

感谢：[给Android开发者的RxJava详解](http://gank.io/post/560e15be2dca930e00da1083#toc_1)
感谢：[RxJava 与 Retrofit 结合的最佳实践](http://gank.io/post/56e80c2c677659311bed9841)
感谢：[RxJava使用介绍（二） RxJava的操作符](http://blog.csdn.net/job_hesc/article/details/46242117)
