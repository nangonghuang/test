# Chapter 41. Writing Custom Plugins  编写自定义插件
## Table of Contents 目录

1. [创建插件](#创建插件)
2. [编写一个简单的插件](#编写一个简单的插件)
3. [使插件可以配置](#使插件可以配置)
4. [在自定义的插件或者任务的文件中工作]
5. [适配扩展属性和任务属性]
6. [在独立项目中使用]
7. [为插件提供DSL配置]


## 创建插件

你可以把插件的源码放在以下几个地方
* Build script  编译脚本以内
    你可以把源码直接放在build脚本里面，这种方式的好处是插件会被自动的编译和引入到脚本的classpath，你什么别的东西都不用做。可是，这个插件在这个build脚本以外就不可见了，因此你不能在定义这个插件的build脚本以外的地方复用它
* buildSrc project  buildSrc文件夹内
    你可以把插件源码放在rootProjectDir/buildSrc/src/main/groovy 目录下.Gradle也将会自动编译和测试这个插件并且把它添加到编译脚本的classpath里。这个插件对编译过程中用到的build脚本都可见。可是，它依然对build脚本以外的地方不可见，这种情况下你也不能复用它。
    在第43章对buildSrc文件夹有更详细的描述。
* Standalone project 单独的项目
    你可以为你的插件单独创建一个项目。这个项目可以导出和发布一个jar，你可以在各种各样的情形下和别人共享这个插件。通常，这个jar可能包含多个插件，or bundle several related task classes into a single library.或者两者都有。
在我们的例子中，会从第一种情况开始。为了简化，我们会先创建一个单独的项目。

## 编写一个简单的插件

要创建一个插件，你需要编写一个类实现插件的Plugin接口。当这个插件在项目中应用的时候，gradle会创建一个插件类的实例然后调用它的Plugin.apply(T)方法。这个项目对象会作为参数传递给插件，用来按照自己的需要来配置。下面是一个插件示例，添加了一个hello任务到项目中。

*Example 41.1. A custom plugin*
build.gradle
~~~Groovy
class GreetingPlugin implements Plugin<Project> {
    void apply(Project project) {
        project.task('hello') {
            doLast {
                println 'Hello from the GreetingPlugin'
            }
        }
    }
}
// Apply the plugin
apply plugin: GreetingPlugin
~~~
 Output of gradle -q hello
> Hello from the GreetingPlugin

值得一提的是对于每一个应用这个插件的项目，都创建了一个插件类实例，而且，插件类是一个泛型类。这个例子中它接受Project类型作为类型参数。插件还可以接受Setting类型，这种情况下插件会被应用到settings脚本，或者是Gradle类型，这种情况下插件会被应用到初始化脚本中。


## 使插件可以配置
大部分的插件都需要从build脚本中获取某些配置。我们可以通过使用extension对象来完成这个功能。Gradle项目会自动和一个ExtensionContainer对象关联，这个对象包含了项目所使用的插件的所有设置和属性。你可以通过添加一个extension对象到这个container来为你的插件提供配置功能。一个extension对象通常是一个简单的java veab对象。Groovy对于实现这种对象很方便，因为它不需要设置getter和setter方法。Java和Kotlin也是可选项。
在这里我们添加了一个extension对象到项目中，用来配置greeting插件。    


*Example 41.2. A custom plugin extension*

build.gradle
~~~Groovy
class GreetingPluginExtension {
    String message = 'Hello from GreetingPlugin'
}

class GreetingPlugin implements Plugin<Project> {
    void apply(Project project) {
        // Add the 'greeting' extension object
        def extension = project.extensions.create('greeting', GreetingPluginExtension)
        // Add a task that uses configuration from the extension object
        project.task('hello') {
            doLast {
                println extension.message
            }
        }
    }
}

apply plugin: GreetingPlugin

// Configure the extension
greeting.message = 'Hi from Gradle'
~~~
Output of gradle -q hello

> Hi from Gradle

在这个例子中，GreetingPluginExtension是一个包含message属性的Groovy对象，这个extension对象被用为greeting这个名字添加到插件中，然后它就在插件项目中可以作为extension对象使用了。
通常，在一个单独的插件中，你会需要特别设置一些相关的属性，Gradle会为每一个extension对象添加一个配置用的闭包块，这样你可以把一组设置放到一起。下面是示例     

*Example 41.3. A custom plugin with configuration closure*

build.gradle
~~~Groovy
class GreetingPluginExtension {
    String message
    String greeter
}

class GreetingPlugin implements Plugin<Project> {
    void apply(Project project) {
        def extension = project.extensions.create('greeting', GreetingPluginExtension)
        project.task('hello') {
            doLast {
                println "${extension.message} from ${extension.greeter}"
            }
        }
    }
}

apply plugin: GreetingPlugin

// Configure the extension using a DSL block
greeting {
    message = 'Hi'
    greeter = 'Gradle'
}
~~~
Output of gradle -q hello

> Hi from Gradle

在这个例子中，多个设置作为一组放到了greeting闭包内。这个闭包的名字需要和extension对象的名字一致。然后，当这个闭包被执行，extension对象的相应属性就会被设置为闭包里的值。