# Chapter 1. Introduction 第一章. 引言
## <font color=green>Table of Contents 目录 </font>
<font color=green>1.1. About this user guide 关于本用户指南</font>
<br />
<br />

We would like to introduce Gradle to you, a build system that we think is a quantum leap for build technology in the Java (JVM) world. Gradle provides:  
我们将会把Gradle介绍给你--一种用于Java (JVM)的构建系统，我们认为它是这个领域的构建技术的一大突破。Gradle可以提供以下内容：

* A very flexible general purpose build tool like Ant.  
像Ant一样非常灵活的通用构建工具。

* Switchable, build-by-convention frameworks a la Maven. But we never lock you in!
* Very powerful support for multi-project builds.  
非常强大的多项目构建支持

* Very powerful dependency management (based on Apache Ivy).  
非常强大的依赖管理(基于Apache Ivy)

* Full support for your existing Maven or Ivy repository infrastructure.  
对你现有的Maven或者Ivy代码仓库的完全支持

* Support for transitive dependency management without the need for remote repositories or pom.xml and ivy.xml files.   
支持依赖管理，而不需要远程仓库或者pom.xml和ivy.xml文件

* Ant tasks and builds as first class citizens.

* *Groovy* build scripts.   
Groovy构建脚本

* A rich domain model for describing your build.

In *Chapter 2, Overview* you will find a detailed overview of Gradle. Otherwise, the tutorials are waiting, have fun :)   
在第二章，你会看到关于Gradle的更详细的概览。

## <font color=green>1.1. About this user guide  关于本用户指南</font>
This user guide, like Gradle itself, is under very active development. Some parts of Gradle aren't documented as completely as they need to be. Some of the content presented won't be entirely clear or will assume that you know more about Gradle than you do. We need your help to improve this user guide. You can find out more about contributing to the documentation at the Gradle web site.   
本用户指南和Gradle一样都处于非常活跃的发展阶段，Gradle的部分内容可能还没有详细的文档列出来，现有文档的部分内容也可能描述不是十分的清晰，或者说我们假设你已经对Gradle有些了解。我们需要你的帮助以改善这份文档的质量，你可以在Gradle网址上找到更多关于怎样向这份文档贡献自己的力量的方法。

Throughout the user guide, you will find some diagrams that represent dependency relationships between Gradle tasks. These use something analogous to the UML dependency notation, which renders an arrow from one task to the task that the first task depends on.   
在整个用户指南中，你会看到一些用于表示Gradle task之间的关系的图表，他们使用了和UML依赖描述相类似的符号，比如从某一个task到第一个task所依赖的task的箭头符号

