Chapter 5. The Gradle Console
第五章 Gradle控制台

[原文地址 v4.4.1](https://docs.gradle.org/current/userguide/console.html)

Table of Contents

* [5.1. 概述](#概述)
* [5.2. 命令行反馈](#命令行反馈)
* [5.3. 在无交互环境下的外观和感想](#在无交互环境下的外观和感受)

## 概述

在某些层面上来说几乎每一个Gradle用户都体验过命令行接口，Gradle控制台输出可以用来显示性能和可用性，只显示相关信息，并且提供反馈。

Figure 5.1. The Gradle command-line in action

![The Gradle command-line in action](https://docs.gradle.org/current/userguide/img/console-animation.gif)


## 命令行反馈

构建在执行的时候Gradle会显示相关信息使你可以关注感兴趣的重要条目。每一个Gradle控制台的输出区域都有助于解决以下问题：

    关于我的构建任务现在是不是有我立刻需要知道的东西，也就是测试是不是失败了，有没有警告信息？

    我的构建任务什么时候会结束？

    Gradle现在在做什么？

    有没有其他感兴趣的输出，也就是是不是有任务被跳过了或者它们是up-to-data状态的？

###  构建过程的输出

build脚本的log信息，任务，产生的新的进程，测试输出，编译警告等输出会显示在build进度条上

Figure 5.2. Build output portion of the Gradle command-line

![Build output portion of the Gradle command-line](https://docs.gradle.org/current/userguide/img/console-build-output.jpg)

从Gradle 4.0开始，命令行控制台输出内容会有所减少。每个任务的开始和结束或者说任务的结果(也就是 UP_TO_DATE)不会再显示，任务的名字只会在任务执行时有信息输出的时候才会显示。Gradle也会通过特定的上下文把输出分组显示，也就是编译，测试执行或者创建进程等产生的所有warnings。分组输出信息对于并行任务执行将会很有用，因为它会阻止未知来源信息的交叉存储(see Section 26.8, “Parallel project execution”)。

> 分组控制台输出和减少控制台输出功能只会在富文本控制台命令行出现。持续集成服务器和使用--console==plain参数构建的将会看到和Gradle 4.0以前的版本一样的控制台输出。你可也可以通过org.gradle.console属性来设置这个选项，查看 Section 12.1, “Configuring the build environment via gradle.properties”获取更多信息。

下面的控制台输出为configuration phase和compileJava任务的输出进行的分组
~~~
> Configure project ':library'
Configuring project version for project ':library'

> Configure project ':consumer'
Configuring project version for project ':consumer'

> Task :compileJava
Note: Some input files use unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.
~~~
Gradle不会一直等到工作完全结束才显示输出。Gradle在短暂的时间后会刷新输出到控制台来确保相关信息尽快显示。当并行构建的时候，长时间运行的任务可能会受到别的任务的影响。每个控制台输出块都会清晰的显示这个输出属于哪个任务。
~~~
> Task :compileJava
Note: Some input files use unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.

> Task :generateCode
Generating JAXB classed from XSD files.

> Task :compileJava
Note: Some input files use or override a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
~~~

### 构建进度条

通过构建进度条你可以很快捷的知道这次构建是不是会很快结束。构建工作开始的时候，进度条会从左到右填充。在任何时间，构建进度条也会渲染出当前构建处于生命周期的哪个阶段和总共用的时间。

Figure 5.3. Build progress bar portion of the Gradle command-line   
![Build progress bar portion of the Gradle command-line](https://docs.gradle.org/current/userguide/img/console-build-progress-bar.jpg)

下面的例子显示了初始化配置和执行阶段的进度条
~~~
<-------------> 0% INITIALIZING [2s]
<==-----------> 25% CONFIGURING [4s]
<=========----> 64% EXECUTING [17s]
~~~

### 显示当前工作

Gradle在构建进度条下剔红了很直观的视图用来查看当前正在进行的工作。每一行都代表一个能并行解析依赖项，执行任务和运行测试的工作线程或者进程。如果一个可用的工作者没有被使用它就会被标记为空闲。默认的工作者数量取决于你执行任务的机器的核心数量

Figure 5.4. Work in-progress portion of the Gradle command-line   
![Work in-progress portion of the Gradle command-line](https://docs.gradle.org/current/userguide/img/console-work-in-progress.jpg)

> 并行测试任务的执行只会在基于JVM的测试任务中由Gradle核心来显示，也就是Junit和TestNG。未来的Gradle版本可能会支持其他类型的测试工具和框架。

下面的控制台输出部分显示了由八个并行工作者运行的工作任务
~~~
<==========---> 77% EXECUTING [10s]
> :codeQuality:classpathManifest > Resolve dependencies :codeQuality:runtimeClasspath
> :ivy:classpathManifest > Resolve dependencies :ivy:runtimeClasspath
> IDLE
> :antlr:classpathManifest > Resolve dependencies :antlr:runtimeClasspath
> :scala:compileJava > Resolve dependencies :scala:compileClasspath
> :buildInit:classpathManifest > Resolve dependencies :buildInit:runtimeClasspath
> :jacoco:classpathManifest > Resolve dependencies :jacoco:runtimeClasspath
> IDLE
~~~

### 构建结果

在构建的结尾，Gradle会显示构建结果(成功或者失败)和执行的任务数量以及跳过的任务数量。构建结果也会显示执行任务整体的消耗时间。执行的任务数量直观的显示了how out-of-date or busy the build was.

Figure 5.5. Build progress bar portion of the Gradle command-line   
![Build progress bar portion of the Gradle command-line](https://docs.gradle.org/current/userguide/img/console-build-result.jpg)

下面的构建结果显示了一次成功的构建结果和任务的数量以及它们的状态
~~~
BUILD SUCCESSFUL in 2m 10s
411 actionable tasks: 381 executed, 30 up-to-date
~~~
Actionable任务只是至少包含一个action的任务。像Lifecycle任务的构建就不会声明任何action，因此是not actionable的。

## 在无交互环境下的外观和感受

默认情况下，Gradle会通过检测构建任务执行的控制台的类型来试着允许富文本控制台输出，这样子可以显示颜色和更多的控制台输出类型。无交互的环境下会回退到纯文本控制台输出。纯文本输出的格式不支持分组输出。任务和结果总是和Gradle 3.x版本保持一致。

> 用过IDE(也就是BuildShip和IntelliJ)执行的Gradle构建或者持续集成产品(也就是Jenkins和TeamCity)默认使用纯文本输出格式

下面展示了纯文本控制台的输出：

~~~
:compileJava
Note: Some input files use unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.
:processResources
:classes
:jar
:assemble
:compileTestJava NO-SOURCE
:processTestResources NO-SOURCE
:testClasses UP-TO-DATE
:test NO-SOURCE
:check UP-TO-DATE
:build

BUILD SUCCESSFUL in 6s
11 actionable tasks: 6 executed, 5 up-to-date
~~~