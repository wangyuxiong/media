#ClassLoader详解 

## ClassLoader是神马？

先回顾一下java程序运行过程和class加载的原理：

**java程序运行过程**：一个java程序都是由一些*.class字节码文件组成。程序的运行，需要从入口函数开始（如main()），调用相关的class文件中的方法，如果该字节码文件找不到，则会抛出系统异常。
 
**class文件加载原理**：程序启动时，并不会一次性加载程序所要用的所有class文件，而是根据程序的需要，通过Java的类加载机制（ClassLoader）来动态加载某个class文件到内存当中的。只有当class文件被载入到了内存之后，才能被其它class引用。
 
**综上：ClassLoader就是用来动态加载class文件到内存当中用的。**
 
## Java默认提供的三个加载器
- **BootStrap ClassLoader**
  
  称为启动类加载器,是Java类加载层次中最顶层的类加载器，负责加载JDK中的核心类库.
  
- **Extension ClassLoader**

  称为扩展类加载器，负责加载Java的扩展类库，默认加载JAVA_HOME/jre/lib/ext/目下的所有jar.
  
- **APP ClassLoader**

  称为系统类加载器，负责加载应用程序classpath目录下的所有jar和class文件.

## ClassLoader加载类的原理
ClassLoader使用的是双亲委托模型来搜索类的，每个ClassLoader实例都有一个父类加载器的引用，虚拟机内置的类加载器（Bootstrap ClassLoader）本身没有父类加载器，但可以用作其它ClassLoader实例的的父类加载器。
 
注：*BootstrapClassLoader是jvm内部由c++实现的,并不继承java.lang.ClassLoader类,所以它不属于“ClassLoader 实例”，也没有办法在Java代码中获取到它。在JVM启动时，就已经产生，负责java平台的核心库加载。*
 
除 BootstrapClassLoader 外的类加载器加载流程:（伪代码说明）

```
// 检查类是否已被装载过
Class c = findLoadedClass(name);
if (c == null ) {
     // 指定类未被装载过
     try {
         if (parent != null ) {
             // 如果父类加载器不为空， 则委派给父类加载
             c = parent.loadClass(name, false );
         } else {
             // 如果父类加载器为空， 则委派给启动类加载加载
             c = findBootstrapClass0(name);
         }
     } catch (ClassNotFoundException e) {
         // 启动类加载器或父类加载器抛出异常后， 当前类加载器将其
         // 捕获， 并通过findClass方法， 由自身加载
         c = findClass(name);
     }
}
```

注：当一个ClassLoader实例需要加载某个类时，它会试图亲自搜索某个类之前，先把这个任务委托给它的父类加载器，这个过程是由上至下依次检查的。

首先由最顶层的类加载器Bootstrap ClassLoader试图加载，如果没加载到，则把任务转交给Extension ClassLoader试图加载，如果也没加载到，则转交给App ClassLoader 进行加载，如果它也没有加载得到的话，则返回给委托的发起者，由它到指定的文件系统或网络等URL中加载该类。

如果它们都没有加载到这个类时，则抛出ClassNotFoundException异常。否则将这个找到的类生成一个类的定义，并将它加载到内存当中，最后返回这个类在内存中的Class实例对象。
 
## ClassLoader的体系架构:
![](/media/content/BlogPost/images/14664907139203.jpg )￼

 
## 类加载器的特性
- 每个ClassLoader都维护了一份自己的名称空间，同一个名称空间里不能出现两个同名的类。
- 为了实现java安全沙箱模型顶层的类加载器安全机制, java默认采用了 ”双亲委派的加载链 “ 结构.


### 1. ClassLoader如何判定两个class是相同的呢？
JVM在判定两个class是否相同时，不仅要判断两个类名是否相同，而且要判断是否由同一个类加载器实例加载的。只有两者同时满足的情况下，JVM才认为这两个class是相同的。就算两个class是同一份class字节码，如果被两个不同的ClassLoader实例所加载，JVM也会认为它们是两个不同class。

比如网络上的一个Java类org.classloader.simple.NetClassLoaderSimple，javac编译之后生成字节码文件NetClassLoaderSimple.class，ClassLoaderA和ClassLoaderB这两个类加载器并读取了NetClassLoaderSimple.class文件，并分别定义出了java.lang.Class实例来表示这个类，对于JVM来说，它们是两个不同的实例对象，但它们确实是同一份字节码文件，如果试图将这个Class实例生成具体的对象进行转换时，就会抛运行时异常java.lang.ClassCaseException，提示这是两个不同的类型。
 
### 2. 为什么要使用双亲委托这种模型呢？

因为这样可以避免重复加载，当父亲已经加载了该类的时候，就没有必要子ClassLoader再加载一次。

也是出于安全因素的考虑，我们试想一下，如果不使用这种委托模式，那我们就可以随时使用自定义的String来动态替代java核心api中定义的类型，这样就会存在非常大的安全隐患，而双亲委托的方式，就可以避免这种情况，因为String已经在启动时就被引导类加载器（Bootstrcp ClassLoader）加载，所以用户自定义的ClassLoader永远也无法加载一个自己写的String，除非你改变JDK中ClassLoader搜索类的默认算法。

##定义自已的ClassLoader
Java中提供的默认ClassLoader，只加载指定目录下的jar和class，如果我们想加载其它位置的类或jar时，比如：我要加载网络上的一个class文件，通过动态加载到内存之后，要调用这个类中的方法实现我的业务逻辑。在这样的情况下，默认的ClassLoader就不能满足我们的需求了，所以需要定义自己的ClassLoader。
 
**定义自已的类加载器分为两步：**
1. 继承java.lang.ClassLoader
2. 重写父类的findClass方法
 
### 父类有那么多方法，为什么偏偏只重写findClass方法？

因为JDK已经在loadClass方法中帮我们实现了ClassLoader搜索类的算法，当在loadClass方法中搜索不到类时，loadClass方法就会调用findClass方法来搜索类，所以我们只需重写该方法即可。 
 
## 参考：
1. [深入分析 Java ClassLoader原理](http://blog.csdn.net/xyang81/article/details/7292380)


