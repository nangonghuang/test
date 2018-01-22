Chapter 7. The Gradle Daemon
## 第七章. Gradle 守护进程

目录

[7.1. 为什么Gradle守护进程对性能很重要](#71-为什么gradle守护进程对性能很重要)

[7.2. 运行中的守护进程状态](#72-运行中的守护进程状态)

[7.3. 禁止守护进程](#73-禁止守护进程)

[7.4. 停止已经存在的守护进程](#74-停止已经存在的守护进程)

[7.5. FAQ](#75-faq)

[7.6. 工具和IDE相关](#76-工具和IDE)

[7.7. Gradle守护进程是怎么加速构建过程的](#77-gradle守护进程是怎么加速构建过程的)


来源于维基百科...
> 守护进程是指作为后台进程运行而不是直接受控于使用者的计算机程序。

Gradle运行在JVM上，并且使用了一些需要很长时间来初始化的库。因此，它有时候看起来启动很慢。解决这个问题的办法就是Gradle守护进程：一个长时间存活的后台进行，用来更快的执行你的构建过程。我们通过避免昂贵的引导过程以及利用缓存来实现这一点，将有关项目的数据保存在内存中。执行Gradle构建的时候启不启动守护进程没什么区别，只需要配置你想不想用它--别的事情都交给Gradle去透明的处理。
### 7.1. 为什么Gradle守护进程对性能很重要

守护进程是一个长期存活的进程，因此我们不仅可以避免每次构建中启动JVM的耗时，而且可以缓存项目结构，文件，任务以及更多的信息在内存里。

推理很简单：通过重复利用以前构建过程的计算可以提高构建速度。带来的优势的显而易见的: 我们经过测量后发现连续构建的时候构建时间能够减少15%-75%。我们推荐使用--profile参数来对你的构建过程进行性能分析，这样可以了解Gradle守护进程对你有多大的影响。

Gradle守护进程在3.0版本以后是默认开启的，因此你不需要做任何事情就可以从中受益。

如果你在不重用任何进行的临时环境里面执行CI构建，守护进行会轻微的降低性能（因为会缓存额外的数据）而没有任何好处，可以禁止掉。

### 7.2. 运行中的守护进程状态

为了获取运行中的守护进程列表和它们的状态，你可以使用--status命令

输出样例:

```
PID VERSION                 STATUS
28411 3.0                     IDLE
34247 3.0                     BUSY
```

目前来说，Gradle只会连接到相同版本的守护进程。这意味着这里的状态输出只会显示和Gradle版本相同的守护进行被激活。未来版本将会移除掉这个限制冰箱会显示所有版本的Gradle的运行中的守护进程信息。

### 7.3. 禁止守护进程
Gradle守护进程是默认允许的，而且我们也推荐一直允许它。有几个方法都可以禁用它，最常用的就是在«USER_HOME»/.gradle/gradle.properties文件里增加一行`org.gradle.daemon=false`，这里的«USER_HOME»是指你的用户目录，根据你的平台不同，下面是一些典型的目录：
```
    C:\Users\<username> (Windows Vista & 7+)

    /Users/<username> (macOS)

    /home/<username> (Linux)
```
如果那个文件不存在，使用文本编辑器创建一个就行。你可以在7.5节FAQ 找到关于允许或者禁止守护进程的更多的信息，那一节也包含了守护进程是怎么运行的。

注意一旦守护进程开启了，你的所有构建都可以利用它来进行加速，跟构建使用的gradle版本没有关系。

> 持续集成   
从Gradle3.0开始，我们默认允许守护进程并且推荐开发着机器和持续集成服务器都使用它。不过如果你怀疑守护进行会使你的CI构建不稳定，你可以禁止它，这样每次构建会使用全新的运行时，因为每次运行都完全和之前的构建无关

### 7.4. 停止已经存在的守护进程
我们提到，守护进程是一个后台进程，你不需要关心你的机器上gradle进程的创建过程。每一个守护进程都会监视自己的内存使用情况，如果可用系统内存不多，在空闲的时候还会停掉自己。如果你想要显式的停止守护进程，使用命令gradle --stop

这会终止所执行的Gradle版本对应的启动的所有守护进程，如果你安装了jdk，你可以用干jps命令很容易的确认守护进程已经停止了，你也可以看到名字叫做GradleDaemon的运行着的守护进程列表

### 7.5. FAQ
#### 7.5.1. 我怎样可以禁止守护进程

有两种推荐方式可以永久的禁止守护进程：

* 通过环境变量: 增加`-Dorg.gradle.daemon=false`到GRADLE_OPTS环境变量里

* 通过properties文件: 添加`org.gradle.daemon=false` 到 «GRADLE_USER_HOME»/gradle.properties 文件

«GRADLE_USER_HOME» 默认指向«USER_HOME»/.gradle，这里的«USER_HOME»是当前用户的home目录。这个地方可以通过-g 和 --gradle-user-home 命令行来配置，也可以通过GRADLE_USER_HOME环境变量和JVN的系统属性org.gradle.user.home来配置。

不管怎么配置都是相同的效果。使用哪一种取决于个人爱好。大部分Gradle用户使用第二种方法，添加一个entry到gradle.properties文件。

在windows上，这个命令会为当前用户禁止守护进程。
```
(if not exist "%USERPROFILE%/.gradle" mkdir "%USERPROFILE%/.gradle") && (echo. >> "%USERPROFILE%/.gradle/gradle.properties" && echo org.gradle.daemon=false >> "%USERPROFILE%/.gradle/gradle.properties")
```
在UNIX相关的系统上， 下面的Bash shell命令可以为当前用户禁止守护进程:
```
mkdir -p ~/.gradle && echo "org.gradle.daemon=false" >> ~/.gradle/gradle.properties
```
一旦通过这种方式禁止了，Gradle守护进程不会再启动除非显式的使用了--daemon选项。

使用Gradle命令行接口的时候，--daemon和--no-daemon命令行选项为单次的构建允许或者禁止守护进程的使用。这些命令行选项相对于环境变量具有最高的处理级别。通常，为用户允许守护进程会更方便，所有的构建都会自动使用守护进程而不需要记住去添加--daemon选项。
#### 7.5.2. 为什么我的机器上都不止一个守护进程？
Gradle创建新的守护进程而不是只使用一个，这样做有多个原因。一个基本的规则是如果没有存在的空闲或者兼容的守护进程可用时，Gradle会启动新的守护进程。如果守护进程空闲了三个小时或者以上，Gradle也会杀掉它，因此你不用担心需要手动的清理它们。

> 空闲状态   
空闲状态的守护进程就是指当前没有执行构建任务或者其他有用的工作的进程。
> 兼容的   
兼容的守护进程是指可以迎合所需求的构建环境的守护进程。Java运行时就是构建环境的一个方面。另一个例子是JVM要求的构建运行时系统属性。

有些需求的构建环境可能不会匹配守护进程。如果守护进程是Java 7运行时启动的，但是请求的构建环境要求Java 8，然后这个守护进程就不兼容因此另一个守护进程必须被启动起来。而且，一旦JVM启动起来特定版本的Java运行时就不会变化，比如，你没有办法改变一个运行着的JVM的内存分配，默认文本编码格式，默认的地区。

需求的构建环境通常是由构建客户端环境（比如Gradle命令行，IDE等等）隐式的创建的，也可以通过命令行显示的更改或者设置。查看第12章，构建环境获取关于控制和指定构建环境的更多细节。

以下的JVM系统属性是永久起效的。如果需求的构建环境请求下面的任意属性，和守护进程的JVM的这个属性带有不同的值，这个守护进程就是不兼容的。
```
    file.encoding

    user.language

    user.country

    user.variant

    java.io.tmpdir

    javax.net.ssl.keyStore

    javax.net.ssl.keyStorePassword

    javax.net.ssl.keyStoreType

    javax.net.ssl.trustStore

    javax.net.ssl.trustStorePassword

    javax.net.ssl.trustStoreType

    com.sun.management.jmxremote
```
下面的JVM属性，通过启动参数控制的，也是永久起效的。需求的构建环境和守护进程所对应的以下参数都必须完全匹配，这样守护进程才会是兼容的。
```
    The maximum heap size (i.e. the -Xmx JVM argument)

    The minimum heap size (i.e. the -Xms JVM argument)

    The boot classpath (i.e. the -Xbootclasspath argument)

    The “assertion” status (i.e. the -ea argument)
```

要求的Gradle版本则是需求构建环境的另外一个方面。守护进程是和一个特定版本的Gradle运行时相关联的。在使用不同的Gradle版本的多个Gradle项目中工作是需要启动多个守护进程的普遍原因之一。
#### 7.5.3. Gradle守护进程使用多少内存，我能设置更多吗
如果需求构建环境没有特别指定最大的堆容量，守护进程会使用最大1GB的堆。它会使用JVM的默认最小堆容量。对于大部分的构建来说1GB都是完全足够的，如果构建有上百个子项目，一大堆配置和许多源码，这种可能会需要更多的内存。

使用合适的flag作为需求构建环境就可以提高守护进程的可用内存。查看第12章,build环境获取更多信息。
#### 7.5.4. 我怎样可以停止守护进程
如果三个小时都没有交互，守护进程会自动终止自己。如果你希望在这之前停止守护进程，你可以通过炒作系统指令也可以执行gradle --stop命令。--stop开关会请求与这个命令相同Gradle版本的所有守护进程终止自己。
#### 7.5.5. 守护进程会出什么错

Considerable engineering effort has gone into making the Daemon robust, transparent and unobtrusive during day to day development. However, Daemon processes can occasionally be corrupted or exhausted. A Gradle build executes arbitrary code from multiple sources. While Gradle itself is designed for and heavily tested with the Daemon, user build scripts and third party plugins can destabilize the Daemon process through defects such as memory leaks or global state corruption.

It is also possible to destabilize the Daemon (and build environment in general) by running builds that do not release resources correctly. This is a particularly poignant problem when using Microsoft Windows as it is less forgiving of programs that fail to close files after reading or writing.

Gradle actively monitors heap usage and attempts to detect when a leak is starting to exhaust the available heap space in the daemon. When it detects a problem, the Gradle daemon will finish the currently running build and proactively restart the daemon on the next build. This monitoring is enabled by default, but can be disabled by setting the org.gradle.daemon.performance.enable-monitoring system property to false.

If it is suspected that the Daemon process has become unstable, it can simply be killed. Recall that the --no-daemon switch can be specified for a build to prevent use of the Daemon. This can be useful to diagnose whether or not the Daemon is actually the culprit of a problem.
### 7.6. 工具和IDE
IDE和其他工具都使用Gradle工具API(查看第14章，使用工具API内嵌Gradle)来集成gradle，它们总是使用Gradle守护进行来执行构建过程，如果你通过IDE来执行Gradle构建，那么你就是在使用Gradle守护进行，不需要专门为你的环境去允许它
###7.7. Gradle守护进程是怎么加速构建过程的
Gradle守护进程是一个长期存活的构建进程。在多次构建之间它会空闲等待下一次构建。这样做对于多次构建有明显的好处，相比于每次构建都单独执行，它只需要加载gradle到内存一次就足够了。这是一个重要的性能优化的点，但不止于此。

现代JVM的性能的一个重要方面就是运行时代码优化，比如HotSpot（Oracle为OpenJDK实现和提供的最基础的虚拟机)就可以在它运行的时候优化代码。优化过程是不断进行的而不是一次进行，在运行期间持续优化代码的意思是，接下来的构建过程会因为这次优化而变的更快。HotSpot的数据显示它进行了5到10次构建优化用于稳定性方面，对于守护进程来说，第一次构建和第10次构建消耗的时间可能会相差巨大。

守护进程也允许在多次构建期间更加有效的缓存内存。比如，构建(比如插件，脚本）所需要的类可以持续的加载在内存中。类似的，Gradle可以维持构建数据的内存缓存，比如用于增量构建的输入输入任务的hash值。