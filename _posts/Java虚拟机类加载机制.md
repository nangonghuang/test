---
title: Java虚拟机类加载机制
date: 2017-11-22 15:51:43
tags:
categories: Java
---

来源：《深入理解Java虚拟机(第2版)》

# Java虚拟机类加载机制

虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验，转换解析和初始化，最终形成可以被虚拟机直接使用的Java类型，这就是虚拟机的类加载机制。

<!--more-->

## 类加载的时机

虚拟机规定了有且只有五种情况需要开始类的加载过程:

1. 遇到new、getstatic、putstatic或invokestatic这4条字节码指令时，如果类没有进行过初始化，则需要先触发其初始化。生成这4条指令的最常见的Java代码场景是：使用new关键字实例化对象的时候，读取或设置一个类的静态字段（被final修饰、已在编译期把结果放入常量池的静态字段除外）的时候，以及调用一个类的静态方法的时候。
2. 使用java.lang.reflect包的方法对类进行反射调用的时候，如果类没有进行过初始化，则需要先触发其初始化。
3. 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
4. 当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类。
5. 当使用JDK 1.7的动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果REF_getStatic、REF_putStatic、REF_invokeStatic的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化。

这五种场景中的行为称为对一个类的主动引用，其他情况称为被动引用，

```Java 
//被动引用示例一: 使用子类引用父类的静态字段，不会导致子类初始化。
public class SuperClass {
    public static int value = 123;
    static {
        System.out.println("super class init.");
    }
}

public class SubClass extends SuperClass {
    static {
        System.out.println("sub class init.");
    }
}

public static void main(String[] args) {    
    System.out.println(SubClass.value);
}

//输出：
super class init.
```

```Java
//被动引用示例二：通过数组定义来引用类，不会触发类的初始化
public static void main(String[] args) {
        SuperClass[] arr = new SuperClass[10];
}

public class SuperClass {
    public static int value = 123;
    static {
        System.out.println("super class init.");
    }
}

//输出
nonthing
```

```Java
//被动引用示例三：常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化。
public static void main(String[] args) {
        System.out.println(ConstClass.Test);
}

public class ConstClass {
    public static final String Test = "Hello world!";

    static {
        System.out.println("const class init.");
    }
}

//输出
Hello world!
```

## 类加载的过程

java虚拟机中类加载的全过程：加载、验证、准备、解析和初始化这5个阶段，注意区别于类的生命周期。

### 加载

在加载阶段，虚拟机需要完成以下3件事情：

> 1）通过一个类的全限定名来获取定义此类的二进制字节流。 
>   2）将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。 
>   3）在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口。

  数组类本身不通过类加载器创建，它是由java虚拟机直接创建的。但数组类与类加载器仍然有密切的关系，因为数组类的元素类型（Element Type，指的是数组去掉所有维度的类型）最终是要靠类加载器去创建。

  加载阶段与连接阶段的部分内容（如一部分字节码文件格式验证动作）是交叉进行的，加载阶段尚未完成，连接阶段可能已经开始，但这些夹在加载阶段之中进行的动作，仍然属于连接阶段的内容，这两个阶段的开始时间仍然保持着固定的先后顺序。

### 验证

  验证是连接阶段（连接阶段包括验证、准备、解析）的第一步，这一阶段的目的是为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。如果验证失败，会抛出java.lang.VerifyError异常。

  验证阶段大致上会完成下面4个阶段的检验动作：文件格式验证、元数据验证、字节码验证、符号引用验证。

### 准备

  准备阶段是正式为类变量分配内存并设置类变量初始值的阶段，这些变量所使用的内存都将在方法区中进行分配。这个阶段中有两个容易产生混淆的概念需要强调一下，首先，这时候进行内存分配的仅包括类变量（被static修饰的变量），而不包括实例变量，实例变量将会在对象实例化时随着对象一起分配在java堆中。其次，这里所说的初始值“通常情况下“是数据类型的零值。假设一个类变量的定义为：

```
public static int value=123;
```

  那变量value在准备阶段过后的初始值为0而不是123

  如果类字段的字段属性表中存在ConstantValue属性，那在准备阶段变量value就会被初始化为ConstantValue属性所指定的值，假设上面类变量value的定义变为：

```
public static final int value=123;
```

编译时javac将会为value生成ConstantValue属性，在准备阶段虚拟机就会根据ConstantValue的设置讲value赋值为123。

### 解析

解析阶段时虚拟机将常量池内的符号引用替换为直接引用的过程。

> **符号引用**：符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可。符号引用与虚拟机实现的内存布局无关，引用的目标并不一定已经加载到内存中。各种虚拟机实现的内存布局可以各不相同，但是它们能接受的符号引用必须都是一致的，因为符号引用的字面量形式明确定义在java虚拟机规范的Class文件格式中。
>
> **直接引用**：直接引用可以是直接指向目标的指针、相对偏移量或是一个能间接定位到目标的句柄。直接引用是和虚拟机实现的内存布局相关的，同一个符号引用在不同虚拟机实例上翻译出来的直接引用一般不会相同。如果有了直接引用，那直接引用的目标必定已经在内存中存在。

解析的动作主要针对类或接口、字段、类方法、接口方法四类符号引用进行解析。

### 初始化

  在准备阶段，变量已经赋过一次系统要求的初始值，而在初始化阶段，则根据程序员制定的主观计划去初始化类变量和其他资源，或者从另一个角度来表达：初始化阶段是执行类构造器< clinit >()方法的过程。

关于< clinit >：

* < clinit >方法是由编译器自动收集类中的所有类变量的赋值动作和静态语句块（static{}块）中的语句合并产生的，编译器收集的顺序是由语句在源文件中出现的顺序所决定的，静态语句块中只能访问到定义在静态语句块之前的变量。


* 类构造器< clinit >方法对于类和接口并不是必须的，如果一个类中没有静态初始化块，也没有类变量赋值操作，则编译器可以不为该类生成类构造器< clinit >方法。
* java虚拟机会保证一个类的< clinit >方法在多线程环境中被正确地加锁和同步，如果多个线程同时去初始化一个类，只会有一个线程去执行这个类的< clinit >方法，其他线程都需要阻塞等待，直到活动线程执行< clinit >方法完毕。 

初始化阶段，当执行完类构造器< clinit >方法之后，才会执行实例构造器的< init >方法，实例构造方法同样是按照先父类，后子类，先成员变量，后实例构造方法的顺序执行。

## 类加载器

### 类与类加载器

对于任何一个类，都需要由加载它的类加载器和这个类来确立其在JVM中的唯一性。也就是说，两个类来源于同一个Class文件，并且被同一个类加载器加载，这两个类才相等。

### 双亲委派模型

从虚拟机的角度来说，只存在两种不同的类加载器：一种是启动类加载器（Bootstrap ClassLoader），该类加载器使用C++语言实现，属于虚拟机自身的一部分。另外一种就是所有其它的类加载器，这些类加载器是由Java语言实现，独立于JVM外部，并且全部继承自抽象类java.lang.ClassLoader。

 

从Java开发人员的角度来看，大部分Java程序一般会使用到以下三种系统提供的类加载器：

1. 启动类加载器（Bootstrap ClassLoader）：负责加载JAVA_HOME\lib目录中并且能被虚拟机识别的类库到JVM内存中，如果名称不符合的类库即使放在lib目录中也不会被加载。该类加载器无法被Java程序直接引用。
2. 扩展类加载器（Extension ClassLoader）：该加载器主要是负责加载JAVA_HOME\lib\，该加载器可以被开发者直接使用。
3. 应用程序类加载器（Application ClassLoader）：该类加载器也称为系统类加载器，它负责加载用户类路径（Classpath）上所指定的类库，开发者可以直接使用该类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

双亲委派模型的工作过程为：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的加载器都是如此，因此所有的类加载请求都会传给顶层的启动类加载器，只有当父加载器反馈自己无法完成该加载请求（该加载器的搜索范围中没有找到对应的类）时，子加载器才会尝试自己去加载。

### 自定义类加载器

```Java
    /**   代码来自 http://blog.csdn.net/boyupeng/article/details/47951037
         * 一、ClassLoader加载类的顺序 
         *  1.调用 findLoadedClass(String) 来检查是否已经加载类。 
         *  2.在父类加载器上调用 loadClass 方法。如果父类加载器为 null，则使用虚拟机的内置类加载器。 
         *  3.调用 findClass(String) 方法查找类。 
         * 二、实现自己的类加载器 
         *  1.获取类的class文件的字节数组 
         *  2.将字节数组转换为Class类的实例 
         */  
        public class ClassLoaderTest {  
            public static void main(String[] args) throws InstantiationException, IllegalAccessException, ClassNotFoundException {  
                //新建一个类加载器  
                MyClassLoader cl = new MyClassLoader("myClassLoader");  
                //加载类，得到Class对象  
                Class<?> clazz = cl.loadClass("classloader.Animal");  
                //得到类的实例  
                Animal animal=(Animal) clazz.newInstance();  
                animal.say();  
            }  
        }  
        class Animal{  
            public void say(){  
                System.out.println("hello world!");  
            }  
        }  
        class MyClassLoader extends ClassLoader {  
            //类加载器的名称  
            private String name;  
            //类存放的路径  
            private String path = "E:\\workspace\\Algorithm\\src";  
            MyClassLoader(String name) {  
                this.name = name;  
            }  
            MyClassLoader(ClassLoader parent, String name) {  
                super(parent);  
                this.name = name;  
            }  
            /** 
             * 重写findClass方法 
             */  
            @Override  
            public Class<?> findClass(String name) {  
                byte[] data = loadClassData(name);  
                return this.defineClass(name, data, 0, data.length);  
            }  
            public byte[] loadClassData(String name) {  
                try {  
                    name = name.replace(".", "//");  
                    FileInputStream is = new FileInputStream(new File(path + name + ".class"));  
                    ByteArrayOutputStream baos = new ByteArrayOutputStream();  
                    int b = 0;  
                    while ((b = is.read()) != -1) {  
                        baos.write(b);  
                    }  
                    return baos.toByteArray();  
                } catch (Exception e) {  
                    e.printStackTrace();  
                }  
                return null;  
            }  
        }  
```

