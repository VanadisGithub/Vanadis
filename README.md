# Java
## JVM
### 内存结构
（运行时数据区）
* 线程共享
    * [堆（Heap）](https://www.cnblogs.com/ibelieve618/p/6380328.html)
        * 新生代（1/3）
            * Eden（伊甸）空间（8/10）
            * From Survivor空间（1/10）
            * To Survivor空间（1/10）
        * 老年代（2/3）
    * 方法区（Method）
     * > 它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据

        * JDK7：永久代（Per Gen）
            * JVM内存
        * JDK8：元空间（MetaSpace）
            * 系统内存
* 线程私有
    * 栈
     * > 与程序计数器一样，Java虚拟机栈（Java Virtual Machine Stacks）也是线程私有的，它的生命周期与线程相同。虚拟机栈描述的是Java方法执行的内存模型：每个方法被执行的时候都会同时创建一个栈帧（Stack Frame）用于存储局部变量表、操作栈、动态链接、方法出口等信息。每一个方法被调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。

    * 本地方法栈
    * 程序计数器
     * > 此内存区域是唯一一个在Java虚拟机规范中没有规定任何OutOfMemoryError情况的区域

### GC
* 对象分配规则
    * 1、对象优先分配在Eden区
        * 如果Eden区没有足够的空间时，虚拟机执行一次Minor GC
    * 2、大对象（大量连续内存空间的对象）直接进入老年代
        * 这样做的目的是避免在Eden区和两个Survivor区之间发生大量的内存拷贝（新生代采用复制算法收集内存）。
    * 3、长期存活的对象进入老年代
        * 虚拟机为每个对象定义了一个年龄计数器，如果对象经过了1次Minor GC那么对象会进入Survivor区，之后每经过一次Minor GC那么对象的年龄加1，知道达到阀值对象进入老年区。
    * 4、动态判断对象的年龄
        * 如果Survivor区中相同年龄的所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象可以直接进入老年代。
    * 5、空间分配担保
        * 每次进行Minor GC时，JVM会计算Survivor区移至老年区的对象的平均大小，如果这个值大于老年区的剩余值大小则进行一次Full GC，如果小于，检查HandlePromotionFailure设置，如果true则只进行Monitor GC,如果false则进行Full GC。
* 对象存活判断
    * 引用计数
     * > 引用计数：每个对象有一个引用计数属性，新增一个引用时计数加1，引用释放时计数减1，计数为0时可以回收。此方法简单，无法解决对象相互循环引用的问题。

        * 强引用
        * 软引用
        * 弱引用
        * 虚引用
    * 可达性分析
     * > 可达性分析（Reachability Analysis）：从GC Roots开始向下搜索，搜索所走过的路径称为引用链。当一个对象到GC Roots没有任何引用链相连时，则证明此对象是不可用的。不可达对象。

* 垃圾收集算法
    * 分代收集算法
     * > 分代收集算法，“分代收集”（Generational Collection）算法，把Java堆分为新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算法
     * > 
     * > 1. JAVA 虚拟机提到过的处于方法区的永生代(Permanet Generation)，它用来存储 class 类，
     * > 常量，方法描述等。对永生代的回收主要包括废弃常量和无用的类。
     * > 2. 对象的内存分配主要在新生代的 Eden Space 和 Survivor Space 的 From Space(Survivor 目
     * > 前存放对象的那一块)，少数情况会直接分配到老生代。
     * > 3. 当新生代的 Eden Space 和 From Space 空间不足时就会发生一次 GC，进行 GC 后，Eden Space 和 From Space 区的存活对象会被挪到 To Space，然后将 Eden Space 和 From Space 进行清理。
     * > 4. 如果 To Space 无法足够存储某个对象，则将这个对象存储到老生代。
     * > 5. 在进行 GC 后，使用的便是 Eden Space 和 To Space 了，如此反复循环。
     * > 6. 当对象在 Survivor 区躲过一次 GC 后，其年龄就会+1。默认情况下年龄到达 15 的对象会被 移到老生代中。

        * 新生代
            * MinorGC
             * > MinorGC的过程：MinorGC采用复制算法。首先，把Eden和ServivorFrom区域中存活的对象复制到ServicorTo区域（如果有对象的年龄以及达到了老年的标准，则赋值到老年代区），同时把这些对象的年龄+1（如果ServicorTo不够位置了就放到老年区）；然后，清空Eden和ServicorFrom中的对象；最后，ServicorTo和ServicorFrom互换，原ServicorTo成为下一次GC时的ServicorFrom区

                * 触发
                    * 当Eden区没有足够的空间进行分配时
        * 老年代
            * MajorGC
             * > MajorGC采用标记—清除算法：首先扫描一次所有老年代，标记出存活的对象，然后回收没有标记的对象。MajorGC的耗时比较长，因为要扫描再回收。MajorGC会产生内存碎片，为了减少内存损耗，我们一般需要进行合并或者标记出来方便下次直接分配。

        * FullGC
            * 触发
                * 调用System.gc时，系统建议执行Full GC，但是不必然执行
                * 老年代空间不足
                * 方法去空间不足
                * 通过Minor GC后进入老年代的平均大小大于老年代的可用内存
                * 由Eden区、From Space区向To Space区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小。
    * 复制算法
     * > 复制算法，“复制”（Copying）的收集算法，它将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。

    * 标记 -清除算法
     * > 标记 -清除算法，“标记-清除”（Mark-Sweep）算法，如它的名字一样，算法分为“标记”和“清除”两个阶段：首先标记出所有需要回收的对象，在标记完成后统一回收掉所有被标记的对象

    * 标记-压缩算法
     * > 标记-压缩算法，标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存

* 垃圾收集器
    * Serial 垃圾收集器(单线程、复制算法)
    * ParNew 垃圾收集器(Serial+多线程)
    * Parallel Scavenge 收集器(多线程复制算法、高效)
    * SerialOld收集器(单线程标记整理算法)
    * ParallelOld收集器(多线程标记整理算法)
    * CMS收集器(多线程标记清除算法)
    * G1 收集器
### JVM调优
* GC日志分析
    * Young GC日志
    * Full GC日志
* [命令](http://www.ityouknow.com/jvm/2017/09/03/jvm-command.html)
    * jps
    * jstat
    * jmap
    * jhat
    * jstack
    * jinfo
* [工具](http://www.ityouknow.com/jvm/2017/09/22/jvm-tool.html)
    * JConsole
    * jvisualvm
    * Memory Analyzer Tool
    * GChisto
### 类加载
* 生命周期
* > 虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验，解析和初始化，最终形成可以被虚拟机直接使用的java类型。

    * 加载
        * 把class字节码文件从各个来源
通过类加载器装载入内存中
            * 1、通过一个类的全限定名来获取其定义的二进制字节流。
            * 2、将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
            * 3、在Java堆中生成一个代表这个类的java.lang.Class对象，作为对方法区中这些数据的访问入口。
    * 连接
        * 验证
            * 确保加载的类信息符合jvm规范，
没有安全方面的问题。
                * 文件格式验证
                * 元数据验证
                * 字节码验证
                * 符号引用验证
        * 准备
            * 正式为类变量（static变量）分配内存并设置类变量初始值的阶段，这些内存都将在方法区中进行分配。
        * 解析
            * 虚拟机常量池内的符号引用替换为直接引用的过程。（比如String s ="aaa",转化为 s的地址指向“aaa”的地址）
    * 初始化
        * 初始化阶段是执行类构造器方法的过程。类构造器方法是由编译器自动收集类中的所有类变量的赋值动作和静态语句块（static块）中的语句合并产生的。
        * 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先初始化其父类的初始化
        * 虚拟机会保证一个类的构造器方法在多线程环境中被正确加锁和同步
        * 当访问一个java类的静态域时，只有真正声明这个静态变量的类才会被初始化。
    * 结束生命周期
        * 执行了System.exit()方法
        * 程序正常执行结束
        * 程序在执行过程中遇到了异常或错误而异常终止
        * 由于操作系统出现错误而导致Java虚拟机进程终止
* 类加载器（继承java.lang.ClassLoader）
    * Bootstrap ClassLoader（启动类加载器）
        * 加载java核心类库(jre/lib/rt.jar)，无法被java程序直接引用
        * 加载扩展类和应用程序类加载器。
        * 并指定他们的父类加载器。
    * Extension ClassLoader（扩展类加载器）
        * 用来加载java的扩展库（jre/ext/*.jar）java虚拟机的实现会自动提供一个扩展目录
    * Application ClassLoader（应用程序类加载器）
        * 根据java应用的类路径（classpath路径），一般来说，java应用的类都是由他来完成加载的
    * 自定义类加载器
        * 通过继承java.lang.ClassLoader类的方式实现自己的类加载器，以满足一些特殊的需求（比如加密代码的解密加载）
* 类加载过程
    * 类的主动引用
（一定会发生类的初始化）
        * new,getstatic,putstatic,invokestatic
        * 调用类的静态成员（除了final常量）和静态方法
        * 使用java.lang.reflect包的方法对类进行反射调用
        * 当初始化一个类，如果其父类没有被初始化，则先初始化他的父类
        * 当要执行某个程序时，一定先启动main方法所在的类
    * 类的被动引用
（不会发生类的初始化）
        * 引用常量(final类型)不会触发此类的初始化（常量在编译阶段就存入调用类的常量池中了）
        * 通过数组定义类应用，不会触发此类的初始化  A[] a = new A[10];
        * 当访问一个静态变量时，只有真正生命这个静态变量的类才会被初始化（通过子类引用父类的静态变量，不会导致子类初始化）
* 类加载机制
    * 全盘负责
    * 双亲委派模型
     * > 如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把请求委托给父加载器去完成，依次向上，因此，所有的类加载请求最终都应该被传递到顶层的启动类加载器中，只有当父加载器在它的搜索范围中没有找到所需的类时，即无法完成该加载，子加载器才会尝试自己去加载该类

        * 破坏双亲委派模型
            * java设计团队曾引入了一个线程上下文类加载器，这个类加载器可以通过java.lang.Thread类的setContextClassloaser()方法进行设置
            * OSGI(动态模型系统)
                * 动态改变构造
                * 模块化编程与热插拔
    * 缓存机制
## JDK
### lang
* 数据类型
    * 基本类型
        * boolean
            * 逻辑
        * char
            * 文本
                * 2字节
        * byte
            * 整数
                * 8位
        * short
            * 整数
                * 16位
        * int
            * 整数
                * 32位
        * long
            * 整数
                * 64位
        * float
            * 浮点
                * 32位
        * double
            * 浮点
                * 64位
        * void
    * 包装类型
        * 基本类型包装类
        * BigInteger
        * BigDecimal
    * 缓存池
        * Integer
            * -128 ~ 127（最大值可以通过-XX:AutoBoxCacheMax=<size>参数）
        * Byte/Short/Long
            * -128 ~ 127（固定）
        * Character
            * 0 ~ 127（固定）
    * 自动拆装箱
        * 操作
            * 装箱
                * 包装类的valueOf()
            * 拆箱
                * 包装类对象的xxxValue()
        * 问题
            * 包装对象的数值比较，equals，除非（-128到127可以使用==）
            * 自动拆箱，如果包装类对象为null，那么自动拆箱时就有可能抛出NPE。
            * 如果一个for循环中有大量拆装箱操作，会浪费很多资源。
* 反射
    * java.lang.reflect
        * Field ：可以使用 get() 和 set() 方法读取和修改 Field 对象关联的字段；
        * Method ：可以使用 invoke() 方法调用与 Method 对象关联的方法；
        * Constructor ：可以用 Constructor 创建新的对象。
    * 优点
        * 可扩展性 ：应用程序可以利用全限定名创建可扩展对象的实例，来使用来自外部的用户自定义类。
        * 类浏览器和可视化开发环境 ：一个类浏览器需要可以枚举类的成员。可视化开发环境（如 IDE）可以从利用反射中可用的类型信息中受益，以帮助程序员编写正确的代码。
        * 调试器和测试工具 ： 调试器需要能够检查一个类里的私有成员。测试工具可以利用反射来自动地调用类里定义的可被发现的 API 定义，以确保一组测试中有较高的代码覆盖率。
    * 缺点
        * 性能开销 ：反射涉及了动态类型的解析，所以 JVM 无法对这些代码进行优化。因此，反射操作的效率要比那些非反射操作低得多。我们应该避免在经常被执行的代码或对性能要求很高的程序中使用反射。
        * 安全限制 ：使用反射技术要求程序必须在一个没有安全限制的环境中运行。如果一个程序必须在有安全限制的环境中运行，如 Applet，那么这就是个问题了。
        * 内部暴露 ：由于反射允许代码执行一些在正常情况下不被允许的操作（比如访问私有的属性和方法），所以使用反射可能会导致意料之外的副作用，这可能导致代码功能失调并破坏可移植性。反射代码破坏了抽象性，因此当平台发生改变的时候，代码的行为就有可能也随着变化。
* String
    * final
        * 不可被继承
    * 实现
        * JDK8：char
        * JDK9：byte
    * 不可变好处
        *  可以缓存 hash 值：做HashMap的key，只需计算一次
        * String Pool
        * 安全性
        * 线程安全
    * String Pool
        * intern() 
    * StringBuffer
        * 线程安全
    * StringBuilder
* Object
* Throwable（异常）
    * Error
        * VirtualMachineError
            * OutOfMemoryError
            * StackOverflowError
                * 当一个应用递归调用的层次太深而导致堆栈溢出或者陷入死循环时抛出该错误
        * AWTError
    * Exception
        * RuntimeException
            * NullPointerException
            * IndexOutOfBoundsException
            * ArithmaticException
            * ClassCastException
            * RejectedExecutionHandler
        * IOException
            * FileNotFoundException
            * EOFException
            * SocketException
* 泛型
### util
* 集合
    * Collection（接口）
单列集合
        * 继承
            * Iterator（Iterable）迭代器接口
                * 方法
                    * next()
                    * hashNext()
                    * remove()
        * 子类
            * Set
                * HashSet
                    * 数组；不保证顺序；集合元素可以为 NULL；
                    * LinkedHashSet
                        * 链表 和 哈希表；有序；
                    * hashCode()
                * SortedSet（接口）
                    * TreeSet
                        * 红黑树
                        * 自定义排序
                            * 实现Comparable接口，用来排序
                * EnumSet
                * 概要: 共同点
                    * 都不允许元素重复
                    * 都不是线程安全的类，解决办法：Set set = Collections.synchronizedSet(set 对象)
            * List
                * ArrayList
                    * 数组
                        * 线程不安全，效率高
                            * 查询快，增删慢
                                * 容量不够扩容，当前容量长度 X 1.5（JDK7：+1）
                * LinkedList
                    * 双向链表
                        * 线程不安全，效率高
                            * 查询慢，增删快
                                * 链表直接在头部尾部新增都可以，所以没有倍数
                * Vector
                    * 数组结构
                        * 线程安全，效率低
                            * 查询快，增删慢
                                * 扩容长度是：当前容量长度*2
            * Queue
                * PriorityQueue
        * 集合工具类
            * Collections
            * Comparable
                * 内比较器
                    * compareTo
                        * d1.compareTo(d2)
            * Comparator
                * 外比较器
                    * compare
                        * dc.compare(d1, d2)
    * Map（接口）
双列集合
        * AbstractMap(接口)
            * HashMap
                * 实现
                    * JDK7
                        * 数组+链表
                    * JDK8
                        * 数组+链表+红黑树
                         * > 时间复杂度取决于链表的长度，为 O(n) 
                         * > 当链表中的元素超过了 8 个以后，会将链表转换为红黑树，时间复杂度为 O(logN)

                * 子类
                    * LinkedHashMap
                    * ConcurrentHashMap
                     * > 在jdk8之前是使用分段加锁的一个方式，分成16个桶，每次只加锁其中一个桶，而在jdk8又加入了红黑树和CAS算法来实现

                        * Segment 数组
                            * 通过继承 ReentrantLock 来进行加锁
                * HashMap线程不安全
                 * > 碰撞与扩容：HashMap在put的时候，插入的元素超过了容量（由负载因子决定）的范围就会触发扩容操作，就是rehash，这个会重新将原数组的内容重新hash到新的扩容数组中，在多线程的环境下，存在同时其他的元素也在进行put操作，如果hash值相同，可能出现同时在同一数组下用链表表示，造成闭环，导致在get时会出现死循环，所以HashMap是线程不安全的

                * 多线程安全方式
                    * Hashtable
                    * Collections.synchronizedMap(new Hashtable())
                    * ConcurrentHashMap
                     * > 在jdk8之前是使用分段加锁的一个方式，分成16个桶，每次只加锁其中一个桶，而在jdk8又加入了红黑树和CAS算法来实现

            * HashTable
                * Properties
            * SortMap（接口）
                * TreeMap
* concurrent（并发）
    * 并发基础
        * CAS
            * Compare And Swap 即比较交换
                * CPU的原子指令
                    * CAS(V,E,N)
                        * 如果V值等于E值，则将V的值设为N。若V值和E值不同，则说明已经有其他线程做了更新，则当前线程什么都不做。
                * 缺陷
                    * 循环时间长开销很大
                    * 只能保证一个共享变量的原子操作
                    * ABA问题
                        * 解决：AtomicStampedReference 时间戳
        * AQS
    * Atomic
        * 基本类型
            * AtomicBoolean
            * AtomicInteger
            * AtomicLong
        * 数组类型
        * 引用类型
        * 字段类型
    * Lock
        * 分类
            * 宏观
                * 乐观锁
                * 悲观锁
            * 顺序
            * 重量级
                * 自旋锁
                * 偏向锁
                * 轻量级锁
                * 重量级锁
        * 优化
            * 适应自旋锁
            * 锁消除
            * 锁粗化
        * ReentrantLock
            * 公平锁
            * 非公平锁
                * 抢占
    * 并发集合
        * ConcurrentHashMap
        * ConcurrentLinkedDeque
        * ConcurrentLinkedQueue
        * ConcurrentSkipListMap
        * ConcurrentSkipListSet
    * 线程
        * 生命周期
            * New
                * 新建
            * Runnable
                * 就绪
                    * start()
            * Running
                * 运行
                    * 获取到cpu
                        * run()
            * Blocked
                * 阻塞
                    * 等待阻塞
                        * wait()
                    * 同步阻塞
                        * synchronized
                    * 其他阻塞
                        * sleep()
                        * join()
                        * I/O
            * Dead
                * 死亡
                    * run()结束
                    * stop()
                    * 抛出异常
        * 线程实现
            * 继承Thread类
            * 实现Runnable接口（推荐）
                * 因为实现接口的方式比继承类的方式更灵活，也能减少程序之间的耦合度
            * 使用Callable和Future接口创建线程
        * 线程调度
            * 优先级
                * Thread.setPriority()
                    * 默认5，总1-10
            * 睡眠
                * Thread.sleep()
                    * 线程不会释放对象锁
            * 让步
                * Thread.yield()
            * 加入
                * Thread.join()
            * 中断
                * Thread.interrupet()
                    * 只是修改中断状态
            * 等待
                * Object.wait()
                    * 线程会放弃对象锁
            * 唤醒
                * Object.notify()
                    * 唤醒线程，进入对象锁池中去竞争
        * 线程同步
            * synchronized
                * 方法同步块
            * volidate
                * 可见性，不具备原子性
                    * 每次都从内存里读
    * 线程池
        * ThreadPoolExecutor
            * ExecutorService
            * 参数
                * corePoolSize
                * maximumPoolSize
                * keepAliveTime
                * BlockingQueue
                    * SynchronousQueue（同步队列）
                     * > 这个队列接收到任务的时候，会直接提交给线程处理，而不保留它（名字定义为 同步队列）。但有一种情况，假设所有线程都在工作怎么办？这种情况下，SynchronousQueue就会新建一个线程来处理这个任务。所以为了保证不出现（线程数达到了maximumPoolSize而不能新建线程）的错误，使用这个类型队列的时候，maximumPoolSize一般指定成Integer.MAX_VALUE，即无限大，去规避这个使用风险。
                     * > 
                     * > 作者：骑小猪看流星
                     * > 链接：https://www.jianshu.com/p/50fffbf21b39
                     * > 来源：简书
                     * > 简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。

                    * LinkedBlockingQueue（链表阻塞队列）
                     * > 这个队列接收到任务的时候，如果当前线程数小于核心线程数，则新建线程(核心线程)处理任务；如果当前线程数等于核心线程数，则进入队列等待。由于这个队列没有最大值限制，即所有超过核心线程数的任务都将被添加到队列中，这也就导致了maximumPoolSize的设定失效，因为总线程数永远不会超过corePoolSize
                     * > 
                     * > 作者：骑小猪看流星
                     * > 链接：https://www.jianshu.com/p/50fffbf21b39
                     * > 来源：简书
                     * > 简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。

                    * ArrayBlockingQueue（数组阻塞队列）
                     * > 可以限定队列的长度（既然是数组，那么就限定了大小），接收到任务的时候，如果没有达到corePoolSize的值，则新建线程(核心线程)执行任务，如果达到了，则入队等候，如果队列已满，则新建线程(非核心线程)执行任务，又如果总线程数到了maximumPoolSize，并且队列也满了，则发生错误
                     * > 
                     * > 作者：骑小猪看流星
                     * > 链接：https://www.jianshu.com/p/50fffbf21b39
                     * > 来源：简书
                     * > 简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。

                    * DelayQueue（延迟队列）
                     * > 队列内元素必须实现Delayed接口，这就意味着你传进去的任务必须先实现Delayed接口。这个队列接收到任务时，首先先入队，只有达到了指定的延时时间，才会执行任务

            * 实现
                * newFixedThreadPool
                * newCachedThreadPool
                * newScheduledThreadPool
                * newSingleThreadExecutor 
    * Future
        * FutureTask
    * ThreadLocal
        * ThreadMap
### io
* IO/NIO
* Serializable
### 版本特性
* Java8
    * Lambda
    * Stream
    * Optional
* Java9
    * 模块系统
## 面向对象
### 特性
* 继承
    * 访问权限
    * 抽象类与接口
    * super
    * 重写与重载
* 封装
* 多态
    * 条件：继承、重写、向上转型
* 抽象
### [设计模式](http://www.runoob.com/design-pattern/design-pattern-tutorial.html)
* Proxy代理模式
    * 定义
        * 由于某些原因需要给某对象提供一个代理以控制对该对象的访问。这时，访问对象不适合或者不能直接引用目标对象，代理对象作为访问对象和目标对象之间的中介。
    * 分类
        * 静态代理
            * 代理类持有具体类的实例，代为执行具体类实例方法
        * 动态代理
            * 代理类在程序运行时创建的代理方式被成为动态代理
    * 应用
        * Spring AOP
* Singleton单例模式
    * 定义
        * 指一个类只有一个实例，且该类能自行创建这个实例的一种模式
    * 分类
        * 懒汉式单例
            * 类加载时没有生成单例
        * 饿汉式单例
            * 类一旦加载就创建一个单例
    * 应用
        * 某类只要求生成一个对象，如一个班班长
        * 当对象需要被共享的场合。由于单例模式只允许创建一个对象，共享该对象可以节省内存，并加快对象访问速度。如 Web 中的配置对象、数据库的连接池等
        * 当某类需要频繁实例化，而创建的对象又频繁被销毁的时候，如多线程的线程池、网络连接池等。
* Prototype原型模式
    * 定义
        * 用一个已经创建的实例作为原型，通过复制该原型对象来创建一个和原型相同或相似的新对象
    * 实现
        * Java 中的 Object 类提供了浅克隆的 clone() 方法，具体原型类只要实现 Cloneable 接口
* Factory工厂模式
    * 定义
        * 定义一个创建产品对象的工厂接口，将产品对象的实际创建工作推迟到具体子工厂类当中
* Delegate委派模式
* Strategy策略模式
    * 定义
        * 该模式定义了一系列算法，并将每个算法封装起来，使它们可以相互替换，且算法的变化不会影响使用算法的客户。
* Template模板模式
* Decorator装饰器模式
* Observer观察者模式

*XMind: ZEN - Trial Version*