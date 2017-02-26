（1）自我介绍。
  
*  ...

（2）JVM如何加载一个类的过程，双亲委派模型中有哪些方法？

* 加载（加载class文件） 连接（验证（验证class对象的有效性），准备（分配内存以及赋予初始化值），解析（将符号引用转化为指针引用）） 初始化（初始化静态函数以及成员） 方法有load方法

（3）HashMap如何实现的？

* HashMap works on the principal of hashing.
* Map.Entry interface - This interface gives a map entry (key-value pair). HashMap in Java stores both key and value object, in bucket, as Entry object which implements this nested interface Map.Entry.
* hashCode() -HashMap provides put(key, value) for storing and get(key) method forretrieving Values from HashMap. When put() method is used to store (Key, Value) pair, HashMap implementation calls hashcode on Key object to calculate a hash that is used to find a bucket where Entry object will be stored. When get() method is used to retrieve value, again key object is used to calculate a hash which is used then to find a bucket where that particular key is stored.
* equals() - equals() method is used to compare objects for equality. In case of HashMap key object is used for comparison, also using equals() method Map knows how to handle hashing collision (hashing collision means more than one key having the same hash value, thus assigned to the same bucket. In that case objects are stored in a linked list.

![hashMap](https://qph.ec.quoracdn.net/main-qimg-8d490b416731e970856e0b051d904441-p)

* Though HashMap implementation provides constant time performance O(1) for get() and put() method but that is in the ideal case when the Hash function distributes the objects evenly among the buckets.But the performance may worsen in the case hashCode() used is not proper and there are lots of hash collisions. As we know now that in case of hash collision entry objects are stored as a node in a linked-list and equals() method is used to compare keys. That comparison to find the correct key with in a linked-list is a linear operation so in a worst case scenario the complexity becomes O(n).To address this issue in Java 8 hash elements use balanced trees instead of linked lists after a certain threshold is reached. Which means HashMap starts with storing Entry objects in linked list but after the number of items in a hash becomes larger than a certain threshold, the hash will change from using a linked list to a balanced tree, this will improve the worst case performance from O(n) to O(log n).

（4）HashMap和Concurrent HashMap区别， Concurrent HashMap 线程安全吗， Concurrent HashMap如何保证 线程安全？

* HashMap不是线程安全的，ConcurrentHashMap实现成安全的，通过ReentrantLock保证线程安全性。通过分散热点的思想将一个热点拆分为多个小热点，这样可以完成使得可以并行的对热点进行访问。ReentrantLock本身采用的是Unsafe的CAS保证线程的安全性的。

（5）HashMap和HashTable 区别，HashTable线程安全吗？

* HashMap不是线程安全的，HashTable是线程安全的。
* HashMap key and value all can be null but in HashTable neither the key and the value can be null。
* HashMap 有一个实现类LinkedHashMap可以有序访问元素 而HashTable则没有对应的实现。

（6）进程间通信有哪几种方式？

* 管道（Pipe）：其中无名管道可以在具有亲属关系的管道间进行通信，有名管道克服了管道名字的问题，可以在无亲属关系的管道间进行通信。
* 信号（signal）：相对复杂的通信方式，信号可以用于通知接受进程有某种事件发生。
* 消息队列（Message Queue）：消息队列是消息的链接表，有写权限的进程可以向消息队列中推入消息，有读取权限的进程可以向消息队列中读出消息。消息队列克服了信号承载信息量少，管道只能承载无格式字节流，以及缓冲区大小受限的等缺点。
* 共享内存 ：建立一块内存区域，不同的进程都可以访问，这个是最快的IPC（interprocess communication）。是针对其它线程运行效率低下而设计的。往往与其它通信机制联合使用，比如信号量，来达到访问进程的互斥以及同步。
* 信号量（semaphores）：用于进程间，或者同一进程不同线程间的同步。
* 套接字（Socket）：用于不同机器的进程间的交互。

（7）JVM分为哪些区，每一个区干吗的？

* 类加载子系统 ： 加载类用的。
* java 堆 ： 对象的存储点。分为新生代，老年代。
* java 栈 ： 局部变量存储点。
* Native 栈 ： 存放本地方法的储存点。
* PC寄存器 ： 存储当前线程指向的方法，如果是native方法，则为指向null。
* 方法区（元数据区）：存储方法的相关信息，静态变量等。
* 直接内存 ：访问速度很快。NIO访问直接内存，经常读写的可以放入到直接内存中。
* GC处理器
* 执行引擎

（8）JVM如何GC，新生代，老年代，持久代，都存储哪些东西？


（9）GC用的引用可达性分析算法中，哪些对象可作为GC Roots对象？

（10）快速排序，过程，复杂度？

（11）什么是二叉平衡树，如何插入节点，删除节点，说出关键步骤。

（12）TCP如何保证可靠传输？三次握手过程？

（13）TCP和UDP区别？

（14）滑动窗口算法？

（15）Linux下如何进行进程调度的？

（16）Linux下你常用的命令有哪些？

（17）操作系统什么情况下会死锁？

（18）常用的hash算法有哪些？

（19）什么是一致性哈希？

（20）如何理解分布式锁？

（21）数据库中的范式有哪些？

（22）数据库中的索引的结构？什么情况下适合建索引？

（23）Java中的NIO，BIO，AIO分别是什么？

（24）用什么工具调试程序？JConsole，用过吗？

（25）现在JVM中有一个线程挂起了，如何用工具查出原因？

（26）线程同步与阻塞的关系？同步一定阻塞吗？阻塞一定同步吗？

（27）同步和异步有什么区别？

（28）线程池用过吗？

（29）如何创建单例模式？说了双重检查，他说不是线程安全的。如何高效的创建一个线程安全的单例？

（30）concurrent包下面，都用过什么？

（31）常用的数据库有哪些？redis用过吗？

（32）了解hadoop吗？说说hadoop的组件有哪些？hdfs，hive,hbase,zookeeper。说下mapreduce编程模型。

（33）你知道的开源协议有哪些？

（34）你知道的开源软件有哪些？

（35）你最近在看的书有哪些？

（36）你有什么问题要问我吗？

（37）了解哪些设计模式？说说都用过哪些设计模式

（38）如何判断一个单链表是否有环？

（39）操作系统如何进行分页调度？

（40）匿名内部类是什么？如何访问在其外面定义的变量？
