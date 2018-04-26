## A New Post
md，这个prose的插入本地图片的功能好蠢啊。。。

这部分是个人笔记中的首页，所以每次拿出笔记时总会看到，不要太熟。。。
大部分内容源自JVM高级特性那本书中的内容，这个topic主要是对Java源码到JVM中可用类，最后到类卸载的一个全过程，后半部分介绍了一下类加载器的相关知识，也是挖掘地比较深。
从Java源码到Class字节码文件(编译期优化)：
![从Java源码到Class字节码文件(编译期优化)]({{site.baseurl}}//%E5%9B%BE%E7%89%871.png)
将class字节码文件加载入JVM中（加载与验证阶段）：
![将class字节码文件加载入JVM中（加载与验证阶段）]({{site.baseurl}}//%E5%9B%BE%E7%89%872.png)
将class字节码文件加载入JVM中（准备与解析阶段）：
![将class字节码文件加载入JVM中（准备与解析阶段）]({{site.baseurl}}//%E5%9B%BE%E7%89%873.png)
将class字节码文件加载入JVM中（初始化与使用阶段）：
![将class字节码文件加载入JVM中（初始化与使用阶段）]({{site.baseurl}}//%E5%9B%BE%E7%89%874.png)
将class字节码文件加载入JVM中（卸载）：
![将class字节码文件加载入JVM中（卸载）]({{site.baseurl}}//%E5%9B%BE%E7%89%875.png)
![图片5.png]({{site.baseurl}}/图片5.png)

类加载器部分：
Java中自带三种类加载器，从上至下为父加载器与子加载器的关系（不为extends继承的父子关系）
Bootstrap ClassLoader：最顶层的加载类，主要加载核心类库，%JRE_HOME%\lib下的rt.jar、resources.jar、charsets.jar和class等。另外需要注意的是可以通过启动jvm时指定-Xbootclasspath和路径来改变Bootstrap ClassLoader的加载目录。
Extention ClassLoader 扩展的类加载器，加载目录%JRE_HOME%\lib\ext目录下的jar包和class文件。还可以加载-D java.ext.dirs选项指定的目录。
Appclass Loader也称为SystemAppClass 加载当前应用的classpath的所有类。
双亲委派体制：当子加载器收到加载类的请求时，先将请求提交给父加载器，若父加载器无法加载时，子加载器才会尝试加载。
![类加载器体系结构]({{site.baseurl}}//%E5%9B%BE%E7%89%876.png)
在JDK中，Bootstrap类加载器不是用java写的，而是用C++写的，其他的类加载器例如ext，app等都是java源码。可以从JVM虚拟机入口sun.misc.Launcher看起。
![Launcher()片段]({{site.baseurl}}//%E5%9B%BE%E7%89%877.png)
这是Laucher的构造方法，其中主要做了两件事：
1、初始化extension classloader
2、初始化application classloader

这里没有出现关于bootstrap classloader的代码，前面也提到bootstrap是c++代码，而在Laucher内会将bootstrap的类加载目录传递进来，如下：
![bootstrap载入路径的指定]({{site.baseurl}}//%E5%9B%BE%E7%89%878.png)

![ClassLoader的公共抽象类]({{site.baseurl}}//%E5%9B%BE%E7%89%879.png)
前面提到父加载器，这个在所有classloader的公共抽象父类中有所体现，
getParent()实际上返回的就是一个ClassLoader对象parent，parent的赋值是在ClassLoader对象的构造方法中，它有两个情况： 
1. 由外部类创建ClassLoader时直接指定一个ClassLoader为parent。

2. 由getSystemClassLoader()方法生成，也就是在sun.misc.Laucher通过getClassLoader()获取，也就是AppClassLoader。直白的说，一个ClassLoader创建时（比如我们自己写的classloader）如果没有指定parent，那么它的parent默认就是AppClassLoader。

Bootstrap ClassLoader是由C/C++编写的，它本身是虚拟机的一部分，所以它并不是一个JAVA类，也就是无法在java代码中获取它的引用，JVM启动时通过Bootstrap类加载器加载rt.jar等核心jar包中的class文件，比如int.class,String.class都是由它加载。前面提到了，JVM初始化sun.misc.Launcher并创建Extension ClassLoader和AppClassLoader实例。并将ExtClassLoader设置为AppClassLoader的父加载器。Bootstrap没有父加载器，但是它却可以作用一个ClassLoader的父加载器。比如ExtClassLoader。所以这也是为什么ExtClassLoader的getParent方法获取为Null的原因。

![loadClass()片段]({{site.baseurl}}//%E5%9B%BE%E7%89%8710.png)
在这个loadClass内，类加载器会首先判断所要加载的类是否已经被加载过了，如果已经被加载过，就不会进入后续的委托父类加载的方法。
![loadClass()片段2]({{site.baseurl}}//%E5%9B%BE%E7%89%8711.png)
如果该类未被resolve，则会调用resolveClass来生成真正的class对象。可以看到，在loadClass内实现了双亲委派体制，所以自定义类加载器时，推荐不去覆盖loadClass，而只是去覆盖findClass来实现自己的寻找类与加载类的方法。

具体讲class文件转化为JVM可用的类是defineClass()方法的工作，它会出现在findClass方法中，大致思路是将class文件的二进制流读入内存数组中，并进行转化为可用的类

由于双亲委派体制是由java源码进行实现的，所以也可以打破双亲委派体制，比如osgi中的直接委托，线程上下文类加载器的父委托子等等。在JVM中，一个类对应其类加载器，同一个class文件被不同的类加载器加载会成为不同的类对象，因为其命名空间不一样。
在各种框架中，对于类加载器可以定义其不同的委托方式、类加载路径等等，来实现不同场合中的需求。

总结：ClassLoader用来加载class文件的。
系统内置的ClassLoader通过双亲委托来加载指定路径下的class和资源。
可以自定义ClassLoader一般覆盖findClass()方法。
ContextClassLoader与线程相关，可以获取和设置，可以绕过双亲委托的机制。双亲委托机制可以使得类与类加载器一起获得带有优先级的层次关系，保护JDK中原有类的安全。
对于类加载器有很多可以自定义与扩展的应用。
