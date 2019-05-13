# JVM

**（1）基本概念**

​	JVM是可运行Java代码的假想计算器，包括一套字节码指令集、一组寄存器、一个栈、一个垃圾回收、堆和一个存储方法域。JVM是运行在操作系统之上，它与硬件没有直接的交互。

![JVM](./assets/jvm/JVM.png)



(2)运行过程

​	我们都知道Java源文件，通过编译器，能够产生相应的.class文件，也就是字节码文件，而字节码文件有通过Java虚拟机中的解释器，编译成特定机器上的机器码。

也就是如下：

​	①Java源文件—>编译器—>字节码文件

​	②字节码文件—>JVM—>机器码

​	每一种平台的解释器是不同的，但是实现的虚拟机是相同的，这也是Java为什么能够跨平台的原因，当一个程序从开始运行，这是虚拟机就凯斯实例化了，多个程序启动就会存在多个虚拟机示例。程序退出或者关闭，则虚拟机示例消亡，多个虚拟机示例之间数据不能共享。

![JVM](./assets/jvm/Java-Runtime.png)



## 2.1.线程

​	这里所说的线程指程序执行过程中的一个线程实体。JVM 允许一个应用并发执行多个线程。Hotspot  JVM 中的 Java 线程与原生操作系统的线程有直接的映射关系。当本地线程存储、缓存区分配、同步对象、栈、程序计数器等准备好后，就会创建一个操作系统原生线程。Java 线程结束，原生线程随之被回收。操作系统负责调度Java线程的 run() 方法。当线程结束时，会释放原生线程和 Java 线程的所有资源。

​	Hotspot JVM 后台运行的系统线程主要有下面几个：

| 虚拟机线程            (VM thread) | 这个线程等待 JVM 到达安全点操作出现。这些操作必须要在独立的线程中里执行，因为当堆修改无法进行时，线程都需要 JVM 位于安全点。这些操作的类型有： stop-the-world 垃圾回收、线程 dump、线程暂停、线程偏向锁 (biased locking) 接触 |
| ---------------------------------- | ------------------------------------------------------------ |
| 周期性任务线程                     | 这些线程负责定时器时间 (也就是中断)，用来调度周期性操作的执行 |
| GC 线程                            | 这些线程支持 JVM 中不同的垃圾回收活动。                      |
| 编译器线程 | 这些线程在运行时将字节码动态编译成本地平台相关的机器码 |
| 信号分发线程 | 这个线程接收发送到 JVM 的信号并调用适当的 JVM 方法处理 |



## 2.2.JVM 内存区域

![JVM](./assets/jvm/JVM-memory.png)

​	JVM 内存区域主要分为线程私有区域【程序计数器、虚拟机栈、本地方法区】、线程共享区域、【JAVA 堆、方法区】、直接内存。

​	线程私有数据区域生命周期与线程相同，依赖用户线程的启动/结束 而 创建/销毁(在 Hotspot VM 内，每个线程都与操作系统的本地线程直接映射，因此这部分内存区域的存/否跟随本地线程的生/死对应)。

​	线程共享区域随虚拟机的启动/关闭而创建/销毁。

​	直接内存并不是 JVM 运行时数据区域的一部分，但也会被频繁的使用：在 JDK 1.4 引入的 NIO 提供了基于 Channel 与 Buffer 的 IO 方式，它可以使用 Native 函数库直接分配堆外内存，然后使用 DirectByteBuffer 对象作为这块内存的引用进行操作(详见：Java I/O 扩展)，这样就避免了再 Java 堆和 Native 堆中来回复制数据，因此在一些场景中可以显著提高性能。

![Java-Memory-Area](./assets/jvm/Java-Memory-Area.png)



### 2.2.1.程序计数器

​	一块较小的内存空间，是当前线程所执行的字节码的行号指示器，每条线程都要有一个独立的程序计数器，这类内存也称为"线程私有"的内存。

​	正在执行的 Java 方法的话，计数器记录的是虚拟机字节码指令的地址 (当前指令的地址) 。如果是 Native 方法，则为空。

​	这个内存区域是唯一一个在虚拟机中没有规定任何 OutOfMemoryError 情况区域。

### 2.2.2.虚拟机栈 (线程私有)

​	虚拟机栈是描述 Java 方法执行的内存模型，每个方法在执行的同事都会创建一个栈帧 (Stack Frame) 用于存储局部变量表、操作数栈、动态链接、方法出口灯信息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机中入栈到出栈的过程。

​	栈帧 (Stack Frame) 是用来存储数据和部分过程结果的数据结构，同时也被用来动态处理动态链接 (Dynamic Linking)、方法返回值和异常分派 (Dispatch Exception) 。栈帧随着方法调用而创建，随着方法结束而销毁——无论方法是正常完成异常完成 (抛出了在方法内未被捕获的异常) 都算做方法的结束。



### 2.2.3.本地方法区 (线程私有)

​	本地方法区和 Java Stack 作用类型，区别是虚拟机栈作为执行 Java 方法服务，而本地方法栈为 Native 方法服务，如果一个 VM 实现适用 C-linkage 模型来支持 Native 调用，那么该栈将会是一个 C 栈，但 HotSpot VM 直接就把本地方法栈和虚拟机栈合二为一。

### 2.2.4. 堆 (Heap-线程共享) -运行时数据区

​	堆是被线程共享的一块内存区域，创建的对象和数组都保存在 Java 堆内存中，也就是垃圾收集器进行垃圾收集的最重要的内存区域。由于现代 VM 采用**分代收集算法**，因此 Java 堆从 GC 的角度还可以细分为：新生代(Eden 区、From Survivor 区和 To Survivor区) 和**老年代**。

### 2.2.5. 方法区/永久代 (线程共享)

​	即我们常说的**永久代 (Permanent Generation)**，用于存储**被 JVM 加载的类信息、常量、静态变量、即时编译器编译后的代码**等数据。HotSpot VM 把 GC 分代收集扩展至方法区，即**使用 Java 堆的永久代来实现方法区**，这样 HotSpot 的垃圾收集器就可以像管理 Java 堆一样管理这部分内存，而不必为方法区开发专门的内存管理器(永久代的内存回收的主要目标是针对**常量池的回收**和**类型的卸载**，因此收益一般很小)。

​	运行时常量池 (Runtime Constant Pool) 是方法区的一部分。 Class 文件中除了有了类版本、字段、方法、接口等描述信息外，还有一项信息是常量池 (Constant Pool Table)， 用于存放编译器生成各种字面量和符号引用，这部分内容将在类加载后存放到方法区的运行时常量池中。Java 虚拟机对 Class 文件的每一部分 (自然也包括常量池) 的格式都有严格的规定，每一个字节用于存储哪种数据都必须符合规范上的要求，这样才会被虚拟机认可、装载和执行。

## 2.3.JVM运行时内存

​	Java 堆从 GC 的角度还可以细分为：**新生代** (Eden区、From Survivor 区和 To Survivor 区) 和**老年代**。



### 2.3.1.新生代

​	新生代是用来放新生的对象。一般占据堆得 1/3 空间。由于频繁创建对象，所以新生代会频繁触发 MinorGC 进行垃圾回收。新生代又分为 Eden 区、ServivorFrom、ServiviorTo三个区。

#### 2.3.1.1. Eden区

​	Java 新对象的出生地 (如果新创建的对象占用内存很大，则直接分配到老年代)。当 Eden 区内存不够的时候回触发 MinorGC，对新生代区进行一次垃圾回收。

#### 2.3.1.2. ServivorFrom

​	上一次 GC 的幸存者，作为这一次 GC 的被扫描者。

#### 2.3.1.3. ServivorTo

​	保留了一次 MinorGC 过程中的幸存者。

#### 2.3.1.4. MinorGC 的过程 (复制->清空->互换)

​	MinorGC 采用复制算法。

**①：eden、servivorFrom 复制到 ServicorTo，年龄+1**

​	首先，把 Eden 和 ServivorFrom 区域中存活的对象复制到 ServicorTo 区域 (如果有对象的年龄以及达到了老年的标准，则复制到老年代区)，同时把这些对象的年龄+1 (如果 ServicorTo 不够位置了就放到老年区)；

**②：清空 eden、servicorFrom**

​	然后，清空 Eden 和 ServicorFrom 中的对象；

**③：ServicorTo 和 ServicorFrom 互换**

​	最后，ServicorTo 和 ServicorFrom 互换，源 ServicorTo 成为下一次 GC 时的 ServicorFrom区。



### 2.3.2. 老年代

​	主要存放在应用程序中生命周期长的内存对象。

​	老年代的对象比较稳定，所以 MajorGC 不会频繁执行。在进行 MajorGC 前一般都先进行了 MinorGC，使得有新生代的对象晋升入老年代，导致空间不够用时才触发。当无法找到足够大的连续空间分配给所创建的较大对象时也会提前触发一次 MajorGC 进行垃圾回收腾出空间。

​	MajorGC 采用标记清除法：首先扫描一次所有老年代，标记出存活的对象，然后回收没有标记的对象。MajorGC 的耗时比较长，因为要扫描在回收。MajorGC 会产生内存碎片，为了减少内存消耗，我们一般需要进行合并或者标记出来方便下次直接分配。当老年代也满了装不下的时候，就会抛出 OOM (Out Of Memory) 异常。

### 2.3.3. 永久代

​	永久代是指内存的永久保存区域，主要存放 Class 和 Meta (元数据) 的信息，Class 在被加载的时候被放入永久区域，它和存放示例的区域不同，GC 不会再主程序运行期对永久区域进行清理。所以这也导致了永久代的区域会随着加载的 Class 的增多而膨胀，最终抛出 OOM 异常。



#### 2.3.3.1. Java8 与元数据

​	在 Java8 中，永久代已经被移除，被一个称为 "元数据区" (元空间) 的区域所取代。元空间的本质和永久代类似，元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。因此，默认情况下，元空间的大小受本地内存限制。类的元数据放入 native memory，字符串池和类的静态变量放入 Java 堆中，这样可以加载多少类的元数据就不再由 MaxPermSize 控制，而由系统的实际可用空间来控制。



## 2.4. 垃圾回收与算法





### 2.4.1. 如何确定垃圾



#### 2.4.1.1. 引用计数法

​	在 Java 中，引用和对象时有关联的。如果要操作对象，必须用引用进行。因此，很显然一个简单的办法是通过引用计数来判断一个对象是否可以回收。简单来说，即一个对象如果没有任务与之关联的引用，即他们的引用计数都不为0，则说明对象不太可能再被用到，那么这个对象就是可回收对象。

#### 2.4.1.2. 可达性分析

​	为了解决引用计数法的循环引用问题， Java 使用了可达性分析的方法。通过一系列的 "GC roots" 对象作为起点搜索。如果在 "GC roots" 和一个对象之间没有可达路劲，则称该对象是不可达的。

​	需要注意的是，不可达对象不等价可回收对象，不可达对象变为可回收对象至少要经过两次标记过程。两次标记后仍然是可回收对象，则将面临回收。

### 2.4.2. 标记清除算法 (Mark-Sweep)

​	最基础的垃圾回收算法，分为两个阶段，标记和清除。标记阶段标记出所有需要回收的对象，清除阶段回收被标记的对象所占用的空间。如图



​	从图中我们就可以发现，该算法最大问题就是内存碎片化严重，后续可能发生大对象不能找到可利用空间的问题。

### 2.4.3. 复制算法 (copying)

​	为了解决 Mark-Sweep 算法内存碎片化的缺陷而被提出的算法。按内存容量将内存划分为等大小的两块，每次只使用其中一块，当这块内存满后将尚存活的对象复制到另一块上去，把已使用的内存清理掉，如图：



​	这种算法虽然实现简单，内存效率高，不易产生碎片，但是最大问题是可用内存被压缩到原来的一半。且存活对象增多的话，Copying 算法的效率会大大的降低。

### 2.4.4. 标记整理 (Mark-Compact)

​	结合了以上两个算法，为了避免缺陷而提出。标记阶段和Mark-Sweep算法相同，标记后不是清理对象，而是将存活的对象移向内存的一段。然后清除端边界外的对象。如图：



### 2.4.5.分代收集算法

​	分代收集法是目前大部分JVM所采用的方法，其核心思想是根据对象存活的不同生命周期将内存划分为不同的域，一般情况下将GC堆划分为老生代(Tenured/Old Generation) 和新生代 (Young Generation)。老生代的特点是每次垃圾回收时只有少量对象需要回收，新生代的特点是每次垃圾回收时都有大量垃圾需要被回收，因此可以根据不同区域选择不同的算法。



#### 2.4.5.1.新生代与复制算法

​	目前大部分 JVM 的 GC 对于新生代都采用 Coping 算法，因为新生代中每次垃圾回收大部分对象，即要复制的操作比较少，但通常并不是按照 1：1 来划分新生代。一般将新生代划分为一块较大的 Eden 空间和两块较小的 Survivor 空间 (From Space，To Space)，每次使用 Eden空间和其中一块 Survivor 空间，当进行回收时，将两块空间中还存活的对象分，复制到另外一块 Survivor空间中。



#### 2.4.5.2. 老年代与标记复制算法

​	而老年代因为每次回收少量对象，因而采用 Mark-Compact算法。

1. Java 虚拟机提到过的处于方法区的永久代 (Permanet Generation)，它用来存储 class类、常量、方法描述等。对永生代的回收主要包括废弃的常量和无用的类。

2. 对象的内存分配主要在新生代的 Eden Space 和 Survivor Space 的 From Space (Survivor 目前存放对象的那一块)，少数情况会直接分配到老生代。

3. 当新生代的 Eden Space 和 From Space 空间不足时就会发生一次 GC，进行 GC 后， Eden Space 和 From Space 区的存活对象会被挪到 To Space，然后将 Eden Space 和 From Space 进行清理。

4. 如果 To Space 无法足够存储某个对象，则将这个对象存储到老生代。

5. 在进行 GC 后，使用的便是 Eden Space 和 To Space了，如此反复循环。

6. 当对象在 Survivor 区躲过一次 GC 后，其年龄就会+1。默认情况下年龄达到 15 的对象会被移到老生代中。

   

   ## 2.5. Java 四种引用类型

   ### 2.5.1. 强引用

   ​	在 Java 中最常见的就是强引用，把一个对象赋给一个引用变量，这个引用变量就是一个强引用。当一个对象被强引用变量引用时，它是处于可达状态，它是不可能被垃圾回收机制回收的，即使该对象以后永远都不会用到 JVM 也不会回收。因此强引用是造成 Java 内存泄漏的主要原因之一。

   ### 2.5.2. 软引用

   ​	软引用需要用 SoftReference 类来实现，对于只有软引用的对象来说，当系统内存足够时它不会被回收，当系统内存不足时它会被回收。软引用通常在对内存敏感的程序中。

### 2.5.3. 弱引用

​	弱引用需要用 WeakReference 类来实现，它比软引用的生存期更短，对于只有弱引用的对象来说，只要垃圾回收机制一运行，不管 JVM 的内存是否足够，总会回收改对象占用的内存。

### 2.5.4. 虚引用

​	虚引用需要用 PhantomReference 类来实现，它不能单独使用，必须和引用队列联合使用。虚引用的主要作用是跟踪对象被垃圾回收的状态。



## 2.6. GC 分代收集算法 VS 分区收集算法



### 2.6.1. 分代收集算法

​	当前主流 VM 垃圾收集都采用" 分代收集 ” (Generational Collection) 算法，这种算法会根据对象存活周期的不同将内存划分为几块，如 JVM 中的 新生代、老年代、永久代，这样就可以根据各年代特点分别采用最适当的 GC 算法。

#### 2.6.1.1. 在新生代-复制算法

​	每次垃圾收集都能发现大批对象已死，只有少量存活。因此选用复制算法，只需要付出少量存活对象的复制成本就可以完成收集。

#### 2.6.1.2. 在老年代-标记整理算法

​	因为对象存活率高、没有额外空间对它进行分配担保，就必须采用 "标记—清理" 或 "标记—整理" 算法来回收，不必进行内存复制，且直接腾出空闲内存。

### 2.6.2. 分区收集算法

​	分区算法则将整个堆空间划分为连续的不同小区间，每个小区间独立使用，独立回收。这样做的好处是可以控制一次回收多少个小区间，根据目标停顿时间，每次合理地回收若干个小区间 (而不是整个堆)，从而减少一次 GC 所产生的停顿。

## 2.7. GC 垃圾收集器

​	Java 堆内存被划分为新生代和年老代，新生代主要使用复制和标记—清理垃圾回收算法；年老代主要使用标记—整理垃圾回收算法，因此 Java 虚拟机中针对新生代和年老代分别提供了多种不同的垃圾收集器，JDK 1.6 中 Sun HotSpot 虚拟机的垃圾收集器如下：





### 2.7.1. Serial 垃圾收集器 (单线程、复制算法)

​	Serial (英文连续) 是最基本垃圾收集器，使用复制算法，曾经是 JDK1.3.1 之前新生代唯一的垃圾收集器。 Serial 是一个单线程的收集器，它不但只会使用一个 CPU 或一条线程去完成垃圾收集工作，并且在进行垃圾收集的同时，必须暂停其他所有的工作线程，直到垃圾收集结束。

​	Serial 垃圾收集器虽然在手机垃圾过程中需要停止所有其他的工作线程，但是它简单高效，对于限定单个 CPU 环境来说，没有线程交互的开销。可以获得最高的单线程垃圾收集效率，因此 Serial 垃圾收集器依然是 Java 虚拟机运行在 Client 模式下默认的新生代垃圾收集器。

### 2.7.2. ParNew 垃圾收集器 （Serial + 多线程）

​	ParNew 垃圾收集器其实是 Serial 收集器的多线程版本，也是用复制算法，除了使用多线程进行垃圾收集之外，其余的行为和 Serial 收集器完全一样， ParNew 垃圾收集器在垃圾收集过程中同样也要暂停所有其他的工作线程。

​	ParNew 收集器默认开启和 CPU 数目相同的线程数，可以通过 -XX:ParallelGCThreads 参数来限制垃圾收集器的线程数。【Parallel：平行的】

​	ParNew 虽然是除了多线程外和 Serial 收集器几乎完全一样，但是 ParNew 垃圾收集器是很多 Java 虚拟机在 Server 模式下新生代的默认垃圾收集器。

### 2.7.3. Parallel Scavenge 收集器 （多线程复制算法、高效）

​	Parallel Scavenge 收集器也是一个新生代垃圾收集器，同样适用复制算法，也是一个多线程垃圾收集器，它重点关注的是程序达到一个可控制的吞吐量 （Thoughput，CPU 用于运行用户代码的时间/CPU 总消耗时间，即吞吐量=运行用户代码的时间/(运行用户代码时间+垃圾收集时间)），高吞吐量可以最高效率的利用 CPU 时间，尽快的完成程序的运算任务，主要适用于在后台运算而不需要太多交互的任务。自适应调节策略也是 ParallelScavenge 收集器与 ParNew 收集器的一个重要区别。

### 2.7.4. Serial Old 收集器 （单线程标记整理算法）

​	Serial Old 是 Serial 垃圾收集器年老代版本，它同样是个单线程的收集器，使用标记—整理算法。这个收集器也主要是运行在 Client 默认的 Java 虚拟机的年老代垃圾收集器。

​	在 Server 模式下，主要有两个用途：

1. 在 JDK1.5 之前版本中与新生代的 Parallel Scavenge 收集器搭配使用。

2. 作为老年代中使用 CMS 收集器的后备垃圾方案。

   新生代 Serial 与年老代 Serial Old 搭配垃圾收集过程图：





​	新生代 Parallel Scavenge 收集器与 ParNew 收集器工作原理相似，都是多线程的收集器，都使用的是复制算法，在垃圾收集过程中都需要暂停所有的工作线程。新生代 Parallel Scavenge/Parnew 与年老代 Serial Old 搭配垃圾收集过程图：



### 2.7.5. Parallel Old 收集器（多线程标记整理算法）

​	Parallel Old 收集器是 Parallel Scavenge 的年老代版本，使用多线程的标记—整理算法，在 JDK1.6 才开始提供。

​	在 JDK1.6 之前，新生代使用 ParallelScavenge 收集器只能搭配年老代的 Serial Old 收集器，只能保证新生代的吞吐量优先，无法保证整体的吞吐量，Parallel Old 正是为了在年老代同样提供吞吐量优先的垃圾收集器，如果系统对吞吐量要求比较高，可以优先考虑新生代 Parallel Scavenge 和年老代 Parallel Old 收集器搭配策略。

​	新生代 Parallel Scavenge 和年老代 Parallel Old 收集器搭配运行过程图：



### 2.7.6. CMS 收集器 （多线程标记清除算法）

​	Concurrent mark sweep （CMS）收集器是一种年老代垃圾收集器，其最主要目标是获取最短垃圾收停顿时间。和其他年老代使用标记—整理算法不同，它使用多线程的标记—清除算法。

​	最短垃圾收集停顿时间可以为交互比较高的程序提高用户体验。

​	CMS 工作机制相比其他垃圾收集器来说更复杂，整个过程分为以下 4 个阶段：

#### 2.7.6.1. 初始标记

​	只是标一下 GC Roots 能直接关联的对象，速度很快，仍然需要暂停所有的工作线程。进行 GC Roots 跟踪的过程，和用户线程一起工作，不需要暂停工作线程。

#### 2.7.6.2. 并发标记

​	进行 GC Roots 跟踪的过程，和用户线程一起工作，不需要暂停工作线程。

#### 2.7.6.3. 重新标记

​	为了修正正在并发标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，仍然需要暂停所有的工作线程。

#### 2.7.6.4. 并发清除

​	清除 GC Roots 不可达对象，和用户线程一起工作，不需要暂停工作线程。由于耗时最长的并发标记和并发清除过程中，垃圾收集线程可以和用户线程一起并发工作，所以总体上来看 CMS 收集器的内存回收和用户线程是一起并发地执行。

CMS 收集器工作过程：





### 2.7.7. G1 收集器

​	Garbage first 垃圾收集器是目前垃圾收集器理论发展的最前沿成果，相比于 CMS 收集器，G1 收集器两个突出的改进是：

1.基于标记—整理算法，不产生内存碎片。

2.可以非常精确控制停顿时间，在不牺牲吞吐量前提下，实现低停顿垃圾回收。

G1 收集器避免全区域垃圾收集，它把堆内存划分为大小固定的几个独立区域，并且跟踪这些区域的垃圾收集进度，同时在后台维护一个优先级列表，每次根据所允许的收集时间，优先回收垃圾最多区域。区域划分和优先级区域回收机制，确保 G1 收集器可以在优先时间获得最高的垃圾收集效率。

## 2.8. JAVA IO/NIO

### 2.8.1. 阻塞 IO 模型

​	最传统的一种 IO 模型，即在读写数据过程中会发生阻塞现象。当用户线程发出 IO 请求之后，内核会去查看数据是否就绪，如果没有就绪就会等待数据就绪，而用户线程就会处于阻塞状态，用户线程交出 CPU。当数据就绪之后，内核会将数据拷贝到用户线程，并返回结果给用户线程，用户线程才解除 block 状态。典型的阻塞 IO 模型的例子为：data = socket.read()；如果数据没有就绪，就会一直阻塞在read方法。

### 	2.8.2. 非阻塞 IO 模型

​	当用户线程发起一个 read 操作后，并不需要等待，而是马上就得到了一个结果。如果结果是一个 error 时，它就知道数据还没有准备好，于是他可以再次发送 read 操作。一旦内核中的数据准备好了，并且又再次收到了用户的线程的请求，那么它马上就将数据拷贝到了用户线程，然后返回。所以事实上，在非阻塞 IO 模型中，用户线程需要不断地询问内核数据是否就绪，也就是说非阻塞 IO 不会交出 CPU，而会一直占用 CPU。典型的非阻塞 IO 模型一般如下：

```java
while(true){
	data = socket.read();
	if(data != error){
    // 处理数据
    break;
	}
}
```

但是对于非阻塞 IO 就有一个非常严重的问题，在 while 循环中需要不断地询问内核数据是否就绪，这样会导致 CPU 占用率非常高，因此一般情况下很少使用 while 循环这种方式来读取数据。

### 2.8.3. 多路复用 IO 模型

​	多路复用 IO 模型是目前使用得比较多的模型。 Java NIO 实际上就是多路复用 IO。在多路复用 IO 模型中，会有一个线程不断的去轮询多个 socket 的状态，只有当 socket 真正有读写事件时，才真正调用实际的 IO 读写操作。因为在多路复用 IO 模型中，只需要使用一个线程就可以管理多个 socket，系统不需要建立新的进程或者线程，也不必维护这些线程和进程，并且只有在真正有 socket 读写事件进行时，才会使用 IO 资源，所以它大大减少了资源占用。在 Java NIO 中，是通过 selector.select() 去查询每个通道是否有到达事件，所以没有事件，则一直阻塞在那里，因此这种方式会导致用户线程阻塞。多路复用 IO 模式，通过一个线程就可以管理多个 socket，只有当 socket 真正有读写事件发生才会占用资源进行实际的读写操作。因此，多路复用 IO 比较适合连接数比较多的情况。

​	另外多路复用 IO 为何比非阻塞 IO 模型的效率高是因为在非阻塞 IO 中，不断地询问 socket 状态时通过用户线程去进行的，而在多路复用 IO 中，轮询每个 socket 状态是内核在进行的，这个效率要比用户线程要高德多。

​	不过需要注意的是，多路复用 IO 模型是通过轮询方式来检测是否有时间到达，并且对到达的事件逐一进行响应。对于多路复用 IO 模型来说，一旦事件响应体很大，那么就会导致后续的事件迟迟得不到处理，并且会影响到新的事件轮询。

### 2.8.4. 信号驱动 IO 模型

​	在信号驱动 IO 模型中，当用户线程发起一个 IO 操作，会给对应的 socket 注册一个信号函数，然后用户线程会继续执行，当内核数据就绪时会发送一个信号给用户线程，用户线程收到信号后，便在信号函数中调用 IO 读写操作来进行实际的 IO 请求操作。

### 2.8.5. 异步 IO 模型

​	异步 IO 模型才是最理想的 IO 模型，在异步 IO 模型中，当用户线程发起 read 操作之后，立刻就可以开始去做其他的事。而另一方面，从内核的角度，当它收到一个 asynchronous read 之后，它会立刻返回，说明 read 请求已经成功发起，因此不会对用户线程产生任何 block。然后，内核会等待数据准备完成，然后将数据拷贝到用户线程，当这一切都完成之后，内核会给用户线程发送一个信号，告诉它 read 操作完成了。也就是说用户线程完全不需要了解整个 IO 操作是如何进行的，只需要发起一个请求，当解说内核返回的成功信号时表示 IO 操作已经完成，可以直接去使用数据了。

​	也就说在异步 IO 模型中，IO 操作的两个阶段都不会阻塞用户线程，这两个阶段都是由内核自动完成，然后发送一个信号告知用户线程操作已完成。用户线程中不需要再次调用 IO 函数进行具体的读写。这点事和信号驱动模型有所不同在信号驱动模型中，当用户线程接收到信号表示数据已经就绪，然后需要用户线程调用 IO 函数进行实际的读写操作；而在异步 IO 模型中，收到信号表示 IO 操作已经完成，不需要再在用户线程中调用 IO 函数进行实际的读写操作。

注意，异步 IO 是需要操作系统的底层支持，在 Java 7 中，提供了 Asynchronous IO 。

## 2.9.1. JAVA IO 包



### 2.9.2.  Java NIO

​	NIO 主要有三大核心部分： Channel（通道），Buffer（缓冲器），Selector。传统 IO 基于字节流和字符流进行操作，而 NIO 基于 Channel 和 Buffer （缓冲区）进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector （选择区）用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个线程可以监听多个数据通道。



​	NIO 和传统 IO 之间最大的区别是，IO 是面向流的，NIO 是面向缓冲的。

### 2.9.2.1. NIO 的缓冲区

​	Java IO 面向流意味着每次从流中读一个或者多个字节，它们没有被缓存在任何地方。此外，它不能前后移动流中的数据。如果需要前后移动从流中读取数据，需要先将它缓存到一个缓冲区。NIO 的缓冲导向和方法不同。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动。这就增加了处理过程中的灵活性。但是，还需要检查是否该缓冲区中包含所有您需要处理的数据。而且，需要确保当更多的数据读入缓冲区时，不要覆盖缓冲区里尚未处理的数据。

### 2.9.2.2. NIO 的非阻塞

​	IO 的各种流是阻塞的。这意味着，当一个线程调用 read() 或者 write() 时，该线程是被阻塞，直到有一些数据被读取，或数据被完全写入。该线程在此期间不能再干任何事情了。NIO 的非阻塞模式，使一个线程线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可以用时，就什么都不会获取。而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。线程通信将非阻塞 IO 的空闲时间用于在其他通道上执行 IO 操作，所以一个单独的线程现在可以管理多个输入和输出通道（channel）。

### 2.9.3. Channel

​	首先说一下 Channel，国内大多翻译成 “通道” 。Channel 和 IO 中的 Stream（流）是差不多一个等级的。只不过 Stream 是单向的，譬如：InputStream，OutputStream，而 Channel 是双向的，即可以用来进行读操作，又可以用来进行写操作。

NIO 中的 Channel 的主要实现有：

1. FileChannel

2. DatagramChannel

3. SocketChannel

4. ServerSocketChannel

   

这里看名字就可以猜出个所以然来：分别对应文件 IO、UDP 和 TCP （Server 和 Client）。

下面演示的案例基本上就是围绕这4个类型的 Channel进行陈述的。

   

### 2.9.4. Buffer

​	Buffer，顾名思义，缓冲区，实际上是一个容器，使一个连续数组。Channel 提供从文件、网络读取数据的渠道，当是读取或写入的数据都必须经由 Buffer。





​	上面的图描述了从一个客户端向服务端发送数据，然后服务端接收数据的过程。客户端发送数据时，必须先将数据存入 Buffer 中，然后将 Buffer 中的内容写入通道。服务端这个接收数据必须通过 Channel 将数据读入到 Buffer 中，然后再从 Buffer 中取出数据来处理。

​	在 NIO 中，Buffer 是一个顶层父类，它是一个抽象类，常用的 Buffer 的子类有：ByteBuffer、IntBuffer、CharBuffer、LongBuffer、DoubleBuffer、FloatBuffer、ShortBuffer

### 2.9.5. Selector

​	Selector 类是 NIO 的核心类，Selector 能够检测多个注册通道上是否有事件发生，如果有事件发生，便获取事件然后针对每个事件进行相应的响应处理。这样一来，只是用一个单线程就可以管理多个通道，也就是管理多个连接。这样使得只有在连接真正有读写事件发生时，才会调用函数来进行读写，就大大的减少了系统开销，并且不必为每个连接都创建一个线程，不用去维护多个线程，避免了多线程之间的上下文切换导致的开销。



## 2.10. JVM 类加载机制

​	JVM 类加载机制分为五个部分：加载、验证、准备、解析、初始化，下面我们就分别来看一下这五个过程。







#### 2.10.1.1. 加载

​	加载时类加载过程中的一个阶段，这个阶段会在内存中生成一个代表这个类的 java.lang.Class 对象，作为方法区这个类的各种数据的入口。注意这里不一定非得要从一个 Class 文件读取，这里即可以从 ZIP 包中读取 （比如 jar 包和 war 包中读取），也可以在运行时计算生成（动态代理），也可以由其他文件生成（比如 JSP 文件转换成对应 Class 类）。

#### 2.10.1.2. 验证

​	这一阶段的主要目的是为了确保 Class 文件的字节流中包含的信息是否符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。

#### 2.10.1.3. 准备

​	准备阶段是正式为类变量分配内存并设置类变量的初始值阶段，即在方法区中分配这些变量所使用的内存空间。注意这里说的初始值概念，比如一个类变量定义为：

```java
public static int v = 8080;
```

​	实际上变量 v 在准备阶段过后的初始值为 0 而不是 8080，将 v 赋值为 8080 的 put static 指令是程序被编译后，存放于类构造器 <client> 方法中。

​	但是注意如果声明为：

```java
public static final int v = 8080;
```

​	在编译阶段会为 v 生成 ConstantValue 属性，在准备阶段虚拟机会根据 ConstantValue 属性将 v 赋值为 8080 。

#### 2.10.1.4. 解析

​	解析阶段是指虚拟机将常量池中的符号引用替换为直接引用的过程。符号引用就是 class 文件中的：

1. CONSTANT_Class_info

2. CONSTANT_Field_info

3. CONSTANT_Method_info

等类型的常量。

#### 2.10.1.5. 符号引用

- 符号引用与虚拟机实现布局无关，引用的目标不一定要已经加载到内存中。各种虚拟机实现的内存布局可以各不相同，当是它们能接受的符号引用必须是一致的，因为符号引用的字面量形式明确定义在 Java 虚拟机规范的 Class 文件格式中。

#### 2.10.1.6. 直接引用
- 直接引用可以是指向目标的指针，相对偏移量或是一个能间接定位到目标句柄。如果有了直接引用，那引用的目标必定已经在内存中存在。
  
#### 2.10.1.7.  初始化

​	初始化阶段是类加载最后一个阶段，前面的类加载之后，除了在加载阶段可以自定义类加载器外，其它操作都由 JVM 主导。到了初始阶段，才开始真正执行勒种定义的 Java 程序代码。

#### 2.10.1.8. 类构造器 <client>

​	初始化阶段是执行类构造器 <client> 方法的过程。<client> 方法是由编译器自动收集类中的类变量的赋值和静态语句块中的语句合并而成的，虚拟机会保证子 <client> 方法执行之前，父类的 <client> 方法已经执行完毕，如果一个类中没有对静态变量赋值也没静态语句块，那么编译器可以不为这个类生成 <client>() 方法。

​	注意以下几种情况不会执行类初始化：

1. 通过子类引用父类的静态字段，只会触发父类的初始化，而不会触发子类的初始化。
2. 定义对象数组，不会触发类的初始化。
3. 常量在编译期间会存入调用类的常量池中，本质上并没有直接引用定义常量的类，不会触发定义常量所在的类。
4. 通过类名获取 Class 对象，不会触发类的初始化。
5. 通过 Class.forName 加载指定类时，如果指定参数 initialize 为 false 时，也不会触发类初始化，其实这个参数是告诉虚拟机，是否要对类进行初始化。
6. 通过 ClassLoader 默认的 loadClass 方法，也不会触发初始化动作。

### 2.10.2. 类加载器

​	虚拟机设计团队吧加载动作放到 JVM 外部实现，以便让应用程序决定如何获取所需要的类，JVM 提供了 3 中类加载器：

#### 2.10.2.1. 启动类加载器(Bootstrap ClassLoader)

1. 负责加载 JAVA_HOME\lib 目录中的，或者通过 -Xbootclasspath 参考指定路径中的，且被虚拟机认可 (按文件名称识别，如 rt.jar) 的类

#### 2.10.2.2. 扩展类加载器(Extension Classloader)

 2. 负责加载 JAVA_HOME\lib\ext 目录中的，或者通过 java.ext.dirs 系统变量指定路径中的类库。

#### 2.10.2.3. 应用程序类加载器

 3. 负责加载用户路径 (classpath) 上的类库。

    JVM 通过双亲委派模型进行类加载，当然我们也可以通过继承 java.lang.ClassLoader 实现自定义的类加载器。



### 2.10.3. 双亲委派

​	当一个类收到了类加载请求，他首先不会尝试自己去加载这个类，而是把这个请求委派给父类去完成，每个层次类加载器都是如此，因此所有的加载请求都应该传送到启动类加载器中，只有当父类加载器反馈自己无法完成这个请求的时候 (在它的加载路径下没有找到所需要加载的 Class)，子类加载器才会尝试自己去加载。

​	采用双亲委派的一个好处就是比如加载位于 rt.jar 包中的 java.lang.Object，不管是哪个加载器加载这个类，最终都是委托给顶层的启动类进行加载，这样就保证了使用不同的类加载器最终得到的都是同一个 Object 对象。









### 2.10.4. OSGI (动态模型熊)

​	OSGI(Open Service Gateway Initiative)，是面向 Java 的动态模型系统，是 Java 动态化模块化系统的一系列规范。



#### 2.10.4.1. 动态改变构造

​	OSGI 服务平台提供在多种网络设备上无需重启的动态改变构造的功能。为了最小化耦合度和促使这些耦合度可管理，OSGI 技术提供一种面向服务的架构，它能使这些组件动态地发下对方。



#### 2.10.4.2. 模块化编程与热插拔

​	OSGI 旨在为实现 Java 程序的模块化编程提供基础条件，基于 OSGI 的程序很可能可以实现模块级的热插拔功能，当程序升级更新时，可以只停用、重新安装然后启动程序的其中一部分，这对企业程序开发来说是非常具有诱惑力的特性。

​	OSGI 描绘了一个很美好的模块化开发目标，而且定义了实现这个目标所需要的服务与框架，同时也有成熟的框架实现支持。但并非所有的应用都适合采用 OSGI 作为基础架构，它在提供强大功能的同时，也引入了额外的复杂度，因为他不遵守了类加载器的双亲委派模型。



# 3. JAVA 集合

## 3.1. 接口继承关系和实现

​	集合类存放在 Java.util 包中，主要有 3 种：set(集)、list(列表包含 Queue)和map(映射)。

1. Collection：Collection 是集合 List、Set、Queue 的最基本的接口。
2. Iterator：迭代器，可以通过迭代器遍历集合中的数据。
3. Map：是映射表的基础接口。















## 3.2. List

​	Java 的 List 是非常常用的数据类型。List 是有序的 Collection。Java List 一共三个实现类：分别是 ArrayList、Vector 和 LinkedList。









### 3.2.1. ArrayList (数组)

​	ArrayList 是最常用的 List 实现类，内部是通过数组实现的，它允许对元素进行快速随机访问。数组的缺点是每个元素之间不能有间隔，当数组大小不满足时需要增加存储能力，就要将已有数组复制到新的存储空间中。当从 ArrayList 的中间位置插入或者删除元素时，需要对数组进行复制、移动的代价比较高。因此，它适合随机查找和遍历，不适合插入和删除。

### 3.2.2. Vector (数组实现、线程同步)

​	Vector 和ArrayList 一样，也是通过数组实现的，不同的是它支持线程同步，即某一时刻只有一个线程能够写 Vector，避免多线程同时写而引起的不一致性，但实现同步需要很高的花费，因此访问它比访问 ArrayList 慢。

### 3.2.3. LinkList (链表)

​	LinkedList 是链表结构存储数据的，很适合数据的动态插入和删除，随机访问和遍历速度比较慢。另外，它还提供了 List 接口中没有定义的方法，专门用于操作表头和表位元素，可以当作堆栈、队列和双向队列使用。



## 3.3. Set

​	Set 注重独一无二的性质，该体系集合用于存储无序(存入和取出的顺序不一定相同)元素，值不能重复。对象的相等性本质是对象 hashCode 值 (java 是依据对象的内存地址计算出的此序号) 判断的，如果想要让两个不同的对象视为相等的，就必须覆盖 Object 的hashCode 方法和 equals 方法。

### 3.3.1.1. HashSet (Hash 表)

​	哈希表里面存放的是哈希值。 HashSet 存储元素的顺序并不是按照存入时的顺序 (和 List 显然不同)，而是按照哈希值来存的所以取数据也是按照哈希值取得。元素的哈希值是通过元素的 hashCode 方法来获取的，HashSet 首先判断两个元素的哈希值，如果哈希值一样，接着会比较 equals 方法来获取的，HashSet 首先判断两个元素的哈希值，如果哈希值一样，接着会比较 equals 方法，如果 equals 结果为true，HashSet 就视为同一个元素。如果 equals 为 false 就不是同一个元素。

​	哈希值相同 equals 为 false 的元素是怎么存储呢，就是在同样的哈希值下顺延 (可以认为哈希值相同的元素放在一个哈希桶中)。也就是哈希一样的存一列。如图 1 表示 hashCode 值不相同的情况；图 2 表示 hashCode 值相同，但 equals 不相同的情况。

​	HashSet 通过 hashCode 值来确定元素在内存中的位置。一个 hashCode 位置上可以存放多个元素。

#### 3.3.1.2. TreeSet (二叉树)

1. TreeSet 是使用二叉树的原理堆新 add() 的对象按照指定的顺序 (升序、降序)，每增加一个对象都会进行排序，将对象插入的二叉树指定的位置。
2. Integer 和 String 对象可以进行默认的TreeSet 排序，而自定义类的对象是不可以的，自定义类必须实现 Comparable 接口，并且覆写响应的 compareTo() 函数，才可以正常使用。
3. 在覆写 compare() 函数视，要返回相应的值才能使 TreeSet 按照一定的规则来排序
4. 比较此对象与指定对象的顺序。如果该对象小于、等于或大于指定对象，则返回负整数、零或正整数。

#### 3.3.1.3. LinkHashSet (HashSet + LinkedHashMap)

​	对于 LinkedHashSet 而言，它继承与 HashSet、又基于 LinkedHashMap 来实现的。LinkedHashSet 底层使用 LinkedHashMap 来保存所有元素，它继承于 HashSet，其所有的方法操作上又与 HashSet 相同，因此 LinkedHashSet 的实现上非常简单，只提供了四个构造方法，并通过传递一个标识参数，调用父类的构造器，底层构造一个 LinkedHashMap 来实现，在相关操作上与父类 HashSet 的操作相同，直接调用父类 HashSet 的方法即可。



## 3.4. Map







### 3.4.1. HashMap (数组+链表+红黑树)

​	HashMap 根据键的 hashCode 值存储数据，大多数情况下可以直接定位到它的值，因而具有很快的访问速度，但遍历顺序却是不确定的。HashMap 最多只允许一条记录的键为 null，允许多条记录的值为 null。HashMap 非线程安全，即任一时刻可以由多个线程同时写 HashMap，可能会导致数据不一致。如果需要满足线程安全，可以用 Collections 的 synchronizedMap 方法是 HashMap 具有线程安全的能力，或者使用 ConcurrentHashMap。我们用下面这张图来介绍 HashMap 的结构。



#### 3.4.1.1. Java 7 实现

**Java7 HashMap 结构**







​	大方向上，HashMap 里面是一个数组，然后数组中每个元素是一个单向链表。上图中，每个绿色的实体是嵌套类 Entry 的实例，Entry 包含四个属性：key，value，hash 值和用于单向链表的 next。

1. capacity：当前数组容量，始终保持在 2^n，可以扩容，扩容后数组大小为当前的 2 倍。
2. loadFactor：负载因子，默认为 0.75.
3. threshold：扩容的阈值，等于 capacity*loadFactor.



#### 3.4.1.2. JAVA8 实现

​	Java8 对 HashMap 进行了一些修改，最大的不同就是利用了红黑树，所以其由数组+链表+红黑树组成。

​	根据 Java7 HashMap 的介绍，我们知道，查找的时候，根据 hash 值我们能够快速的定位到数组的具体下标，但是之后的话，需要顺着链表一个个比较下去才能找到我们需要的，时间复杂度取决于链表的长度，为 O(n)。为了降低这部分的开销，在 Java8 中，当链表中的元素超过了 8 个以后，会将链表转换为红黑树，在这些位置进行查找的时候可以降低时间复杂度为 O(logn)。

**Java8 HashMap 结构**







### 3.4.2. ConcurrentHashMap

#### 3.4.2.1. Segment 段

​	ConcurrentHashMap 和 HashMap 思路差不多，当是因为它支持并发操作，所以要复杂一些。整个 ConcurrentHashMap 由一个个 Segment 组成，Segment 代表"部分"或"一段"的意思，所以很多地方将其描述为分段锁。注意，行文中，我很多地方用了"槽"来代表一个 segment。

#### 3.4.2.2. 线程安全 (Segment 继承 ReentrantLock)

​	简单理解就是，ConcurrentHashMap 是一个 Segment 数组，Segment 通过继承 ReentrantLock 来进行加锁，所以每次需要加锁的操作锁住的是一个 Segment，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全。

**Java7 ConcurrentHashMap**







#### 3.4.2.3. 并行度 (默认16)

​	concurrentLevel: 并行级别、并发数、Segment数，怎么翻译不重要，理解它。默认是16，也就是说 ConcurrentHashMap 有 16 个 Segment，所以理论上，这个时候，最多可以同时支持 16 个线程并发，只要它们分别分布在不同的 Segment 上。这个值可以在初始化的时候设置为其他值，但是一旦初始化后，它是不可以扩容的。在具体到每个 Segment 内部，其实每个 Segment 很像之间介绍的 HashMap，不过它要保证线程安全，所以处理起来要麻烦些。

#### 3.4.2.4. Java8 实现 (引入了红黑树)

​	Java8 对 ConcurrentHashMap 进行了比较大的改动，Java8 也引入了红黑树。

**Java8 ConcurrentHashMap 结构**







### 3.4.3. HashTable (线程安全)

​	Hashtable 遗留类，很多映射的常用功能与 HashMap 类似，不同的是它继承自 Dictionary 类，并且是线程安全的，任一时间只有一个线程能写 HashTable，并发性不如 ConcurrentHashMap，因为 ConcurrentHashMap 引入了分段锁。HashTable 不建议在新代码中使用，不需要线程安全的场合可以用 HashMap 替换，需要线程安全的场合可以用 ConcurrentHashMap 替换。

### 3.4.4. TreeMap (可排序)

​	TreeMap 实现 SortedMap 接口，能够把它保存的记录根据键排序，默认是按值的升序排序，也可以指定排序的比较器，当用 Iterator 遍历 TreeMap 时，得到的记录是排过序的。

​	如果使用排序的映射，建议使用 TreeMap。

​	在使用 TreeMap 时，key 必须实现 Comparable 接口或者在构造 TreeMap 传入自定义的 Comparator，否则会在运行时抛出 java.lang.ClassCastException 类型异常。

### 3.4.5. LinkHashMap (记录插入顺序)

​	LinkedHashMap 是 HashMap 的一个子类，保存了记录的插入顺序，在用 Iterator 遍历 LinkedHashMap 时，先得到的记录肯定是先插入，也可以在构造时带参数，按照访问次序排序。



# 4. Java 多线程并发

### 4.1.1.  Java 并发知识库







### 4.1.2. Java 线程实现/创建方式

#### 4.1.2.1. 继承 Thread 类

​	Thread 类本质上是实现了 Runnable 接口的一个实例。启动线程的唯一方法就是通过 Thread 类的 start() 实例方法。start() 方法是一个 native 方法，它将启动一个新线程，并执行 run() 方法。

```java
public class MyThread extends Thread {
		public void run() {
				System.out.println("MyThread.run()");
		}
}
MyThread myThread1 = new MyThread();
myThread1.start();

```

#### 4.1.2.2. 实现 Runnable 接口

​	如果自己的类已经 extends 另外一个类，就无法直接 extends Thread，此时实现一个 Runnable 接口。

```java
public 	class MyThread extends OtherClass Implements Runnable {
  public void run() {
    System.out.println("MyThread.run()");
  }
}
// 启动一个 MyThread，需要先实例化一个 Thread，并传入自己的 MyThread 实例：
MyThread mythread = new MyThread();
Thread thread = new Thread(myThread);
thread.start();
// 事实上，当传入一个 Runnable target 参数给 Thread 后，Thread 的 run() 方法就会调用 target.run()
public void run() {
  if (target != null ){
    target.run();
  }
}
```

#### 4.1.2.3. ExecutorService、Callable<Class>、Future 有返回值线程

​	有返回值的任务必须实现 Callable 接口，类似的，无返回值的任务必须 Runnable 接口。执行 Callable 任务后，可以获取一个 Future 的对象，在该对象上调用 get 就可以获取到 Callable 任务返回的 Object 了，再结合 ExcutorService 就可以实现传说中有返回结果的多线程了。

```java
public class MyCallable implements Callable<Object> {
  Object call() throws Exception{
    // 处理业务逻辑 TODO
    Object obj = new Object();
    // 返回处理结果
    return obj;  
  }
}


// 创建一个线程
ExecutorService pool = Executors.newFixedThreadPool(taskSize);
// 创建多个有返回值的任务
List<Future> list = new ArrayList<Futrue>();
for(int i = 0 ; i < taskSize; i++){
  Callable c = new MyCallable(i+"");
  // 执行任务并获取 Future 对象
  Future f = pool.submit(c);
  list.add(f);
}
// 关闭线程池
pool.shutdown();
// 获取所有并发任务的运行结果
for (Future f : list) {
  // 从 Future 对象上获取任务的返回值，并输出到控制台
  System.out.println("res: " + f.get().toString());
}
```



### 4.1.3. 4 种线程池

​	Java 里面线程池的顶级接口是 Executor，当是严格意义上讲 Executor 并不是一个线程池，而是一个执行线程的工具。真正的线程池接口是 **ExecuteService**。









#### 4.1.3.1. newCachedThreadPool

​	创建一个可根据需要创建新线程的线程池，但是在以后构造的线程可用时将重用它们。对于执行很多短期异步任务的程序而言，这些线程池通常可提高程序的性能。调用 execute 将重用以前构造的线程 (如果线程可用)。如果现在线程没有可用的，则创建一个新线程并添加到池中。终止并从缓存中移除那些已有 60 秒钟未被使用的线程。因此，长时间保持空闲的线程池不会使用任务资源。



#### 4.1.3.2. newFixThreadPool

​	创建一个可重用固定线程数的线程池，以共享的无界队列方式来运行这些线程。在任意点，在大多数 Threads 线程会处于处理任务的活动状态。如果在所有线程处于活动状态时提交附加任务，则在有可用线程之前，附加任务将在队列中等待。如果在关闭前的执行期间由于失败而导致任何线程终止，那么一个新线程将代替它执行后续的任务 (如果需要)。在某个线程被显式地关闭之前，池中的线程将一直存在。

#### 4.1.3.3.  newScheduledThreadPool

​	创建一个线程池，它可安排在给定延迟后运行命令或者定期地执行

```java
ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(3);
scheduledThreadPool.schedule(new Runnable(){
  @Override
  public void run() {
    System.out.println("延迟三秒！");
  }
}, 3, TimeUnit.SECONDS);

scheduledThreadPool.scheduleAtFixedRate(new Runnable(){
  @Override
  public void run(){
    System.out.println("延迟1秒后每三秒执行一次！");
  }
}, 1, 3, TimeUnit.SECONDS);
```

#### 4.1.3.4. newSingleThreadExecutor

​	Executors.newSingleThreadExecutor() 返回一个线程池 (这个线程池只有一个线程)，这个线程池可以在线程死后 (或发生异常时) 重新启动一个线程来替代原来的线程继续执行下去！

### 4.1.4. 线程生命周期(状态)

​	当线程被创建并启动以后，它既不是一启动就进入执行状态，也不是一直处于执行状态。在线程的生命周期中，它要经过新建 (New)、就绪 (Runnable)、运行 (Running)、阻塞 (Blocked) 和死亡 (Dead) 5中状态。尤其是当多线程启动以后，它不可能一直 "霸占" 着 CPU 独自运行，所以 CPU 需要在多条线程之间切换，于是线程状态也会多次在运行、阻塞之间切换。



#### 4.1.4.1. 新建状态 (NEW)

​	当一程序使用 new 关键字创建一个线程之后，改线程就处于新建状态，此时仅由 JVM 为其分配内存，并初始化其成员变量的值

#### 4.1.4.2. 就绪状态 (RUNNABLE)

​	当线程对象调用了 start() 方法之后，该线程处于就绪状态。 Java 虚拟机会为其创建方法调用栈和程序计数器，等待调度运行。

#### 4.1.4.3. 运行状态 (RUNNING)

​	如果处于就绪状态的线程获得了 CPU，开始执行 run() 方法的线程执行体，则该线程处于运行状态。

#### 4.1.4.4. 阻塞状态 (BLOCKED)

​	阻塞状态时指线程因为某种原因放弃了 CPU 使用权，也立即让出了 CPU timeslice，暂时停止运行。直到线程进入可运行 (runnable) 状态，才有机会再次获得 CPU timeslice 转到运行 (running) 状态。阻塞的情况分三种：

**等待阻塞 (o.wait->等待队列)：**

​	运行 (running) 的线程执行 o.wait() 方法，JVM 会把该线程放入等待队列 (waitting queue) 中。

**同步阻塞 (lock->锁池)：**

​	运行 (running) 的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则 JVM 会把该线程放入锁池 (lock pool) 中。

**其他阻塞 (sleep|join)**

​	运行 (running) 的线程执行 Thread.sleep(long ms) 或 t.join() 方法，或者发出 I/O 请求时，JVM 会把该线程置为阻塞状态。当 sleep() 状态超时、join() 等待线程终止或者超时、或者 I\O处理完毕，线程重新转入可运行 (running) 状态。

#### 4.1.4.5. 线程死亡 (DEAD)

​	线程会以下面三种方式结束，结束后就是死亡状态。

**正常结束**

1. run() 或 call() 方法执行完成，线程正常结束。

**异常结束**

2. 线程抛出一个未捕获的 Exception 或 Error。

**调用 stop**

3. 直接调用该线程的 stop() 方法来结束该线程——该方法容易导致死锁，不推荐使用。









### 4.1.5. 终止线程4中方式

#### 4.1.5.1. 正常运行结束

​	程序运行结束，线程自动结束。

#### 4.1.5.2. 使用退出标志退出线程

​	一般 run() 方法执行完，线程就会正常结束，然而，常常有一些线程是私服线程。它们需要长时间的运行，只有在外部某些条件满足的情况下，才能关闭这些线程。使用一个变量来控制循环，例如：最直接的方法就是设置一个 Boolean 类型标志，并通过设置这个标志为 true 或 false 来控制 while 循环是否退出，代码示例：

```java
public class ThreadSafe extends Thread {
  public volatile boolean exit = false;
  public void run() {
    while(!exit){
      // do something
      
    }
  }
}
```

定义了一个退出标志 exit，当 exit 为 true 时，while 循环退出，exit 的默认值为 false。在定义 exit时，使用了一个 Java 关键字 volatile，这个关键字的目的是使 exit 同步，也就是说在同一时刻只能由一个线程来修改 exit 的值。



#### 4.1.5.3. Interrupt 方法结束线程

​	使用 interrupt() 方法来中断线程有两种情况：

1. 线程处于阻塞状态：如使用了 sleep，同步锁的 wait，socket 中的 receiver，accept 等方法时，会使线程处于阻塞状态。当调用线程的 interrupt() 方法时，会抛出 InterruptException 异常。阻塞中的那个方法抛出这个异常，通过代码捕获该异常，然后 break 跳出循环状态，从而让我们有机会结束这个线程的执行。通常很多人认为只要调用 interrupt 方法线程就会结束，实际上是错的，一定要先捕获 InterruputedException 异常之后通过 break 来跳出循环，才能正常结束 run 方法。

2. 线程未处于阻塞状态：使用 isInterrupted() 判断线程的中断标志来退出循环。当使用 interrupt() 方法时，中断标志就会置true，和使用自定义的标志来控制循环是一样的道理。

   ```java
   public class ThreadSafe extends Thread {
     public void run() {
       while (!isInterrupted()){// 非阻塞过程通过判断中断标志来退出
         try{
           Thread.sleep(5*1000);// 阻塞过程捕获中断异常来退出
         }catch(InterruptedException e){
           e.printStackTrace();
           break;// 捕获到异常之后，执行 break 跳出循环
         }
       }
     }
   }
   ```

   #### 4.1.5.4. stop 方法终止线程 (线程不安全)

   ​	程序中可以直接使用 thread.stop() 来强行终止线程，但是 stop 方法是很危险的，就象突然关闭计算机电源，而不是按正常程序关机一样，可能会产生不可预料的结果，不安全主要是：thread.stop() 调用之后，创建子线程的线程会抛出 ThreadDeatherror 的错误，并且会释放子线程持有的所有锁。一般任何进行加锁的代码块，都是为了保护数据的一致性，如果在调用 thread.stop() 后导致了该线程所持有的所有锁突然释放(不可控制)，那么被保护数据就有可能呈现不一致，其他线程在使用这些被破坏的数据时，有可能导致一些很奇怪的应用程序错误。因此，并不推荐使用 stop 方法来终止线程。

   ### 4.1.6. sleep 和 wait 区别

   1. 对于 sleep() 方法，我们首先要知道该方法是属于 Thread 类中的。而 wait() 方法，则是属于 Object 类中的。
   2. sleep() 方法导致了程序暂停执行指定的时间，让出 cpu 给其他线程，但是它的监控状态依然保持着，当指定的时间到了又会恢复运行状态。
   3. 在调用 sleep() 方法过程中，线程不会释放对象锁。
   4. 而当调用 wait() 方法的时候，线程会放弃对象锁，进入等待此对象的等待锁定池，只有针对此对象调用 notify() 方法后本线程才进入对象锁进入运行状态。

