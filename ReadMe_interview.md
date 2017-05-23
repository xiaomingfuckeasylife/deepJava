### 自我介绍。
  
*  ...

### JVM如何加载一个类的过程，双亲委派模型中有哪些方法？

* 加载（加载class文件） 连接（验证（验证class对象的有效性），准备（分配内存以及赋予初始化值），解析（将符号引用转化为指针引用）） 初始化（初始化静态函数以及成员） 方法有load方法

### HashMap如何实现的？

* HashMap works on the principal of hashing.
* Map.Entry interface - This interface gives a map entry (key-value pair). HashMap in Java stores both key and value object, in bucket, as Entry object which implements this nested interface Map.Entry.
* hashCode() -HashMap provides put(key, value) for storing and get(key) method forretrieving Values from HashMap. When put() method is used to store (Key, Value) pair, HashMap implementation calls hashcode on Key object to calculate a hash that is used to find a bucket where Entry object will be stored. When get() method is used to retrieve value, again key object is used to calculate a hash which is used then to find a bucket where that particular key is stored.
* equals() - equals() method is used to compare objects for equality. In case of HashMap key object is used for comparison, also using equals() method Map knows how to handle hashing collision (hashing collision means more than one key having the same hash value, thus assigned to the same bucket. In that case objects are stored in a linked list.

![hashMap](https://qph.ec.quoracdn.net/main-qimg-8d490b416731e970856e0b051d904441-p)

* Though HashMap implementation provides constant time performance O(1) for get() and put() method but that is in the ideal case when the Hash function distributes the objects evenly among the buckets.But the performance may worsen in the case hashCode() used is not proper and there are lots of hash collisions. As we know now that in case of hash collision entry objects are stored as a node in a linked-list and equals() method is used to compare keys. That comparison to find the correct key with in a linked-list is a linear operation so in a worst case scenario the complexity becomes O(n).To address this issue in Java 8 hash elements use balanced trees instead of linked lists after a certain threshold is reached. Which means HashMap starts with storing Entry objects in linked list but after the number of items in a hash becomes larger than a certain threshold, the hash will change from using a linked list to a balanced tree, this will improve the worst case performance from O(n) to O(log n).

### HashMap和Concurrent HashMap区别， Concurrent HashMap 线程安全吗， Concurrent HashMap如何保证 线程安全？

* HashMap不是线程安全的，ConcurrentHashMap实现成安全的，通过ReentrantLock保证线程安全性。通过分散热点的思想将一个热点拆分为多个小热点，这样可以完成使得可以并行的对热点进行访问。ReentrantLock本身采用的是Unsafe的CAS保证线程的安全性的。

### HashMap和HashTable 区别，HashTable线程安全吗？

* HashMap不是线程安全的，HashTable是线程安全的。
* HashMap key and value all can be null but in HashTable neither the key and the value can be null。
* HashMap 有一个实现类LinkedHashMap可以有序访问元素 而HashTable则没有对应的实现。

### 进程间通信有哪几种方式？

* 管道（Pipe）：其中无名管道可以在具有亲属关系的管道间进行通信，有名管道克服了管道名字的问题，可以在无亲属关系的管道间进行通信。
* 信号（signal）：相对复杂的通信方式，信号可以用于通知接受进程有某种事件发生。
* 消息队列（Message Queue）：消息队列是消息的链接表，有写权限的进程可以向消息队列中推入消息，有读取权限的进程可以向消息队列中读出消息。消息队列克服了信号承载信息量少，管道只能承载无格式字节流，以及缓冲区大小受限的等缺点。
* 共享内存 ：建立一块内存区域，不同的进程都可以访问，这个是最快的IPC（interprocess communication）。是针对其它线程运行效率低下而设计的。往往与其它通信机制联合使用，比如信号量，来达到访问进程的互斥以及同步。
* 信号量（semaphores）：用于进程间，或者同一进程不同线程间的同步。
* 套接字（Socket）：用于不同机器的进程间的交互。

### JVM分为哪些区，每一个区干吗的？

* 类加载子系统 ： 加载类用的。
* java 堆 ： 对象的存储点。分为新生代，老年代。
* java 栈 ： 局部变量存储点。
* Native 栈 ： 存放本地方法的储存点。
* PC寄存器 ： 存储当前线程指向的方法，如果是native方法，则为指向null。
* 方法区（元数据区）：存储方法的相关信息，静态变量等。
* 直接内存 ：访问速度很快。NIO访问直接内存，经常读写的可以放入到直接内存中。
* GC处理器
* 执行引擎

### JVM如何GC，新生代，老年代，持久代，都存储哪些东西？

* GC的过程，首先根据GC的次数，初始被new出来的对象都放入到新生代中，然后进过一定次数的GC后，仍然存活的会转入到持久代，再进行一定次数的GC后，如果这些对象仍然存在则会被转移到老年代中。

### GC用的引用可达性分析算法中，哪些对象可作为GC Roots对象？

#### 首先需要了解的一点是：GC会回收的不适GC Roots对象，而是那些没有被GC root引用到的对象。GC roots有以下多种情况：
* Class ： 由类加载器加载的对象，这些类是不能被回收的，他们以静态字段的方式保存持有其它对象。我们需要注意的一点就是，通过用户自定义的类加载器加载的类，除非相应的Class实例以其他的某种方式称为roots，否则他们不是roots。
* Threads 活着的线程
* Stack local ： 栈桢局部变量表中的local变量
* JNI local ： 本地的局部变量
* JNI global ： 本地全局变量
* Monitor Used ： 用于同步的监控对象
* Held By JVM : 被JVM保留使用的对象

### 快速排序，过程，复杂度？

* 快速排序的过程是通过分治的思想来解决问题。所谓分，就是将我们的数组分为两个部分分别为大于我们的特定值的一部分以及小于特定值的一部分。之后进行递归继续延用上述思想，我们就可以将一个数组完全分割完成。 复杂度为O(nlogn) 最坏的情况为O(n^2)

（11）什么是二叉平衡树，如何插入节点，删除节点，说出关键步骤。

（12）TCP如何保证可靠传输？三次握手过程？

* 1. client => server 2. server => client 3. client => server 

（13）TCP和UDP区别？



（14）滑动窗口算法？

（15）Linux下如何进行进程调度的？

### Linux下你常用的命令有哪些？

（17）操作系统什么情况下会死锁？

### 常用的hash算法有哪些？

### 什么是一致性哈希？

### 如何理解分布式锁？

（21）数据库中的范式有哪些？

### 数据库中的索引的结构？什么情况下适合建索引？

* Btree ,  小字段，选择性高，where条件经常查询的。

（23）Java中的NIO，BIO，AIO分别是什么？

（24）用什么工具调试程序？JConsole，用过吗？

### 现在JVM中有一个线程挂起了，如何用工具查出原因？

* Jconsole,jps,jmap

### 线程同步与阻塞的关系？同步一定阻塞吗？阻塞一定同步吗？

### 同步和异步有什么区别？

### 线程池用过吗？

（29）如何创建单例模式？说了双重检查，他说不是线程安全的。如何高效的创建一个线程安全的单例？

### concurrent包下面，都用过什么？

* 

（31）常用的数据库有哪些？redis用过吗？

（32）了解hadoop吗？说说hadoop的组件有哪些？hdfs，hive,hbase,zookeeper。说下mapreduce编程模型。

（33）你知道的开源协议有哪些？

（34）你知道的开源软件有哪些？

（35）你最近在看的书有哪些？

（36）你有什么问题要问我吗？

（37）了解哪些设计模式？说说都用过哪些设计模式

### 如何判断一个单链表是否有环？

*  进行遍历看是否循环重复

（39）操作系统如何进行分页调度？

（40）匿名内部类是什么？如何访问在其外面定义的变量？
    
（41）Java Bean是线程安全的吗？

（42）i++是线程安全的吗
    
    
    
### 值传递和引用传递

```java
Dog aDog = new Dog("Max");
  foo(aDog);
  aDog.getName().equals("Max"); // true

  public void foo(Dog d) {
      d.getName().equals("Max"); // true 
      d = new Dog("Fifi");
      d.getName().equals("Fifi"); // true
  }
```

### JDBC用过吗？ 原生的那种。 都用过那些方法。

### transaction的理解 

### AOP的实现方式

### spring常用的一些Annotation

算法
### 什么是深度优先搜索，广度优先搜索

### 树的遍历

数据库
### 子查询的优缺点


## java基础   

Arrays.sort实现原理和Collection实现原理   
foreach和while的区别(编译之后)   
线程池的种类，区别和使用场景   
分析线程池的实现原理和线程的调度过程   
线程池如何调优   
线程池的最大线程数目根据什么确定   
动态代理的几种方式   
HashMap的并发问题   
了解LinkedHashMap的应用吗  
反射的原理，反射创建类实例的三种方式是什么？   
cloneable接口实现原理，浅拷贝or深拷贝   
Java NIO使用   
hashtable和hashmap的区别及实现原理，hashmap会问到数组索引，hash碰撞怎么解决   
arraylist和linkedlist区别及实现原理   
反射中，Class.forName和ClassLoader区别   
String，Stringbuffer，StringBuilder的区别？   
有没有可能2个不相等的对象有相同的hashcode   
简述NIO的最佳实践，比如netty，mina   
TreeMap的实现原理   

### JVM相关   

类的实例化顺序，比如父类静态数据，构造函数，字段，子类静态数据，构造函数，字段，他们的执行顺序   
JVM内存分代   
Java 8的内存分代改进   
JVM垃圾回收机制，何时触发MinorGC等操作   
jvm中一次完整的GC流程（从ygc到fgc）是怎样的，重点讲讲对象如何晋升到老年代，几种主要的jvm参数等   
你知道哪几种垃圾收集器，各自的优缺点，重点讲下cms，g1   
新生代和老生代的内存回收策略   
Eden和Survivor的比例分配等   
深入分析了Classloader，双亲委派机制   
JVM的编译优化   
对Java内存模型的理解，以及其在并发中的应用   
指令重排序，内存栅栏等   
OOM错误，stackoverflow错误，permgen space错误   
JVM常用参数   
tomcat结构，类加载器流程   
volatile的语义，它修饰的变量一定线程安全吗    
g1和cms区别,吞吐量优先和响应优先的垃圾收集器选择    
说一说你对环境变量classpath的理解？如果一个类不在classpath下，为什么会抛出ClassNotFoundException异常，如果在不改变这个类路径的前期下，怎样才能正确加载这个类？   
说一下强引用、软引用、弱引用、虚引用以及他们之间和gc的关系      

### JUC/并发相关   

ThreadLocal用过么，原理是什么，用的时候要注意什么   
Synchronized和Lock的区别   
synchronized 的原理，什么是自旋锁，偏向锁，轻量级锁，什么叫可重入锁，什么叫公平锁和非公平锁   
concurrenthashmap具体实现及其原理，jdk8下的改版   
用过哪些原子类，他们的参数以及原理是什么   
cas是什么，他会产生什么问题（ABA问题的解决，如加入修改次数、版本号）   
如果让你实现一个并发安全的链表，你会怎么做   
简述ConcurrentLinkedQueue和LinkedBlockingQueue的用处和不同之处   
简述AQS的实现原理    
countdowlatch和cyclicbarrier的用法，以及相互之间的差别?   
concurrent包中使用过哪些类？分别说说使用在什么场景？为什么要使用？  
LockSupport工具   
Condition接口及其实现原理   
Fork/Join框架的理解   
jdk8的parallelStream的理解   
分段锁的原理,锁力度减小的思考    

### Spring   

Spring AOP与IOC的实现原理   
Spring的beanFactory和factoryBean的区别   
为什么CGlib方式可以对接口实现代理？   
RMI与代理模式   
Spring的事务隔离级别，实现原理   
对Spring的理解，非单例注入的原理？它的生命周期？循环注入的原理，aop的实现原理，说说aop中的几个术语，它们是怎么相互工作的？  
Mybatis的底层实现原理      
MVC框架原理，他们都是怎么做url路由的   
spring boot特性，优势，适用场景等   
quartz和timer对比   
spring的controller是单例还是多例，怎么保证并发的安全   

### 分布式相关    

Dubbo的底层实现原理和机制   
描述一个服务从发布到被消费的详细过程   
分布式系统怎么做服务治理   
接口的幂等性的概念   
消息中间件如何解决消息丢失问题    
Dubbo的服务请求失败怎么处理   
重连机制会不会造成错误   
对分布式事务的理解   
如何实现负载均衡，有哪些算法可以实现？  
Zookeeper的用途，选举的原理是什么？      
数据的垂直拆分水平拆分。  
zookeeper原理和适用场景   
zookeeper watch机制   
redis/zk节点宕机如何处理    
分布式集群下如何做到唯一序列号   
如何做一个分布式锁   
用过哪些MQ，怎么用的，和其他mq比较有什么优缺点，MQ的连接是线程安全的吗   
MQ系统的数据如何保证不丢失   
列举出你能想到的数据库分库分表策略；分库分表后，如何解决全表查询的问题。   

### 算法&数据结构&设计模式   

海量url去重类问题（布隆过滤器）  
数组和链表数据结构描述，各自的时间复杂度   
二叉树遍历   
快速排序   
BTree相关的操作    
在工作中遇到过哪些设计模式，是如何应用的   
hash算法的有哪几种，优缺点，使用场景   
什么是一致性hash   
paxos算法   
在装饰器模式和代理模式之间，你如何抉择，请结合自身实际情况聊聊   
代码重构的步骤和原因，如果理解重构到模式？   

### 数据库   

MySQL InnoDB存储的文件结构   
索引树是如何维护的？   
数据库自增主键可能的问题  
MySQL的几种优化   
mysql索引为什么使用B+树   
数据库锁表的相关处理   
索引失效场景    
高并发下如何做到安全的修改同一行数据，乐观锁和悲观锁是什么，INNODB的行级锁有哪2种，解释其含义     
数据库会死锁吗，举一个死锁的例子，mysql怎么解决死锁     

### Redis&缓存相关   

Redis的并发竞争问题如何解决了解Redis事务的CAS操作吗   
缓存机器增删如何对系统影响最小，一致性哈希的实现   
Redis持久化的几种方式，优缺点是什么，怎么实现的   
Redis的缓存失效策略    
缓存穿透的解决办法   
redis集群，高可用，原理     
mySQL里有2000w数据，redis中只存20w的数据，如何保证redis中的数据都是热点数据   
用Redis和任意语言实现一段恶意登录保护的代码，限制1小时内每用户Id最多只能登录5次   
redis的数据淘汰策略   

### 网络相关   

http1.0和http1.1有什么区别   
TCP/IP协议   
TCP三次握手和四次挥手的流程，为什么断开连接要4次,如果握手只有两次，会出现什么   
TIME_WAIT和CLOSE_WAIT的区别   
说说你知道的几种HTTP响应码   
当你用浏览器打开一个链接的时候，计算机做了哪些工作步骤   
TCP/IP如何保证可靠性，数据包有哪些数据组成    
长连接与短连接     
Http请求get和post的区别以及数据包格式   
简述tcp建立连接3次握手，和断开连接4次握手的过程；关闭连接时，出现TIMEWAIT过多是由什么原因引起，是出现在主动断开方还是被动断开方。    

### 其他  

maven解决依赖冲突,快照版和发行版的区别   
Linux下IO模型有几种，各自的含义是什么  
实际场景问题，海量登录日志如何排序和处理SQL操作，主要是索引和聚合函数的应用   
实际场景问题解决，典型的TOP K问题   
线上bug处理流程   
如何从线上日志发现问题    
linux利用哪些命令，查找哪里出了问题（例如io密集任务，cpu过度）   
场景问题，有一个第三方接口，有很多个线程去调用获取数据，现在规定每秒钟最多有10个线程同时调用它，如何做到。    
用三个线程按顺序循环打印abc三个字母，比如abcabcabc。   
常见的缓存策略有哪些，你们项目中用到了什么缓存系统，如何设计的   
设计一个秒杀系统，30分钟没付款就自动关闭交易（并发会很高）   
请列出你所了解的性能测试工具   
后台系统怎么防止请求重复提交？   
有多个相同的接口，我想客户端同时请求，然后只需要在第一个请求返回结果的时候返回给客户端     

