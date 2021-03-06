# Java虚拟机

Java虚拟机：Java Virture Machine，简称`JVM`，是一种能够运行Java字节码的虚拟机。只要生成的编译文件匹配jvm对加载编译文件的格式要求，任何语言都可以有JVM编译执行，如：kotlin、Scala等。

## 一、JVM基本结构

JVM包含两个子系统两个组件：两个字系统为：

- 类加载子系统：`ClassLoader SubSystem`，根据给定的全限定类名来装载对应的class文件到`Runtime Data Area`中的`Methood Area`;
- 执行引擎：`Execution Engine`，执行classpath中的指令；

两个组件为：

- 本地接口：`Native Interface`，与`native libraries`交互，是其它编程语言的接口；
- 运行时数据区（内存结构）：`Runutime Data Area`，JVM内存区域。

JVM运行流程：

1. 通过类加载器把类文件（Java代码）转换成字节码文件，并把字节码文件加载到`运行时数据区`的`方法区`；
2. 执行引擎将字节码翻译成机器指令，再交由CPU去执行；
3. CPU执行翻译好的机器指令时，会调用其他语言的本地库接口，以此实现整个程序的功能。

![image-20200328143101538](img/image-20200328143101538.png)

### 类加载子系统

#### 类的生命周期

![image-20200328153719629](./img/image-20200328153719629.png)

##### 1. 加载

将`.class`文件从磁盘读到内存中；

##### 2. 连接

###### 2.1 验证

验证字节码文件的正确性；

###### 2.2 准备

给类的静态变量分配内存，并赋予默认值；

###### 2.3 解析

类装载器装入类所引用的其他所有类；

##### 3. 初始化

为类的静态变量赋予正确的初始值，上述的准备阶段为静态变量赋予的是虚拟机默认的初始值，此处赋予的才是程序编写者为变量分配的真正值，执行静态代码块。

##### 4. 使用

##### 5. 卸载

#### 类加载器种类

![image-20200328145640395](img/image-20200328145640395.png)

##### 1. 启动类加载器（Bootstrap ClassLoader）

负责加载JRE的核心类库，如JRE目标下的 `rt.jar`、`charsets.jar`等；

##### 2. 扩展类加载器（Extension ClassLoader）

负责加载JRE扩展目录`ext`中的jar包；

##### 3. 系统类加载器（Application ClassLoader）

负责加载 `classpath`路径下的类；

##### 4. 用户自定义加载器（User Custom ClassLoader）

负责加载用户自定义路径下的类包。

#### 类加载机制

##### 1.  全盘负责委托机制

当一个ClassLoader加载一个类时，除非显式的使用另一个ClassLoader，否则，该类所依赖和引用的类也由这个ClassLoader来加载。

##### 2. 双亲委派机制

先委托父类加载器寻找目标类，在父类加载器找不到目标的情况下，再到自己的路径下查找并加载目标类；

###### 双亲委派模式的优势

- 沙箱安全机制：比如自己写的String.class类不会被加载，可防止核心库被随意篡改；
- 避免类的重复加载：当父类加载器已经加载该类时，就不需要在子类加载器中再次加载。

### 运行时数据区（内存结构）

Java虚拟机在执行Java程序过程中，会把它所管理的内存区域划分为若干个不同的数据区域。运行时数据区由五个部分组成：

- Method Area：方法区
- Heap：堆
- Stack：栈
- Native Method Stack：本地方法栈
- Program Counter Register：程序计数器

#### 1. 方法区

用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译后的代码等数据。

虽然`Java虚拟机规范`将方法区描述为堆的一个逻辑部分，但它的别名`Non-Heap`，是为了和Java的堆区分开。其中，JDK8以前的`Hotspot虚拟机`叫`永久代`、`持久代`，在JDK8时叫`元空间`。

#### 2. 堆

Java 虚拟机中内存最大的一块，是被所有线程共享的，几乎所有的对象实例都在这里分配内存，当对象无法在该空间申请到内存时，将抛出异常：`OutOfMemoryError`，另外，堆也是垃圾收集器的主要作用区域。

![image-20200328151514559](img/image-20200328151514559.png)

##### 2.1 新生代（Young Generation）

类出生、成长、消亡的区域，一个类在这里产生、应用，最后被垃圾回收器收集，结束生命。

新生代又可分为两部分：

- 伊甸区（Eden Space）：所有的类都是从伊甸区被`new`出来的；
- 幸存区（Survivor Space）：分为`From区`和`To区`。当伊甸区的空间用完时，如果程序又需要创建新对象，JVM的垃圾回收器将伊甸区进行垃圾回收（Minor GC），将伊甸区中的不再被其他对象引用的对象进行销毁处理，然后将Eden区中剩余的对象转移到幸存区的`From区`，若From区也满了，在对此处的对象进行垃圾回收，然后将剩余对象转移到To区。

##### 2.2 老年代（Old Generation）

新生代对象经过多次GC仍然存活会移动到老年区，若老年代也满了，此时将发生`Major GC`（也叫`Full GC`），进行老年区的内存清理。若老年区执行了Full GC后发现依然无法进行对象的保存操作，就会抛出`OOM`（OutOfMemoryError）错误。

##### 2.3 元空间（Meta Space）

从jdk1.8开始，元空间代替了永久代，它是对JVM虚拟机规范中方法区的实现，区别在于元数据区不在虚拟机中，而是用的本地内存，永久代在虚拟机中，永久代逻辑结构上也属于堆，但在物理结构上不属于。

###### 为什么移除永久代？

官方解释：http://openjdk.java.net/jeps/122

移除永久代是为融合HotSpot与JRockit做出的努力，因为JRockit没有永久代，不需要配置永久代。

![image-20200328153020081](img/image-20200328153020081.png)

#### 3. 栈

Java线程执行方法的内存模型，一个线程对于一个栈，每个方法在执行的同时都会创建一个栈帧（用于存储局部变量表、操作数栈、动态链接、方法出口等信息），该区域不存在垃圾回收问题，只要线程执行结束，该区域的生命周期和线程的生命周期保持一致。

###### 堆和栈的区别：

- 物理地址的连续性：
  - 堆上对象的物理地址分配是不连续的，性能较低；
  - 栈使用的是数据结构中的栈，先进先出的原则，物理地址分配是连续的，性能较高。
- 内存大小不同：
  - 堆的物理地址分配由于是不连续的，给对象或数组分配的内存空间需要在运行时才能确定，因此，堆的大小不固定，但一般堆的大小往往远大于栈；
  - 栈的物理地址分配是连续的，分配的内存大小在编译期就可以确定，因此，栈的大小固定；
- 存放内容不同
  - 堆中存放对象的实例或数组。更关注数据的存储；
  - 栈中存放局部变量表、操作数栈、返回结果等。更关注方法的执行；
- 程序的可见性不同
  - 堆对于整个应用程序都是共享、可见的；
  - 栈只对线程可见，因此是线程私有的，栈的生命周期与线程相同。

#### 4. 本地方法栈

和栈的作用类似，区别不过是Java栈为JVM执行Java方法服务，而本地方法栈为JVM执行native方法服务。主要是登记native方法，在`Execution Engine`执行时加载本地方法库。

#### 5. 程序计数器

当前线程所执行的字节码的行号指示器。字节码解析器的工作是通过改变这个计数器的值，来选取下一条需要执行的字节码指令，像分支、循环、跳转、异常处理、线程恢复等基础功能，都需要依赖程序计数器。是一个非常小的空间，内存占用几乎可以忽略。

## 二、GC算法

### 1. 如何判断对象可以被回收

堆中存放着几乎所有的对象实例，垃圾回收前的第一步操作就是判断哪些对象可以被回收。

#### 引用计数法

给对象添加一个`引用计数器`，对象每次被引用时，该计数器加1，当引用失效时，计数器减1，计数器为0的对象就是不会再被使用的对象，可以被回收。

此方法效率高，但由于存在`循环引用`的问题，所以目前主流虚拟机都不采用这种方法来管理堆内存。

#### 可达性分析算法

基本思想：当一个对象到`GC Roots`之间没有任何引用链相连时，此对象就是不可用的！

- `GC Roots`：根节点，类加载器、Thread、虚拟机栈的局部变量表、static成员变量、常量的引用、本地方法栈等；
- `引用链`：以`GC Roots`为起始节点，向下进行搜索指定对象，节点到对象所走过的路径称为引用链。

##### 如何判断一个常量是废弃常量？

运行时常量池主要回收的是废弃常量。假如在常量池中存在字符串`abc`，如果当前没有任何String对象引用该字符串常量的话，说明常量`abc`就是废弃常量，如果这时发生内存回收，且有必要回收的话，`abc`会被系统清理出常量池。

##### 如何判断一个类是无用的类？

- 该类所有的实例都已经被回收，即Java堆中不存在该类的任何实例；
- 加载该类的ClassLoader已经被回收；
- 该类对应的`java.lang.Class`对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

JVM虚拟机可以对满足上述3个条件的无用类进行回收。但注意这里说的是`可以`，不是`一定`。

### 2. 垃圾回收算法

```mermaid
graph LR
A[垃圾回收算法] --> B(标记-清除算法)
A --> C(复制算法)
A --> D(标记-整理算法)
A --> E(分代收集算法)
```

#### 2.1 标记-清除算法

最基础的收集算法。分为两个阶段：

- 标记：标记需要回收的对象；
- 清除：标记完成后统一回收所有被标记的对象。

【缺点】：

- 效率问题，标记和清除两个过程的效率都不高；
- 空间问题，标记清除后会产生大量不连续的内存碎片。

#### 2.2 复制算法

为了解决标记-清除算法的效率问题，出现了复制算法。它可以把内存分为大小相同的两块，每次只使用其中一块，当这块内存使用完后，就将还存活的对象复制到另一边，之后再把使用的空间一次性清理掉。这样一来，每次的内存回收都是对一般的内存空间进行回收。

#### 2.3 标记-整理算法

针对老年代特点提出的一种回收算法。标记过程与`标记-清除算法`一样，但标记之后，不是直接对可回收对象进行回收的操作，而是让所有存活的对象向同一边移动，然后再清理掉边界以外的内存。

#### 2.4 分代收集算法

商用虚拟机的垃圾收集器基本都采用分代收集算法。这种算法可以根据对象存活周期的不同将内存划分为几块。一般讲Java堆分为新生代和老年代，这样就可以根据各个年代的特点选择合适的垃圾收集算法。

在新生代中，每次收集都有大量对象死去，可选择`复制算法；只需付出少量对象的复制成本就可完成垃圾收集；`

而老年代的对象存活率较高，且没有额外的空间在对这些对象进行分配担保，必须选择“标记-清除”或“标记-整理”算法进行垃圾收集。

### 3. 垃圾收集器

Java虚拟机规范没有规定垃圾收集器应该如何实现，只有根据实际的应用场景来选择合适的垃圾收集器。