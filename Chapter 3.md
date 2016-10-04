# Chapter 3. Installing Gradle 第三章 安装Gradle
## Table of Contents 目录

* [3.1.Prerequisites 运行要求](#prerequisites)
* [3.2.Download 下载](#download)
* [3.3.Unpacking 解压](#unpacking)
* [3.4.Environment variables 设置环境变量](#environment-variables)
* [3.5.Running and testing your installation 运行并测试](#running-and-testing-your-installation)
* [3.6.JVM options JVM 选项](#jvm-options)

## Prerequisites 运行要求

Gradle requires a Java JDK or JRE to be installed, version 7 or higher (to check, use java -version). Gradle ships with its own Groovy library, therefore Groovy does not need to be installed. Any existing Groovy installation is ignored by Gradle.  
Gradle运行需要先安装Java JDK或者Java JRE 7或者更高（可以用java -version 来检查）。Gradle自带了Groovy库，因此不需要先安装Groovy，如果Groovy已经安装了将会被忽略。

Gradle uses whatever JDK it finds in your path. Alternatively, you can set the JAVA_HOME environment variable to point to the installation directory of the desired JDK.  
Gradle将会使用它在系统Path路径里面发现的JDK。另外你也可以设置JAVA_HOME环境变量来使用指定版本的JDK 。

## Download 下载
You can download one of the Gradle distributions from the Gradle web site.  
你可以在Gradle官网上下载Gradle的发行版本

## Unpacking 解压
The Gradle distribution comes packaged as a ZIP. The full distribution contains:  
Gradle的发行版已经打包成了zip格式，完整的安装包包括：

* The Gradle binaries. Gradle二进制文件
* The user guide (HTML and PDF). 用户指南（HTML和PDF版本）
* The DSL reference guide. DSL参考指南
* The API documentation (Javadoc). API文档

Extensive samples, including the examples referenced in the user guide, along with some complete and more complex builds you can use as a starting point for your own build.
The binary sources. This is for reference only. If you want to build Gradle you need to download the source distribution or checkout the sources from the source repository. See the Gradle web site for details.  
扩展示例，包括用户指南里面的实力代码，有一些已经构建完成的样品你可以用来作为你自己构建的起点。二进制的源码仅仅用来引用，如果你想自己编译Gradle你需要自己去下载源码或者从仓库检出源码。在官网上有更详细的说明。

## Environment variables 设置环境变量
For running Gradle, firstly add the environment variable GRADLE_HOME. This should point to the unpacked files from the Gradle website. Next add GRADLE_HOME/bin to your PATH environment variable. Usually, this is sufficient to run Gradle.  
为了运行Gradle，我们需要先添加GRADLE_HOME到环境变量，它应该指向Gradle官网上下载下来的二进制文件的路径，然后添加GRADLE_HOME/bin到环境变量。通常来说这样子就足够运行Gradle了

## Running and testing your installation 运行并测试
You run Gradle via the gradle command. To check if Gradle is properly installed just type gradle -v. The output shows the Gradle version and also the local environment configuration (Groovy, JVM version, OS, etc.). The displayed Gradle version should match the distribution you have downloaded.  
你可以通过gradle命令来运行Gradle，输入gradle -v来检查Gradle是不是已经正确安装。界面上会显示Gradle的版本信息和本地环境的配置（Groovy，JVM版本，系统信息等等）。显示出来的Gradle版本应该会和你下载的Gradle版本一致。

## JVM options JVM选项
JVM options for running Gradle can be set via environment variables. You can use either GRADLE_OPTS or JAVA_OPTS, or both. JAVA_OPTS is by convention an environment variable shared by many Java applications. A typical use case would be to set the HTTP proxy in JAVA_OPTS and the memory options in GRADLE_OPTS. Those variables can also be set at the beginning of the gradle or gradlew script.  
你可以通过环境变量来设置Gradle运行时的JVM选项，使用GRADLE_OPTS和JAVA_OPTS之一或者都设置。JAVA_OPTS是Java应用程序的通用环境变量设置，一个典型的应用是在它里面设置HTTP代理，在GRADLE_OPTS里面设置内存选项。这些值也可以在gradle或者gradlew脚本的开头进行设置

Note that it's not currently possible to set JVM options for Gradle on the command line.  
需要注意的是现在暂时不可以通过Gradle命令行来设置JVM选项