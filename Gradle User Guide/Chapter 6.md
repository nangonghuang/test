Chapter 6. The Gradle Wrapper

[原文地址v 4.4.1](https://docs.gradle.org/current/userguide/gradle_wrapper.html)

Table of Contents

* [6.1. 使用Wrapper执行一次构建任务](#使用wrapper执行一次构建任务)
* [6.2. 为项目添加Wrapper](#62-为项目添加wrapper)
* [6.3. 配置](#63-配置)
* [6.4. 下载认证的gradle发行版](#64-下载认证的gradle发行版)
* [6.5. 已经下载的gradle发行版的校验](#65-已经下载的gradle发行版的校验)
* [6.6. Unix文件权限](#66-unix文件权限)

大部分的工具在你能使用它们之前都需要先安装，如果安装过程很简单，你会觉得那还行。但是对于执行构建任务的用户来说，这是一种没必要的负担，同样重要的是，用户是不是安装了正确的工具版本？如果它们使用老版本的软件执行构建会怎样？
Gradle Wrapper（后面简称为Wrapper)就是用来解决这种问题的，而且我们更推荐用这种方式来执行构建任务。

## 使用Wrapper执行一次构建任务
如果一个Gradle项目设置了Wrapper(并且我们推荐所有的项目都这样做)，你可以在项目的根目录下使用下面的命令执行构建：
~~~
    ./gradlew <task> (on Unix-like platforms such as Linux and macOS)

    gradlew <task> (on Windows using the gradlew.bat batch file)
~~~

每个Wrapper都和一个特定版本的Gradle绑定了，因此当你第一次运行上面的命令时，它会下载对应的Gradle发行版并且用它来执行构建任务。
> IDEs   
> 通过Wrapper导入一个gradle项目的时候，你的IDE可能会请求使用Gradle的'all'发行版，这样做挺好的，并且可以帮助IDE为build脚本提供代码提示功能

这样做不仅意味着你不用手动去安装Gradle，而且可以保证你使用的一定是这次构建所指定的版本。这会使你以前的构建更加可靠。当你看到一个命令行以gradle开头的时候，使用上面提到的语法就可以了，不管是在user guidel,Stack Overflow，文章里面或者其他任何地方。

为了保证完整性，确保你没有删掉任何重要的文件，这里是在一个Gradle项目中组成Wrapper的文件和目录：

* gradlew (Unix Shell script)

* gradlew.bat (Windows batch file)

* gradle/wrapper/gradle-wrapper.jar (Wrapper JAR)

* gradle/wrapper/gradle-wrapper.properties (Wrapper properties)

如果你在想Gradle的发行版文件在哪里存储的，你可以在你的用户目录下找到它们($USER_HOME/.gradle/wrapper/dists)

## 6.2. 为项目添加Wrapper

Wrapper是你应该添加到版本控制里的东西。通过为你的项目添加wrapper,任何人都可以直接使用它而不需要事先安装gradle。更好的是，用户能确保使用设定版本的gradle来进行构建。当然，这对于持续集成服务器来说也比较好，因为它不需要在服务器上进行环境的配置。

你可以通过wrapper任务来安装Wrapper到你的项目里面去(这个任务总是可用的，即使你没有在构建里面添加它)。在命令行里面使用--gradle-version来指定Gradle版本。默认情况下，Wrapper会使用一个bin发行版，这是最小的Gradle发行版。你可以用过使用--distribution-type选择不同版本。你也可以通过--gradle-distribution-url直接指定下载Gradle的URL，如果没有指定版本或者发行版URL，Wrapper将会被配置成使用wrapper任务的gradle版本。因此如果你使用Gradle2.4执行wrapper,那么Wrapper配置会默认设置为2.4版本。

**Example 6.1. Running the Wrapper task**

Output of gradle wrapper --gradle-version 2.0
~~~
> gradle wrapper --gradle-version 2.0
:wrapper

BUILD SUCCESSFUL in 0s
1 actionable task: 1 executed
~~~

通过在你的build脚本添加和配置Wrapper任务然后执行它。这样你可以进行更加详细的个人定制。

Example 6.2. Wrapper task

build.gradle
~~~
task wrapper(type: Wrapper) {
    gradleVersion = '2.0'
}
~~~
通过执行这个任务你可以在项目目录下发现下面的文件(假设你的Wrapper任务使用的是默认的配置)

**Example 6.3. Wrapper generated files**

Build layout
~~~
simple/
  gradlew
  gradlew.bat
  gradle/wrapper/
    gradle-wrapper.jar
    gradle-wrapper.properties
~~~
所有的这些文件都应该提交到版本控制里面去。这只需要做一次。当这些文件被添加到项目里面的时候，项目就应该可以使用gradlew命令行了。gradlew命令和gradle命令的使用方式一摸一样

如果你想切换到Gradle的新版本，你不需要重新运行wrapper任务，直接更改gradle-wrapper.properties文件里面相对应的条目就可以了，但是如果你想使用Gradle wrapper的新功能，那么你还是需要重新生成wrapper文件。

## 6.3. 配置

如果你通过gradlew使用Gradle，Wrapper会检查对应的Gradle发行版是不是可用的，如果是，它会把gradlew命令所包含所有的参数都原始的传给gradle命令。如果没有找到可用的Gradle发行版，它会首先去下载。

当你配置Wrapper任务的时候，你可以指定你想要使用的Gradle版本。gradlew命令会从Gradle仓库下载合适的发行版，你也可以指定Gradle发行版的下载链接。gradlew命令会使用这个链接去下载发行版。如果你既没有指定版本也没有指定链接，gradlew会下载生成wrapper文件的Gradle的版本.

更详细的Wrapper配置细节,可以查看API文档里面的Wrapper类

如果你不想在使用gradlew构建项目的时候进行任何下载,那么把Gradle发行版的zip文件添加到你的Wrapper配置中指定的目录并且添加它到版本控制里面去就行了.
我们支持相对路径,你可以指定一个相对于gradle-wrapper.properties文件的发行版文件.

如果你使用这样的Wrapper,那么这台机器上任何存在的Gradle发行版都会被忽略.

## 6.4. 下载认证的Gradle发行版

> 安全警告
> HTTP Basic Authentication should only be used with HTTPS URLs and not plain HTTP ones. With Basic Authentication, the user credentials are sent in clear text.

Gradle Wrapper可以使用HTTP Basic Authentication从服务器上下载Gradle发行版.This enables you to host the Gradle distribution on a private protected server. You can specify a username and password in two different ways depending on your use case: as system properties or directly embedded in the distributionUrl. Credentials in system properties take precedence over the ones embedded in distributionUrl.

Using system properties can be done in the .gradle/gradle.properties file in the user’s home directory, or by other means, see Section 12.1, “Configuring the build environment via gradle.properties”.

Example 6.4. Specifying the HTTP Basic Authentication credentials using system properties

gradle.properties. 

systemProp.gradle.wrapperUser=username
systemProp.gradle.wrapperPassword=password

Embedding credentials in the distributionUrl in the gradle/wrapper/gradle-wrapper.properties file also works. Please note that this file is to be committed into your source control system. Shared credentials embedded in distributionUrl should only be used in a controlled environment.

Example 6.5. Specifying the HTTP Basic Authentication credentials in distributionUrl

gradle-wrapper.properties. 

distributionUrl=https://username:password@somehost/path/to/gradle-distribution.zip

This can be used in conjunction with a proxy, authenticated or not. See Section 12.3, “Accessing the web via a proxy” for more information on how to configure the Wrapper to use a proxy.

## 6.5. 已经下载的Gradle发行版的校验

The Gradle Wrapper allows for verification of the downloaded Gradle distribution via SHA-256 hash sum comparison. This increases security against targeted attacks by preventing a man-in-the-middle attacker from tampering with the downloaded Gradle distribution.

To enable this feature, download the .sha256 file associated with the Gradle distribution you want to verify.
### 6.5.1. 下载SHA-256文件

You can download the .sha256 file by clicking on one of the sha256 links on whichever page you used to download your distribution:

    https://gradle.org/install

    https://gradle.org/releases

    https://gradle.org/release-candidate

    https://gradle.org/nightly

The format of the file is a single line of text that is the SHA-256 hash of the corresponding zip file.

Add the downloaded hash sum to the gradle-wrapper.properties using the distributionSha256Sum property.

Example 6.6. Configuring SHA-256 checksum verification

gradle-wrapper.properties. 

distributionSha256Sum=371cb9fbebbe9880d147f59bab36d61eee122854ef8c9ee1ecf12b82368bcf10

## 6.6. Unix文件权限

Wrapper任何添加了合适的文件权限来允许gradle 命令的执行.Subversion允许了这些文件权限.我们还不清楚其他的版本控制系统是怎么处理的.执行"sh gradlew"应该总是可行的.