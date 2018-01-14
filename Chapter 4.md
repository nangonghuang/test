# Chapter 4. Using the Gradle Command-Line

[原文地址 v4.4.1](https://docs.gradle.org/current/userguide/tutorial_gradle_command_line.html)

## Table of Contents

* [4.1. 执行多项任务](#执行多项任务)
* [4.2. 排除任务](#排除任务)
* [4.3.在失败发生的时候依然继续执行](#在失败发生的时候依然继续执行)
* [4.4.任务名字缩写](#任务名字缩写)
* [4.5. 选择执行哪个构建脚本](#选择执行哪个构建脚本)
* [4.6.强制任务执行](#强制任务执行)
* [4.7. 获取你的构建信息](#获取你的构建信息)
* [4.8. Dry Run模式](#Dry-Run模式)
* [4.9. 总结](#总结)

这一章介绍Gradle命令行的一些基本用法。在前面的章节你已经有见到，使用gradle命令行来执行一次构建的过程。

### 执行多项任务

通过在命令行列出每一项任务，你可以在一次构建中执行多个任务。比如，gradle compile test命令会执行compile和test两个任务。Gradle会按照命令行里面的顺序执行任务，并且为每一个任务也会执行它们的依赖项。每一个任务都只执行一次，不管它是怎么被包含在build脚本里面的，这通常由两种方式，一是通过命令行显示制定二是作为其他任务的依赖项，或者两种都有。我们来看看一个例子。

在下面定义了四个任务。dist和test任务都依赖于compile，但是执行gradle dist test时，compile任务只会执行一次

Figure 4.1. Task dependencies
Task dependencies

Example 4.1. Executing multiple tasks

build.gradle
~~~groovy
task compile {
    doLast {
        println 'compiling source'
    }
}

task compileTest(dependsOn: compile) {
    doLast {
        println 'compiling unit tests'
    }
}

task test(dependsOn: [compile, compileTest]) {
    doLast {
        println 'running unit tests'
    }
}

task dist(dependsOn: [compile, test]) {
    doLast {
        println 'building the distribution'
    }
}
~~~
Output of gradle dist test
~~~
> gradle dist test
:compile
compiling source
:compileTest
compiling unit tests
:test
running unit tests
:dist
building the distribution

BUILD SUCCESSFUL in 0s
4 actionable tasks: 4 executed
~~~
每个任务都只执行了一次，因此gradle test test等同于gradle test

### 排除任务

你可以通过-x命令行选项和任务名来排除一个任务的执行。我们可以用前面的例子来试试。

Example 4.2. Excluding tasks

Output of gradle dist -x test
~~~
> gradle dist -x test
:compile
compiling source
:dist
building the distribution

BUILD SUCCESSFUL in 0s
2 actionable tasks: 2 executed
~~~

你可以在这个例子的输出里面看到，test任务没有执行，即使它是dist任务的依赖之一。你也可以注意到，test任务的依赖，比如compileTest也没有执行。不过像compile这种被其他任务以来的，还是执行了。

### 在失败发生的时候依然继续执行

默认情况下，一旦遇到任务失败的情况，Gradle就会放弃执行并且中断构建过程，这样可以让构建任务迅速结束，但是也会隐藏调其他可能会发生的失败。为了在一次构建中尽可能的发现更多的任务失败情况，你可以使用--continue选项。

当使用--continue选项执行的时候，Gradle会执行每一个待执行的任务，并且它们的依赖项也会不中止的执行完毕而不是遇到第一个失败就停止构建过程。在构建过程
的结束的时候每一项失败都会报告出来。

如果一个任务失败了，出于安全的考虑，任何接下来依赖它的任务都不会被执行。(这里好像有点矛盾，没看明白，不翻译了)
If a task fails, any subsequent tasks that were depending on it will not be executed, as it is not safe to do so. For example, tests will not run if there is a compilation failure in the code under test; because the test task will depend on the compilation task (either directly or indirectly).


### 任务名字缩写

当你在命令行里面指定执行某些任务的时候，你不一定需要提供任务的完整名字。你只需要提供能够唯一指定任务的名字的足够信息。比如，在上面的例子里面，你可以通过执行gradle d来执行dist任务。

Example 4.3. Abbreviated task name

Output of gradle di
~~~
> gradle di
:compile
compiling source
:compileTest
compiling unit tests
:test
running unit tests
:dist
building the distribution

BUILD SUCCESSFUL in 0s
4 actionable tasks: 4 executed
~~~
如果任务名字是驼峰命名，你也可以简化它们。比如，你可以通过gradle compTest甚至是gradle cT来执行 compileTest任务。

Example 4.4. Abbreviated camel case task name

Output of gradle cT
~~~
> gradle cT
:compile
compiling source
:compileTest
compiling unit tests

BUILD SUCCESSFUL in 0s
2 actionable tasks: 2 executed
~~~

你也可以在-x选项里面使用这种缩写。


### 选择执行哪个构建脚本
当你执行gradle命令的时候，它会去找当前目录下的build脚本去执行。你也可以使用-b选项来选择执行另外一个Build脚本。

Example 4.5. Selecting the project using a build file

subdir/myproject.gradle
~~~groovy
task hello {
    doLast {
        println "using build file '$buildFile.name' in '$buildFile.parentFile.name'."
    }
}
~~~
Output of gradle -q -b subdir/myproject.gradle hello
~~~
> gradle -q -b subdir/myproject.gradle hello
using build file 'myproject.gradle' in 'subdir'.
~~~
另外，你可以使用-p选项来指定使用的项目目录，对于多模块项目你的构建你应该使用-p而不是-b选项。

Example 4.6. Selecting the project using project directory

Output of gradle -q -p subdir hello
~~~
> gradle -q -p subdir hello
using build file 'build.gradle' in 'subdir'.
~~~
### 强制任务执行
4.6. Forcing tasks to execute
许多任务都支持增量构建，尤其是gradle自己提供的。根据上次构建以来它们的输入或者输出是否有变化，这些任务可以决定自己是不是要执行。当Gradle在任务附近显示UP-TO-DATE的时候你可以轻松指定任务参与增量构建。
你可能会想要Gradle强制执行所有任务的情况，不管是不是有任何up-to-data检查。使用--rerun-tasks选项就可以轻松满足你。这里是一个使用和不使用--rerun-tasks选项的例子

Example 4.7. Forcing tasks to run

Output of gradle doIt
~~~
> gradle doIt
:doIt UP-TO-DATE
~~~
Output of gradle --rerun-tasks doIt
~~~
> gradle --rerun-tasks doIt
:doIt
~~~
需要注意的是这会强制执行所有需要的任务，而不只是你命令行里指定的那个任务。它有点像执行一个clean，但是不会删除产生的输出。

### 获取你的构建信息
Gradle提供了几个内建任务用来显示你的构建细节。这对于调试问题，理解项目结构和依赖将会很有帮助。

除了下面显示的几个内建任务以外，你还可以使用项目报告插件来把可以产生这些报告的任务加到你的项目里面去。

4.7.1. Listing projects

执行gradle projects会通过层次树来显示所选的项目的子项目列表，这里是一个例子：

Example 4.8. Obtaining information about projects

Output of gradle -q projects
~~~
> gradle -q projects

------------------------------------------------------------
Root project
------------------------------------------------------------

Root project 'projectReports'
+--- Project ':api' - The shared API for the application
\--- Project ':webapp' - The Web application implementation
~~~
想要看一个项目的tasks列表，执行gradle <project-path>:tasks命令，比如，执行gradle :api:tasks报告会显示所有的项目描述信息，你可以设置描述信息属性的值。

Example 4.9. Providing a description for a project

build.gradle
~~~
description = 'The shared API for the application'
~~~
### 显示任务列表
4.7.2. Listing tasks
执行gradle tasks任务会显示所选项目的主要任务列表。这些报告会显示项目的默认任务，如果有描述信息的话也会显示出来。

Example 4.10. Obtaining information about tasks

Output of gradle -q tasks
~~~
> gradle -q tasks

------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Default tasks: dists

Build tasks
-----------
clean - Deletes the build directory (build)
dists - Builds the distribution
libs - Builds the JAR

Build Setup tasks
-----------------
init - Initializes a new Gradle build.
wrapper - Generates Gradle wrapper files.

Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in root project 'projectReports'.
components - Displays the components produced by root project 'projectReports'. [incubating]
dependencies - Displays all dependencies declared in root project 'projectReports'.
dependencyInsight - Displays the insight into a specific dependency in root project 'projectReports'.
dependentComponents - Displays the dependent components of components in root project 'projectReports'. [incubating]
help - Displays a help message.
model - Displays the configuration model of root project 'projectReports'. [incubating]
projects - Displays the sub-projects of root project 'projectReports'.
properties - Displays the properties of root project 'projectReports'.
tasks - Displays the tasks runnable from root project 'projectReports' (some of the displayed tasks may belong to subprojects).

To see all tasks and more detail, run gradle tasks --all

To see more detail about a task, run gradle help --task <task>
~~~
默认情况下，这个报告只会显示付赋值到一个任务组里的任务，我们称为可见任务。你可以通过设置任务的group属性来做到这些，你可以可以设置description属性，这样在报告里面就会显示它们

Example 4.11. Changing the content of the task report

build.gradle
~~~
dists {
    description = 'Builds the distribution'
    group = 'build'
}
~~~~
你可以使用--all选项来获得更多任务信息。指定此选项的时候，task报告会显示项目里面所有的任务，包括没有赋值到任务组里的任务，我们称这种为隐藏任务。

Example 4.12. Obtaining more information about tasks

Output of gradle -q tasks --all

~~~
> gradle -q tasks --all

------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Default tasks: dists

Build tasks
-----------
clean - Deletes the build directory (build)
api:clean - Deletes the build directory (build)
webapp:clean - Deletes the build directory (build)
dists - Builds the distribution
api:libs - Builds the JAR
webapp:libs - Builds the JAR

Build Setup tasks
-----------------
init - Initializes a new Gradle build.
wrapper - Generates Gradle wrapper files.

Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in root project 'projectReports'.
api:buildEnvironment - Displays all buildscript dependencies declared in project ':api'.
webapp:buildEnvironment - Displays all buildscript dependencies declared in project ':webapp'.
components - Displays the components produced by root project 'projectReports'. [incubating]
api:components - Displays the components produced by project ':api'. [incubating]
webapp:components - Displays the components produced by project ':webapp'. [incubating]
dependencies - Displays all dependencies declared in root project 'projectReports'.
api:dependencies - Displays all dependencies declared in project ':api'.
webapp:dependencies - Displays all dependencies declared in project ':webapp'.
dependencyInsight - Displays the insight into a specific dependency in root project 'projectReports'.
api:dependencyInsight - Displays the insight into a specific dependency in project ':api'.
webapp:dependencyInsight - Displays the insight into a specific dependency in project ':webapp'.
dependentComponents - Displays the dependent components of components in root project 'projectReports'. [incubating]
api:dependentComponents - Displays the dependent components of components in project ':api'. [incubating]
webapp:dependentComponents - Displays the dependent components of components in project ':webapp'. [incubating]
help - Displays a help message.
api:help - Displays a help message.
webapp:help - Displays a help message.
model - Displays the configuration model of root project 'projectReports'. [incubating]
api:model - Displays the configuration model of project ':api'. [incubating]
webapp:model - Displays the configuration model of project ':webapp'. [incubating]
projects - Displays the sub-projects of root project 'projectReports'.
api:projects - Displays the sub-projects of project ':api'.
webapp:projects - Displays the sub-projects of project ':webapp'.
properties - Displays the properties of root project 'projectReports'.
api:properties - Displays the properties of project ':api'.
webapp:properties - Displays the properties of project ':webapp'.
tasks - Displays the tasks runnable from root project 'projectReports' (some of the displayed tasks may belong to subprojects).
api:tasks - Displays the tasks runnable from project ':api'.
webapp:tasks - Displays the tasks runnable from project ':webapp'.

Other tasks
-----------
api:compile - Compiles the source files
webapp:compile - Compiles the source files
docs - Builds the documentation
~~~

### 显示任务使用细节
4.7.3. Show task usage details

执行gradle help --task <someTask>会显示你的多项目构建里匹配你所指定的任务或者多任务的细节信息。下面是一个例子：

Example 4.13. Obtaining detailed help for tasks

Output of gradle -q help --task libs
~~~
> gradle -q help --task libs
Detailed task information for libs

Paths
     :api:libs
     :webapp:libs

Type
     Task (org.gradle.api.Task)

Description
     Builds the JAR

Group
     build
~~~
这里的信息包括了任务完整路径，类型，可能的命令行选项和指定任务的描述信息

### 显示项目依赖
执行gradle dependencies会列出项目的依赖列表。对于每一个配置，直接的和透明的依赖都会显示在一个树状结构里面，下面是一个例子：

Example 4.14. Obtaining information about dependencies

Output of gradle -q dependencies api:dependencies webapp:dependencies

~~~
> gradle -q dependencies api:dependencies webapp:dependencies

------------------------------------------------------------
Root project
------------------------------------------------------------

No configurations

------------------------------------------------------------
Project :api - The shared API for the application
------------------------------------------------------------

compile
\--- org.codehaus.groovy:groovy-all:2.4.10

testCompile
\--- junit:junit:4.12
     \--- org.hamcrest:hamcrest-core:1.3

------------------------------------------------------------
Project :webapp - The Web application implementation
------------------------------------------------------------

compile
+--- project :api
|    \--- org.codehaus.groovy:groovy-all:2.4.10
\--- commons-io:commons-io:1.2

testCompile
No dependencies
~~~


因为依赖报告可能会很大，严格限制指定显示某个配置将会很有用，你可以通过--configuration选项来实现这个目的。

Example 4.15. Filtering dependency report by configuration

Output of gradle -q api:dependencies --configuration testCompile
~~~
> gradle -q api:dependencies --configuration testCompile

------------------------------------------------------------
Project :api - The shared API for the application
------------------------------------------------------------

testCompile
\--- junit:junit:4.12
     \--- org.hamcrest:hamcrest-core:1.3
~~~

### 显示项目build脚本的依赖

执行 gradle buildEnvironment 会显示象奴的build脚本的依赖，显示方式和gradle dependencies类似

### 获取特定依赖的insight信息

执行gradle denpendencyInsight会显示匹配特定输入的依赖的insight信息。下面是一个例子：

Example 4.16. Getting the insight into a particular dependency

Output of gradle -q webapp:dependencyInsight --dependency groovy --configuration compile
~~~
> gradle -q webapp:dependencyInsight --dependency groovy --configuration compile
org.codehaus.groovy:groovy-all:2.4.10
\--- project :api
     \--- compile
~~~
这个任务对于深究依赖信息特别有用，尤其是找到某个依赖是怎么来的和为什么选择了这个版本。更多的信息请查看API文档里面的DependencyInsightReportTask类

这个内置的dependencyInsight任务是Help任务组的一部分。这个任务需要先配置依赖信息和配置信息。这个报告会寻找在特定配置里面匹配指定依赖的依赖项。如果涉及到Java香港的插件的话，dependencyInsight任务是事先通过complile任务的配置配置好的，因为通常来说我们感兴趣的都是编译依赖。你应该通过命令行 --dependency来指定你感兴趣的依赖项。如果你不喜欢默认的你可以通过--configuration来指定配置。更多信息请查看APiece文档里面的DependencyInsightReportTask类


### 列出项目属性

执行gradle properties会显示项目的属性列表。下面是输出信息的一部分内容：

Example 4.17. Information about properties

Output of gradle -q api:properties
~~~
> gradle -q api:properties

------------------------------------------------------------
Project :api - The shared API for the application
------------------------------------------------------------

allprojects: [project ':api']
ant: org.gradle.api.internal.project.DefaultAntBuilder@12345
antBuilderFactory: org.gradle.api.internal.project.DefaultAntBuilderFactory@12345
artifacts: org.gradle.api.internal.artifacts.dsl.DefaultArtifactHandler_Decorated@12345
asDynamicObject: DynamicObject for project ':api'
baseClassLoaderScope: org.gradle.api.internal.initialization.DefaultClassLoaderScope@12345
buildDir: /home/user/gradle/samples/userguide/tutorial/projectReports/api/build
buildFile: /home/user/gradle/samples/userguide/tutorial/projectReports/api/build.gradle
~~~

### 构建的性能分析

当你执行构建任务的时候，--profile命令行选项会记录一些有用的时间信息，并且在build/reports/profile目录下生成一个报告。这个报告会使用构建的启动时间命名。

这个报告列出了configuration phase和任务执行的时间和细节。时间方面按照最耗时的操作来进行排序。任务执行结果也会显示是否有任务跳过以及跳过原因或者是否有不能正常工作的任务。

使用buildSrc目录的构建将会在buildSrc/build目录下产生另外一个profile报告。

### Dry Run模式

有时候你会对命令行里面指定的任务集的执行顺序感兴趣，但是你不想它们真的执行。你可以使用-m选项。比如，如果你执行gradle -m clean compile命令，你将会看到clean和compile任务的所有子任务。这是对于任务集里的任务的补充，它会显示有哪些子任务是可以执行的。

### 总结

在这一章，你看到了可以使用命令行执行的一些gradle任务。你可以在附录D，Gradle命令行 里面看到更多的gradle命令