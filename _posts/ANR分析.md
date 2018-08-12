---
title: ANR分析
date: 2017-11-24 10:28:58
tags:
categories: Android_性能优化
---

[来源](http://blog.csdn.net/dadoneo/article/details/8270107),在这里看到的一些总结，觉得有些收获

<!--more-->

## ANR类型

ANR一般有三种类型：

1. KeyDispatchTimeout(5 seconds) --**主要类型**

   按键或触摸事件在特定时间内无响应

2. BroadcastTimeout(10 seconds)

   BroadcastReceiver在特定时间内无法处理完成

3. ServiceTimeout(20 seconds) --**小概率类型**

   Service在特定的时间内无法处理完成

## 如何去分析ANR

先看个LOG:

04-01 13:12:11.572** I/InputDispatcher( 220): Application is not responding**:Window{2b263310com.android.email/com.android.email.activity.SplitScreenActivitypaused=false}.  5009.8ms since event, 5009.5ms since waitstarted

04-0113:12:11.572 I/WindowManager( 220): Input event dispatching timedout sending tocom.android.email/com.android.email.activity.SplitScreenActivity

04-01 **13:12:14.123 I/Process(  220): Sending signal. PID: 21404 SIG: 3---**发生**ANR**的时间和生成**trace.txt**的时间

04-01 13:12:14.123 I/dalvikvm(21404):threadid=4: reacting to signal 3 

……

04-0113:12:15.872 E/ActivityManager(  220): ANR in com.android.email(com.android.email/.activity.SplitScreenActivity)

04-0113:12:15.872 E/ActivityManager(  220): Reason:keyDispatchingTimedOut

04-0113:12:15.872 E/ActivityManager(  220): Load: 8.68 / 8.37 / 8.53

04-0113:12:15.872 E/ActivityManager(  220): **CPUusage from 4361ms to 699ms ago** ----CPU在ANR发生前的使用情况

04-0113:12:15.872 E/ActivityManager(  220):   5.5%21404/com.android.email: 1.3% user + 4.1% kernel / faults: 10 minor

04-0113:12:15.872 E/ActivityManager(  220):   4.3%220/system_server: 2.7% user + 1.5% kernel / faults: 11 minor 2 major

04-0113:12:15.872 E/ActivityManager(  220):   0.9%52/spi_qsd.0: 0% user + 0.9% kernel

04-0113:12:15.872 E/ActivityManager(  220):   0.5%65/irq/170-cyttsp-: 0% user + 0.5% kernel

04-0113:12:15.872 E/ActivityManager(  220):   0.5%296/com.android.systemui: 0.5% user + 0% kernel

04-0113:12:15.872 E/ActivityManager(  220): **100%TOTAL: 4.8% user + 7.6% kernel + 87% iowait**

04-0113:12:15.872 E/ActivityManager(  220): **CPUusage from 3697ms to 4223ms later**:-- ANR后CPU的使用量

04-0113:12:15.872 E/ActivityManager(  220):   25%21404/com.android.email: 25% user + 0% kernel / faults: 191 minor

04-0113:12:15.872 E/ActivityManager(  220):    16% 21603/__eas(par.hakan: 16% user + 0% kernel

04-0113:12:15.872 E/ActivityManager(  220):    7.2% 21406/GC: 7.2% user + 0% kernel

04-0113:12:15.872 E/ActivityManager(  220):    1.8% 21409/Compiler: 1.8% user + 0% kernel

04-0113:12:15.872 E/ActivityManager(  220):   5.5%220/system_server: 0% user + 5.5% kernel / faults: 1 minor

04-0113:12:15.872 E/ActivityManager(  220):    5.5% 263/InputDispatcher: 0% user + 5.5% kernel

04-0113:12:15.872 E/ActivityManager(  220): **32%TOTAL: 28% user + 3.7% kernel**

从LOG可以看出ANR的类型，CPU的使用情况，如果CPU使用量接近100%，说明当前设备很忙，有可能是CPU饥饿导致了ANR

如果CPU使用量很少，说明主线程被BLOCK了

如果IOwait很高，说明ANR有可能是主线程在进行I/O操作造成的

除了看LOG，解决ANR还得需要trace.txt文件，data/anr/traces.txt 

## 如何调查并解决ANR

1. 首先分析log，根据Log里面ANR发生的时间去trace.txt文件里面查到对应的段落
2. 从trace.txt文件对应的段落里查看调用stack.
3. 看代码
4. 仔细查看ANR的成因（iowait?block?memoryleak?）



例子：

打开log文件 ， 由于是ANR错误，因此搜索"ANR " ， 为何要加空格呢，你加上和去掉比较一下就知道了 。 可以屏蔽掉不少保存到anr.log文件的无效信息。



定位到关键的事件信息如下：

01-15 16:49:02.433 E/ActivityManager( 2466): ANR in com.android.mms (com.android.mms/.ui.SlideshowActivity)

01-15 16:49:02.433 E/ActivityManager( 2466): Reason: keyDispatchingTimedOut

01-15 16:49:02.433 E/ActivityManager( 2466): Load: 0.6 / 0.61 / 0.42

01-15 16:49:02.433 E/ActivityManager( 2466): CPU usage from 1337225ms to 57ms ago:

01-15 16:49:02.433 E/ActivityManager( 2466):   sensorserver_ya: 8% = 0% user + 8%
 kernel / faults: 40 minor

......





01-15 16:49:02.433 E/ActivityManager( 2466):  -com.android.mms: 0% = 0% user + 0%
 kernel

01-15 16:49:02.433 E/ActivityManager( 2466):  -flush-179:8: 0% = 0% user + 0% kernel

01-15 16:49:02.433 E/ActivityManager( 2466): TOTAL: 25% = 10% user + 14% kernel +
 0% iowait + 0% irq + 0% softirq

01-15 16:49:02.436 I/        ( 2466): dumpmesg > "/data/log/dumpstate_app_anr.log"





我们用自然语言来描述一下日志，这也算是一种能力吧 。 

01-15 16:49:02.433 E/ActivityManager( 2466): ANR in com.android.mms (com.android.mms/.ui.SlideshowActivity)

翻译：在16:49分2秒433毫秒的时候 ActivityManager （进程号为2466) 发生了如下错误：com.android.mms包下面的.ui.SlideshowActivity
 无响应 。



01-15 16:49:02.433 E/ActivityManager( 2466): Reason: keyDispatchingTimedOut

翻译：原因 ， keyDispatchingTimeOut - 按键分配超时 



01-15 16:49:02.433 E/ActivityManager( 2466): Load: 0.6 / 0.61 / 0.42

翻译：5分钟，10分钟，15分钟内的平均负载分别为：0.6 , 0.61 , 0.42



在这里我们大概知道问题是什么了，结合我们之前的操作流程，我们知道问题是在点击按钮某时候可能处理不过来按钮事件，导致超时无响应 。那么现在似乎已经可以进行工作了 。
 我们知道Activity中是通过重载dispatchTouchEvent(MotionEvent ev)来处理点击屏幕事件  。 然后我们可以顺藤摸瓜，一点点分析去查找原因 。 但这样够了么 ？

其实不够 ， 至少我们不能准确的知道到底问题在哪儿 ， 只是猜测 ，比如这个应用程序中，我就在顺藤摸瓜的时候发现了多个IO操作的地方都在主线程中，可能引起问题，但不好判断到底是哪个  ，所以我们目前掌握的信息还不够
 。 



于是我们再分析虚拟机信息 ， 搜索“Dalvik Thread”关键词，快速定位到本应用程序的虚拟机信息日志，如下：

----- pid 2922 at 2011-01-13 13:51:07 -----

Cmd line: com.android.mms



DALVIK THREADS:

"main" prio=5 tid=1 NATIVE

  | group="main" sCount=1 dsCount=0 s=N obj=0x4001d8d0 self=0xccc8

  | sysTid=2922 nice=0 sched=0/0 cgrp=default handle=-1345017808

  | schedstat=( 3497492306 15312897923 10358 )

  at android.media.MediaPlayer._release(Native Method)

  at android.media.MediaPlayer.release(MediaPlayer.java:1206)

  at android.widget.VideoView.stopPlayback(VideoView.java:196)

  at com.android.mms.ui.SlideView.stopVideo(SlideView.java:640)

  at com.android.mms.ui.SlideshowPresenter.presentVideo(SlideshowPresenter.java:443)

  at com.android.mms.ui.SlideshowPresenter.presentRegionMedia(SlideshowPresenter.java:219)

  at com.android.mms.ui.SlideshowPresenter$4.run(SlideshowPresenter.java:516)

  at android.os.Handler.handleCallback(Handler.java:587)

  at android.os.Handler.dispatchMessage(Handler.java:92)

  at android.os.Looper.loop(Looper.java:123)

  at android.app.ActivityThread.main(ActivityThread.java:4627)

  at java.lang.reflect.Method.invokeNative(Native Method)

  at java.lang.reflect.Method.invoke(Method.java:521)

  at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:858)

  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:616)

  at dalvik.system.NativeStart.main(Native Method)



"Binder Thread #3" prio=5 tid=11 NATIVE

  | group="main" sCount=1 dsCount=0 s=N obj=0x4837f808 self=0x242280

  | sysTid=3239 nice=0 sched=0/0 cgrp=default handle=2341032

  | schedstat=( 32410506 932842514 164 )

  at dalvik.system.NativeStart.run(Native Method)



"AsyncQueryWorker" prio=5 tid=9 WAIT

  | group="main" sCount=1 dsCount=0 s=N obj=0x482f4b80 self=0x253e10

  | sysTid=3236 nice=0 sched=0/0 cgrp=default handle=2432120

  | schedstat=( 3225061 26561350 27 )

  at java.lang.Object.wait(Native Method)

  - waiting on <0x482f4da8> (a android.os.MessageQueue)

  at java.lang.Object.wait(Object.java:288)

  at android.os.MessageQueue.next(MessageQueue.java:146)

  at android.os.Looper.loop(Looper.java:110)

  at android.os.HandlerThread.run(HandlerThread.java:60)



"Thread-9" prio=5 tid=8 WAIT

  | group="main" sCount=1 dsCount=0 s=N obj=0x4836e2b0 self=0x25af70

  | sysTid=2929 nice=0 sched=0/0 cgrp=default handle=2370896

  | schedstat=( 130248 4389035 2 )

  at java.lang.Object.wait(Native Method)

  - waiting on <0x4836e240> (a java.util.ArrayList)

  at java.lang.Object.wait(Object.java:288)

  at com.android.mms.data.Contact$ContactsCache$TaskStack$1.run(Contact.java:488)

  at java.lang.Thread.run(Thread.java:1096)



"Binder Thread #2" prio=5 tid=7 NATIVE

  | group="main" sCount=1 dsCount=0 s=N obj=0x482f8ca0 self=0x130fd0

  | sysTid=2928 nice=0 sched=0/0 cgrp=default handle=1215968

  | schedstat=( 40610049 1837703846 195 )

  at dalvik.system.NativeStart.run(Native Method)



"Binder Thread #1" prio=5 tid=6 NATIVE

  | group="main" sCount=1 dsCount=0 s=N obj=0x482f4a78 self=0x128a50

  | sysTid=2927 nice=0 sched=0/0 cgrp=default handle=1201352

  | schedstat=( 40928066 928867585 190 )

  at dalvik.system.NativeStart.run(Native Method)



"Compiler" daemon prio=5 tid=5 VMWAIT

  | group="system" sCount=1 dsCount=0 s=N obj=0x482f1348 self=0x118960

  | sysTid=2926 nice=0 sched=0/0 cgrp=default handle=1149216

  | schedstat=( 753021350 3774113668 6686 )

  at dalvik.system.NativeStart.run(Native Method)



"JDWP" daemon prio=5 tid=4 VMWAIT

  | group="system" sCount=1 dsCount=0 s=N obj=0x482f12a0 self=0x132940

  | sysTid=2925 nice=0 sched=0/0 cgrp=default handle=1255680

  | schedstat=( 2827103 29553323 19 )

  at dalvik.system.NativeStart.run(Native Method)



"Signal Catcher" daemon prio=5 tid=3 RUNNABLE

  | group="system" sCount=0 dsCount=0 s=N obj=0x482f11e8 self=0x135988

  | sysTid=2924 nice=0 sched=0/0 cgrp=default handle=1173688

  | schedstat=( 11793815 12456169 7 )

  at dalvik.system.NativeStart.run(Native Method)



"HeapWorker" daemon prio=5 tid=2 VMWAIT

  | group="system" sCount=1 dsCount=0 s=N obj=0x45496028 self=0x135848

  | sysTid=2923 nice=0 sched=0/0 cgrp=default handle=1222608

  | schedstat=( 79049792 1520840200 95 )

  at dalvik.system.NativeStart.run(Native Method)



----- end 2922 -----



每一段都是一个线程 ，当然我们还是看线程号为1的主线程了。通过分析发现关键问题是这样：

  at com.android.mms.ui.SlideshowPresenter$3.run(SlideshowPresenter.java:531)

定位到代码：

mHandler.post(new Runnable() {

​                    public void run() {

​                        try {

​                            presentRegionMedia(view, (RegionMediaModel) model, dataChanged);

​                        } catch (OMADRMException e) {

​                            Log.e(TAG, e.getMessage(), e);

​                            Toast.makeText(mContext,

​                                    mContext.getString(R.string.insufficient_drm_rights),

​                                    Toast.LENGTH_SHORT).show();

​                        } catch (IOException e){

​                            Log.e(TAG, e.getMessage(), e);

​                            Toast.makeText(mContext,

​                                    mContext.getString(R.string.insufficient_drm_rights),

​                                    Toast.LENGTH_SHORT).show();

​                        



​                        }

​                    }







很清楚了， Handler.post 方法之后执行时间太长的问题 。 继续看presentRegionMedia(view, (RegionMediaModel)
 model, dataChanged);方法 ， 发现最终是调用的framework 中MediaPlayer.stop方法 。

至此，我们的日志分析算是告一段落 。 可以开始思考解决办法了



---

上次面试的时候，对方问我：怎么分析和处理ANR。我断断续续也基本上都说到了上面提到的对象，可是对方还是不满意。现在想想，可能是自己对于对应的问题没有固定的处理流程和方法，只靠感觉，想到哪里了找哪里，根据日志文件去猜哪个地方有问题。自己以前英语考试也完全不懂语法，靠感觉选答案，虽然考试成绩还挺好的，可是到了工作面试的时候，没有一个套路，有条理的说出解决办法，肯定无法让对方满意。