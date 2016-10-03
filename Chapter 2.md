# Chapter2.概览
## <font color=green>Table of Contents 目录 </font>

<font color=green>2.1. Features 特性</font>  
<font color=green>2.2. Why Groovy?  为什么会选择Groovy</font>     
## <font color=green>2.1. Features</font>  
Here is a list of some of Gradle's features.   
以下是Gradle的一些特性的列表

**Declarative builds and build-by-convention**     
At the heart of Gradle lies a rich extensible Domain Specific Language (DSL) based on Groovy. Gradle pushes declarative builds to the next level by providing declarative language elements that you can assemble as you like. Those elements also provide build-by-convention support for Java, Groovy, OSGi, Web and Scala projects. Even more, this declarative language is extensible. Add your own new language elements or enhance the existing ones, thus providing concise, maintainable and comprehensible builds.   
Gradle的核心区域包含了基于Groovy的非常易于扩展的领域特定语言（DSL）。Gradle把declarative builds提高到了另一个水平，它提供了可声明的语言元素，你可以任意组合它们来进行构建。这些元素也为Java, Groovy, OSGi, Web和Scala项目提供了build-by-convention支持，甚至这种声明性的语言也是可以扩展的。添加你自己的新的语言元素或者加强已经存在的元素，通过这样Gradle就可以提供精确的，可维护的，易理解的构建过程。

**Language for dependency based programming** **基于依赖来编程的语言**   
The declarative language lies on top of a general purpose task graph, which you can fully leverage in your builds. It provides utmost flexibility to adapt Gradle to your unique needs.   
这种语言通过一系列的任务来完成构建过程，它提供了极大的灵活性来使Gradle适配到你的独特需求上

**Structure your build** **结构化你的构建过程**   
The suppleness and richness of Gradle finally allows you to apply common design principles to your build. For example, it is very easy to compose your build from reusable pieces of build logic. Inline stuff where unnecessary indirections would be inappropriate. Don't be forced to tear apart what belongs together (e.g. in your project hierarchy). Avoid smells like shotgun changes or divergent change that turn your build into a maintenance nightmare. At last you can create a well structured, easily maintained, comprehensible build.   
Gradle的灵活丰富的特性可以使你在构建过程中应用一些通用的设计原则。比如，你可以非常轻易的通过一些可以重用的构建逻辑片段来编写自己的构建过程。包含一些不必要的重定向之类的内部语法是不合适的。也不要强行分开有紧密联系的部分（比如在你的项目层级上）。避免到处乱改或者发散式变化这样的坏味道把你的构建方式的后续维护变成一个噩梦。最后你就可以创建一个结构良好的，易维护和理解的构建方法。

**Deep API** **非常深入的api**   
From being a pleasure to be used embedded to its many hooks over the whole lifecycle of build execution, Gradle allows you to monitor and customize its configuration and execution behavior to its very core.   
从可以愉快的内嵌到它在运行构建过程的整个生命周期的许多hooks,Grade允许你在很核心的层面监控和自定义它的配置和运行时行为。

**Gradle scales**   
Gradle scales very well. It significantly increases your productivity, from simple single project builds up to huge enterprise multi-project builds. This is true for structuring the build. With the state-of-art incremental build function, this is also true for tackling the performance pain many large enterprise builds suffer from.   
Gradle scales very well.无论是简单的单个项目的构建还是大企业级别的多项目构建，它都可以极大的提高你的生产效率。对于结构化的构建过程这没什么不可能。通过一些精巧的增量构建的函数，它也可以用来应对许多大型的企业级别的项目构建所遭受的性能问题。

**Multi-project builds** **多项目联合构建的支持**   
Gradle's support for multi-project build is outstanding. Project dependencies are first class citizens. We allow you to model the project relationships in a multi-project build as they really are for your problem domain. Gradle follows your layout not vice versa.   
Gradle对于多项目联合构建的支持也很出色。项目依赖是它最重点关照的问题。我们允许你在多项目的构建过程中，按照各个项目在你的问题上的所起的作用来进行模块化。Gradle会依照你的设计而不是反之依然。

Gradle provides partial builds. If you build a single subproject Gradle takes care of building all the subprojects that subproject depends on. You can also choose to rebuild the subprojects that depend on a particular subproject. Together with incremental builds this is a big time saver for larger builds.   
Gralde还提供了部分构建的能力。如果你仅仅是构建一个小的子项目Gradle会小心的构建所有这个子项目所依赖的子项目。你也可以选择重新构建依赖某些特定子项目的子项目。再加上增量构建，这种方式会节省很多的时间。

**Many ways to manage your dependencies** **多种方式管理你的项目依赖**   
Different teams prefer different ways to manage their external dependencies. Gradle provides convenient support for any strategy. From transitive dependency management with remote Maven and Ivy repositories to jars or directories on the local file system.   
不同的团队会有不同的方式来管理他们的外部依赖项。Gradle对任何管理策略都提供了很方便的支持,不管是带有远程Maven和Ivy仓库的transitive dependency management还是jar包或者本地系统文件目录。   

**Gradle is the first build integration tool** **Gradle是第一个构建集成工具**   
Ant tasks are first class citizens. Even more interesting, Ant projects are first class citizens as well. Gradle provides a deep import for any Ant project, turning Ant targets into native Gradle tasks at runtime. You can depend on them from Gradle, you can enhance them from Gradle, you can even declare dependencies on Gradle tasks in your build.xml. The same integration is provided for properties, paths, etc ...   
Ant任务是第一关注对象，Ant项目也是。Gradle提供了对任何Ant项目的深度导入方法，在运行时把Ant目标转换成原生的Gradle任务。你可以通过Gradle来依赖它们或者改善他们，你甚至可以在你的build.xml文件里面直接声明依赖Gradle任务，对于properties，paths等我们也做了同样的集成。

Gradle fully supports your existing Maven or Ivy repository infrastructure for publishing and retrieving dependencies. Gradle also provides a converter for turning a Maven pom.xml into a Gradle script. Runtime imports of Maven projects will come soon.   
Gradle完全支持用于发布或者获取依赖库的已有的Maven或者Ivy仓库。Gradle还提供了一个转换工具，可以把Maven pom.xml转化成Gradle脚本文件。对于Maven项目的运行时导入功能也将会马上提供。

**Ease of migration** **方便转移**   
Gradle can adapt to any structure you have. Therefore you can always develop your Gradle build in the same branch where your production build lives and both can evolve in parallel. We usually recommend to write tests that make sure that the produced artifacts are similar. That way migration is as less disruptive and as reliable as possible. This is following the best-practices for refactoring by applying baby steps.   
Gradle可以适配你现有的任何项目文件结构。因此你总是可以在你的生产环境里的同一个分支里进行Gradle构建的开发，同时我们也支持多分支同时开发。我们通常建议写测试来保证转移后的项目产品输出是一致的。转移过程对项目影响越少并且越可靠越好。

**Groovy** **Groovy语言**   
Gradle's build scripts are written in Groovy, not XML. But unlike other approaches this is not for simply exposing the raw scripting power of a dynamic language. That would just lead to a very difficult to maintain build. The whole design of Gradle is oriented towards being used as a language, not as a rigid framework. And Groovy is our glue that allows you to tell your individual story with the abstractions Gradle (or you) provide. Gradle provides some standard stories but they are not privileged in any form. This is for us a major distinguishing feature compared to other declarative build systems. Our Groovy support is not just sugar coating. The whole Gradle API is fully Groovy-ized. Adding Groovy results in an enjoyable and productive experience.   
Gradle的构建脚本是用Groovy而不是XML写的，和其他的脚本编写方式相比，这样做并不仅仅是为了展示动态语言对原生脚本的编写能力，而是后者很容易使后续的维护变得很困难。整个Gradle的设计方向是为了使你可以像使用语言一样使用它，而不是一个死板的框架结构，Groovy就是我们的粘合剂，它使你可以通过Gradle（或者你自己）提供的抽象来进行项目的构建。Gradle提供了一些标准过程但他们没有任何形式上的优先级。这是Gradle和其他的声明性构建系统相比的一大显著特点。Groovy支持也不仅仅是语法糖，整个Gradle的API都已经完全Groovy化了。增加Groovy语言的结果是得到了一个愉快并且高效的体验。

**The Gradle wrapper**   
The Gradle Wrapper allows you to execute Gradle builds on machines where Gradle is not installed. This is useful for example for some continuous integration servers. It is also useful for an open source project to keep the barrier low for building it. The wrapper is also very interesting for the enterprise. It is a zero administration approach for the client machines. It also enforces the usage of a particular Gradle version thus minimizing support issues.   
Gradle Wrapper允许你在没有安装Gradle的机器上执行Gradle构建过程，这会在比如一些持续集成服务器上非常有用，它也能有效的降低开源项目的构建难度。对于企业来说也是如此，在客户端机器上运行的时候它是不需要管理员权限的。它也可以强制使用某个特定版本的Gradle来最小化可能的版本支持问题。

**Free and open source** **免费并且开源**   
Gradle is an open source project, and is licensed under the ASL.   
Gradle是一个开源软件，它遵循ASL协议。

## <font color=green>2.2. Why Groovy? 为什么会选择Groovy</font>    
We think the advantages of an internal DSL (based on a dynamic language) over XML are tremendous when used in build scripts. There are a couple of dynamic languages out there. Why Groovy? The answer lies in the context Gradle is operating in. Although Gradle is a general purpose build tool at its core, its main focus are Java projects. In such projects the team members will be very familiar with Java. We think a build should be as transparent as possible to all team members.   
我们认为在构建脚本的时候，内置一个DSL（基于动态语言）和使用xml相比有着巨大的优势。但是现在已经有很多的动态语言了，为什么会选择Groovy呢?这是Gradle的运行环境所决定的。虽然Gradle本质上是一个通用的构建工具，但它主要关注的还是Java项目，这种项目的团队成员肯定都很熟悉Java，我们觉得一个构建系统应该对所有的团队成员尽可能的透明，清晰。

In that case, you might argue why we don't just use Java as the language for build scripts. We think this is a valid question. It would have the highest transparency for your team and the lowest learning curve, but because of the limitations of Java, such a build language would not be as nice, expressive and powerful as it could be. [1] Languages like Python, Groovy or Ruby do a much better job here. We have chosen Groovy as it offers by far the greatest transparency for Java people. Its base syntax is the same as Java's as well as its type system, its package structure and other things. Groovy provides much more on top of that, but with the common foundation of Java.   
在这种情况下，你可能说为什么我们为什么不直接使用java语言来写编译脚本。这是一个有效的问题，这样做对团队成员来说最透明并且需要的学习成本最低，但是因为Java语言的限制，这样做并不会使Gradle尽可能的nice, expressive and powerful。[1]   
像Python, Groovy 或者 Ruby在这种情况下会更有效。我们最后选择了Groovy因为对于使用Java语言的开发者来说目前它是最直接的。它的基础语法，类型系统，包结构和其他的东西跟Java一模一样，Groovy在这之上还有更多的功能，并且和Java有相同基础结构。

For Java developers with Python or Ruby knowledge or the desire to learn them, the above arguments don't apply. The Gradle design is well-suited for creating another build script engine in JRuby or Jython. It just doesn't have the highest priority for us at the moment. We happily support any community effort to create additional build script engines.   
对于有Python 或者 Ruby基础或者想学它们的Java开发者来说，上述原因并不适用。在Gradle的设计上就很适合使用JRuby或者Jython创建一个新的构建脚本引擎，但是目前对我们来说这件事并没有最高的优先级，我们很乐意为任何想要创建新的构建脚本引擎的社区力量提供支持。
    
[1] At http://www.defmacro.org/ramblings/lisp.html you find an interesting article comparing Ant, XML, Java and Lisp. It's funny that the 'if Java had that syntax' syntax in this article is actually the Groovy syntax.