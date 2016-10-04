Chapter 4. Using the Gradle Command-Line
Table of Contents

4.1. Executing multiple tasks
4.2. Excluding tasks
4.3. Continuing the build when a failure occurs
4.4. Task name abbreviation
4.5. Selecting which build to execute
4.6. Forcing tasks to execute
4.7. Obtaining information about your build
4.8. Dry Run
4.9. Summary
This chapter introduces the basics of the Gradle command-line. You run a build using the gradle command, which you have already seen in action in previous chapters.

4.1. Executing multiple tasks
You can execute multiple tasks in a single build by listing each of the tasks on the command-line. For example, the command gradle compile test will execute the compile and test tasks. Gradle will execute the tasks in the order that they are listed on the command-line, and will also execute the dependencies for each task. Each task is executed once only, regardless of how it came to be included in the build: whether it was specified on the command-line, or as a dependency of another task, or both. Let's look at an example.

Below four tasks are defined. Both dist and test depend on the compile task. Running gradle dist test for this build script results in the compile task being executed only once.

Figure 4.1. Task dependencies

Task dependencies
Example 4.1. Executing multiple tasks

build.gradle
task compile << {
    println 'compiling source'
}

task compileTest(dependsOn: compile) << {
    println 'compiling unit tests'
}

task test(dependsOn: [compile, compileTest]) << {
    println 'running unit tests'
}

task dist(dependsOn: [compile, test]) << {
    println 'building the distribution'
}
Output of gradle dist test
> gradle dist test
:compile
compiling source
:compileTest
compiling unit tests
:test
running unit tests
:dist
building the distribution

BUILD SUCCESSFUL

Total time: 1 secs
Each task is executed only once, so gradle test test is exactly the same as gradle test.

4.2. Excluding tasks
You can exclude a task from being executed using the -x command-line option and providing the name of the task to exclude. Let's try this with the sample build file above.

Example 4.2. Excluding tasks

Output of gradle dist -x test
> gradle dist -x test
:compile
compiling source
:dist
building the distribution

BUILD SUCCESSFUL

Total time: 1 secs
You can see from the output of this example, that the test task is not executed, even though it is a dependency of the dist task. You will also notice that the test task's dependencies, such as compileTest are not executed either. Those dependencies of test that are required by another task, such as compile, are still executed.

4.3. Continuing the build when a failure occurs
By default, Gradle will abort execution and fail the build as soon as any task fails. This allows the build to complete sooner, but hides other failures that would have occurred. In order to discover as many failures as possible in a single build execution, you can use the --continue option.

When executed with --continue, Gradle will execute every task to be executed where all of the dependencies for that task completed without failure, instead of stopping as soon as the first failure is encountered. Each of the encountered failures will be reported at the end of the build.

If a task fails, any subsequent tasks that were depending on it will not be executed, as it is not safe to do so. For example, tests will not run if there is a compilation failure in the code under test; because the test task will depend on the compilation task (either directly or indirectly).

4.4. Task name abbreviation
When you specify tasks on the command-line, you don't have to provide the full name of the task. You only need to provide enough of the task name to uniquely identify the task. For example, in the sample build above, you can execute task dist by running gradle d:

Example 4.3. Abbreviated task name

Output of gradle di
> gradle di
:compile
compiling source
:compileTest
compiling unit tests
:test
running unit tests
:dist
building the distribution

BUILD SUCCESSFUL

Total time: 1 secs
You can also abbreviate each word in a camel case task name. For example, you can execute task compileTest by running gradle compTest or even gradle cT

Example 4.4. Abbreviated camel case task name

Output of gradle cT
> gradle cT
:compile
compiling source
:compileTest
compiling unit tests

BUILD SUCCESSFUL

Total time: 1 secs
You can also use these abbreviations with the -x command-line option.

4.5. Selecting which build to execute
When you run the gradle command, it looks for a build file in the current directory. You can use the -b option to select another build file. If you use -b option then settings.gradle file is not used. Example:

Example 4.5. Selecting the project using a build file

subdir/myproject.gradle
task hello << {
    println "using build file '$buildFile.name' in '$buildFile.parentFile.name'."
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

You may on occasion want to force Gradle to run all the tasks, ignoring any up-to-date checks. If that's the case, simply use the --rerun-tasks option. Here's the output when running a task both without and with --rerun-tasks:

Example 4.7. Forcing tasks to run

Output of gradle doIt
> gradle doIt
:doIt UP-TO-DATE
Output of gradle --rerun-tasks doIt
> gradle --rerun-tasks doIt
:doIt
Note that this will force all required tasks to execute, not just the ones you specify on the command line. It's a little like running a clean, but without the build's generated output being deleted.

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
init - Initializes a new Gradle build. [incubating]
wrapper - Generates Gradle wrapper files. [incubating]

Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in root project 'projectReports'.
components - Displays the components produced by root project 'projectReports'. [incubating]
dependencies - Displays all dependencies declared in root project 'projectReports'.
dependencyInsight - Displays the insight into a specific dependency in root project 'projectReports'.
help - Displays a help message.
model - Displays the configuration model of root project 'projectReports'. [incubating]
projects - Displays the sub-projects of root project 'projectReports'.
properties - Displays the properties of root project 'projectReports'.
tasks - Displays the tasks runnable from root project 'projectReports' (some of the displayed tasks may belong to subprojects).

To see all tasks and more detail, run gradle tasks --all

To see more detail about a task, run gradle help --task <task>
By default, this report shows only those tasks which have been assigned to a task group. You can do this by setting the group property for the task. You can also set the description property, to provide a description to be included in the report.

Example 4.11. Changing the content of the task report

build.gradle
dists {
    description = 'Builds the distribution'
    group = 'build'
}
You can obtain more information in the task listing using the --all option. With this option, the task report lists all tasks in the project, grouped by main task, and the dependencies for each task. Here is an example:

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
dists - Builds the distribution [api:libs, webapp:libs]
    docs - Builds the documentation
api:libs - Builds the JAR
    api:compile - Compiles the source files
webapp:libs - Builds the JAR [api:libs]
    webapp:compile - Compiles the source files

Build Setup tasks
-----------------
init - Initializes a new Gradle build. [incubating]
wrapper - Generates Gradle wrapper files. [incubating]

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
4.7.3. Show task usage details

Running gradle help --task someTask gives you detailed information about a specific task or multiple tasks matching the given task name in your multiproject build. Below is an example of this detailed information:

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
This information includes the full task path, the task type, possible commandline options and the description of the given task.

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
\--- org.codehaus.groovy:groovy-all:2.4.7

testCompile
\--- junit:junit:4.12
     \--- org.hamcrest:hamcrest-core:1.3

------------------------------------------------------------
Project :webapp - The Web application implementation
------------------------------------------------------------

compile
+--- project :api
|    \--- org.codehaus.groovy:groovy-all:2.4.7
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
org.codehaus.groovy:groovy-all:2.4.7
\--- project :api
     \--- compile
This task is extremely useful for investigating the dependency resolution, finding out where certain dependencies are coming from and why certain versions are selected. For more information please see the DependencyInsightReportTask class in the API documentation.

The built-in dependencyInsight task is a part of the 'Help' tasks group. The task needs to configured with the dependency and the configuration. The report looks for the dependencies that match the specified dependency spec in the specified configuration. If Java related plugin is applied, the dependencyInsight task is pre-configured with 'compile' configuration because typically it's the compile dependencies we are interested in. You should specify the dependency you are interested in via the command line '--dependency' option. If you don't like the defaults you may select the configuration via '--configuration' option. For more information see the DependencyInsightReportTask class in the API documentation.

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


4.8. Dry Run
Sometimes you are interested in which tasks are executed in which order for a given set of tasks specified on the command line, but you don't want the tasks to be executed. You can use the -m option for this. For example, if you run “gradle -m clean compile”, you'll see all the tasks that would be executed as part of the clean and compile tasks. This is complementary to the tasks task, which shows you the tasks which are available for execution.

4.9. Summary
In this chapter, you have seen some of the things you can do with Gradle from the command-line. You can find out more about the gradle command in Appendix D, Gradle Command Line.