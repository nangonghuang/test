Chapter 8. Dependency Management Basics
# 第八章 依赖管理基础
 目录

[8.1. 依赖管理是什么](#81-依赖管理是什么)

[8.2. 声明你的依赖](#82-声明你的依赖)

[8.3. 依赖配置](#83-依赖配置)

[8.4. 外部依赖](#84-外部依赖)

[8.5. 仓库](#85-仓库)

[8.6. Publishing artifacts](#86-publication-artifacts)

[8.7. 接下来是什么](#87-接下来是什么)

这一章介绍了Gradle的一些依赖管理的基础知识。

## 8.1. 依赖管理是什么

大体上来说，依赖管理可以分为两部分。首先，Gradle要知道知道你的项目构建和运行缺少什么东西，这样才能找到它们，我们把这些输入文件叫做项目依赖。第二，Gradle需要构建和上传你的项目所产生的输出，我们把这些输出叫做项目产出。让我们详细的看下这两部分：

大部分的项目都不是完全的自我完备的，它们需要别的项目产生的文件来编译和测试等等。比如，为了在项目中使用Hibernate，我在编译我的代码的时候需要包含一些Hibernate的jar包在classpath，为了运行我的测试，我也需要在classpath包含一些额外的jar包。

这些输入文件组成了项目的依赖。Gradle允许你告诉它你的项目依赖是什么，因此它才能找到这些依赖，然后使它们在你的构建过程中可用。这些依赖有可能需要从远程Maven仓库或者Ivy仓库下载，也可能就在本地目录，或者由另一个项目的构建产生。我们把这个过程叫做dependency resolution。

这个特性相对于ant来说是一个主要的优势。使用ant的时候，你只能指定绝对或者相对路径来加载，而使用Gradle，你只需要声明你的依赖的名字和一些其他的用来决定依赖位置的东西。你可以通过给ant添加Ivy仓库来获得类似的功能，但是Gradle可以做的更好。

项目的依赖自身也经常有可能还依赖于别人。比如，Hibernate core运行的时候需要几个其他的库声明在classpath上，因此，当Gradle为你的项目运行测试时，它也会要找到这些依赖然后使它们可用，我们叫这种为transitive依赖。

大部分项目的主要目的都是为了产生可以在项目以外使用的文件，比如，如果你的项目想要产生一个Java库，你需要构建一个jar包，可能还会有源码和一些文档，然后把它们发布到某个地方。

这些输出文件组成了项目的产出。Gradle当然也会为你处理这件重要的事情。你声明你的项目的产出，然后gradle开始构建它们并且发布它们到某个地方。想要精确的描述“发布”的意思取决于你想做什么。你也许想要拷贝文件到一个本地目录，或者上传它们到一个远程maven或者ivy仓库，或者你可能使用另一个项目的文件来构建，我们把这个过程叫做publication。
## 8.2. 声明你的依赖

我们先看以下依赖的声明过程，这里是一个基本的build脚本:

**Example 8.1. 声明依赖**

build.gradle
```groovy
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
    compile group: 'org.hibernate', name: 'hibernate-core', version: '3.6.7.Final'
    testCompile group: 'junit', name: 'junit', version: '4.+'
}
```
这里发生了什么？这个build脚本描述了项目的一些东西。首先，它声明的编译项目的文件需要Hibernate core 3.6.7.Final版本，隐式的说明，Hibernate core和它自己的依赖也是运行时必须的。这个build脚本也声明了任何大于4.0版本的junit也是编译项目测试的必要条件之一。它也告诉了Gradle在maven中央仓库里面去找所需要的任何依赖，接下来的内容将会描述细节部分。

## 8.3. 依赖配置

配置是指一个依赖的名字集合。这里的配置有三个主要的目的：

* 声明依赖   
在执行gradle插件所定义的各种各样的任务的时候，使用配置可以更清晰的描述构建过程需要哪些子项目或者外部产出。
* 解析依赖  
插件使用配置来发现(有可能需要下载)它所定义的任务需要的输入文件。
* 暴露消费者所需要的产出   
插件使用配置来定义它会产生什么文件，用来给别的项目使用。

带着着三个目的，让我们来看看java库插件定义的一些标准配置。你可以找到在49.4节“Java库插件配置”了解到更详细的信息。

* implementation

这部分描述的是编译项目源码所需要的依赖，但不属于项目暴露的api部分，这个配置是声明依赖的配置例子之一
* runtimeClasspath

这部分描述的是项目的类运行时所需要的依赖。默认情况下，包括了api,implementation，和runtimeOnly配置所声明的所有依赖。这个配置是解析依赖的例子之一，而且这样子的话，用户就不应该直接在runtimeClasspath配置中声明依赖

* apiElements
这部分描述的这个项目的外部依赖API的一部分，而且也包含了这个项目定义的应该提供给别的项目的类。这个配置是暴露消费者所需要的产出配置的例子之一。

各种各样的插件提供了更多的标准特性，你也可以在你的构建中定义自己的自定义配置。请阅读25.3节，“依赖配置”获取更多的关于定义和自定义依赖配置的细节。

## 8.4. 外部依赖

你可以声明各种各样的依赖。一种类型是外部依赖，它依赖于当前构建以外的构建产生的文件，这些文件存储于某些仓库中，比如maven central或者公司的maven，ivy仓库，或者本地文件系统目录上。

你需要添加外部依赖到依赖配置里面去，这样才能定义它：

**Example 8.2. 声明外部依赖**

build.gradle
```groovy
dependencies {
    compile group: 'org.hibernate', name: 'hibernate-core', version: '3.6.7.Final'
}
```
这里使用group,name和version属性指定了一个外部依赖，根据你使用的仓库种类的不同，group和version选项可能是可选项。

这里声明的外部依赖的快捷组成格式应该类似于"group:name:version"

**Example 8.3. 外部依赖的快捷定义**

build.gradle
```groovy
dependencies {
    compile 'org.hibernate:hibernate-core:3.6.7.Final'
}
```
想知道更多的关于定义和使用依赖进行工作的例子，查看25.4节"怎样声明你的依赖"
## 8.5. 仓库
Gradle怎样找到外部依赖文件的？Gradle在仓库里面找到它们。仓库是一个由group,name和version描述的文件的集合。Gradle能够识别不同的仓库格式，比如maven和ivy,也可以通过不同的方式访问仓库，比如使用本地文
件系统或者http。

默认情况下，Gradle不会预定义任何仓库。在你使用外部依赖的时候需要自己去指定至少一个。一个选项是maven中央仓库:

**Example 8.4. 使用maven中央仓库**

build.gradle
```groovy
repositories {
    mavenCentral()
}
```
或者 Bintray’s JCenter:

**Example 8.5. 使用JCenter仓库**

build.gradle
```groovy
repositories {
    jcenter()
}
```
或者任何其他的maven远程仓库:

**Example 8.6. 使用maven远程仓库**

build.gradle
```groovy
repositories {
    maven {
        url "http://repo.mycompany.com/maven2"
    }
}
```
或者 Ivy远程仓库:

**Example 8.7. 使用Ivy远程仓库**

build.gradle
```groovy
repositories {
    ivy {
        url "http://repo.mycompany.com/repo"
    }
}
```
你也可以使用本地文件系统上的仓库，Maven或者ivy仓库都可以

**Example 8.8. 使用本地Ivy目录**

build.gradle
```groovy
repositories {
    ivy {
        // URL can refer to a local directory
        url "../local-repo"
    }
}
```
一个项目可以使用多个仓库。Gradle会在每个仓库里面去找指定的依赖项，在一个仓库里面找到需要的模块之后就不再寻找。

想要知道更多关于仓库的定义和使用的信息，查看25.6节仓库。

## 8.6. publication artifacts

发布文件需要进行依赖配置。我们把这些文件叫做publication artifacts，或者简单的叫做artifacts。

插件已经为定义artifacts做了很多工作，因此要告诉Gradle需要什么你不需要做什么特殊的事情。不过你必须告诉Gradle去那里发布这些artifacts，你可以通过给uploadArchives任务附加仓库来做到这些。这里是一个发布到远程ivy仓库的例子：

**Example 8.9. Publishing to an Ivy repository**

build.gradle
```groovy
uploadArchives {
    repositories {
        ivy {
            credentials {
                username "username"
                password "pw"
            }
            url "http://repo.mycompany.com"
        }
    }
}
```
现在，你可以运行gradle uploadArchives，Gradle将会构建和上传你的jar，它也会生成和上传ivy.xml.

你也可以发布到maven仓库，不过语法会有一点不同，注意的是要想发布到maven仓库的话你需要使用maven插件。当这个做好了，Gradle就会生成和上传pom.xml。

**Example 8.10. Publishing to a Maven repository**

build.gradle
```groovy
apply plugin: 'maven'

uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: "file://localhost/tmp/myRepo/")
        }
    }
}
```
想要知道更多关于publication的信息，查看32章Publishing artifacts。
## 8.7. 接下来是什么

想要获取所有关于依赖方案的细节，查看25章，依赖管理，如果是artifact publication，查看32章。

如果你对这里提到的DSL元素感兴趣的话，查看Project.configurations{}， Project.repositories{}和Project.dependencies{}。

否则，继续别的Guide章节把。