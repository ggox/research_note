### 54个官方术语



1. 自适应自旋锁(adaptive spinning)

   自适应自旋锁是一个允许线程在特定点自旋等待特定事件发生而不是直接进行block并等待该事件发生或条件变化的通知的优化机制."自适应"是指决定在block之前自旋的时间.

2. 偏向锁(biased locking)

   偏向锁也是一种优化机制,它使得一个线程在释放锁之后,vm仍旧在逻辑上让该线程"持有"一个对象的锁,它的优化前提点是该线程会在稍后重新获取该锁(这是一个常见的事件),如果此时有其他线程争抢该锁,则必须撤消偏向锁持有者的偏向锁.

3. 块起始表(block start table)

   它表示一段堆内存区域,对象在该内存区域的起始地址从较低的地址开始,它的一个样例是remembered set中的card table变体.

4. 启动类加载器(bootstrap classloader)

   它是负责加载启动路径(尤其核心java平台类)下的类或资源的加载器,一般由虚拟机实现,在JAVA api层面,用classloader获取该实例会返回null.

   顺便提一下JAVA9之后的类加载机制变化,在JAVA9之前,类的加载是简单的"双亲委托"模式,应用类加载器委托给扩展类加载器,再向上委托到启动类加载器,父加载器不能加载时再自行加载,但在JAVA9之后开启了模块化系统,出现了"模块路径"的概念,并移除了"签名重写"和"扩展"机制,相应的移除了扩展类加载器,取而代之的是平台类加载器,相应的变化详述如下:

   应用类加载器不再URLClassLoader的实现类,而成为它的一个内部类,它是非JAVA SE或JDK模块之外的具名模块的默认加载器.

   扩展类加载器也不再是URLClassLoader的实例,而成为它的一个内部类,在JEP220之后,它不再通过扩展机制来加载类.它可以用来定义JAVA SE 和JDK中的可选模块,它现在被称为平台类加载器,使用ClassLoader::getPlatformClassLoader可以获取该加载器的实例.

   在JAVA9之后,启动类加载器由虚拟机和核心库共同实现,出于兼容性,使用ClassLoader api获取该类加载器仍旧会返回null,它定义了核心java se和jdk的模块.

   加载按如下顺序进行:

   应用类加载器首先搜索它的内建加载器定义的所有"具名模块",如果对这些加载器找到了合适的模块定义,将会使用该加载器加载class.如果class并没有在这些加载器定义的具名模块中找到,那么应用类加载器将会委托给双亲.如果双亲也没有找到,则应用类加载器搜索类路径,在类路径下找到的类将成为这些加载器的无名模块.

   原扩展类加载器由平台类加载器替代,平台类加载器会搜索所有内建加载器的具名模块定义,如果找到合适的模块,那么加载器会加载该类.(平台加载器现在可以委托给应用类加载器,它在一个模块位于更新模块路径且依赖了应用模块路径下的模块时会很有用)如果一个类没有在平台类加载器的的有加载器下定义的具名模块中找到,委托给父加载器加载.

   启动类加载器会检索自己定义的具名模块,如果一个类未能在启动加载器定义的具名模块中找到,则查找添加到启动类路径下的文件和目录(可通过-Xbootclasspath/选项指定).在该路径下找到的类将成为这些加载器的无名模块.

   三种加载器分别负责jdk中的不同模块,如下:

   平台类加载器负责的jdk模块:

   ```java
   java.activation* jdk.accessibility
   
   java.compiler* jdk.charsets
   
   java.corba* jdk.crypto.cryptoki
   
   java.scripting jdk.crypto.ec
   
   [java.se](http://java.se) jdk.dynalink
   
   java.se.ee jdk.incubator.httpclient
   
   java.security.jgss jdk.internal.vm.compiler*
   
   java.smartcardio jdk.jsobject
   
   java.sql jdk.localedata
   
   java.sql.rowset jdk.naming.dns
   
   java.transaction* jdk.scripting.nashorn
   
   java.xml.bind* jdk.security.auth
   
   java.xml.crypto jdk.security.jgss
   
   java.xml.ws* jdk.xml.dom
   
   java.xml.ws.annotation* jdk.zipfs
   
   (带'*'表示为"可更新模块".)
   ```

   应用类加载器负责加载的jdk模块:

   ```java
   jdk.aot jdk.jdeps
   
   jdk.attach jdk.jdi
   
   jdk.compiler jdk.jdwp.agent
   
   jdk.editpad jdk.jlink
   
   jdk.hotspot.agent jdk.jshell
   
   jdk.internal.ed jdk.jstatd
   
   jdk.internal.jvmstat jdk.pack
   
   jdk.internal.le jdk.policytool
   
   jdk.internal.opt jdk.rmic
   
   jdk.jartool jdk.scripting.nashorn.shell
   
   jdk.javadoc jdk.xml.bind*
   
   jdk.jcmd jdk.xml.ws*
   
   jdk.jconsole
   ```

   其他JAVA SE和jdk模块由启动类加载器负责:

   ```java
   java.base java.security.sasl
   
   java.datatransfer java.xml
   
   java.desktop jdk.httpserver
   
   java.instrument jdk.internal.vm.ci
   
   java.logging jdk.management
   
   java.management jdk.management.agent
   
   java.management.rmi jdk.naming.rmi
   
   java.naming [jdk.net](http://jdk.net)
   
   java.prefs jdk.sctp
   
   java.rmi jdk.unsupported
   ```

5. 字节码校验(bytecode verification)

   类的链接过程中的一个步骤,在这个步骤中分析方法字节码保证类型安全.

6. C1编译器(C1 compiler)

   C1编译器是一个快速轻量级的优化字节码编译器.它会执行一些值的编号,内联,类分析.它使用简单的面向cfg的SSA高级信息检索、面向机器的低级信息检索,一个线性扫描寄存器分配以及一个模板样式的代码生成器。

7. C2编译器(C2 compiler)

   它是高度的优化字节码编译器,也被称之为'opto',它使用"节点海洋" SSA "理想化" 信息检索,它会下沉到同一种机器规格的信息检索.它有一个图着色的寄存器,可给所有机器状态进行着色(包含本地的,全局的,参数寄存器和栈).C2编译器能做出的优化包含全局变量值编号,状态常量类型传递,常量折叠,全局代码移动,代数身份,方法内联(聚合的优化的和/或多态),内部替换,循环转换(去switch去轮循等),数组边界检查的消除等.

8. 卡表(card table)

   它是一种记录了一个代中oops改变的记录的remembered set.

9. 类数据共享(class data sharing)

   类数据共享是一个启动优化,它记录了一些类的内存结构,使虚拟机在后续的运行中不用再从class文件中去载入相应的类,而是直接映射到内存结构中的数据.

10. 类层级分析(class hierachy analysis)

    也被称之为'CHA',编译器会分析类树,以找出虚拟调用点的接收者是否有一个单一的实现者,如果存在,可以内联被调用者,编译器也可使用一些其他的静态调用机制.

11. 代码缓存(code cache)

    它是一个特殊的持有编译后代码的堆.这些对象不会被gc搬移,但它们可能会包含服务于gc roots 的oops.

12. 整理(compaction)

    整理是一个gc中常见的技术,它会将存活的对象密集地放置在一个虚拟地址空间,同时使得其他地址空间成为连续可用的空闲空间.

13. 并发(concurrency)

    并发或者说并发编程,是逻辑上多个指令流的同时执行,如果有可用的多处理器,那么逻辑上的同时执行可以物理上同是地执行,也就是我们所知的'并行'.

14. 并发gc(concurrent garbage collection)

    它是在java应用线程保持运行态的同时进行大部分工作的gc算法.

15. 拷贝gc(copying garbage collection)

    一种垃圾收集期间移动对象的gc算法.

16. 反优化(deoptimization)

    反优化是一个将编译的(或者更优化的)栈桢转化为解释的(或者弱优化的)栈桢的过程.它也被解释为放弃依赖条件被打破(或者假定被打破)的nmethod的过程.反优化nmethod一般会被重新编译以便适配应用行为的变化.举个例子,编译器初始假定一个引用的值从不为null,并且使用"捕获内存访问"的方法进行测试.后续程序运行过程中,程序使用了null值,那么方法必须进行反优化和重编译,使用显示的test-and-branch方式发现此类null值.

17. 依赖(dependency)

    它是nmethod关联的一个优化假定条件,它允许编译器在nmethod中值入优化代码.举例:一个类无子类,那么它可以简化方法转发和类型测试.载入一个新的类型(或者替换老类型)可能导致这个依赖条件为为false,这就需要舍弃掉并反优化这些依赖nmethod.

18. 伊甸园(eden)

    堆内存的一部分,特点是对象可以在其中高效地创建.

    注:是分代垃圾收集器所有的特性,在一些新的垃圾收集器中出现弱化(G1中为逻辑上不相连的区域,zgc和Shenandoah可算为'无代'的垃圾收集器,也就不存在eden的概念).

19. 空闲列表(free list)

    空闲列表是一种内存管理技术,它使无用的部分java对象堆彼此连接,而不是将这些无用堆部分放置在同一个block中.

20. 垃圾收集(garbage collection)

    即常说的内存的自动管理.

21. 垃圾收集根(garbage collection root)

    它是一个从外部指向java对象堆的指针.举例:从类的静态字段或活化栈桢的本地对堆中对象的引用.

22. GC图(GC map)

    gc映射是一个由JIT(C1或C2)在编译的栈桢内的寄存器或在栈内的对象指针的位置上插入的描述.每一个可能执行安全点操作的位置都有关联的gc映射图.gc知道如何去在一个栈中解析一个桢,怎么去从一个桢的nmethod请求一个gc映射,以及如何去取出栈桢内的gc映射和管理对象指针.

23. 分代垃圾收集(generational garbage collection)

    分代垃圾收集是一种对于不同堆区按存活时间长度不同分离对象的存储管理技术,它可以令这些不同的区域使用不同的算法进行收集.

24. 句柄(handle)

    句柄是一个包含对象指针的内存原语(word),它对gc完全可知,gc视之为根引用.c/c++代码通过句柄间接地引用对象指针,为了让gc更容易地找到和管理根集合.任何时刻,c/c++代码块进入安全点时,gc可能改变句柄中存放的对象指针.句柄只能是'局部的'(属于一个线程,受线程堆栈规则制约,但在线程线上并非必须的)或'全局的'(长期存活且显式取消分配),虚拟机实现了大量的句柄实现,它们全部对gc可识别.

25. 热锁(hot lock)

    高度竞态的锁.

26. 解释器(interpreter)

    解释器是由单个执行字节码实现方法调用的虚拟机模块.解释器具有一个高度特化栈桢部局和寄存器使用图的有限集,使用它们作用于所有的方法活化.Hotspot虚拟器会在启动时生成自己的解释器.

27. JIT编译器(JIT compilers)

    JIT编译器是一个应用运行时为应用(或类库)自生成代码的在线编译器.JIT即just in time.JIT编译器可以在java方法执行前非常快速地创建机器码.Hotspot编译器允许解释器执行java方法数千次以采样运行数据并热身,因为它可以在类加载初始化后观测到完整的类层级,热身周期使编译器有充足的依据做出优化决策.编译器也可以检视解释器收集的分支和类型的剖析信息.

28. jni接口(JNI)

    java本地方法接口.它定义了一组api,用于java代码调用native c代码,以及它怎么调用java虚拟机代码.

29. jvm工具接口(JVM TI)

    用于开发和监测jvm的工具.

30. 类指针(klass pointer)

    Java中对象头有两个word(在JAVA12推出的体验版Shenandoah垃圾收集器模型中新增了一个'间接指针',故有三个word).第二个word指向一个描述了原生对象的部局和行为的(一个元数据对象),对于java对象来主产,"klass"包含c++风格的"vtable".

31. 标记语(mark word)

    每个对象头的第一个word,它是一组按位划分的字段,包含同步状态和hash码,也可能有一个与同步有关信息关联的指针(低位编码),在gc过程中,也可能包含gc状态位.

32. nmethod

    nmethod是一个实现了一些java字节码的可执行代码块.它可能是一个完整的java方法,或者一个'osr'(当前栈替换)方法,它通常包含编译器内联的附加方法的对象代码.

33. 对象头(object header)

    对象头是每个gc管理的堆对象的通用开始结构.(每个对象指针指向一个对象的头部)包含关于堆对象部局,类型,gc状态,同步状态,身份标识哈希码等属性.对象头包含两个word(前述Shenandoah包含三个word),在数组中它后面紧随一个长度字段.注意java对象和vm内部对象具备同样的对象头格式.

34. 对象晋升(object promotion)

    把对象从一个代拷贝到另一个代的过程.

35. 老年代(old generation)

    一个存放存活较持久对象的堆区.

36. 当前栈替换(on-stack replacement)

    即前面说过的'OSR',它是一个把解释栈桢(弱优化)转换为一个编译的栈桢(强优化)的过程.这发生在编译器发现一个方法处于循环中的情景时,这会请求编译器在循环中的一个特殊入口点(特殊情况在后向分支)上生成一个特殊的nmethod,并转化控制到该nmethod,它的反过程即前述'反优化'.

37. 对象指针(oop)

    即object pointer,一般来说是一个指向gc管理堆的指针(这是一个传统的协议,o代表ordinary).它的实现是一个本地机器地址而不是一个句柄.对象指针可以由编译器或解释器java代码直接组装,因为gc知道这些代码中的对象指针的存活情况和位置.(参见gc映射)对象指针可以直接用c/c++代码来组装,但在跨越安全点时相应的代码一定要把它保持在每个句柄之内.

38. 并行类加载(parallel classloading)
39. 同一类加载器同一时刻加载多个class/type.

39. 并行gc(parallel garbage collection)

    并行gc是一个可以在多处理器场景下使用多线程保证更高的效率的gc算法.

40. 永久代(permanent generation)

    永久代是由gc管理的虚拟机自身分配对象的一个地址空间,它其实是被"误解称"为永久代的,几乎其中的所有对象都不会被回收,因为它们会存活很长一段时间,所以极少的回收(但不是不回收).

41. remembered set

    在代之间记录指针的一种数据结构.

42. 安全点(safepoint)

    在安全点,所有的GC roots已知且所有堆对象内容是一致的.从全局安全点的角度看,所有线程一定要在gc可以运行前在安全点阻塞.(特殊情况,运行JNI代码的线程可以继续运行,因为它们只用句柄,在安全点期间他们不能载入句柄的内容,只能阻塞)从局部角度看,安全点即是一个线程可能因为GC而阻塞执行过程的代码中的一个点.大多数的调用点可以作为安全点,有一些强不变量会在任何一个安全点保持true,但在非空全点期间会被无视.编译的java代码和c/c++代码都针对安全点间进行了优化,但对于跨安全点做的优化很少.JIT编译器会在每个安全点安插一个GC映射.在虚拟机中的c/c++代码会使用程式化的基于宏的规约(如TRAPS)来标记潜在安全点.

    目前作者从散碎的若干文档中可以确定安全点的影响点有:

    最常见的基于安全点的操作是gc,或者更精确地说为gc的"stop the world"阶段,但是在虚拟机中仍旧有很多依赖于安全点的操作.

    偏向锁的移除.

    线程挂起或停止(Thread.stop())

    JVMTI请求的内检操作.

    代码反优化.

    刷新代码缓存.

    类重定义(如执交换或在线监测instrumentation)

    偏向锁移除

    各种debug操作(栈dump,死锁检测等)

    从JAVA10开始，官方对于安全点做出了至少一个优化：即线程本地握手/或线程局部握手.

    该功能的核心思想在于在每一个java线程的安全点执行相应的回调操作(java线程或vm线程).

    虚拟机线程将会协调握手操作,它会在握手期间阻止全局安全点的发生,通过这样至少实现了以下几点优化目标:

    提升偏向锁的撤消操作,不再需要停掉所有的线程,只需要停止单个线程并执行偏向锁操作撤消即可.

    减少一些如获取栈迹等操作对虚拟机全局延迟影响.在执行安全的栈迹采样时可减少对信号量的依赖.

    使用异步的Dekker同步技术实现一些内在屏障操作,这可以通过使用java线程执行握手操作的方式实现.如G1和CMS需要使用的状态卡标记代码将不再需要内存屏障,结果就是G1的后处理写屏障可以被优化,有关避免内存屏障的代码分支也可移除.

43. 节点海洋(sea-of-nodes)

    它是C2编译器中的一个高等的中间表现形式,以SSA格式表现,数据流和控制流表现均以节点间的直线表示.与传统编译器的格式不同,传统编译器的控制流图不以代码块为边限.IR(猜测是intermediate representation的意思)允许节点在海洋中流动(受到节点之间的边线约束),直到它们在稍后的编译过程中被预定.

44. 可维护的代理(Serviceability Agent (SA))

    可维护的代理一一组sun公司的内部用来debug Hotspot虚拟机问题的代码.它也被用在一些JDK的工具包中,如jstack,jmap,jinfo,jdb等.

45. 栈图(stackmap)

    它是栈映射表(StackMapTable)属性的引用或者表中的一个特殊栈映射桢(StackMapFrame)的引用.

46. 栈映射表(StackMapTable)

    是在一个包含验证阶段由验证器使用的类型信息的类文件的一个代码属性.它包含一个StackMapFrames数组,它在JDK6中由javac命令自动生成.

47. 幸存者空间(survivor space)

    它是java对象堆中存储对象的一个区域,通常有一对幸存者区,回收一个区时,将一个幸存者区的被引用的(存活)对象拷贝到另一个存活着区.

48. 同步(synchronization)

    它可以用来协调并发职能,保证这些工作的安全性和活跃性.举例:通过锁保护所有访问主某共享数据的访问路径.

49. 线程本地分配缓存(TLAB)

    线程本地分配缓存(Thread-local allocation buffer),用来快速无同步地分配堆空间.编译后的代码对一些尝试用在当前线程本地分配缓存的方式达到"高位标记"的几个指令有一个"快速执行路径",当尝试分配对象时,如果这个标记位于线程本地分配缓存指定的限定地址之前,则代表分配空间成功.

50. 不常见的陷阱(uncommon trap)

    当C2生成的代码回滚到解释器以备后续执行时.C2一般按照常见案例进行编译,这允许它专注于最频繁常见执行路径的优化.举个例子,当一个类在编译时处于未初始化态而需要运行时初始值时,C2编译器在生成的代码中插入一个"不常见的陷阱".

51. 验证器(verifier)

    虚拟机中执行字节码校验的软件代码.

52. 虚拟机操作(VM Operations)

    虚拟机中可以被java线程请求的,但是一定要由虚拟机线程以串行方式执行的操作,这些操作通常是同步的,因此请求者将会block直到虚拟机线程完成旧有操作.很多这些操作通常需要虚拟机到达安全点.gc是一个简单的例子.

53. 写屏障(write barrier)

    写屏障是在每个对象指针存储时执行的代码,举个例子,如维护一个remembered set.

54. 年轻代(young generation)

    java堆中的用来存放最近分配的对象的区域.11