
#Proguard

[proguard文档](https://www.guardsquare.com/en/proguard)

在android sdk目录下tools\proguard也有

## 引言

proguard处理过程,下面的每一个选项都是可选的
![proguard处理过程](https://www.guardsquare.com/files/media/guardsquare2016/Website/ProGuard/ProGuard_build_process_b.png)


Proguard首先读取输入jars(或者aars, wars, ears, zips, apks,或者目录),然后按顺序压缩,优化,混淆,预校验它们.你可以选择性的让proguard跳过一些步骤.

proguard会把输出结果写到一个或者多个输出jars(或者 aars, wars, ears, zips, apks, or directories).输入可能会包含资源文件,它们的名字和内容可能会被选择性的更新以对应于混淆的class名字.   

Proguard需要指定输入jars所需要的库jars包(或者 aars, wars, ears, zips, apks, or directories),它们是你在编译代码时所需要的库文件.proguard会在处理过程中使用它们重新构造类依赖关系,这些库jars自身会一直保持不变,你应该在你的应用里面仍然保留它们.
 
Entry points

为了决定哪些代码需要保留哪些代码需要丢弃或者混淆,你需要为你的代码指定一个或者多个entry points.这些entry points是一些常用类,比如带有main方法, applets, midlets或者activities等等.

* 在压缩阶段,proguard从这些起点开始然后递归的判断哪些类和成员会被使用,其他的类和成员会被丢弃掉.
* 优化阶段,proguard会优化代码.在这些优化中,不是entry points的类和方法可能会被改为private,static或者final,没有使用的参数可能被移除掉,有些方法可能会被内联
* 在混淆阶段,proguard会重命名不是entry points的类和类成员,在这个处理过程中,会保存entry points并且确认它们仍然可以通过原始名字访问.
* 预校验阶段是唯一的一个不需要知道entry points的阶段

## 使用 

你可以在Proguard发行版的Lib目录里面找到proguard的jar包.bin目录下有一些包含这个命令的简单的Linux和windows脚本,通常你需要把大部分的选项放在一个配置文件里面(比如myconfig.pro),然后调用:

~~~
java -jar proguard.jar @myconfig.pro
~~~

选项的顺序通常无关紧要

### 输入输出选项
~~~
 @filename 是-include filename简写

-include filename
从给定的文件名中递归的读取配置选项

-basedirectory directoryname
为在这些配置参数里面的后续的相对文件名或这些配置文件指定一个目录。

-injars class_path
指定应用程序的要处理的jars(or wars, ears, zips, or directories)路径，这些jar里面的class文件将被处理和写入到输出jar中。默认的，所有的没有包含.class的文件将会被原风不动的复制，请注意任何临时文件，特别是你直接从目录里导入文件。class路径中的条目会像filters描述的那样被过滤掉。为了更好的可读性，可以使用多个指定的类路径条目 -injars 选项。

-outjars class_path
指定处理完后要输出的jar,war,ear和目录的名称 ,在先前-injars 输入的文件将会被重写到指定的jar中，这将允许你选择输入的jar到指定输出的jar。另外，输出条目可以按filters的规则过滤掉，每一个被处理过的class文件和资源文件会被写入到第一个匹配过滤规则的输入文件中。你必须避免让输出文件重写任何的输入文件中。为了更好的可读性，可以使用多个类路径条目 -outjars选项，如果没有任何的-outjars 选项，将没有jar会被重写。

-libraryjars class_path
指定要程序所引用的 jar（比如jdk的rt.jar）( 或 war, ears,zips,或者目录)。这些jar里面的文件不会输入到输出jar中，指定的library jars 至少包含application的子类。Library class files 只被调用而不必呈现，虽然它们的存在可以提升优化的结果，这些class路径会filter的要求来过滤。为了更好的可读性，可以使用多个类路径条目 -libraryjars选项。请注意当寻找library class 时，引导路径和progurad的运行路径是不会被考虑在内的，这就意味着你要明确地指出你的代码会用到的run-time jar。虽然这会看起来比较笨重，但是它允许你运行自己的程序在不同的运行时环境中。比如，你可以通过指定合适的run-time jar就可以运行J2SE applications和JME midlets。

-skipnonpubliclibraryclasses
为了加快运行和减少proguard的使用内存，当读取library jars指定跳过non-public 类。在默认情况下ProGuard读取non-public 和public类一样，然而，如果它们没有影响输入jar里面的的程序代码，non-public 类通常是不相关的。在没有影响输出的情况下，忽略它们来加速Proguard.不幸的是，一些库，包括最近 的JSE run-time 库，包含一些非public class继承自公共的library classes，那在这种情况下你不能使用这个option。在使用这个选项ProGuard将打印出警告当找不到类的时候。

-dontskipnonpubliclibraryclasses
指定不去忽略非公共库类，在4.5以上这个是默认设置。

-dontskipnonpubliclibraryclassmembers
指定不忽略包可见的库类成员(字段和方法)。默认地，当解析库类的时候ProGuard会跳过这些类成员，项目类一般不会去引用它们。然而有的时候，程序里的类相当于库类存在于相同包。此时它们会引用他们的包可见的类成员。在这种情况下为了保持程序代码保持一致性去读取这些类的成员是有用的。

-keepdirectories [directory_filter]
指定要保存在输出jars(或wars ,ears,或directories)的目录。默认地，目录部分是会被移除的，这样可以减少jar的大小，但是当程序尝试用构造器找寻它们时会出现不愿看见的结果比如：“MyClass.class.getResource(“”)”如果这个选项没过滤器，所有的目录都会保存下来，有过滤器时只有符合过滤器的目录会保存下来。

-target version
在被处理的class文件中指定版本号。版本号可以是1.0,1.1,1.2,1.3,1.4,1.5(或者就是5),1.6(或者就是6),1.7(或者就是7)，默认地，class文件的版本号是保持不变的。比如，你可能想更新class file到Java 6,通过改变他们的版本让他们预编译。

-forceprocessing
指定输入的过程，即使输出看起来最新。这个最新的测试是基于比较指定输入，输出和配置文件或目录的时间戳。
~~~
### 保留选项
~~~
-keep [,modifier,…] class_specification
不混淆某些类。
如：-keep public class mypackage.MyMain {public static void main(java.lang.String[]);}

-keepclassmembers [,modifier,…] class_specification
不混淆类的成员，如果它们的类也被保护了它们会被保护的更好。比如你想保护可以序列化类的变量和方法。
如：-keepclassmembers class * implements java.io.Serializable {private static final java.io.ObjectStreamField[] serialPersistentFields;private void writeObject(java.io.ObjectOutputStream);private void readObject(java.io.ObjectInputStream);java.lang.Object writeReplace();java.lang.Object readResolve();}

-keepclasseswithmembers [,modifier,…] class_specification
不混淆类及其成员，但条件是所有指定的类和类成员是要存在。比如，你想保持似有的包含主方法 的application而无需显示的列出它们。
如：-keepclasseswithmembers public class * {public static void main(java.lang.String[]);}

-keepnames class_specification
是-keep,allowshrinking class_specification的缩写，保护指定的类和类的成员的名称。

keepclassmembernames class_specification
相当于-keepclassmembers,allowshrinking class_specification只保留成员名称，混淆内容。

keepclasseswithmembernames class_specification
相当于-keepclasseswithmembers,allowshrinking class_specification如果这些指定的类在压缩阶段后存在， 保护指定的类和类的成员的名称。
保持 native 方法不被混淆：-keepclasseswithmembernames class * {native ;}

-printseeds [filename]
列出匹配各种-keep选项的类和类的成员，标准输出到给定的文件。
~~~
### 压缩选项
~~~
-dontshrink
产压缩输入类文件，默认是要压缩的，除了-keep列出的class和这些类文件直接或间接引用的文件间外其它类文件都会被移除，压缩步骤在优化之后执行，因为优化可能会移除更多的类文件和类成员。

-printusage [filename]
指定输入文件中的死代码输出到标准文件中，适用于使用了压缩。

-whyareyoukeeping class_specification
打印出为什么在压缩过程中保留了这些类文件和类成员的具体原因，也适用了使用了压缩的情况。
~~~

### 优化选项
~~~
-dontoptimize
指定不优化输入类文件。默认情况下，已启用优化，所有的方法都在字节码级优化。

-optimizations optimization_filter
指定要启用和禁用的优化，在更精细的水平。只有当优化适用。

-optimizationpasses n
指定代码的优化压缩级别

-assumenosideeffects class_specification
优化试某个类中的方法不被执行，通常用来优化移除日志

-allowaccessmodification
优化时允许访问并修改有修饰符的类和类的成员

-mergeinterfacesaggressively
指定接口可以合并，即使实现类没实现所有的方法。该选项可以通过减少类的总数减少输出文件的大小。只有开启优化时可用。
~~~

### 混淆选项
~~~
-dontobfuscate
指定不混淆类文件，默认开启。

-printmapping [filename]
输出类和类成员新旧名字之间的映射到指定文件中。只有开启混淆时可用。

-applymapping filename
重用映射，映射文件未列出的类和类成员会使用随机的名称。如果代码结构从根本上发生变化，ProGuard 可能会输出映射会引起冲突警告。你可以通过添加-useuniqueclassmembernames选项来降低风险。只能指定一个映射文件。只有开启混淆时可用。

-obfuscationdictionary filename
使用文件中的关键字作为方法及字段混淆后的名称。默认使用 ‘a’，’b’ 等短名称作为混淆后的名称。你可以指定保留关键字或不相关的标识符。文件中的空格、标点符号、重复的单词及注释会被忽略。只有开启混淆时可用。

-classobfuscationdictionary filename
使用文件中的关键字作为类混淆后的名称，类似于-obfuscationdictionary。只有开启混淆时可用。

-packageobfuscationdictionary filename
使用文件中的关键字作为包混淆后的名称，类似于-obfuscationdictionary。只有开启混淆时可用。

-overloadaggressively
开启侵入性重载混淆。多个字段及方法允许同名，只要它们的参数及返回值类型不同。该选项可使处理后的代码更小(及更难阅读)。只有开启混淆时可用。注：Dalvik 不能处理重载的静态字段。

-useuniqueclassmembernames
方法同名混淆后亦同名，方法不同名混淆后亦不同名。不使用该选项时，类成员可被映射到相同的名称。因此该选项会增加些许输出文件的大小。只有开启混淆时可用。

-dontusemixedcaseclassnames
混淆时不会产生大小写混合的类名。默认混淆后的类名可以包含大写及小写。如果 jar 被解压到非大小写敏感的系统（比如 Windows），解压工具可能会将命名类似的文件覆盖另一个文件。只有开启混淆时可用。

-keeppackagenames [package_filter]
不混淆指定的包名。过滤器是由逗号分隔的包名列表。包名可以包含 ？、*、** 通配符，并且可以在包名前加上 ! 否定符。只有开启混淆时可用。

-flattenpackagehierarchy [package_name]
重新打包所有重命名的包到给定的包中。如果没参数或字符串为空，包移动到根包下。该选项是进一步混淆包名的例子，可以使处理后的代码更小更难阅读。只有开启混淆时可用。

-repackageclasses [package_name]
重新打包所有重命名的类到给定的包中。如果没参数或字符串为空，类的包会被完全移除。该选项覆盖-flattenpackagehierarchy，是进一步混淆包名的另一个例子，可以使处理后的代码更小更难阅读。曾用名为-defaultpackage。只有开启混淆时可用。

-keepattributes [attribute_filter]
保留任何可选属性。过滤器是由逗号分隔的 JVM 及 ProGuard 支持的属性列表。属性名可以包含 ？、*、** 通配符，并且可以在属性名前加上 ! 否定符。例如：处理库文件时应该加上Exceptions,InnerClasses,Signature属性。同时保留SourceFile及LineNumberTable属性使混淆后仍能获取准确的堆栈信息。同时如果你的代码有使用注解你可能会保留annotations属性。只有开启混淆时可用。

-keepparameternames
保留已保留方法的参数的名称及类型。只有开启混淆时可用。

-renamesourcefileattribute [string]
指定一个常量字符串作为SourceFile（和SourceDir）属性的值。需要被-keepattributes选项指定保留。只有开启混淆时可用。

-adaptclassstrings [class_filter]
混淆与完整类名一致的字符串。没指定过滤器时，所有符合现有类的完整类名的字符串常量均会混淆。只有开启混淆时可用。

-adaptresourcefilenames [file_filter]
以混淆后的类文件作为样本重命名指定的源文件。没指定过滤器时，所有源文件都会重命名。只有开启混淆时可用。

-adaptresourcefilecontents [file_filter]
以混淆后的类文件作为样本混淆指定的源文件中与完整类名一致的内容。没指定过滤器时，所有源文件中与完整类名一致的内容均会混淆。只有开启混淆时可用。
~~~

### 预校验选项
~~~
-dontpreverify
指定不对处理后的类文件进行预校验。默认情况下如果类文件的目标平台是 JavaMicro Edition 或 Java 6 或更高时会进行预校验。目标平台是 Android时没必要开启，关闭可减少处理时间。

-microedition
指定处理后的类文件目标平台是 Java Micro Edition。
~~~

### 常规选项
~~~
-verbose
指定处理期间打印更多相关信息。

-dontnote [class_filter]
指定配置中潜在错误或遗漏时不打印相关信息。类名错误或遗漏选项时这些信息可能会比较有用。class_filter 是一个可选的正则表达式。类名匹配时 ProGuard 不会输出这些类的相关信息。

-dontwarn [class_filter]
指定找不到引用或其他重要问题时不打印警告信息。class_filter 是一个可选的正则表达式。类名匹配时 ProGuard 不会输出这些类的相关信息。*注意：如果找不到引用的类或方法在处理过程中是必须的，处理后的代码将会无法正常运行。请明确该操作的影响时使用该选项。

-ignorewarnings
打印找不到引用或其他重要问题的警告信息，但继续处理代码。注意：如果找不到引用的类或方法在处理过程中是必须的，处理后的代码将会无法正常运行。请明确该操作的影响时使用该选项。

-printconfiguration [filename]
将已解析过的配置标准输出到指定的文件。该选项可用于调试配置。
-dump [filename]
标准输出类文件的内部结构到给定的文件中。例如，你可能要输出一个 jar 文件的内容而不需要进行任何处理。
~~~

## keep选项的modifier

~~~
includedescriptorclasses
通常在保留native方法的时候使用,确保native方法的参数类型不会被重命名,因而方法签名也会保持不变,和natibe库相兼容.

allowshrinking
不常用

allowoptimization
不常用

allowobfuscation
不常用
~~~

## keep选项总览

|保留	 |不被移除或者重命名	 |不被重命名
| - | :-: | -: | 
|类和类成员	|-keep	|-keepnames
|仅仅是类成员	|-keepclassmembers	|-keepclassmembernames
|如果拥有某成员，保留类和类成员	|-keepclasseswithmembers	|-keepclasseswithmembernames

如果你不确定需要用哪个选项,就直接使用-keep.这会保证指定的类和类成员在压缩阶段不会被移除,在混淆阶段不会被重命名

## class_specification

完整的格式模板
~~~
[@annotationtype] [[!]public|final|abstract|@ ...] [!]interface|class|enum classname
    [extends|implements [@annotationtype] classname]
[{
    [@annotationtype] [[!]public|private|protected|static|volatile|transient ...] <fields> |
                                                                      (fieldtype fieldname);
    [@annotationtype] [[!]public|private|protected|static|synchronized|native|abstract|strictfp ...] <methods> |
                                                                                           <init>(argumenttype,...) |
                                                                                           classname(argumenttype,...) |
                                                                                           (returntype methodname(argumenttype,...));
    [@annotationtype] [[!]public|private|protected|static ... ] *;
    ...
}]
~~~

* 方括号[]表示内容是可选的,省略号...表示任意数量的条目都可以,竖线|表示多选1,()表示指定为一组.

* **class**表示类或者接口. **interface** 只匹配接口. The **enum** 匹配枚举类. 带有!符号表示不匹配此类型.
* 每一个类名都需要使用完整名.内部类使用"$"符号间隔,比如java.lang.Thread$State.类名可以包含匹配以下通配符的正则表达式:
  *  ?   匹配类名里面的任意单个字符,不包括包名分隔符.比如"mypackage.Test?" 匹配 "mypackage.Test1" 和 "mypackage.Test2", 不匹配"mypackage.Test12".
  *  \*  匹配类名的任意一部分,不包括包名分隔符.比如,"mypackage.\*Test\*" 匹配 "mypackage.Test" 和 "mypackage.YourTestApplication", 不匹配"mypackage.mysubpackage.MyTest",或者,更普遍的,"mypackage.*" 匹配所有的 "mypackage"包下的类,但是不包含子包下的类.
  *  \** 匹配类名的任意部分,也许包含任意数量的包分隔符,比如 \"\**.Test\"匹配除了根目录以外所有包下的Test类. 或者"mypackage.**" 匹配"mypackage"包和它的子包下的所有类.
* 为了方便和向下兼容,\*表示任意的类,无论是哪个包的
* @符号表示被限制为指定注解标记的类和类成员.
* 成员变量和方法
    * \<init\>    匹配任意的构造函数
    * \<fields\>  匹配任意成员变量
    * \<methods\> 匹配任意方法
    * \*        匹配任意成员变量或者方法

    成员变量和方法也可以使用正则匹配,名字里面可以包含以下通配符:
    * ? 匹配方法里面的单个字符
    * \* 匹配方法的任意一部分
    类型里面可以包含以下通配符:
    * % 匹配任意的原始类型(boolean,int等,不包括void)
    * ? 匹配类名里面单个字符
    * \* 匹配类名里面的一部分,不包括包分隔符
    * \** 匹配类名的一部分,可能包含任意数量的包分隔符
    * \*** 匹配任意类型(原始或者非原始,数组或者非数组)
    * ... 匹配任意类型的任意数量的参数

## 示例代码

### 移除Android日志
~~~
-assumenosideeffects class android.util.Log { 
    public static boolean isLoggable(java.lang.String, int); 
    public static int v(...); 
    public static int i(...); 
    public static int w(...); 
    public static int d(...); 
    public static int e(...); 
} 
~~~
### 处理native方法
~~~
-keepclasseswithmembernames,includedescriptorclasses class * { 
    native <methods>; 
} 
~~~

### 处理序列化方法
~~~
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    static final java.io.ObjectStreamField[] serialPersistentFields;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}

~~~
### 处理bean
~~~
-keep class mybeans.** { 
    void set*(***); 
    void set*(int, ***); 
    boolean is*(); 
    boolean is*(int); 
    *** get*(); 
    *** get*(int); 
} 
~~~
### 混淆包名
~~~
-repackageclasses 'myobfuscated' 
-allowaccessmodification
~~~

混淆前
~~~
mycompany.myapplication.MyMain 
mycompany.myapplication.Foo
mycompany.myapplication.Bar 
mycompany.myapplication.extra.FirstExtra 
mycompany.myapplication.extra.SecondExtra 
mycompany.util.FirstUtil mycompany.util.SecondUtil
~~~
混淆后
~~~
mycompany.myapplication.MyMain 
myobfuscated.a 
myobfuscated.b 
myobfuscated.c 
myobfuscated.d 
myobfuscated.e 
myobfuscated.f 
~~~
### 注解处理
~~~
-keep @proguard.annotation.Keep class *

-keepclassmembers class * {
    @proguard.annotation.Keep *;
}
~~~

### Android的四大组件等等
~~~

# RemoteViews might need annotations.
-keepattributes *Annotation*

# Preserve all fundamental application classes.
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Application
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider

# Preserve all View implementations, their special context constructors, and
# their setters.
-keep public class * extends android.view.View {
    public <init>(android.content.Context);
    public <init>(android.content.Context, android.util.AttributeSet);
    public <init>(android.content.Context, android.util.AttributeSet, int);
    public void set*(...);
}

# Preserve all classes that have special context constructors, and the
# constructors themselves.
-keepclasseswithmembers class * {
    public <init>(android.content.Context, android.util.AttributeSet);
}

# Preserve all classes that have special context constructors, and the
# constructors themselves.
-keepclasseswithmembers class * {
    public <init>(android.content.Context, android.util.AttributeSet, int);
}

# Preserve the special fields of all Parcelable implementations.
-keepclassmembers class * implements android.os.Parcelable {
    static android.os.Parcelable$Creator CREATOR;
}

# Preserve static fields of inner classes of R classes that might be accessed
# through introspection.
-keepclassmembers class **.R$* {
  public static <fields>;
}

# Preserve the required interface from the License Verification Library
# (but don't nag the developer if the library is not used at all).
-keep public interface com.android.vending.licensing.ILicensingService
-dontnote com.android.vending.licensing.ILicensingService

# The Android Compatibility library references some classes that may not be
# present in all versions of the API, but we know that's ok.
-dontwarn android.support.**

# Preserve all native method names and the names of their classes.
-keepclasseswithmembernames class * {
    native <methods>;
}

# Preserve the special static methods that are required in all enumeration
# classes.
-keepclassmembers class * extends java.lang.Enum {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

# Explicitly preserve all serialization members. The Serializable interface
# is only a marker interface, so it wouldn't save them.
# You can comment this out if your application doesn't use serialization.
# If your code contains serializable classes that have to be backward 
# compatible, please refer to the manual.

-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    static final java.io.ObjectStreamField[] serialPersistentFields;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}

# Your application may contain more items that need to be preserved; 
# typically classes that are dynamically created using Class.forName:

# -keep public class mypackage.MyClass
# -keep public interface mypackage.MyInterface
# -keep public class * implements mypackage.MyInterface
~~~


在ANDROID_SDK的tools\proguard\examples 目录下,有更多的示例用法

感谢:
http://blog.csdn.net/byhook/article/details/52529617