Chapter 6. The Gradle Wrapper

[原文地址v 4.4.1](https://docs.gradle.org/current/userguide/gradle_wrapper.html)

Table of Contents

6.1. Executing a build with the Wrapper
6.2. Adding the Wrapper to a project
6.3. Configuration
6.4. Authenticated Gradle distribution download
6.5. Verification of downloaded Gradle distributions
6.6. Unix file permissions

Most tools require installation on your computer before you can use them. If the installation is easy, you may think that’s fine. But it can be an unnecessary burden on the users of the build. Equally importantly, will the user install the right version of the tool for the build? What if they’re building an old version of the software?

The Gradle Wrapper (henceforth referred to as the “Wrapper”) solves both these problems and is the preferred way of starting a Gradle build.
6.1. Executing a build with the Wrapper

If a Gradle project has set up the Wrapper (and we recommend all projects do so), you can execute the build using one of the following commands from the root of the project:

    ./gradlew <task> (on Unix-like platforms such as Linux and macOS)

    gradlew <task> (on Windows using the gradlew.bat batch file)

Each Wrapper is tied to a specific version of Gradle, so when you first run one of the commands above for a given Gradle version, it will download the corresponding Gradle distribution and use it to execute the build.
IDEs

When importing a Gradle project via its wrapper, your IDE may ask to use the Gradle 'all' distribution. This is perfectly fine and helps the IDE provide code completion for the build files.

Not only does this mean that you don’t have to manually install Gradle yourself, but you are also sure to use the version of Gradle that the build is designed for. This makes your historical builds more reliable. Just use the appropriate syntax from above whenever you see a command line starting with gradle …​ in the user guide, on Stack Overflow, in articles or wherever.

For completeness sake, and to ensure you don’t delete any important files, here are the files and directories in a Gradle project that make up the Wrapper:

    gradlew (Unix Shell script)

    gradlew.bat (Windows batch file)

    gradle/wrapper/gradle-wrapper.jar (Wrapper JAR)

    gradle/wrapper/gradle-wrapper.properties (Wrapper properties)

If you’re wondering where the Gradle distributions are stored, you’ll find them in your user home directory under $USER_HOME/.gradle/wrapper/dists.
6.2. Adding the Wrapper to a project

The Wrapper is something you should check into version control. By distributing the Wrapper with your project, anyone can work with it without needing to install Gradle beforehand. Even better, users of the build are guaranteed to use the version of Gradle that the build was designed to work with. Of course, this is also great for continuous integration servers (i.e. servers that regularly build your project) as it requires no configuration on the server.

You install the Wrapper into your project by running the wrapper task. (This task is always available, even if you don’t add it to your build). To specify a Gradle version use --gradle-version on the command-line. By default, the Wrapper will use a bin distribution. This is the smallest Gradle distribution. Some tools, like Android Studio and Intellij IDEA, provide additional context information when used with the all distribution. You may select a different Gradle distribution type by using --distribution-type. You can also set the URL to download Gradle from directly via --gradle-distribution-url. If no version or distribution URL is specified, the Wrapper will be configured to use the gradle version the wrapper task is executed with. So if you run the wrapper task with Gradle 2.4, then the Wrapper configuration will default to version 2.4.

Example 6.1. Running the Wrapper task

Output of gradle wrapper --gradle-version 2.0

> gradle wrapper --gradle-version 2.0
:wrapper

BUILD SUCCESSFUL in 0s
1 actionable task: 1 executed

The Wrapper can be further customized by adding and configuring a Wrapper task in your build script, and then executing it.

Example 6.2. Wrapper task

build.gradle

task wrapper(type: Wrapper) {
    gradleVersion = '2.0'
}

After such an execution you find the following new or updated files in your project directory (in case the default configuration of the Wrapper task is used).

Example 6.3. Wrapper generated files

Build layout

simple/
  gradlew
  gradlew.bat
  gradle/wrapper/
    gradle-wrapper.jar
    gradle-wrapper.properties

All of these files should be submitted to your version control system. This only needs to be done once. After these files have been added to the project, the project should then be built with the added gradlew command. The gradlew command can be used exactly the same way as the gradle command.

If you want to switch to a new version of Gradle you don’t need to rerun the wrapper task. It is good enough to change the respective entry in the gradle-wrapper.properties file, but if you want to take advantage of new functionality in the Gradle wrapper, then you would need to regenerate the wrapper files.
6.3. Configuration

If you run Gradle with gradlew, the Wrapper checks if a Gradle distribution for the Wrapper is available. If so, it delegates to the gradle command of this distribution with all the arguments passed originally to the gradlew command. If it didn’t find a Gradle distribution, it will download it first.

When you configure the Wrapper task, you can specify the Gradle version you wish to use. The gradlew command will download the appropriate distribution from the Gradle repository. Alternatively, you can specify the download URL of the Gradle distribution. The gradlew command will use this URL to download the distribution. If you specified neither a Gradle version nor download URL, the gradlew command will download whichever version of Gradle was used to generate the Wrapper files.

For the details on how to configure the Wrapper, see the Wrapper class in the API documentation.

If you don’t want any download to happen when your project is built via gradlew, simply add the Gradle distribution zip to your version control at the location specified by your Wrapper configuration. A relative URL is supported - you can specify a distribution file relative to the location of gradle-wrapper.properties file.

If you build via the Wrapper, any existing Gradle distribution installed on the machine is ignored.
6.4. Authenticated Gradle distribution download
Security Warning

HTTP Basic Authentication should only be used with HTTPS URLs and not plain HTTP ones. With Basic Authentication, the user credentials are sent in clear text.

The Gradle Wrapper can download Gradle distributions from servers using HTTP Basic Authentication. This enables you to host the Gradle distribution on a private protected server. You can specify a username and password in two different ways depending on your use case: as system properties or directly embedded in the distributionUrl. Credentials in system properties take precedence over the ones embedded in distributionUrl.

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
6.5. Verification of downloaded Gradle distributions

The Gradle Wrapper allows for verification of the downloaded Gradle distribution via SHA-256 hash sum comparison. This increases security against targeted attacks by preventing a man-in-the-middle attacker from tampering with the downloaded Gradle distribution.

To enable this feature, download the .sha256 file associated with the Gradle distribution you want to verify.
6.5.1. Downloading the SHA-256 file

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

6.6. Unix file permissions

The Wrapper task adds appropriate file permissions to allow the execution of the gradlew *NIX command. Subversion preserves this file permission. We are not sure how other version control systems deal with this. What should always work is to execute “sh gradlew”.