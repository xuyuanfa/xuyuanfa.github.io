---
title: JDK特性收集整理
categories:
- Java
tags:
- JDK
---


- Java 5 主要新特性
- Java 6 主要新特性
- Java 7 主要新特性
- Java 8 主要新特性
- Java 9 主要新特性
- Java 10 主要新特性
- Java 11 主要新特性
- Java 12 主要新特性
<!--more-->

# Java 5 主要新特性
[查看Java 5 官方参考](https://docs.oracle.com/javase/1.5.0/docs/relnotes/features.html)
## 泛型
泛型，即“参数化类型”。将类型由原来的具体的类型参数化。在泛型使用过程中，操作的数据类型被指定为一个参数，这种参数类型可以用在类、接口和方法中，分别被称为泛型类、泛型接口、泛型方法。
https://blog.csdn.net/s10461/article/details/53941091
如果静态方法要使用泛型的话，必须将静态方法也定义成泛型方法 。
泛型的上下边界添加，必须与泛型的声明在一起。
通配符?表示未知类型，可以结合上下边界。
list的通配符上下界，extends上边界只支持读，super下边界支持写，读时要强制转型。

## 增强for循环(For-Each)
foreach语句是java5的新特征之一，在遍历数组、集合方面，foreach为开发人员提供了极大的方便。

### 自动装箱和拆箱
包装类是引用传递而基本类型是值传递， 基本类型的变量和对象的引用变量都是在函数的栈内存中分配，而实际的对象是在存储堆内存中。
基本数据(Primitive)类型的自动装箱(autoboxing)、拆箱(unboxing)是自J2SE 5.0开始提供的功能。
整型池：如果不是-128到127之间的 int 类型转换为 Integer 对象，则直接返回一个新的对象。否则直接从cache数组中获得。
优先选择基本数据类型，例如Integer传递给Long，自动拆箱经过int再加宽到long才装箱为Long。

## 枚举
Java中Enum的本质其实是在编译时期转换成对应的类的形式。
枚举（enum）类型是Java 5新增的特性，它是一种新的类型，允许用常量来表示特定的数据片断，而且全部都以类型安全的形式来表示。
枚举本质上是通过普通的类来实现的，只是编译器为我们进行了处理。每个枚举类型都继承自java.lang.Enum，并自动添加了values和valueOf方法。 而每个枚举常量是一个静态常量字段，使用内部类实现，该内部类继承了枚举类。所有枚举常量都通过静态代码块来进行初始化，即在类加载期间就初始化。 另外通过把clone、readObject、writeObject这三个方法定义为final的，同时实现是抛出相应的异常。这样保证了每个枚举类型及枚举常量都是不可变的。可以利用枚举的这两个特性来实现线程安全的单例。 

## 可变长度参数
在 Java 5 中提供了变长参数，允许在调用方法时传入不定长度的参数。变长参数是 Java 的一个语法糖，本质上还是基于数组的实现。
Java的可变参数，会被编译器转型为一个数组。变长参数在编译为字节码后，在方法签名中就是以数组形态出现的。这两个方法的签名是一致的，不能作为方法的重载。如果同时出现，是不能编译通过的。

## 静态导入
静态导入之后在类中就可以直接用方法名调用静态方法，而不必用 类名.方法名 的方式调用，同样静态的变量也可以直接使用。
注意：这样的写法虽然可以简化代码的书写，但是代码的可读性下降。

## 注解（annotation）
注解可以声明在包、类、字段、方法、局部变量、方法参数等的前面，用来对这些元素进行说明和注释。 注解都不会直接影响到程序的语义，也不会改变程序的编译方式，只是作为注解（标识）存在，相当于给它们添加了额外的辅助信息，我们可以通过反射机制编程实现对这些元数据的访问。 
作用分类：
编写文档：通过代码里标识的元数据生成文档【生成文档doc文档】
代码分析：通过代码里标识的元数据对代码进行分析【使用反射】
编译检查：通过代码里标识的元数据让编译器能够实现基本的编译检查【例如Override】 


# Java 6 主要新特性
[查看Java 6 官方参考](https://www.oracle.com/technetwork/java/javase/features-141434.html)

## 新增 Desktop类 和 SystemTray类
前者可以用来打开系统默认浏览器浏览指定的URL，打开系统默认邮件客户端给指定的邮箱发邮件，用默认应用程序打开或编辑文件(比如，用记事本打开以txt为后缀名的文件)，用系统默认的打印机打印文档；后者可以用来在系统托盘区创建一个托盘程序。

## 使用JAXB2实现对象与XML之间的映射
JAXB(Java Architecture for XML Binding)可以将一个 Java 对象转变成为 XML 格式，反之亦然。

## STAX
STAX(JSR 173) 是 JDK1.6 中除了 DOM 和 SAX 之外的又一种处理XML文档的API。
StAX(The Streaming API for XML) 是一种利用拉模式解析(pull-parsing)XML文档的API。StAX 通过提供一种基于事件迭代器(Iterator)的API让程序员去控制xml文档解析过程，程序遍历这个事件迭代器去处理每一个解析事件，解析事件可以看做是程序拉出来的，也就是程序促使解析器产生一个解析事件然后处理该事件，之后又促使解析器产生下一个解析事件，如此循环直到碰到文档结束符；
SAX 也是基于事件处理xml文档，但却是用推模式解析，解析器解析完整个xml文档后，才产生解析事件，然后推给程序去处理这些事件；
DOM 采用的方式是将整个xml文档映射到一颗内存树，这样就可以很容易地得到父节点和子结点以及兄弟节点的数据，但如果文档很大，将会严重影响性能。

## Compiler API
动态编译 Java 源文件，Compiler API结合反射功能就可以实现动态的产生Java代码并编译执行这些代码，有点动态语言的特征。
步骤：
通过 Controller 接口，提交Java代码，代码统一实现Callable接口；
动态编译Java代码为字节码；
使用类加载器加载编译的字节码；
使用反射拿到类的元数据信息，执行call方法，完成对应的功能。 

## 轻量级Http Server API
JDK1.6 提供了一个简单的 Http Server API，据此我们可以构建自己的嵌入式Http Server，它支持Http和Https协议，提供了HTTP1.1的部分实现，没有被实现的那部分可以通过扩展已有的Http Server API来实现，程序员必须自己实现HttpHandler接口，HttpServer会调用HttpHandler实现类的回调方法来处理客户端请求，在这里，我们把一个Http请求和它的响应称为一个交换，包装成HttpExchange类，HttpServer负责将HttpExchange传给HttpHandler实现类的回调方法。

## 插入式注解处理API(Pluggable Annotation Processing API)
插入式注解处理API(Pluggable Annotation Processing API) - (JSR 269)：提供一套标准API来处理Annotations(JSR 175)
JSR 269 用 Annotation Processor 在编译期间而不是运行期间处理注解【Annotation】，Annotation Processor相当于编译器的一个插件，所以称为插入式注解处理。举例，类库lombok，@Setter@Getter使编译代码添加set和get方法。

## 新增 Console 类
JDK1.6 中提供了 java.io.Console 类专用来访问基于字符的控制台设备。

## 对脚本语言的支持
JDK1.6 增加了对脚本语言的支持(JSR 223)，原理上是将脚本语言编译成 bytecode，这样脚本语言也能享用 Java 平台的诸多优势包括可移植性、安全等，另外由于现在是编译成 bytecode 后再执行，所以比原来边解释边执行效率要高很多。

## Common Annotations
Common annotations原本是Java EE 5.0(JSR 244)规范的一部分，现在SUN把它的一部分放到了Java SE 6.0中.随着Annotation元数据功能(JSR 175)加入到Java SE 5.0里面，很多Java 技术(比如EJB,Web Services)都会用Annotation部分代替XML文件来配置运行参数（或者说是支持声明式编程,如EJB的声明式事务）, 如果这些技术为通用目的都单独定义了自己的Annotations,显然有点重复建设, 所以,为其他相关的Java技术定义一套公共的Annotation是有价值的，可以避免重复建设的同时，也保证Java SE和Java EE 各种技术的一致性.

## 增加Java DB数据库
新安装了 JDK 6 的程序员们也许会发现，除了传统的 bin、jre 等目录，JDK 6 新增了一个名为 db 的目录。这便是 Java 6 的新成员：Java DB。这是一个纯 Java 实现、开源的数据库管理系统（DBMS），源于 Apache 软件基金会（ASF）名下的项目 Derby。它只有 2MB 大小，对比动辄上 G 的数据库来说可谓袖珍。但这并不妨碍 Derby 功能齐备，支持几乎大部分的数据库应用所需要的特性。更难能可贵的是，依托于 ASF 强大的社区力量，Derby 得到了包括 IBM 和 Sun 等大公司以及全世界优秀程序员们的支持。这也难怪 Sun 公司会选择其 10.2.2 版本纳入到 JDK 6 中，作为内嵌的数据库。这就好像为 JDK 注入了一股全新的活力：Java 程序员不再需要耗费大量精力安装和配置数据库，就能进行安全、易用、标准、并且免费的数据库编程。
https://www.ibm.com/developerworks/cn/java/j-lo-jse65/index.html

## JDBC 4.0版本升级
自动加载驱动，在 JDBC 4.0 之前，编写 JDBC 程序都需要加上以下这句有点丑陋的代码： Class.forName("org.apache.derby.jdbc.EmbeddedDriver").newInstance();
JDBC 4.0 的规范规定，所有 JDBC 4.0 的驱动 jar 文件必须包含一个 java.sql.Driver，它位于 jar 文件的 META-INF/services 目录下。基于SPI机制实现自动加载驱动。
其他，RowId、SQLXML、SQLExcpetion的增强。

# Java 7 主要新特性
[查看Java 7 官方文档](https://www.oracle.com/technetwork/java/javase/jdk7-relnotes-418459.html)


## 支持动态类型语言（InvokeDynamic） 
其实在 JDK6 中就已经支持 JSR223：Java 平台上的脚本语言，通过一个脚本语言引擎在 JVM 上执行 JavaScript 等脚本语言。但由于 JVM 本身的设计原来是针对 Java 这种静态类型语言的，所以脚本语言无论是解释执行，或者是编译时用虚拟类型，还是运用反射机制，都会对执行效率产生很大程度的影响。 JSR292 的实现增加了一个 InvokeDynamic 的字节码指令来支持动态类型语言，使得在把源代码编译成字节码时并不需要确定方法的签名，即方法参数的类型和返回类型。当运行时执行 InvokeDynamic 指令时，JVM 会通过新的动态链接机制 Method Handles，寻找到真实的方法。 

## G1 垃圾回收器（Garbage-First Collector）
G1 垃圾回收器是一个服务器端的垃圾回收器，针对大内存多核 CPU 的环境，目的在于减少 Full GC 带来的暂停次数，增加吞吐量。从长远来看，G1 会代替 Concurrent Mark-Sweep Collector（CMS）。实现上，G1 在堆上分配一系列相同大小的连续区域，然后在回收时先扫描所有的区域，按照每块区域内存活对象的大小进行排序，优先处理存活对象小的区域，即垃圾对象最多的区域，这也是 Garbage First 这个名称的由来。G1 把要收集的区域内的存活对象合并并且复制到其他区域，从而避免了 CMS 遇到的内存碎片问题。此外，G1 采用了一个可预测暂停时间模型来达到软实时的要求。

## 语言改进（Project Coin）
Coin 项目提供了一系列语言上的改进，为 Java 开发者提供了更多的便利。其中包括了支持 String 的 switch 语句，在 try 之后自动关闭资源（try-with-resources），更简洁的泛型，数字可以用下划线分割和多重 catch 的改进等等。
支持String的switch语句
原理是基于hashCode和equal，hashCode返回整型，equal防碰撞做安全校验，从反编译看switch的case只支持整型。
二进制整型字面值
数值字面值中的下划线
扩展的try语句，称为带资源的try(try-with-resources)语句，这种try语句支持自动资源管理。
对异常处理进行了增强，单个catch子句能够捕获两个或更多个异常(multi-catch)，并且对重新抛出的异常提供了更好的类型检查。
对与某些方法（参数的长度可变）类型关联的编译器警告进行了改进，尽管语法没有发生变化，并且警告具有更大的控制权。
构造泛型实例时的类型推断

## 核心类库改进
ClassLoader 新增 API
为了防止自定义多线程 ClassLoad 产生的死锁问题，java.lang.ClassLoader 类增加了以下 API。
protected Object getClassLoadingLock(String className) 
protected static boolean registerAsParallelCapable()
URLClassLoader 新增 API
URLClassLoader 新增 close 方法可以关闭该类加载器打开的资源。
Concurrent 包的改进
java.util.concurrent 包引入了一个轻量级的 fork/join 的框架来支持多核多线程的并发计算。此外，实现了 Phaser 类，它类似于 CyclicBarrier 和 CountDownLatch 但更灵活。最后，ThreadLocalRandom 类提供了线程安全的伪随机数生成器。
国际化（i18n）
支持 Unicode 6.0。改进 java.util.Locale 以支持 IETF BCP 47 和 UTR 35，并且在 get/set locale 的时候分成了用于显示的 locale 和用于格式化的 locale。

# Java 8 主要新特性
[查看Java 8 官方参考](https://www.oracle.com/technetwork/java/javase/8-whats-new-2157071.html)

## 函数式接口
Java 8 引入的一个核心概念是函数式接口（Functional Interfaces）。通过在接口里面添加一个抽象方法，这些方法可以直接从接口中运行。如果一个接口定义个唯一一个抽象方法，那么这个接口就成为函数式接口。同时，引入了一个新的注解：@FunctionalInterface。可以把他它放在一个接口前，表示这个接口是一个函数式接口。这个注解是非必须的，只要接口只包含一个方法的接口，虚拟机会自动判断，不过最好在接口上使用注解 @FunctionalInterface 进行声明。在接口中添加了 @FunctionalInterface 的接口，只允许有一个抽象方法，否则编译器也会报错。

## Lambda表达式
在 Java 8 之前，匿名内部类，监听器和事件处理器的使用都显得很冗长，代码可读性很差，Lambda 表达式的应用则使代码变得更加紧凑，可读性增强；Lambda 表达式使并行操作大集合变得很方便，可以充分发挥多核 CPU 的优势，更易于为多核处理器编写代码；
Lambda 表达式由三个部分组成：第一部分为一个括号内用逗号分隔的形式参数，参数是函数式接口里面方法的参数；第二部分为一个箭头符号：->；第三部分为方法体，可以是表达式和代码块。
为了减少过量的函数式接口，Java 8 在 java.util.function 中增加了不少新的函数式通用接口。例如：
Function<T, R>：将 T 作为输入，返回 R 作为输出，他还包含了和其他函数组合的默认方法。
Predicate<T> ：将 T 作为输入，返回一个布尔值作为输出，该接口包含多种默认方法来将 Predicate 组合成其他复杂的逻辑（与、或、非）。
Consumer<T> ：将 T 作为输入，不返回任何内容，表示在单个参数上的操作。

## 方法引用
方法引用使得开发者可以直接引用现存的方法、Java类的构造方法或者实例对象。方法引用和Lambda表达式配合使用，使得java类的构造方法看起来紧凑而简洁，没有很多复杂的模板代码。

方法引用在Java8中使用方式相当灵活，总的来说，一共有以下几种形式：

静态方法引用：ClassName::methodName;
实例上的实例方法引用：instanceName::methodName;
超类上的实例方法引用：supper::methodName;
类的实例方法引用：ClassName:methodName;
构造方法引用Class:new;
数组构造方法引用::TypeName[]::new

## 接口的默认方法和静态方法
默认方法使得开发者可以在 不破坏二进制兼容性的前提下，往现存接口中添加新的方法，即不强制那些实现了该接口的类也同时实现这个新加的方法。
在接口中，还允许定义静态的方法。接口中的静态方法可以直接用接口来调用。

## 改进的类型推断
Java 8编译器在类型推断方面有很大的提升，在很多场景下编译器可以推导出某个参数的数据类型，从而使得代码更为简洁。

## 重复注解
自从Java 5引入了注解机制，这一特性就变得非常流行并且广为使用。然而，使用注解的一个限制是相同的注解在同一位置只能声明一次，不能声明多次。Java 8打破了这条规则，引入了重复注解机制，这样相同的注解可以在同一地方声明多次。 重复注解机制本身必须用@Repeatable注解。事实上，这并不是语言层面上的改变，更多的是编译器的技巧，底层的原理保持不变。


## 类型注解
Java 8 的类型注解扩展了注解使用的范围。在该版本之前，注解只能是在声明的地方使用。现在几乎可以为任何东西添加注解：局部变量、类与接口，就连方法的异常也能添加注解。新增的两个注释的程序元素类型 ElementType.TYPE_USE 和 ElementType.TYPE_PARAMETER 用来描述注解的新场合。ElementType.TYPE_PARAMETER 表示该注解能写在类型变量的声明语句中。而 ElementType.TYPE_USE 表示该注解能写在使用类型的任何语句中（例如声明语句、泛型和强制转换语句中的类型）。 对类型注解的支持，增强了通过静态分析工具发现错误的能力。原先只能在运行时发现的问题可以提前在编译的时候被排查出来。

## Streams 和 Parallel Streams
Java 8 引入了流式操作（Stream），通过该操作可以实现对集合（Collection）的并行处理和函数式操作。根据操作返回的结果不同，流式操作分为中间操作和最终操作两种。最终操作返回一特定类型的结果，而中间操作返回流本身，这样就可以将多个操作依次串联起来。根据流的并发性，流又可以分为串行和并行两种。流式操作实现了集合的过滤、排序、映射等功能。

Stream 和 Collection 集合的区别：Collection 是一种静态的内存数据结构，而 Stream 是有关计算的。前者是主要面向内存，存储在内存中，后者主要是面向 CPU，通过 CPU 实现计算。

流有串行和并行两种，串行流上的操作是在一个线程中依次完成，而并行流则是在多个线程上同时执行。并行与串行的流可以相互切换：通过 stream.sequential() 返回串行的流，通过 stream.parallel() 返回并行的流。相比较串行的流，并行的流可以很大程度上提高程序的执行效率。
常见的中间操作有：
filter()：对元素进行过滤；
sorted()：对元素排序；
map()：元素的映射；
distinct()：去除重复元素；
subStream()：获取子 Stream 等。 
常见的终止操作有：
forEach()：对每个元素做处理；
toArray()：把元素导出到数组；
findFirst()：返回第一个匹配的元素；
anyMatch()：是否有匹配的元素等。 

## Optional
Java 开发中最常见的 bug 就是空值异常【NullPointerException】。在Java 8之前，Google Guava引入了Optionals 类来解决 NullPointerException，从而避免源码被各种 null 检查污染，以便开发者写出更加整洁的代码。Java 8也将 Optional 加入了官方库。

## 新的日期和时间的API
Java 8新的Date-Time API (JSR 310)在很大程度上受到Joda-Time的影响，并且吸取了其精髓。新的java.time包涵盖了所有处理日期，时间，日期/时间，时区，时刻（instants），过程（during）与时钟（clock）的操作。

重要的类
LocalTime：提供简单的时间，并不包含年月日信息。
LocalDate：提供简单的日期，并不包含当天的时间信息。
LocalDateTime：LocalTime和LocalDate的合体，同时表明日期和时间。
Instant：以Unix元年时间开始所经历的秒数进行计算。
时间间隔 
Duration：以秒和纳秒为单位
Period：以年月日为时间单位
TemporalAdjuster接口：日期调整 
TemporalAdjusters类的为我们提供了大量预定义的TemporalAdjuster。


## 提升HashMaps的性能
当hash冲突时，以前都是用链表存储，在java8里头，当节点个数>=TREEIFY_THRESHOLD - 1时，HashMap将采用红黑树存储，这样最坏的情况下即所有的key都Hash冲突，采用链表的话查找时间为O(n)，而采用红黑树为O(logn)。 


# Java 9 (过渡版) 主要新特性
[查看Java 9 官方参考](https://docs.oracle.com/javase/9/whatsnew/toc.htm)

## Java原生态支持模块化
Java 平台模块系统，也就是 Project Jigsaw，把模块化开发实践引入到了 Java 平台中。在引入了模块系统之后，JDK 被重新组织成 94 个模块。Java 应用可以通过新增的 jlink 工具，创建出只包含所依赖的 JDK 模块的自定义运行时镜像。这样可以极大的减少 Java 运行时环境的大小。这对于目前流行的不可变基础设施的实践来说，镜像的大小的减少可以节省很多存储空间和带宽资源 。

## G1成为默认垃圾回收装置
Java原生封装方便的操作http协议的工具
Jshell，Java开始原生支持交互脚本，或者简称命令行
集合框架进一步优化
提供类的工厂化构建方法
支持docker 
多版本兼容 JAR 

# Java 10 (过渡版) 主要新特性
[查看Java 10 官方参考](https://www.oracle.com/technetwork/java/javase/10-relnote-issues-4108729.html)

## 局部变量类型推断 
var list = new ArrayList<String>(); // ArrayList<String>
var stream = list.stream(); // Stream<String>

## 并行全垃圾回收器 G1 
G1 垃圾回收器是 Java 9 中 Hotspot 的默认垃圾回收器，是以一种低延时的垃圾回收器来设计的，旨在避免进行 Full GC，但是当并发收集无法快速回收内存时，会触发垃圾回收器回退进行 Full GC。之前 Java 版本中的 G1 垃圾回收器执行 GC 时采用的是基于单线程标记扫描压缩算法（mark-sweep-compact）。为了最大限度地减少 Full GC 造成的应用停顿的影响，Java 10 中将为 G1 引入多线程并行 GC，同时会使用与年轻代回收和混合回收相同的并行工作线程数量，从而减少了 Full GC 的发生，以带来更好的性能提升、更大的吞吐量。
Java 10 中将采用并行化 mark-sweep-compact 算法，并使用与年轻代回收和混合回收相同数量的线程。具体并行 GC 线程数量可以通过：-XX：ParallelGCThreads 参数来调节，但这也会影响用于年轻代和混合收集的工作线程数。


# Java 11 主要新特性
[查看Java 11 官方参考](https://www.oracle.com/technetwork/java/javase/11-relnote-issues-5012449.html#NewFeature)

## Local Var
在Lambda表达式中，可以使用var关键字来标识变量，变量类型由编译器自行推断。
Arrays.asList("Java", "Python", "Ruby")
    .forEach((var s) -> {
        System.out.println("Hello, " + s);
    });

## HttpClient
长期以来，如果要访问Http资源，JDK的标准库中只有一个HttpURLConnection，这个古老的API使用非常麻烦，而且已经不适用于最新的HTTP协议。
JDK11的新的HttpClient支持HTTP/2和WebSocket，并且可以使用异步接口：
    HttpRequest request = HttpRequest.newBuilder().uri(URI.create("https://www.qq.com/")).GET().build();
    HttpResponse.BodyHandler<String> bodyHandler = HttpResponse.BodyHandlers.ofString();
    HttpClient client = HttpClient.newHttpClient();
    CompletableFuture<HttpResponse<String>> future = client.sendAsync(request, bodyHandler);
    future.thenApply(HttpResponse::body).thenAccept(System.out::println).join();

## List API
对于List接口，新增了一个of(T...)接口，用于快速创建List对象：
List<String> list = List.of("Java", "Python", "Ruby");
List的toArray()还新增了一个重载方法，可以更方便地把List转换为数组。可以比较一下两种转换方法：
// 旧的方法:传入String[]:
String[] oldway = list.toArray(new String[list.size()]);
// 新的方法:传入IntFunction:
String[] newway = list.toArray(String[]::new);

## 读写文件
对Files类增加了writeString和readString两个静态方法，可以直接把String写入文件，或者把整个文件读出为一个String：
Files.writeString(
    Path.of("./", "tmp.txt"), // 路径
    "hello, jdk11 files api", // 内容
    StandardCharsets.UTF_8); // 编码
String s = Files.readString(
    Paths.get("./tmp.txt"), // 路径
    StandardCharsets.UTF_8); // 编码
这两个方法可以大大简化读取配置文件之类的问题。

## String API
String新增了strip()方法，和trim()相比，strip()可以去掉Unicode空格，例如，中文空格：
String s = " Hello, JDK11!\u3000\u3000";
System.out.println("     original: [" + s + "]");
System.out.println("         trim: [" + s.trim() + "]");
System.out.println("        strip: [" + s.strip() + "]");
System.out.println(" stripLeading: [" + s.stripLeading() + "]");
System.out.println("stripTrailing: [" + s.stripTrailing() + "]");
输出如下：
     original: [ Hello, JDK11!　　]
         trim: [Hello, JDK11!　　]
        strip: [Hello, JDK11!]
 stripLeading: [Hello, JDK11!　　]
stripTrailing: [ Hello, JDK11!]

新增isBlank()方法，可判断字符串是不是“空白”字符串：
String s = " \u3000"; // 由一个空格和一个中文空格构成
System.out.println(s.isEmpty()); // false
System.out.println(s.isBlank()); // true

新增lines()方法，可以非常方便地按行分割字符串：
String s = "Java\nPython\nRuby";
s.lines().forEach(System.out::println);

新增repeat()方法，可以指定重复次数：
System.out.println("-".repeat(10)); // 打印----------



## 标准 HTTP Client 升级
Java 11 对 Java 9 中引入并在 Java 10 中进行了更新的 Http Client API 进行了标准化，在前两个版本中进行孵化的同时，Http Client 几乎被完全重写，并且现在完全支持异步非阻塞。
新版 Java 中，Http Client 的包名由 jdk.incubator.http 改为 java.net.http，该 API 通过 CompleteableFutures 提供非阻塞请求和响应语义，可以联合使用以触发相应的动作，并且 RX Flow 的概念也在 Java 11 中得到了实现。

## Epsilon：低开销垃圾回收器
Epsilon 垃圾回收器的目标是开发一个控制内存分配，但是不执行任何实际的垃圾回收工作。它提供一个完全消极的 GC 实现，分配有限的内存资源，最大限度的降低内存占用和内存吞吐延迟时间。




# Java 12 (过渡版) 主要新特性
[查看Java 12 官方参考](https://www.oracle.com/technetwork/java/javase/12-relnote-issues-5211422.html#NewFeature)

## Switch表达式扩展
使用 Java 12 中 Switch 表达式的写法，省去了 break 语句，避免了因少些 break 而出错，同时将多个 case 合并到一行，显得简洁、清晰也更加优雅的表达逻辑分支，其具体写法就是将之前的 case 语句表成了：case L ->，即如果条件匹配 case L，则执行 标签右侧的代码 ，同时标签右侧的代码段只能是表达式、代码块或 throw 语句。表达式默认可返回结果。




# 引用
https://blog.csdn.net/fanxiaobin577328725/article/details/53066450



TODO问题：
1、jdk9模块化后，类加载机制的变化


