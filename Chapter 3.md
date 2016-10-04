Chapter 3. Installing Gradle
Table of Contents

[3.1.Prerequisites](#3.1.prerequisites)
3.2. [Download](#download)
3.3. [Unpacking](#jump)
3.4. [Environment variables](#environment-variables)
3.5. Running and testing your installation
3.6. JVM options

* [3.1.Prerequisites](#3.1.prerequisites)
* 3.2. [Download](#download)
* [Environment variables](#environment-variables)
* [Download](#download)
* [横线](#横线)

## 3.1.Prerequisites
## Environment-variables
Gradle requires a Java JDK or JRE to be installed, version 7 or higher (to check, use java -version). Gradle ships with its own Groovy library, therefore Groovy does not need to be installed. Any existing Groovy installation is ignored by Gradle.

横线

Gradle uses whatever JDK it finds in your path. Alternatively, you can set the JAVA_HOME environment variable to point to the installation directory of the desired JDK.
## Download
## 3.2. Download
You can download one of the Gradle distributions from the Gradle web site.
## Environment variables
## <span id="jump">3.3. Unpacking</span>
The Gradle distribution comes packaged as a ZIP. The full distribution contains:

The Gradle binaries.
The user guide (HTML and PDF).
The DSL reference guide.
The API documentation (Javadoc).
Extensive samples, including the examples referenced in the user guide, along with some complete and more complex builds you can use as a starting point for your own build.
The binary sources. This is for reference only. If you want to build Gradle you need to download the source distribution or checkout the sources from the source repository. See the Gradle web site for details.
## Environment variables
For running Gradle, firstly add the environment variable GRADLE_HOME. This should point to the unpacked files from the Gradle website. Next add GRADLE_HOME/bin to your PATH environment variable. Usually, this is sufficient to run Gradle.

## 3.5. Running and testing your installation
You run Gradle via the gradle command. To check if Gradle is properly installed just type gradle -v. The output shows the Gradle version and also the local environment configuration (Groovy, JVM version, OS, etc.). The displayed Gradle version should match the distribution you have downloaded.

## 3.6. JVM options
JVM options for running Gradle can be set via environment variables. You can use either GRADLE_OPTS or JAVA_OPTS, or both. JAVA_OPTS is by convention an environment variable shared by many Java applications. A typical use case would be to set the HTTP proxy in JAVA_OPTS and the memory options in GRADLE_OPTS. Those variables can also be set at the beginning of the gradle or gradlew script.

Note that it's not currently possible to set JVM options for Gradle on the command line.