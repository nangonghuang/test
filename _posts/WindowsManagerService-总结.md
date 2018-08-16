---
title: WindowsManagerService 总结
date: 2018-08-14 18:40:04
tags:
categories: Android_基础
---

Android 中的窗口是屏幕上一块用于 显示各种视图 并且可以响应用户输出的 矩形区域。从原理上来说，窗口的是独占一个Surface实例的显示区域。而WindowsManagerService则是运行在系统进程中的用于 为窗口分配Surface,控制Surface的Z-order，位置，尺寸，窗口动画等功能的 一项重要服务，并且还是输入系统的一个中转站，负责转发输入系统的消息到具体的窗口进行响应。


Android中显示出一个窗口内容主要经过以下几个步骤：
1. 获取wms服务：`IWindowManager wms = IWindowManager.Stub.asInterface(ServiceManager.getService(Context.WINDOW_SERVICE))`
2. 获取窗口的Session：`mSession = WindowManagerGlobal.getWindowSession(Looper.myLooper)`
3. 添加窗口: `wms.addWindowToken() ; mSession.add(...)`
4. 确定窗口大小，尺寸:`mSession.relayout(...)`
5. renderFrames : `mChoreographer.postCallback(renderRunnable,...)`

在后面，我们分析RootView的时候会看到这些过程


Android的显示层次可以分为三层:
1. UI框架层，这里是在surface上绘制的ui元素和相应输入事件
2. wms, 这里是管理surface的分配，层级顺序等
3. surfaceflinger , 负责将多个Suface混合并输出


## wms的重要成员
1. mInputManager  , InputManagerService的实例
2. mChoreographer  , Choreographer的实例
3. mPolicy , WindowsPolicyManager的实例，目前只有PhoneWindowManager一个实现类。负责窗口的相关策略，比如告诉wms一个窗口的z-order是多少，为wms监听屏幕的旋转状态，矫正窗口属性，预处理Home,Back按键的默认行为等
4. mTokenMap , 一个Hashmap, 保存了窗口的令牌，一个窗口必须隶属于某个令牌。其中还有一些子集，比如有些是保存的windowToken,有些是保存的AppWindowToken等，其中AppWindowToken是有序的，和AMS中的mHistory列表保持一直，反映了系统中的Activity的顺序
5. mWindowMap , 保存了 windowState 实例，用于窗口的管理，它也有一些自己，窗口在这个列表中的位置决定了窗口最终显示的Z序
6. mSessions , IWindowSession的Bn端，主要用于客户端和wms的交互