# Chapter 4. Using the Gradle Command-Line

[原文地址 v4.4.1](https://docs.gradle.org/current/userguide/tutorial_gradle_command_line.html)

## Table of Contents

* 4.1. [执行多项任务](#执行多项任务)
* 4.2. Excluding tasks
* 4.3. Continuing the build when a failure occurs
* 4.4. Task name abbreviation
* 4.5. Selecting which build to execute
* 4.6. Forcing tasks to execute
* 4.7. Obtaining information about your build
* 4.8. Dry Run
* 4.9. Summary

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

4.5. Selecting which build to execute

When you run the gradle command, it looks for a build file in the current directory. You can use the -b option to select another build file. Example:

Example 4.5. Selecting the project using a build file

subdir/myproject.gradle

task hello {
    doLast {
        println "using build file '$buildFile.name' in '$buildFile.parentFile.name'."
    }
}

Output of gradle -q -b subdir/myproject.gradle hello

> gradle -q -b subdir/myproject.gradle hello
using build file 'myproject.gradle' in 'subdir'.

Alternatively, you can use the -p option to specify the project directory to use. For multi-project builds you should use -p option instead of -b option.

Example 4.6. Selecting the project using project directory

Output of gradle -q -p subdir hello

> gradle -q -p subdir hello
using build file 'build.gradle' in 'subdir'.

4.6. Forcing tasks to execute

Many tasks, particularly those provided by Gradle itself, support incremental builds. Such tasks can determine whether they need to run or not based on whether their inputs or outputs have changed since the last time they ran. You can easily identify tasks that take part in incremental build when Gradle displays the text UP-TO-DATE next to their name during a build run.

You may on occasion want to force Gradle to run all the tasks, ignoring any up-to-date checks. If that’s the case, simply use the --rerun-tasks option. Here’s the output when running a task both without and with --rerun-tasks:

Example 4.7. Forcing tasks to run

Output of gradle doIt

> gradle doIt
:doIt UP-TO-DATE

Output of gradle --rerun-tasks doIt

> gradle --rerun-tasks doIt
:doIt

Note that this will force all required tasks to execute, not just the ones you specify on the command line. It’s a little like running a clean, but without the build’s generated output being deleted.
4.7. Obtaining information about your build

Gradle provides several built-in tasks which show particular details of your build. This can be useful for understanding the structure and dependencies of your build, and for debugging problems.

In addition to the built-in tasks shown below, you can also use the project report plugin to add tasks to your project which will generate these reports.
4.7.1. Listing projects

Running gradle projects gives you a list of the sub-projects of the selected project, displayed in a hierarchy. Here is an example:

Example 4.8. Obtaining information about projects

Output of gradle -q projects

> gradle -q projects

------------------------------------------------------------
Root project
------------------------------------------------------------

Root project 'projectReports'
+--- Project ':api' - The shared API for the application
\--- Project ':webapp' - The Web application implementation

To see a list of the tasks of a project, run gradle <project-path>:tasks
For example, try running gradle :api:tasks

The report shows the description of each project, if specified. You can provide a description for a project by setting the description property:

Example 4.9. Providing a description for a project

build.gradle

description = 'The shared API for the application'

4.7.2. Listing tasks

Running gradle tasks gives you a list of the main tasks of the selected project. This report shows the default tasks for the project, if any, and a description for each task. Below is an example of this report:

Example 4.10. Obtaining information about tasks

Output of gradle -q tasks

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

By default, this report shows only those tasks which have been assigned to a task group, so-called visible tasks. You can do this by setting the group property for the task. You can also set the description property, to provide a description to be included in the report.

Example 4.11. Changing the content of the task report

build.gradle

dists {
    description = 'Builds the distribution'
    group = 'build'
}

You can obtain more information in the task listing using the --all option. With this option, the task report lists all tasks in the project, including tasks which have not been assigned to a task group, so-called hidden tasks. Here is an example:

Example 4.12. Obtaining more information about tasks

Output of gradle -q tasks --all

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

4.7.3. Show task usage details

Running gradle help --task someTask gives you detailed information about a specific task or multiple tasks matching the given task name in your multi-project build. Below is an example of this detailed information:

Example 4.13. Obtaining detailed help for tasks

Output of gradle -q help --task libs

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

This information includes the full task path, the task type, possible command line options and the description of the given task.
4.7.4. Listing project dependencies

Running gradle dependencies gives you a list of the dependencies of the selected project, broken down by configuration. For each configuration, the direct and transitive dependencies of that configuration are shown in a tree. Below is an example of this report:

Example 4.14. Obtaining information about dependencies

Output of gradle -q dependencies api:dependencies webapp:dependencies

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

Since a dependency report can get large, it can be useful to restrict the report to a particular configuration. This is achieved with the optional --configuration parameter:

Example 4.15. Filtering dependency report by configuration

Output of gradle -q api:dependencies --configuration testCompile

> gradle -q api:dependencies --configuration testCompile

------------------------------------------------------------
Project :api - The shared API for the application
------------------------------------------------------------

testCompile
\--- junit:junit:4.12
     \--- org.hamcrest:hamcrest-core:1.3

4.7.5. Listing project buildscript dependencies

Running gradle buildEnvironment visualises the buildscript dependencies of the selected project, similarly to how gradle dependencies visualises the dependencies of the software being built.
4.7.6. Getting the insight into a particular dependency

Running gradle dependencyInsight gives you an insight into a particular dependency (or dependencies) that match specified input. Below is an example of this report:

Example 4.16. Getting the insight into a particular dependency

Output of gradle -q webapp:dependencyInsight --dependency groovy --configuration compile

> gradle -q webapp:dependencyInsight --dependency groovy --configuration compile
org.codehaus.groovy:groovy-all:2.4.10
\--- project :api
     \--- compile

This task is extremely useful for investigating the dependency resolution, finding out where certain dependencies are coming from and why certain versions are selected. For more information please see the DependencyInsightReportTask class in the API documentation.

The built-in dependencyInsight task is a part of the 'Help' tasks group. The task needs to be configured with the dependency and the configuration. The report looks for the dependencies that match the specified dependency spec in the specified configuration. If Java related plugins are applied, the dependencyInsight task is pre-configured with the 'compile' configuration because typically it’s the compile dependencies we are interested in. You should specify the dependency you are interested in via the command line '--dependency' option. If you don’t like the defaults you may select the configuration via the '--configuration' option. For more information see the DependencyInsightReportTask class in the API documentation.
4.7.7. Listing project properties

Running gradle properties gives you a list of the properties of the selected project. This is a snippet from the output:

Example 4.17. Information about properties

Output of gradle -q api:properties

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

4.7.8. Profiling a build

The --profile command line option will record some useful timing information while your build is running and write a report to the build/reports/profile directory. The report will be named using the time when the build was run.

This report lists summary times and details for both the configuration phase and task execution. The times for configuration and task execution are sorted with the most expensive operations first. The task execution results also indicate if any tasks were skipped (and the reason) or if tasks that were not skipped did no work.

Builds which utilize a buildSrc directory will generate a second profile report for buildSrc in the buildSrc/build directory.
Profile
4.8. Dry Run

Sometimes you are interested in which tasks are executed in which order for a given set of tasks specified on the command line, but you don’t want the tasks to be executed. You can use the -m option for this. For example, if you run “gradle -m clean compile”, you’ll see all the tasks that would be executed as part of the clean and compile tasks. This is complementary to the tasks task, which shows you the tasks which are available for execution.
4.9. Summary

In this chapter, you have seen some of the things you can do with Gradle from the command-line. You can find out more about the gradle command in Appendix D, Gradle Command Line.