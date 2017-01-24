
# 初探JAVA虚拟机
## 基础
* java和其他的脚本语言相比，有着烦锁的语法，并且动态也很弱，因为这个java引入了一个新的特性invokeDynamic 这种方式可以稍微增加一些java的动态性问题。因为这些问题所以出现了类似Scala（高并发语言），Jython，Groovy（脚本语言）等依托于Java虚拟机的编程语言。

* 首先什么是虚拟机呢？虚拟机就是一台虚拟的计算机，用于执行计算机指令。虚拟机分为系统虚拟机，包括VisualBox , Vmware 等等，以及程序虚拟机，比如JVM。

* 为什么JVM可以跨平台。首先比如C语言，在windows系统上写的程序，依赖于windows系统支持。那么将同样的c代码放入到linux系统中，由于缺乏某些windows系统的包等等，则必然运行不了。但是同样的java字节码文件。由于它们运行在jvm中，不需要任何系统的支持。所以只要我们在对应的系统中安装好对应的JVM则，同样的代码一定都可以在不同的系统完成同样的效果。

* java的基础类型包括，byte(8位) short（16位） char（16位） int（32位） long（64位） float（32位浮点数） double（64位浮点数） boolean（8位／32位）. 其中 byte short int long 都是整型，char是无符号整型。 1byte ＝ 8 bit  . 引用类型分为类或者接口引用，泛型引用，数组引用。

* Java和JVM的区别？首先java是一门语言，通过编译可以生成java字节码文件。而JVM则是可以运行这种文件的运行环境。JVM是一个性能优异的，商用级别的软件运行和开发平台。

* jvm的主要内容 包括虚拟机的内部结构，虚拟机执行的字节码类型。class文件的结构。类的装载连接和初始化。

* 整数在计算机中一般都是实用补码表示的，jvm也是。用补码表示的好处有亮点（1）可以解决0的正负问题，因为补码情况下不论0是正数还是负数都是一样的。（2）可以加减运算时候需要判断正负的问题。实用补码就直接进行加运算就行了。

# 认识虚拟机的基本结构
## JVM 的架构
![JVM架构](https://i.imgsafe.org/1c808eb6fe.png) 

* `类加载子系统`负责从文件系统或者网络中加载Class信息。加载的类信息存放于一块称为`方法区`的内存空间。除了类的信息外，方法区中可能还存放运行时常量池信息，包括字符串字面量和数字常量。

* `Java堆`在虚拟机启动的时候建立，它是Java程序最主要的内存工作区域。几乎所有的java对象实例都是放在这个区域，堆空间对所有线程是共享的。

*  Java的NIO库可以使用`直接内存`，直接内存是java堆外的直接向系统申请的内存。通常直接内存的访问速度大于java堆，因此出于性能考虑，经常读写的，放到直接内存中由于直接内存存在于java堆外，所以Xmx设置的堆内存大小不能控制直接内存大小。但是系统内存的大小是有限的，所以他依然受限于系统能给出的最大内存。

* `垃圾回收系统`是JVM的重要组成成分。它可以对`方法区`，`java堆`，以及`直接内存`的绝大部份内容进行垃圾回收，其中`java堆`是回收重点。jvm的垃圾回收都是隐式的。不像c或者c++它们有free 和 delete方法可以显示的回收内存。

* 每一个JVM线程都有自己的`Java栈`。一个线程的Java栈在线程创建的时候被创建。Java栈中保存着局部变量，方法参数，同时和java方法的调用以及返回密切相关。

* `本地方法栈`和Java栈类似，只不过Java栈管理着java方法的调用，而本地方法栈管理着本地方法的调用，JVM是允许Java程序调用本地方法的。通常本地方法都是由c实现的。

* `PC(program counter) 寄存器`，每个线程都有自己的寄存器空间。在任意时刻java线程总是在执行一个方法，这个方法被称为当前方法。如果当前方法不是本地方法，PC寄存器就回指向当前正在被执行的命令。如果是，则PC寄存器的值就是undefined。

* `执行引擎`是Java虚拟机的最核心组件之一，他负责执行虚拟机的字节码，现代虚拟机为了提高执行效率，会使用即时编译技术（JIT）将方法编译成机器码然后再执行。

## Java虚拟机的参数

```bash
java [-options] class [args...]
```
其中 -option是指在这个地方可以设置jvm参数 后面的class是我们带有main的字节码文件，args是这个main函数的参数。例如：
```java
public class Main{
  public static void main(String args[]){
    for(int i=0;i<args.length;i++){
      System.out.println("参数 " + (i+1) + " : " + args[i]);
    }
    System.out.println("Xmx" + Runtime.getRuntime().maxMemory()/1000/1000 +"M");
  }
}
```
```bash
$>java -Xmx32 cn.Main a 
参数 1 ：a
Xmx32M
```
eclipse中jvm参数的设置在run as->run configurations->Arguments中进行配置即可
提醒大家可能很久没有用过java 以及javac命令了，如果大家在用java命令的时候找不到`Error: Could not find or load main class`  出现这种错误首先可以看下classpath有没有将当前目录设置到里面。linux中应该用引号区隔，windows中用分号区隔。如果这也做了还是有问题。请看你的当前位置在哪里，跳出到包所在的目录然后执行命令。因为如果你在包里面的目录，这个时候你执行命令，java命令还是找不到你的class文件的。

## 辨清Java堆
根据回收机制实现的不同java堆也有所不同，常见的java堆可以分为新生代和老年代两个部分，其中新生代用于存放刚生成不久的对象，老年代用于存放年龄较大的对象，其中新生代还可以分为eden区,s0区以及s1区，其中s0和s1大小一样，也被称为from和to区域。大多数情况下对象出世的时候首先被放入eden区域，之后没经过一次新生垃圾回收，所有存活下来的对象年龄就涨了一岁，这个时候就从eden区域转化到，s0或者s1区域。经过一定次数的回收循环，当对象到达一定的年龄以后就会转到老年区域（tenured）。下面的一个实例用于展示方法区，java堆，java栈的区别，
![堆明细图](https://i.imgsafe.org/1c89869282.png) 

```java
public class SimpleHeap{
  private int id;
  public SimpleHead(int id){
    this.id = id;
  }
  public void show(){
    System.out.println("My Id is " + id);
  }
  
  public static void main(String[] args){
    SimpleHeap s1 = new SimpleHeap(1);
    SimpleHeap s2 = new SimpleHeap(2);
    s1.show();
    s2.show();
  }
}
```
![内存图](https://i.imgsafe.org/1c7d7d4958.png) 

## Java栈
java栈和程序线程的执行有密切的关系。线程的执行基本行为都是方法调用，每次函数调用的数据都是通过java栈传递的。java栈和数据结构中的栈是类似的，也只有两个行为，出栈和入栈，java栈中保存的主要内容是栈桢，每一次函数的调用，都会有一个栈桢被压入栈中，每一次函数调用的结束都会有栈桢被压出栈。函数的嵌套调用中，当前正在执行的函数成为栈顶。函数返回有两种形式分别为正常通过return的返回或者是函数抛出异常。不论是其中那种方式，都会有栈桢被压出。一个栈桢通常包含三个部分，分别是局部变量表，操作数栈，桢数据区。如图例子：
![栈详情图](https://i.imgsafe.org/1c78391fbc.png) 

当请求的栈的深度大于最大的栈深度的时候，会报出stackoverflow的错误。java可以通过-Xss堆最大的栈空间大小进行设置。
```java
public class Main {
	private static int depth;
	
	private static void doStack() {
		depth++;
		doStack();
	}

	public static void main(String[] args) {
		try {
			doStack();
		} catch (Throwable ex) {
			System.out.println("the bigest depth is " + depth);
		}
	}
}
```
```bash
$> java -Xss200k Main 
the bigest depth is 1359
$> java -Xss400k Main
the bigest depth is 6109
```
### `局部变量表` 局部变量表只有在当前函数调用中有效，当前函数调用结束后，随着函数的销毁，局部变量表随之销毁。由于局部变量表在栈桢，所以如果局部变量变量和参数较多，会使得局部变量表膨胀，从而导致栈桢占的内存变大，从而栈能够嵌套的函数变少。

```java
public class Main {
	private static int depth;
	
	private static void doStack(int a ,int b ,int c) {
		int d = 10;int e = 20 ;int  f = 30 ; int g = 50;
		depth++;
		doStack();
	}

	public static void main(String[] args) {
		try {
			doStack(1,1,1);
		} catch (Throwable ex) {
			System.out.println("the bigest depth is " + depth);
		}
	}
}
```
```bash
$> java -Xss200k Main 
the bigest depth is 799
```
局部变量表的槽位是可以充用的，如果一个局部变量过了作用域（出了括号）那么再之后，如果有新的局部变量，就很可能复用过期的局部变量的槽位，从而达到节省资源的目的。局部变量表中的变量也是垃圾回收的重要的根结点，只要被局部变量表直接或者间接联系的对象都是不会被回收的。
```java
public class LocalVarGC{
	public void localVar1(){
  		Byte[] bytes = new Byte[6 * 1024 * 1024];
		System.gc();
  	}
	public void localVar2(){
		Byte[] bytes = new Byte[6 * 1024 * 1024];
		bytes = null;
		System.gc();
	}
	public void localVar3(){
		{
			Byte[] bytes = new Byte[6 * 1024 * 1024];
		}
		System.gc();
	}
	
	public void localVar4(){
		{
			Byte[] bytes = new Byte[6 * 1024 * 1024];
		}
		int a = 10;
		System.gc();
	}
	public void localVar5(){
		localVar1();
		System.gc();
	}
	public static void main(String[] args){
		LocalVarGC gc = new LocalVarGC();
		gc.localVar1(); // [Full GC 7506K->6519K(83008K), 0.0201924 secs]
		gc.localVar2(); // [Full GC 7506K->375K(83008K), 0.0146760 secs]
		gc.localVar3(); // [Full GC 7506K->6519K(83008K), 0.0207434 secs]
		gc.localVar4(); // [Full GC 13345K->369K(83008K), 0.0136461 secs]
		gc.localVar5(); // [Full GC 7506K->6519K(83008K), 0.0214124 secs] [Full GC 6860K->375K(83008K), 0.0135594 secs]

	}
}

```
通过设置jvm参数 -XX:+PrintGC 可以打印出方法调用前后堆的大小。

localVar1 在方法内部调用gc回收的时候，这个时候局部变量还在局部变量表中所以并不能回收垃圾。但是我们可以对比看localVar5当推出方法1后，对内存马上就叫少了差不多6兆。localVar2中将局部变量复制null就是销毁在局部变量表中的局部变量，所以这个局部变量对应的内存是可以回收的了。localVar3在括号中的局部变量虽然在局部变量外没有任何效果了，但是这个时候这个局部变量仍然在局部变量表中，所以不能销毁，但是对比4可以看出在括号外，又重新复了一个值，这个时候这个新的局部变量复用了无用的局部变量空间，这个时候括号内的局部变量已经从局部变量表中消除了，这个时候，那部分空间可以回收了。

### `操作数桢` 主要用于保存计算结果，同时作为计算过程中的临时储存空间。

### `桢数据区` 主要用于支持常量池解析，正常方法返回，异常处理，存储着常量池的指针，异常处理表等等。

### `栈上分配` 对于那些私有化（其他线程无法访问到的对象）分配到栈上，而不分配到堆上，这样的好处，出栈的时候这些对象就会自行销毁，而不用gc，提高效率。这种技术有一个基础就是`逃逸分析`，也就是分析对象是否能逃逸出方法体。例子：
```java
public class onStackTest{
	public static class User{
		public User(int id){
			this.id = id;
		}
		private int id;
	}
	
	public void alloc(){
		User u = new User(1);
	}
	
	public static void main(String[] args){
		for(int i=0;i<10000000;i++){
			alloc();
		}
		System.gc();
	}
}
```
我们可以知道一个User占有8个字节(引用4个字节＋int类型4个字节)，10 000 000万次，大概可以申请80兆内存在堆上。我们执行一下 使用以下参数
```bash
-server -Xmx10M -Xms10M -XX:+DoEscapeAnalysis -XX:+PrintGC -XX:-UseTLAB -XX:+EliminateAllocations
```	
-server : 因为必须在server模式下，才可以开启逃逸分析。-XX:DoEscapeAnalysis ： 启用逃逸分析。 -Xmx10M -Xms10M ：执行最大堆内存以及初始化堆内存。-XX:+PrintGC 打印gc日志。-XX:+ElimateAllocations：开启标量替换，允许将对象打散到栈上。这个时候id会被当作一个局部变量-XX:-UseTLAB：关闭了TLAN。
我们可以看到没有答应什么日志，`[Full GC 408K->386K(9856K), 0.0078271 secs]` 基本上没有堆内存的清理。 栈上分配，一般针对小对象，并且依赖于启用逃逸分析，以及开启标量替换。

## 识别方法区
和java堆一样，方法区（永久区）也是所有线程都能访问的区域，方法区域的大小决定了，jvm可以加载的类的数量。如果类太多，也会导致内存溢出。在jdk1.6,1.7中可以通过设置，-XX:PermSize -XX:MaxPermSize指定。如果系统使用了动态代理，那么可能会产生很多的类。就可能导致永久区内存溢出。

```java
	public static void main(String[] args) {
		Interceptor c = new Interceptor();
		for(int i=0;i<1000000;i++){
			CglibBean proxy = c.createProxyObject();
		}
	}
```
然后将jvm参数设置：默认的永久区内存大小为64兆
```bash
-XX:+PrintGCDetails -XX:PermSize=5M -XX:MaxPermSize=5M
```
然后执行代码后发现永久区内存溢出：
```java
Exception in thread "Reference Handler" java.lang.OutOfMemoryError: PermGen space
```
在JDK1.8中，彻底废除了方法区（永久区），取而代之的为元数据区，元数据区大小可以通过使用参数-XX:MaxMetaspaceSize进行设置。这是一块堆外的直接内存。与方法区不同，默认情况下，他会一直使用知道系统内存使用完毕为止。如果元数据区内存溢出同样会报错。`Metaspace`

# 常用虚拟机参数
## 跟踪调试参数

#### 1. 最简单的一个GC参数 -XX:+PrintGC 使用这个参数，是要程序调用了GC，则会打印
```java
[Full GC 1362K->375K(83008K), 0.0197301 secs]
```
这个表示执行了一次Full垃圾回收，堆内存使用从1362K到回收后的375k，总的堆内存为83M左右，这词gc使用的时间为0.019秒
#### 2. 如果觉得这个显示的不是那么清楚，可以使用 -XX:+PrintGCDetails 
```java
[Full GC (System) [CMS: 0K->375K(63872K), 0.0162591 secs] 1362K->375K(83008K), [CMS Perm : 4629K->4628K(21248K)], 0.0226283 secs] [Times: user=0.02 sys=0.00, real=0.02 secs] 
Heap
 par new generation   total 19136K, used 1021K [7f3000000, 7f44c0000, 7f44c0000)
  eden space 17024K,   6% used [7f3000000, 7f30ff658, 7f40a0000)
  from space 2112K,   0% used [7f40a0000, 7f40a0000, 7f42b0000)
  to   space 2112K,   0% used [7f42b0000, 7f42b0000, 7f44c0000)
 concurrent mark-sweep generation total 63872K, used 375K [7f44c0000, 7f8320000, 7fae00000)
 concurrent-mark-sweep perm gen total 21248K, used 4689K [7fae00000, 7fc2c0000, 800000000)
```
老年代使用了63872k内存的375k，新生代没有使用内存，gc后，总的使用内存从1362k到现在的375k，也就是新生代现在所有区域都没有使用内存。方法区使用了4628k内存，gc总共花费了0.02秒，用户态系统耗时0.02，系统态系统耗时0，实际耗时0.02最后的三个地址，分别为内存的下界，当前上界，上届。
#### 3. 使用-XX:+PrintHeapAtGC 可以打印GC前后的堆内存状况
```java
{Heap before GC invocations=0 (full 0):
 par new generation   total 19136K, used 1362K [7f3000000, 7f44c0000, 7f44c0000)
  eden space 17024K,   8% used [7f3000000, 7f3154a00, 7f40a0000)
  from space 2112K,   0% used [7f40a0000, 7f40a0000, 7f42b0000)
  to   space 2112K,   0% used [7f42b0000, 7f42b0000, 7f44c0000)
 concurrent mark-sweep generation total 63872K, used 0K [7f44c0000, 7f8320000, 7fae00000)
 concurrent-mark-sweep perm gen total 21248K, used 4629K [7fae00000, 7fc2c0000, 800000000)
Heap after GC invocations=1 (full 1):
 par new generation   total 19136K, used 0K [7f3000000, 7f44c0000, 7f44c0000)
  eden space 17024K,   0% used [7f3000000, 7f3000000, 7f40a0000)
  from space 2112K,   0% used [7f40a0000, 7f40a0000, 7f42b0000)
  to   space 2112K,   0% used [7f42b0000, 7f42b0000, 7f44c0000)
 concurrent mark-sweep generation total 63872K, used 375K [7f44c0000, 7f8320000, 7fae00000)
 concurrent-mark-sweep perm gen total 21248K, used 4628K [7fae00000, 7fc2c0000, 800000000)
}
```
#### 4.由于GC会引起程序停顿，因此我们可以通过-XX:+PrintGCApplicationConcurrentTime -XX:+PrintGCApplicationStoppedTime观察程序的执行时间，和停顿时间
```java
Application time: 0.5324587 seconds
Total time for which application threads were stopped: 0.0160692 seconds
Application time: 0.0262697 seconds
```
#### 5. 将GC产生的日志输入到日志文件。 -Xloggc:gc.log

#### 6. 使用-XX:+TraceClassLoading  -XX:+TraceClassUnloading 用于监测加载的类以及卸载的类
```java
[Opened /Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Classes/classes.jar]
[Loaded java.lang.Object from /Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Classes/classes.jar]
[Loaded java.io.Serializable from /Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Classes/classes.jar]
[Loaded java.lang.Comparable from /Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Classes/classes.jar]
[Loaded java.lang.CharSequence from /Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Classes/classes.jar]
[Loaded java.lang.String from /Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Classes/classes.jar]
......
```

#### 7. 使用-XX:+PrintVMOptions 打印JVM接受的显示参数。 -XX:+PrintCommandLineFlags 打印JVM接受的显示和隐式参数 :
```bash
(显示和隐式)
-XX:MaxNewSize=87244800 -XX:MaxTenuringThreshold=4 -XX:NewRatio=7 -XX:NewSize=21811200 -XX:OldPLABSize=16 -XX:OldSize=65433600 -XX:+PrintCommandLineFlags -XX:+PrintGCDetails -XX:+UseCompressedOops -XX:+UseConcMarkSweepGC -XX:+UseParNewGC 
```

#### 8. 最大堆初始堆的设置：-Xmx-Xms 一般来说虚拟机会维持在初始堆空间，但是如果堆空间不足，则会扩展堆空间内容，扩展的上线为最大堆空间 , 在工作中我们可以将初始和最大堆空间设置成一样的这样可以减少GC，提高性能。

#### 9. 使用参数-Xmn 可以设置新生代的内存大小，这个参数对GC行为有很大的影响，新生代的大小一般设置为整个堆空间的1/3到1/4左右。 -XX:SurvivorRatio用来设置新生代中eden和from/to的比例。
```java
public class NewSizeDemo{
	public static void main(String[] args){
		byte[] b = null;
		for(int i=0;i<10;i++){
			b = new byte[1  *  1024 * 1024];
		}
	}
}
```
使用不同的堆空间配置可以得到不同的gc效果。
`-Xmx20m -Xms20m -Xmn1m -XX:SurvivorRadio=2 -XX:+PrintGCDetails` 运行上述程序
```java
[GC [ParNew: 512K->245K(768K), 0.0070901 secs] 512K->245K(20224K), 0.0096377 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
[GC [ParNew: 757K->204K(768K), 0.0037639 secs] 757K->412K(20224K), 0.0038154 secs] [Times: user=0.01 sys=0.01, real=0.00 secs] 
Heap
 par new generation   total 768K, used 495K [7f9a00000, 7f9b00000, 7f9b00000)
  eden space 512K,  56% used [7f9a00000, 7f9a48c70, 7f9a80000)
  from space 256K,  79% used [7f9a80000, 7f9ab32c8, 7f9ac0000)
  to   space 256K,   0% used [7f9ac0000, 7f9ac0000, 7f9b00000)
 concurrent mark-sweep generation total 19456K, used 10448K [7f9b00000, 7fae00000, 7fae00000)
 concurrent-mark-sweep perm gen total 21248K, used 4690K [7fae00000, 7fc2c0000, 800000000)
```
我们看到申请的10兆内存基本上都在老年代上面因为新生代没有足够的空间用于储存1兆的内存，所以触发新生代gc。
下面我们进行另外一种配置`-Xmx20m -Xms20m -Xmn7m -XX:SurvivorRadio=2 -XX:+PrintGCDetails` :
```java
[GC [ParNew: 3340K->1414K(5376K), 0.0025629 secs] 3340K->1414K(18688K), 0.0059159 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 
[GC [ParNew: 4534K->1024K(5376K), 0.0037942 secs] 4534K->1408K(18688K), 0.0038327 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
[GC [ParNew: 4199K->1042K(5376K), 0.0020668 secs] 4584K->1427K(18688K), 0.0021145 secs] [Times: user=0.00 sys=0.01, real=0.00 secs] 
Heap
 par new generation   total 5376K, used 3254K [7f9a00000, 7fa100000, 7fa100000)
  eden space 3584K,  61% used [7f9a00000, 7f9c29168, 7f9d80000)
  from space 1792K,  58% used [7f9f40000, 7fa0449a8, 7fa100000)
  to   space 1792K,   0% used [7f9d80000, 7f9d80000, 7f9f40000)
 concurrent mark-sweep generation total 13312K, used 384K [7fa100000, 7fae00000, 7fae00000)
 concurrent-mark-sweep perm gen total 21248K, used 4690K [7fae00000, 7fc2c0000, 800000000)
```
老年代基本上区域没有被使用，新生代的内存空间完全足够使用。除此之外我们还可以通过设置`-XX:NewRatio`来对老年代和新生代的比例进行配置。我们在这里就不赘述了：
![新生代老年代](https://i.imgsafe.org/1c64787836.png) 

#### 10. 堆内存溢出的时候，我们可以使用-XX:HeapDumpOnOutofMemoryError -XX:HeapDumpPath=XX/a.dump 使用这两个参数如果发生堆内存溢出可以将整个堆信息导出到对应的文件中。
```java
[GC [ParNew: 1024K->128K(1152K), 0.0122162 secs] 1024K->367K(6464K), 0.0150368 secs] [Times: user=0.01 sys=0.01, real=0.02 secs] 
[GC [ParNew: 367K->117K(1152K), 0.0017592 secs][CMS: 345K->376K(5312K), 0.0167783 secs] 606K->376K(6464K), [CMS Perm : 4629K->4628K(21248K)], 0.0186347 secs] [Times: user=0.02 sys=0.01, real=0.02 secs] 
[Full GC [CMS: 376K->338K(6912K), 0.0094079 secs] 376K->338K(8064K), [CMS Perm : 4628K->4617K(21248K)], 0.0094780 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
java.lang.OutOfMemoryError: Java heap space
Dumping heap to /Users/clark/heap.log ...
Heap dump file created [1106333 bytes in 0.024 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at test.HeapOutofMemory.main(HeapOutofMemory.java:6)
Heap
 par new generation   total 1152K, used 80K [7fa600000, 7fa740000, 7fa740000)
  eden space 1024K,   7% used [7fa600000, 7fa614068, 7fa700000)
  from space 128K,   0% used [7fa700000, 7fa700000, 7fa720000)
  to   space 128K,   0% used [7fa720000, 7fa720000, 7fa740000)
 concurrent mark-sweep generation total 6912K, used 338K [7fa740000, 7fae00000, 7fae00000)
 concurrent-mark-sweep perm gen total 21248K, used 4680K [7fae00000, 7fc2c0000, 800000000)
```
除此之外，JVM还允许我们在发生指定错误的时候，执行一个脚本程序 : printstack.sh
`jstack -F %1 > /Users/clark/a.txt` `-XX:+PrintGCDetails -Xmx5m-Xms5m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/Users/clark/heap.dump "-XX:OnOutOfMemoryError=/Users/clark/printstack.sh %p"`

#### 11. 配置方法区（永久区）内存 可以通过 -XX:PermSize -XX:PermMaxSize 配置元数据区内存可以使用 -XX:MaxMetaspaceSize 指定元数据区的大小。

#### 12. 栈内存的配置可以使用 -XX:Xss 

#### 13. 直接内存的配置可以使用 ， -XX:MaxDirectoryMemorySize 如果不设置默认值为最大的堆内存。当直接内存的使用到达最大值的时候，这个时候就会触发垃圾回收机制，如果这个时候仍然不能释放足够的空间就会报出OutOfMemory的异常，一般来说直接内存的读写速度快于java堆，但是申请内存的速度小于java堆。

首先来测试堆内存和直接内存的访问：
```java
public class AccessDirectBuffer{
	
	public void directAccess(){
		long start = System.currentTimeMillis();
		ByteBuffer b= ByteBuffer.allocateDirect(500);
		
		for(int i=0;i<1000000;i++){
			
			for(int j=0;j<100;j++){
				b.putInt(j);
			}
			b.flip();
			for(int j=0;j<100;j++){
				b.getInt(j);
			}
			b.clear();
		}
		long end = System.currentTimeMillis();
		System.out.println("direct Access : " + (end - start));
	}
	
	public void accessBuffer(){
		long start = System.currentTimeMillis();
		ByteBuffer b= ByteBuffer.allocate(500);
		
		for(int i=0;i<1000000;i++){
			
			for(int j=0;j<100;j++){
				b.putInt(j);
			}
			b.flip();
			for(int j=0;j<100;j++){
				b.getInt(j);
			}
			b.clear();
		}
		long end = System.currentTimeMillis();
		System.out.println("buffer Access : " + (end - start));
	}
	
	public static void main(String[] args) {
		
		AccessDirectBuffer adb = new AccessDirectBuffer();
		adb.directAccess();
		adb.accessBuffer();
		
		adb.directAccess();
		adb.accessBuffer();
		
		
	}
	
}
```
结果
```java
direct Access : 177
buffer Access : 529
direct Access : 417
buffer Access : 511
```
再来测试直接内存和java堆的内存申请速度
```java
public class AccessDirectBuffer{
	
	public void directAccess(){
		long start = System.currentTimeMillis();
		for(int i=0;i<10000;i++){
			ByteBuffer b= ByteBuffer.allocateDirect(500);
		}
		
		long end = System.currentTimeMillis();
		System.out.println("direct Access : " + (end - start));
	}
	
	public void accessBuffer(){
		long start = System.currentTimeMillis();
		for(int i=0;i<10000;i++){
			ByteBuffer b= ByteBuffer.allocate(500);
		}
		long end = System.currentTimeMillis();
		System.out.println("buffer Access : " + (end - start));
	}
	
	public static void main(String[] args) {
		
		AccessDirectBuffer adb = new AccessDirectBuffer();
		adb.directAccess();
		adb.accessBuffer();
		
		adb.directAccess();
		adb.accessBuffer();
		
		
	}
	
}
```
结果
```java
direct Access : 44
buffer Access : 16
direct Access : 10
buffer Access : 4
```

#### 14. JVM的两种模式，client 模式和server模式 。 其中client模式相对server模式启动块，针对用户界面程序，运行时间不长，追求启动速度的。但是server模式，启动缓慢，但是稳定，执行速度快。这两种模式下的各种参数大小也有很大的不同。

## 垃圾回收概念与算法

### 讨论常用的垃圾回收算法

#### 1. 引用指数法
 * 实现：通过计算对象当前的引用数，如果为0则可以回收，如果不为0则不能回收。
 * 问题：1 无法处理循环的情况，比如，对象A引用对象B，然后对象B引用对象A，这种情况下，A和B都不会被回收。但是A，B已经没有其他的对象引用了，这个已经说明这两个，对象都是有问题的。都应该被回收。2 由于每个对象都有一个计数器跟着，这样每次对象引用的添加以及对象引用的销毁都会计数，这样堆系统的性能来说并不是很好。因此java并没有使用这种垃圾回收方式。
 
#### 2. 标记清除法

 * 实现：将垃圾回收分为两个阶段，标记阶段，清除阶段。标记从根结点（java栈的局部变量表）开始向上寻找对象找到的对象进行标记。在清除阶段，清除掉所有的没有被标记的对象。
 * 问题：容易产生空间碎片（也就是由于被清除的对象不是一整块一整块的，导致被清除的空间也是不连续的）
 
#### 3. 复制算法
 
 * 实现：将内存分为两份，一份使用，一份暂时不使用。创立对象后，将需要回收的对象移到未使用区域，同时清空第一个区域的内存即可。
 * 缺点：内存减半。
 * 扩展：jvm也使用复制算法。在新生代的 eden from to 三个区域中 from和to就是复制区域。存活的放入到它们的内存里面，然后清除eden区域。
 
#### 4 标记压缩法
 * 实现：基本思想和标记清除法一样，只不过多了一个步骤，将标记的内存压缩到内存的一边，然后以此为边界清楚不需要的内存。
 * 扩展：jvm的老年代的垃圾回收使用的就是这种方式，这种方式既避免了将内存减半，有避免了空间碎片。由于老年区的对象一般很稳定所以并不适合复制算法。
 
#### 5 分代算法
 * 实现：由于不同的算法适用于不同的场景，所以，通常情况下，将内存分为新生代和老年代，新生代的对象基本上百分之90的会被销毁所以适用于复制算法，而老年代的对象，基本上很少需要被回收，所以适合使用标记清除法。
 
#### 6 分区算法
 * 实现：由于GC会是程序停顿，所以，我们如果每次都对整个堆内存进行gc的话，可能是程序陷入较长时间的停顿，这个时候，如果我们将内存分为几个不同的块，这个时候，如果我们只是对个别几个小的区块进行gc，就会减少一次gc的时间。

### 判断可触及性
对象的状态可以分为三种：
 * 可触及的，从根结点开始，可以到达的对象。
 * 可复活的，对象的所有引用都被释放，但是对象仍然可能在finalize函数中复活。 
 * 不可触及的，对象的finalize函数被调用，并且没有复活。因为finalize只可能被调用一次。
 
#### 对象的复活
```java
public class CanRelieveObj{
	private static CanRelieveObj obj ; 
	@Override
	protected void finalize() throws Throwable {
		super.finalize();
		System.out.println("finallize recorvery");
		obj = this;
	}
	
	public static void main(String[] args) throws InterruptedException {
		obj = new CanRelieveObj();
		obj = null;
		System.gc();   // after gc . jvm will invoke finallize method .
		// recorvery needs time 
		Thread.sleep(1000);
		System.out.println("first gc");
		if(obj == null){
			System.out.println("obj is null");
		}else {
			System.out.println("obj is usable");
		}
		System.gc();
		Thread.sleep(1000);
		obj = null;
		System.out.println("second gc");
		if(obj == null){
			System.out.println("obj is null");
		}else {
			System.out.println("obj is usable");
		}
	}
}
```
```java
[Full GC (System) [CMS: 0K->376K(63872K), 0.0166137 secs] 1362K->376K(83008K), [CMS Perm : 4632K->4631K(21248K)], 0.0211313 secs] [Times: user=0.01 sys=0.01, real=0.03 secs] 
finallize recorvery
first gc
obj is usable
[Full GC (System) [CMS: 376K->369K(63872K), 0.0122512 secs] 1057K->369K(83008K), [CMS Perm : 4634K->4634K(21248K)], 0.0123162 secs] [Times: user=0.02 sys=0.00, real=0.01 secs] 
second gc
obj is null
Heap
 par new generation   total 19136K, used 681K [7f3000000, 7f44c0000, 7f44c0000)
  eden space 17024K,   4% used [7f3000000, 7f30aa510, 7f40a0000)
  from space 2112K,   0% used [7f40a0000, 7f40a0000, 7f42b0000)
  to   space 2112K,   0% used [7f42b0000, 7f42b0000, 7f44c0000)
 concurrent mark-sweep generation total 63872K, used 369K [7f44c0000, 7f8320000, 7fae00000)
 concurrent-mark-sweep perm gen total 21248K, used 4692K [7fae00000, 7fc2c0000, 800000000)
```
可以看到第一次gc，finallize复活了对象。由于finallize可能存在引用外泄的隐患，所以我们一般不通过finallize释放内存。而是通过try...catch()..finally。

#### 可触及性的强度之强引用
强引用的特点：
1. 可以直接访问目标对象
2. 强引用所指向的内存，任何时候都不会被内存释放，就算是爆出OOM也不会释放。
3. 强引用可能产生内存泄漏

#### 可触及性的强度之软引用
弱引用的特点：
1. 弱引用只有在内存不足的时候才会被回收。并且每一个引用软引用都可以附带一个引用队列，当这个对象的可达性状态发生改变的时候，软引用就会进入引用队列。通过这个引用队列可以跟踪对象的回收状态。
```java
public class SoftRef{
	
	private static ReferenceQueue<User> rq ;
	
	public static class User{
		public int id;
		public String name;
		public User(int id,String name){
			this.id = id;
			this.name = name;
		}
	}
	
	static class CheckSoftRefQueue extends Thread{
		@SuppressWarnings("unchecked")
		@Override
		public void run() {
			while(true){
				if(rq != null){
					MySoftReference<User> u;
					try {
						u = (MySoftReference<User>) rq.remove();
						if(u != null){
							System.out.println("User " + u.getId() + " is been removed");
						}
					} catch (InterruptedException e) {
						e.printStackTrace();
					} 
				}
			}
		}
	}
	
	static class MySoftReference<user> extends SoftReference<User>{
		private int id;
		public MySoftReference(User referent, ReferenceQueue<? super User> q) {
			super(referent, q);
			id = referent.id;
		}
		
		public int getId(){
			return id;
		}
	}
	
	public static void main(String[] args) throws InterruptedException {
		// listen the queue .
		CheckSoftRefQueue check = new SoftRef.CheckSoftRefQueue();
		check.setDaemon(true);
		check.start();
		
		User u = new User(1,"xiaoming");
		rq = new ReferenceQueue<User>();
		MySoftReference<User> sr = new MySoftReference<User>(u,rq);
		u = null;
		System.out.println("before gc:" + sr.get());
		System.gc();
		System.out.println("when there are enough space after gc :" + sr.get());
		byte[] b = new byte[ 9 * 940 * 1024];
		System.gc();
		System.out.println("when there are not enough space :" + sr.get());
		
		Thread.sleep(1000);
	}
}
```

以上的内容的JVM参数为`-Xmx10m-Xms10m` 
```java
before gc:test.SoftRef$User@68f99ff5
when there are enough space after gc :test.SoftRef$User@68f99ff5
when there are not enough space :null
User 1 is been removed
```

#### 可触及性的强度之弱引用
1. 特点发现即回收，和软引用一样，他也有一个注册队列。一旦弱引用被回收也会被注册到这个队列中。这个类通过WeakReference 来实现。 具体的实现过程。这里就省去了。基本上和软引用一样。
软引用，弱引用十分适合保存那些可有可无的缓存数据。如果那么做，当内存不足的时候，这些内存就会被回收。当内存充足的时候又可以在内存中存活相当长的一段时间。

#### 可触及性的强度之虚引用
1. 虚引用是最弱的引用。用虚引用获取强引用，基本上就不会成功。使用虚引用只有一个目的就是记录跟踪强引用。所以他只能和引用队列一起使用才行。具体的实现代码也喝第一个例子差不多。

### 垃圾回收的停顿现象：stop-the-world(STW)
当gc工作的时候，所有的线程全部停止工作。

## 垃圾回收器和内存分配

#### 串行回收器
是指使用单线程进行垃圾回收的回收器。根据作用的空间不同分为新生代串行回收器和老年代串行回收器。
* 新生代串行回收器：特点，独占式的（STW）以及使用单线程进行垃圾回收。由于它会停止所有线程的当前活动，所以在实时性要求很高的场景下基本上不能使用这种回收器，这种回收器，适合使用在单核的计算机中，对实时性要求不高的场景下。 可以使用 +XX:+UseSerialGC参数进行设置。在client模式下，默认的回收器为串行回收器。`[GC [DefNew: 1267K->192K(1856K), 0.0063437 secs] 1267K->372K(65344K), 0.0088598 secs] [Times: user=0.00 sys=0.00, real=0.01 secs]`

* 老年代串行回收器：使用标记压缩进行垃圾回收，老年代的垃圾回收一般会比新生代花的时间长很多，这个时候可能导致程序停顿很长时间。虽然如此但是由于它的稳定性，所以我们经常将它与其他的垃圾回收器一起使用。作为备用回收器。可以通过一下参数设置老年代串行回收器。-XX:+UseSerialGC (新生代老年代都使用串行回收器)，-XX:+UseParNewGC（新生代使用ParNew回收器，老年代使用串行回收器）,-XX:+UseParallelGC(新生代使用ParallelGC，老年代使用串行回收器)`[Full GC (System) [Tenured: 0K->372K(43712K), 0.0130356 secs] 2438K->372K(63360K), [Perm : 4628K->4628K(21248K)], 0.0200884 secs] [Times: user=0.01 sys=0.01, real=0.02 secs]`

#### 并行回收器
是指通过多个线程同时进行垃圾处理的回收器。对于并行能力强的计算机，可以有效缩短垃圾回收所需要的实际内容。注意虽然是并行并不是并发。下面的这些并行回收器都是STW的独断式的垃圾回收器，也就是说再用这些垃圾回收器进行垃圾处理的时候，所有的线程停止工作。

* 新生代ParNew回收器：只是简单的将串行回收器从单线程转为多线程仅此而已。所以他也是独占式的(STW)。可以通过-XX:+UseParNewGC(新生代使用ParNew回收器，老年代使用串行回收器) -XX:+UseConcMarkSweepGC:(新生代使用ParNew回收器，老年代使用CMS)使用ParNew回收器的时候，可以使用-XX:ParallelGCThreads对并行线程数量进行设定。当CPU数量大于8的时候，可以使用公式 3 + ((5*CPU_Count)/8) `[GC [ParNew: 1267K->192K(1856K), 0.0040987 secs] 1267K->395K(65344K), 0.0103077 secs] [Times: user=0.01 sys=0.01, real=0.01 secs]`

* 新生代Parallel回收器：这个回收器和ParNew很类似，可以通过，-XX:+UseParallelGC（新生代使用ParallelGC，老年代使用串行回收器），-XX:+UseParallelOldGC(新生代使用ParallelGC，老年代使用paralalOld回收器),只不过它对GC的吞吐量十分关注，可以通过-XX:MaxGCPauseMillis设置，设置最大垃圾收集停顿时间。它的值是一个大于0的值。一般是通过降低吞吐量达到要求的时间。 可以通过-XX:GCTimeRatio设置吞吐量大小，它是一个0到99的值，假设它的值为n，那么最大的一次gc的时间将不会超过 1/(n+1) . 除此之外，他还支持一种自适应的GC调节模式，可以使用-XX:UseAdaptiveSizePolicy,  这种模式下我们可以只需要设置最大堆，最大吞吐量，每次gc最大消耗时间，然后让JVM自行调优.`[GC [PSYoungGen: 1288K->240K(1792K)] 2312K->1428K(65280K), 0.0092242 secs] [Times: user=0.01 sys=0.01, real=0.01 secs]`

* 老年代ParallelOld回收器：Old是相对于新生代来讲的，他和新生代基本一样， 可以通过-XX:+UseParallelOldGC (新生代使用ParallelGC 老年代使用ParallelOldGC) `[Full GC (System) [PSYoungGen: 496K->0K(19136K)] [ParOldGen: 0K->372K(43712K)] 496K->372K(62848K) [PSPermGen: 4628K->4626K(21248K)], 0.0118866 secs] [Times: user=0.02 sys=0.00, real=0.01 secs]`

#### CMS回收器
这个垃圾回收器是Concurrent Marked Sweep 的缩写，使用的是标记清除算法。同时它又是一个多线程并行的回收器。
* CMS回收器包含以下步骤：初始标记，并发标记，预清理，重新标记，并发清除和并发重置。其中初始标记和重新标记是独占式的，其余都是和用户线程并发的。所以从总体上来说，CMS并不是独占式的垃圾回收机制。可以通过 -XX:+UseConcMarkSweepGC 可以通过-XX:+ConcGCThreads 或者 -XX:+ParallelCMSThreads 手工设定并行的线程数量。![CMS回收器](https://i.imgsafe.org/33aae83123.png)

* 因为CMS不是独占式的GC，所以他并不会等到内存用完的时候才进行GC，而是在内存进行到一定的使用率的时候就进行GC，预留部分内存在gc的时候程序使用。可以通过-XX:CMSInitiatingOccupancyFraction来设定。默认为68。也就是当使用率到达68%的时候回启用gc。由于算法为标记清除，标记清除最大的缺点就是空间碎片，为了解决空间碎片问题。我们可以使用-XX:+UseCMSCompactAtFullCollection (进行完垃圾回收后，进行碎片清除) 或者-XX:CMSFullGCsBeforeCompaction(多少次cms后进行一次碎片清除)`[Full GC (System) [CMS: 0K->376K(63872K), 0.0149420 secs] 11602K->376K(83008K), [CMS Perm : 4629K->4628K(21248K)], 0.0176247 secs] [Times: user=0.01 sys=0.00, real=0.02 secs]` 

* 有关class的回收，如果开启了-XX:+CMSClassUnloadingEnabled 那么在条件允许的情况下，还会进行一次FUll GC对Perm区的内容进行回收。

#### G1回收器
* G1:(garbage-first):特点，并行性（充分利用多核），并发性（不是STW的回收器），分代GC，空间整理，可预见性（预判垃圾回收）。G1的回收过程分为四个阶段：1. 新生代GC，2. 并发标记周期， 3.混合收集 4. 如果需要再进行一次Full GC 。 可以通过 -XX:+UseG1GC标记打开垃圾回收。通过一下三个参数对其进行设置。-XX:MaxGCPauseMills 以及 -XX:ParallelGCThreads -XX:InitiatingHeapOccupancyPercent . 我们可以看到G1回收器基本上集合了上述的那些回收器的所有的优点。

### 对象分配和回收的一些细节问题

#### 禁用System.gc()
1. 手动调用gc方法，会直接触发Full GC，同时对老年代和新生代进行垃圾回收。 可以通过参数 -XX:+DisableExplicitGC 来控制是否手工触发GC。
2. 默认情况下调用gc方法，不会产生并发的垃圾回收机制，也就是说我们使用 CMS 或者是 G1的垃圾回收机制不起任何作用。这个时候我们可以通过配置，-XX:+ExplicitGCInvokesConcurrent 来改变这种默认的行为。
3. 并行GC在Full GC的时候，都会首先进行一次，新生代GC，这样做的目的是为了让第一次的新生代GC，尽可能的分担一部分GC人物，从而尽可能的缩短GC总时间。这个可以通过 -XX:-ScavengeBeforeFullGC 进行去除。默认他是设置为true的。
4. 新生代对象通过一定次数的GC循环后，如果仍然存活可以进入到老年区。默认次数为15次GC，这个数字可以通过，-XX:MaxTenuringThreshold进行设置。但是这个数字还收到survivor区域的使用率的影响，也就是说，如果到达了存活区使用率的上限仍然没有叨叨MaxTenuringThreshold的话，去小的值作为新的晋升老年区的年龄。survivor区的使用率可以通过-XX:TargetSurvivorRatio进行设定。
5. survivor区存方不下的对象通常会被放入到老年代。可以通过参数-XX:PretenureSizeThreshold设置但是有时候却发现没有任何效果，这是因为他被分配到了TLAB区域，所以可以使用参数 -XX:-UserTLAB 然后在看GC信息。
6. TLAB（Thread local allocation buffer）,线程本地缓存，这是一个线程专用的内存区域，由于一般情况下对象是分配在堆上的，而堆是对所有的线程可见的，那么，每次分配内存就需要做同步，这样就会降低效率，因此，JVM使用这种每个线程的专属区域避免线程冲突，TLAB本身占用了eden的空间。在TLAB启动的情况下，是为每一个线程分配自己的TLAB区域的。（其实这个和栈上分配很类似，只不过TLAB在堆上）![对象分配简要说明](https://i.imgsafe.org/36a88a6b5a.png)

### Finallize对GC的影响
1. Java提供了一个和c++析构函数类似的函数叫做finallize，默认情况下是不建议使用finallize进行垃圾回收的。之前也说过，他可能会外泄this引用，复活对象，造成内存泄漏，函数finallize是由FinallizerThread线程处理的。也就是说每一个对象即将被回收的时候都会加入到FinalizeThread的执行队列中。这个和我们之前说过的软引用是一样的。

2. 注意我们的gc在进行垃圾回收的时候，只会收集不可达到的对象，如果我们在finallize方法中外泄了this引用，就会导致那一块内存永远释放不了。并且由于对象在释放会时候会进入到FinallizeThread监测的队列中，只有从检测队列中出队列的对象才能被释放掉，因为Finallizer有一个强引用指向对象，所以如果finallize方法使用的时间过长，则可能从整体上拖垮程序的性能，降低垃圾回收的能力。所以一定注意不要轻易使用finallize。具体的流程如图：![finallize监控队列](https://i.imgsafe.org/3767963b53.png)

## 垃圾回收器对TOMCAT性能的影响实验

#### 采用JMeter进行性能监测
1.  我用15个线程，每个线程请求1000次总共15000次请求。来压测我们的tomcat每秒能够介绍多少个请求。bash```CATALINA_OPTS="Xloggc:/var/tmp/gc.log -Xmx128m -Xms64m -XX:+PrintGCDetails -XX:+UseSerialGC```注意将参数设置在catalina.sh(linux)/catalina.bat(windows) 中，一般设置在第一行即可。我们看看这样的参数得到的结果，`36/s`大概在36个请求每秒左右。

2. bash ```CATALINA_OPTS="Xloggc:/var/tmp/gc.log -Xmx1024m -Xms512m -XX:+PrintGCDetails -XX:+UseSerialGC``` => `82/s`

3. bash ```CATALINA_OPTS="Xloggc:/var/tmp/gc.log -Xmx1024m -Xms512m -XX:+UsePrintGCDetails -XX:+UseParallelGC -XX:+ParallelOldGC -XX:ParallelGCThread=4``` => `156/s`

4. bash ```CATALINA_OPTS="Xloggc:/var/tmp/gc.log -Xmx1024m -Xms512m -XX:+PrintGCDetails -XX:+ParallelGC``` => `158/s`


## 性能监测工具

### linux 性能监控工具

#### 显示系统整体的使用情况 top
![top命令行](https://i.imgsafe.org/45bd8b2ace.png)

1. 第一行分别表示，当前时间，系统运行时间，当前用户，平均负载 第二行分别表示，当前总进程，正在运行进程，睡眠进程，停止的进程，僵尸进程。后几行显示CPU，内存，交换区内存的使用情况。
2. 下面的显示的是具体的进程详情。 [CPU和内存的区别](http://www.differencebetween.com/difference-between-cpu-and-vs-ram/)
 
#### 类似的还有vmstat , iostat

#### pidstat 一个强大的监测工具，不仅可以监测进程，还可以监测线程
```java
/**
 * 
 * @author clark
 * 
 * Jan 22, 2017
 * 开启四个线程，其中一个占用忙碌，另外三个空闲状态
 */
public class PidstatTest{
	
	public static class BusyThread extends Thread{
		@Override
		public void run() {
			while(true){
				System.out.print("hei , i am busy\t");
			}
		}
	}
	
	public static class IdleThread extends Thread{
		@Override
		public void run() {
			while(true){
				try {
					Thread.sleep(1000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}
	
	public static void main(String[] args) {
		
		new PidstatTest.BusyThread().start();
		new PidstatTest.IdleThread().start();
		new PidstatTest.IdleThread().start();
		new PidstatTest.IdleThread().start();
		
	}
}
```
通过jps找到执行的进程
```
15502 PidstatTest
```
然后使用`pidstat -p 15502 -u 1 3` 对15502号线程的CPU进行每1秒钟一次的采样连续三次。
```bash
[root@SextoyTRC ~]# pidstat -p 15502 -u 1 3
Linux 2.6.32-042stab113.11 (SextoyTRC) 	22.1.2017 	_i686_	(24 CPU)

20:28:19          PID    %usr %system  %guest    %CPU   CPU  Command
20:28:20        15502    0,00    1,00    0,00    1,00     0  java
20:28:21        15502    1,00    0,00    0,00    1,00     0  java
20:28:22        15502    0,00    0,00    0,00    0,00     0  java
Average:        15502    0,33    0,33    0,00    0,67     -  java
```
如果想要看进一步的线程信息可以通过添加参数`-t`完成对具体的线程的监控
```bash
[root@SextoyTRC ~]# pidstat -p 15502 -u 1 3 -t
Linux 2.6.32-042stab113.11 (SextoyTRC) 	22.1.2017 	_i686_	(24 CPU)

20:44:46         TGID       TID    %usr %system  %guest    %CPU   CPU  Command
20:44:47        15502         -    0,00    0,00    0,00    0,00     0  java
20:44:47            -     15502    0,00    0,00    0,00    0,00     0  |__java
20:44:47            -     15503    0,00    0,00    0,00    0,00     1  |__java
20:44:47            -     15504    0,00    0,00    0,00    0,00     0  |__java
20:44:47            -     15505    0,00    0,00    0,00    0,00     0  |__java
20:44:47            -     15506    0,00    0,00    0,00    0,00     0  |__java
20:44:47            -     15507    0,00    0,00    0,00    0,00     0  |__java
20:44:47            -     15508    0,00    0,00    0,00    0,00     0  |__java
20:44:47            -     15509    0,00    0,00    0,00    0,00     1  |__java
20:44:47            -     15510    0,00    0,00    0,00    0,00     0  |__java
20:44:47            -     15511    0,00    0,00    0,00    0,00     0  |__java
20:44:47            -     15512    0,00    0,00    0,00    0,00     0  |__java
20:44:47            -     15513    0,00    0,00    0,00    0,00     0  |__java
20:44:47            -     15514    0,00    0,00    0,00    0,00     0  |__java
20:44:47            -     16035    0,00    0,00    0,00    0,00     1  |__java

20:44:47         TGID       TID    %usr %system  %guest    %CPU   CPU  Command
20:44:48        15502         -    1,00    0,00    0,00    1,00     0  java
20:44:48            -     15502    0,00    0,00    0,00    0,00     0  |__java
20:44:48            -     15503    0,00    0,00    0,00    0,00     1  |__java
20:44:48            -     15504    0,00    0,00    0,00    0,00     0  |__java
20:44:48            -     15505    0,00    0,00    0,00    0,00     0  |__java
20:44:48            -     15506    0,00    0,00    0,00    0,00     0  |__java
20:44:48            -     15507    0,00    0,00    0,00    0,00     0  |__java
20:44:48            -     15508    0,00    0,00    0,00    0,00     0  |__java
20:44:48            -     15509    0,00    0,00    0,00    0,00     1  |__java
20:44:48            -     15510    0,00    0,00    0,00    0,00     1  |__java
20:44:48            -     15511    1,00    0,00    0,00    1,00     1  |__java
20:44:48            -     15512    0,00    0,00    0,00    0,00     0  |__java
20:44:48            -     15513    0,00    0,00    0,00    0,00     0  |__java
20:44:48            -     15514    0,00    0,00    0,00    0,00     0  |__java
20:44:48            -     16035    0,00    0,00    0,00    0,00     1  |__java

20:44:48         TGID       TID    %usr %system  %guest    %CPU   CPU  Command
20:44:49        15502         -    0,00    0,00    0,00    0,00     0  java
20:44:49            -     15502    0,00    0,00    0,00    0,00     0  |__java
20:44:49            -     15503    0,00    0,00    0,00    0,00     1  |__java
20:44:49            -     15504    0,00    0,00    0,00    0,00     0  |__java
20:44:49            -     15505    0,00    0,00    0,00    0,00     0  |__java
20:44:49            -     15506    0,00    0,00    0,00    0,00     0  |__java
20:44:49            -     15507    0,00    0,00    0,00    0,00     0  |__java
20:44:49            -     15508    0,00    0,00    0,00    0,00     0  |__java
20:44:49            -     15509    0,00    0,00    0,00    0,00     1  |__java
20:44:49            -     15510    0,00    0,00    0,00    0,00     0  |__java
20:44:49            -     15511    0,00    0,00    0,00    0,00     1  |__java
20:44:49            -     15512    0,00    0,00    0,00    0,00     0  |__java
20:44:49            -     15513    0,00    0,00    0,00    0,00     0  |__java
20:44:49            -     15514    0,00    0,00    0,00    0,00     0  |__java
20:44:49            -     16035    0,00    0,00    0,00    0,00     1  |__java

Average:         TGID       TID    %usr %system  %guest    %CPU   CPU  Command
Average:        15502         -    0,33    0,00    0,00    0,33     -  java
Average:            -     15502    0,00    0,00    0,00    0,00     -  |__java
Average:            -     15503    0,00    0,00    0,00    0,00     -  |__java
Average:            -     15504    0,00    0,00    0,00    0,00     -  |__java
Average:            -     15505    0,00    0,00    0,00    0,00     -  |__java
Average:            -     15506    0,00    0,00    0,00    0,00     -  |__java
Average:            -     15507    0,00    0,00    0,00    0,00     -  |__java
Average:            -     15508    0,00    0,00    0,00    0,00     -  |__java
Average:            -     15509    0,00    0,00    0,00    0,00     -  |__java
Average:            -     15510    0,00    0,00    0,00    0,00     -  |__java
Average:            -     15511    0,33    0,00    0,00    0,33     -  |__java
Average:            -     15512    0,00    0,00    0,00    0,00     -  |__java
Average:            -     15513    0,00    0,00    0,00    0,00     -  |__java
Average:            -     15514    0,00    0,00    0,00    0,00     -  |__java
Average:            -     16035    0,00    0,00    0,00    0,00     -  |__java
20:29:51            -     15508    0,00    0,00    0,00    0,00     0  |__java
20:29:51            -     15509    0,00    0,00    0,00    0,00     1  |__java
20:29:51            -     15510    0,00    0,00    0,00    0,00     1  |__java
20:29:51            -     15511    0,00    0,00    0,00    0,00     0  |__java
20:29:51            -     15512    0,00    0,00    0,00    0,00     0  |__java
20:29:51            -     15513    0,00    0,00    0,00    0,00     0  |__java
20:29:51            -     15514    0,00    0,00    0,00    0,00     1  |__java

Average:         TGID       TID    %usr %system  %guest    %CPU   CPU  Command
Average:        15502         -    0,33    0,33    0,00    0,67     -  java
Average:            -     15502    0,00    0,00    0,00    0,00     -  |__java
Average:            -     15503    0,00    0,00    0,00    0,00     -  |__java
Average:            -     15504    0,00    0,00    0,00    0,00     -  |__java
Average:            -     15505    0,00    0,00    0,00    0,00     -  |__java
Average:            -     15506    0,00    0,00    0,00    0,00     -  |__java
Average:            -     15507    0,00    0,00    0,00    0,00     -  |__java
Average:            -     15508    0,00    0,00    0,00    0,00     -  |__java
Average:            -     15509    0,00    0,00    0,00    0,00     -  |__java
Average:            -     15510    0,00    0,33    0,00    0,33     -  |__java
Average:            -     15511    0,00    0,00    0,00    0,00     -  |__java
Average:            -     15512    0,00    0,00    0,00    0,00     -  |__java
Average:            -     15513    0,00    0,00    0,00    0,00     -  |__java
Average:            -     15514    0,00    0,00    0,00    0,00     -  |__java
```
我们可以看到线程号为15510的消耗了很多资源。我们在通过jstack将程序的线程信息全部导出来分析`jstack -l 1187 > t.txt`
```bash
[root@SextoyTRC ~]# cat t.txt 
2017-01-22 20:34:51
Full thread dump Java HotSpot(TM) Client VM (25.111-b14 mixed mode):

"Attach Listener" #12 daemon prio=9 os_prio=0 tid=0xabd04c00 nid=0x3ea3 runnable [0x00000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"DestroyJavaVM" #11 prio=5 os_prio=0 tid=0xb6906800 nid=0x3c8f waiting on condition [0x00000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Thread-3" #10 prio=5 os_prio=0 tid=0xb6992400 nid=0x3c9a waiting on condition [0xab909000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
	at java.lang.Thread.sleep(Native Method)
	at PidstatTest$IdleThread.run(PidstatTest.java:17)

   Locked ownable synchronizers:
	- None

"Thread-2" #9 prio=5 os_prio=0 tid=0xb6991000 nid=0x3c99 waiting on condition [0xab95a000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
	at java.lang.Thread.sleep(Native Method)
	at PidstatTest$IdleThread.run(PidstatTest.java:17)

   Locked ownable synchronizers:
	- None

"Thread-1" #8 prio=5 os_prio=0 tid=0xb698fc00 nid=0x3c98 waiting on condition [0xab9ab000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
	at java.lang.Thread.sleep(Native Method)
	at PidstatTest$IdleThread.run(PidstatTest.java:17)

   Locked ownable synchronizers:
	- None

"Thread-0" #7 prio=5 os_prio=0 tid=0xb698e800 nid=0x3c97 runnable [0xab9fc000]
   java.lang.Thread.State: RUNNABLE
	at java.io.FileOutputStream.writeBytes(Native Method)
	at java.io.FileOutputStream.write(FileOutputStream.java:326)
	at java.io.BufferedOutputStream.flushBuffer(BufferedOutputStream.java:82)
	at java.io.BufferedOutputStream.flush(BufferedOutputStream.java:140)
	- locked <0xaf2b8868> (a java.io.BufferedOutputStream)
	at java.io.PrintStream.write(PrintStream.java:482)
	- locked <0xaf2b2360> (a java.io.PrintStream)
	at sun.nio.cs.StreamEncoder.writeBytes(StreamEncoder.java:221)
	at sun.nio.cs.StreamEncoder.implFlushBuffer(StreamEncoder.java:291)
	at sun.nio.cs.StreamEncoder.flushBuffer(StreamEncoder.java:104)
	- locked <0xaf2b2300> (a java.io.OutputStreamWriter)
	at java.io.OutputStreamWriter.flushBuffer(OutputStreamWriter.java:185)
	at java.io.PrintStream.write(PrintStream.java:527)
	- locked <0xaf2b2360> (a java.io.PrintStream)
	at java.io.PrintStream.print(PrintStream.java:669)
	at PidstatTest$BusyThread.run(PidstatTest.java:7)

   Locked ownable synchronizers:
	- None

"Service Thread" #6 daemon prio=9 os_prio=0 tid=0xb6984000 nid=0x3c95 runnable [0x00000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"C1 CompilerThread0" #5 daemon prio=9 os_prio=0 tid=0xb6980800 nid=0x3c94 waiting on condition [0x00000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Signal Dispatcher" #4 daemon prio=9 os_prio=0 tid=0xb697f000 nid=0x3c93 runnable [0x00000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Finalizer" #3 daemon prio=8 os_prio=0 tid=0xb6964400 nid=0x3c92 in Object.wait() [0xabefe000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0xaf2b29a8> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:143)
	- locked <0xaf2b29a8> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:164)
	at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:209)

   Locked ownable synchronizers:
	- None

"Reference Handler" #2 daemon prio=10 os_prio=0 tid=0xb6961400 nid=0x3c91 in Object.wait() [0xac0af000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0xaf2b2b48> (a java.lang.ref.Reference$Lock)
	at java.lang.Object.wait(Object.java:502)
	at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
	- locked <0xaf2b2b48> (a java.lang.ref.Reference$Lock)
	at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

   Locked ownable synchronizers:
	- None

"VM Thread" os_prio=0 tid=0xb695cc00 nid=0x3c90 runnable 

"VM Periodic Task Thread" os_prio=0 tid=0xb6986800 nid=0x3c96 waiting on condition 

JNI global references: 7
```
我们可以看到线程NID为0x3c97的转化为10进制后，刚好是15511，磁盘I/O也是性能的一大瓶颈。使用pidstat也可以监测进程内线程的IO。
```
public class PidstatTest{
	
	public static class BusyIOThread extends Thread{
		@Override
		public void run() {
			while(true){
				try {
					PrintWriter pw = new PrintWriter(new File("temp.txt"));
					for(int i=0;i<10;i++){
						pw.println("i am a good man " + i);
					}
					pw.close();
					BufferedReader br = new BufferedReader(new FileReader(new File("temp.txt")));
					String str = null;
					try {
						while((str = br.readLine())  != null){
							System.out.println(str);
						}
					} catch (IOException e) {
						e.printStackTrace();
					}finally{
						try {
							br.close();
						} catch (IOException e) {
						}
					}
					
				} catch (FileNotFoundException e) {
					e.printStackTrace();
				}
				
			}
		}
	}
	
	public static class IdleIOThread extends Thread{
		@Override
		public void run() {
			while(true){
				try {
					Thread.sleep(1000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}
	
	public static void main(String[] args) {
		
		new PidstatTest.BusyIOThread().start();
		new PidstatTest.IdleIOThread().start();
		new PidstatTest.IdleIOThread().start();
		new PidstatTest.IdleIOThread().start();
		
	}
}
```
通过加上参数`-d`表明添加监控对象磁盘IO
```bash
[root@SextoyTRC ~]# pidstat -p 17154 -d -t 1 3
Linux 2.6.32-042stab113.11 (SextoyTRC) 	22.1.2017 	_i686_	(24 CPU)

20:58:48         TGID       TID   kB_rd/s   kB_wr/s kB_ccwr/s  Command
20:58:49        17154         -      0,00   1376,00      0,00  java
20:58:49            -     17154      0,00      0,00      0,00  |__java
20:58:49            -     17155      0,00      0,00      0,00  |__java
20:58:49            -     17156      0,00      0,00      0,00  |__java
20:58:49            -     17157      0,00      0,00      0,00  |__java
20:58:49            -     17158      0,00      0,00      0,00  |__java
20:58:49            -     17159      0,00      0,00      0,00  |__java
20:58:49            -     17160      0,00      0,00      0,00  |__java
20:58:49            -     17161      0,00      0,00      0,00  |__java
20:58:49            -     17162      0,00      0,00      0,00  |__java
20:58:49            -     17163      0,00   1376,00      0,00  |__java
20:58:49            -     17164      0,00      0,00      0,00  |__java
20:58:49            -     17165      0,00      0,00      0,00  |__java
20:58:49            -     17166      0,00      0,00      0,00  |__java

20:58:49         TGID       TID   kB_rd/s   kB_wr/s kB_ccwr/s  Command
20:58:50        17154         -      0,00   1392,00      0,00  java
20:58:50            -     17154      0,00      0,00      0,00  |__java
20:58:50            -     17155      0,00      0,00      0,00  |__java
20:58:50            -     17156      0,00      0,00      0,00  |__java
20:58:50            -     17157      0,00      0,00      0,00  |__java
20:58:50            -     17158      0,00      0,00      0,00  |__java
20:58:50            -     17159      0,00      0,00      0,00  |__java
20:58:50            -     17160      0,00      0,00      0,00  |__java
20:58:50            -     17161      0,00      0,00      0,00  |__java
20:58:50            -     17162      0,00      0,00      0,00  |__java
20:58:50            -     17163      0,00   1392,00      0,00  |__java
20:58:50            -     17164      0,00      0,00      0,00  |__java
20:58:50            -     17165      0,00      0,00      0,00  |__java
20:58:50            -     17166      0,00      0,00      0,00  |__java

20:58:50         TGID       TID   kB_rd/s   kB_wr/s kB_ccwr/s  Command
20:58:51        17154         -      0,00   1376,00      0,00  java
20:58:51            -     17154      0,00      0,00      0,00  |__java
20:58:51            -     17155      0,00      0,00      0,00  |__java
20:58:51            -     17156      0,00      0,00      0,00  |__java
20:58:51            -     17157      0,00      0,00      0,00  |__java
20:58:51            -     17158      0,00      0,00      0,00  |__java
20:58:51            -     17159      0,00      0,00      0,00  |__java
20:58:51            -     17160      0,00      0,00      0,00  |__java
20:58:51            -     17161      0,00      0,00      0,00  |__java
20:58:51            -     17162      0,00      0,00      0,00  |__java
20:58:51            -     17163      0,00   1376,00      0,00  |__java
20:58:51            -     17164      0,00      0,00      0,00  |__java
20:58:51            -     17165      0,00      0,00      0,00  |__java
20:58:51            -     17166      0,00      0,00      0,00  |__java

Average:         TGID       TID   kB_rd/s   kB_wr/s kB_ccwr/s  Command
Average:        17154         -      0,00   1381,33      0,00  java
Average:            -     17154      0,00      0,00      0,00  |__java
Average:            -     17155      0,00      0,00      0,00  |__java
Average:            -     17156      0,00      0,00      0,00  |__java
Average:            -     17157      0,00      0,00      0,00  |__java
Average:            -     17158      0,00      0,00      0,00  |__java
Average:            -     17159      0,00      0,00      0,00  |__java
Average:            -     17160      0,00      0,00      0,00  |__java
Average:            -     17161      0,00      0,00      0,00  |__java
Average:            -     17162      0,00      0,00      0,00  |__java
Average:            -     17163      0,00   1381,33      0,00  |__java
Average:            -     17164      0,00      0,00      0,00  |__java
Average:            -     17165      0,00      0,00      0,00  |__java
Average:            -     17166      0,00      0,00      0,00  |__java
```
我们可以看到线程ID为17163的读写的比较频繁，我们可以通过导出线程信息，来查看这个线程具体是做什么的。这里就不在赘述了，和之前的一样。除此之外，我们还可以通过`-r`参数查看进程的内存的具体详情。
```bash
[root@SextoyTRC ~]# pidstat -p 17154 -r 1 3
Linux 2.6.32-042stab113.11 (SextoyTRC) 	22.1.2017 	_i686_	(24 CPU)

21:05:16          PID  minflt/s  majflt/s     VSZ    RSS   %MEM  Command
21:05:17        17154      8,00      0,00  195168  15836   3,02  java
21:05:18        17154      9,00      0,00  195168  15836   3,02  java
21:05:19        17154      0,00      0,00  195168  15836   3,02  java
Average:        17154      5,67      0,00  195168  15836   3,02  java
```
其中 minflt 表示每秒不需要从瓷盘中调出内存页的总数，majflt表示需要从瓷盘中调出的内存页总数，VSZ表示使用的虚拟内存，RSS表示使用的物理内存，后面是使用的MEM占比。

### JDK 性能监测工具
我们上面使用到的诸如jps jstack都是，它们在bin目录下，实际实现实在$JAVA_HOME/lib/tools.jar中。
* jps ,类似于ps只不过但用于显示java进程。可以直接使用jps，会显示进程以及主函数段名称。也可以添加 -m(显示传入参数) -l(显示完整的主函数名称) -v(显示JVM参数)

* jstat,查看虚拟机运行时信息，拥有很多的参数可以很方便的查看jvm堆的使用情况以及GC情况。

* jinfo, 可以查看JVM参数的当前值，以及动态添加修改某些特定的参数值。 

* jmap, 导出堆。可以查看堆内对象实例的统计信息。查看classloader信息，以及finallize队列。

* jhat, 用于分析java堆快照内容，比如jmap导出的堆文件，可以通过jhat进行分析。jhat命令分析堆快照，启动了一个http服务器。我们可以通过http://localhost:7000 进行访问。

* jstack, 可以打印出线程相关的堆信息以及栈信息。可以找到死锁。

* jstatd, RMI服务器程序。帮助其他命令完成远程监控。比如jps jstack等。

* jcmd, 是一个1.7后添加的新的命令，它可以完成上面的命令的很多功能，比如导出堆，打印java进程，查看jvm参数，等等。
上面知识简单的介绍了这些命令，最主要的还是要大家去看源码，然后一个命令一个命令的消化。否则只是会用，没有任何意义。

### 虚拟机监控工具JConsole
使用JConsole可以监控java堆，永久区使用情况，类加载情况。

#### 使用JConsole进行远程监控 
如果要进行远程连接，那么我们必须首先在启动程序前，配置参数，启动命令jstatd，开启一个RMI服务。然后我们的远程客户端才能连接。
```java
-Djava.rmi.server.hostname=127.0.0.1
-Dcom.sun.management.jmxremote 
-Dcom.sun.management.jmxremote.port=8081 
-Dcom.sun.management.jmxremote.authenticate=false 
-Dcom.sun.management.jmxremote.ssl=false
```
使用Jconsole连接上以后的图形是这样的
![Jconsole连接](https://i.imgsafe.org/56275ba763.png)

* 内存监控：eden，survivor等区域的内存都可以看到，并且还可以点击perform gc进行一次强制的full gc.
* 线程监控：可以查看具体的每个线程的栈信息，以及检查死锁等。
* 类加载监控：查看已经加载的类数量，卸载的类数量。
* 虚拟机信息：jvm的一些参数信息。

类似于Jconsole的工具还有virtual VM 以及  Java Misson Console .  如果大家在工作觉得Jconsole的内容还不够细，可以使用刚才说的两款工具。

## 其他的点什么

### 深堆和浅堆
#### 深堆和浅堆是两个非常重要的概念：它们分别表示一个对象结构所占有的内存大小和一个对象被GC回收后，可以释放的实际大小。浅堆是指一个对象所消耗的内存。在32位系统中一个对象引用占4个字节，一个int类型占4个字节，long占8个字节。每个对象头需要占有8个字节。深堆：是指对象的保留集中所有对象的浅堆大小之和。保留集：对象A被垃圾回收后，只能被A对象访问到的所有的对象的集合。浅堆指对象本身占的内存大小，不包括引用的大小。一个对象的深堆指只能通过该对象访问到的所有对象的浅堆之和。
```java
public class A{
	int id = 0;
	private B b = new B();
	private D d = D.d;
}

public class B{
	int id = 0;
	private C c = new C();
	private D d = D.d;
}

public class D{
	int id = 0;
	public static D d = new D();
	
}
```
* A浅堆: 4（引用大小） ＋ 4（int类型大小） ＋ 8（对象头大小） ＋ 4（向8对齐后的大小） ＝ 24
* A深堆: 浅堆B ＝ 24 （由于A中的d对象不只只有A对象可以访问到，b也可以引用到，所以它并不是保留集中的一员）

## 锁与并发在JVM中的应用

### 锁的基本概念与实现
* 锁，用于保护临界区资源不会因为多个线程的同时访问而遭到破坏。如果由于多线程的访问导致对象数据的不一致，那么系统运行将会得到错误的结果。通过锁，让线程排队，一个个进入到临界区访问目标对象。是目标对象的状态总是保持一致。这也是锁存在的意义。

* 对象头：在java虚拟机的实现中，每个对象都有一个对象头，用于保存对象的系统信息。对象头中有一个被称为`Mark Word`的部分。它是实现锁的关键。他是一个多功能区域。存放着对象的hash值，对象年龄，锁的指针等信息。

### 为了避免将所有的多线程问题都交由操作系统去解决，JVM会在虚拟机层面上尽量减少竞争。这些手法就是下面我们要说的锁

#### 细说锁的类型
* 偏向锁：如果程序没有竞争则取消之前获取的线程同步操作。也就是说当线程第一次进入同步中，然后将锁的信息放入到`Mark Word`后，第二次同样的线程再次进行访问的时候，对象就会不进行相关的同步操作，从而增加访问速度。 可以通过 -XX:+UseBiasedLocking 进行设置。 注意偏向锁紧紧真对竞争不激烈的场景，如果在竞争激烈的场景下，那么会导致持有锁的线程不停的切换，这个时候锁很难保持偏向状态，这样不仅不能增加性能，反而有可能降低系统性能。可以使用-XX:-UseBiasedLocking参数禁用偏向锁。

* 轻量级锁，重量级锁：在偏向锁失败后，线程会申请轻量级锁，如果轻量级锁失败后锁开始膨胀线程开始申请重量级锁。

* 自旋锁 ： 锁膨胀后线程很可能会在操作系统层面上被挂起。这样[上下文切换](https://en.wikipedia.org/wiki/Context_switch)性能的损失就比较大。一种解决方案就是使用一个空循环，如果在不长的时间内，获取到了锁，那么就避免了线程被挂起。但是如果对于竞争很激烈的场景的话，使用自选旋锁则不好，因为获取可能很长时间自旋之后还是获取不到锁，这样白白占用了cpu时间。

* 锁消除，jvm在进行JIT编译时，通过扫描上下文去除掉那些不可能有资源共享的锁。通过消除这些锁，可以节省毫无意义的请求锁时间。
```java
public String appendStr(String str){
	StringBuffer sb = new StringBuffer();
	sb.append(str);
	return sb.toString();
}
```
我们可以看到上面的代码不可能产生线程竞争的场景。都是局部变量。科室StringBuffer内部是使用到了同步操作的。那么锁消除会把锁干掉。

#### 我们如何使用锁
* 减小锁的持有时间：在使用锁操作的时候，尽量在最小单元处加入锁。

* 减小锁粒度：举个例子就是一个只有一个门的大厕所，分成多个门，然后这样每次就可以有多个人上厕所，而不是一个个的来。将一把大锁，换成多个小锁。其中的一个经典的实现案例，就是ConcurrentHashMap.其中的一个特例就是锁分离，其实核心思想和减小锁力度是一样的。

#### 如果无招胜有招：无锁
* 使用ThreadLocal 每个线程拥有自己的对象。从而达到无锁的效果，仍然可以保持一致性。

* CAS算法：CAS(V,E,N) 三个参数分别表示需要修改的变量，期待的当前变量的值，N，如果当前值与期待值相同那么就会更新当前变量为为新的值，否则表示有别的线程已经修改了当前变量值，这个时候返回修改后的值。目前大部分的现代处理器都支持原子化的CAS指令。并且在JVM的实现中也用的十分广泛。我们atomic包下面的AtomicInteger，AtomicLong，AtomicReference 等等的信息全部都是通过CAS算法实现的。
```java
public final int getAndSet(int newValue){
	for(;;){
		int current = get();
		if(compareAndSet(current,newValue)){ 
		// compareAndSet 是原子操作的所以不用担心不一致问题
			return current;
		}
	}
}
```
* CAS算法和减小锁粒度。 由于我们的CAS实现原子操作是通过类似自旋来实现的。但是我们前面也说过使用自选的坏处。如果在竞争激烈的场景下，可能很长时间都在循环中。因此我们向是不是可以对那些值比如AtomicInteger的value不把value作为一个整体，而是将value进行拆分，把她们的值分别放入到一个数组里面。然后我们对这个数组保证原子性。这种分离热点的思想。一个经典的例子就是LongAdder的实现。
![LongAdder](https://i.imgsafe.org/5bf09cd914.png)
 
### java内存模型中的一些关于所得应用

#### 原子性
指不可分割的操作，不能被多线程打扰。比如对int，byte的赋值就是原子操作的，但是比如a++这种高操作就不是原子操作的，因为他有三个步骤1读取，2自加，3赋值。中间是很可能被其他线程干扰导致错误的结果。在32位JVM中，对long和double的赋值操作，由于它们都是64bit的，无法一次性操作，因此对于它们的操作都不是原子的，在并发环境下，可能会出现一些意想不到的错误。 这种问题可以添加volatile，保证变量写入的原子性。

#### 有序性
现代CPU都支持指令流水线执行，为了保证流水线的顺畅执行，有时候会对指令进行重排序，重排序不会导致单线程的语义错误。但是在多线程操作中就并不能保证语义的正确性。
```java
class orderExample{
	private int flag;
	private int a;
	
	public void write(){
		a = 10;
		flag = true;
	}
	public void read(){
		if(flag){
			System.out.println(a);
		}
	}
}
```
这个时候如果有两个线程访问，一个读一个写，那么可能的结果为`0`,避免重排序问题的手段可以通过将上述的读写操作都加synchronized 第二种方法就是给a加volatile。volatile就是保证对象的一致性。这个时候系统不会对a进行重排序，就不会产生问题。

#### 可见性
可见性是指一个线程修改了某个变量的值，在另一个线程中马上就可以知道，可以使用volatile保证一致性。 但是也可以使用synchronized 。 由此可见synchronized不仅可以解决同步问题，还可以解决可见性问题。

####Happen-before原则
 * 程序顺序原则：一个线程内保证语义的串行性。
 * volatile原则：volatile的写会先发生于读，保证了它的可见性。
 * 锁规则：解锁必定是发生在之后的加锁前。
 * 传递性，A先于B，B先于C，则A先于C
 * 线程的start方法先于他的每一个动作
 * 线程的所有操作都先于Thread.join()
 * 线程的中断先于被中断线程的代码
 * 对象的构造函数执行结束在finalize()之前

## Class 

### Class文件
![class文件](https://i.imgsafe.org/6b789a6b0f.png)
```cpp
ClassFile {
  u4			magic;  // 魔头
  u2			minor_version; //  小版本号
  u2			major_version; //  大版本号
  u2			constant_pool_count;  // 常量池
  cp_info		constant_pool[constant_pool_count -1];  // 常量池的表项
  u2			access_flags;	//访问修饰符
  u2			this_class;	//代表自身类的修饰符
  u2			super_class;	//父类引用
  u2			interface_count;//接口数量
  u2			interface[interfaces_count];//接口实现数量
  u2			fields_count; //字段数量 
  field_info		fields[fields_count]; //字段描述
  u2			methods_count;  // 方法数量
  method_info		methods[methods_count]; //方法描述
  u2			attribute_count; // 属性数量
  attribute_info	attributes[attribute_count]; //属性描述
}
```
其中u1,u2,u4,u8分别表示无符号整型单子节，2字节，4字节，8字节。由于我们在实际开发过程中直接使用class文件的场景十分稀少，所以这个地方我就不将上述结构体的字段一个个的说明了。

### 操作字节码ASM

ASM是一款字节码的操作库。很多有名的软件都是基于它开发出来的。比如AspectJ，Clojure，Eclipse，spring，以及CGlib都是ASM的使用者。与CGlib以及Javaassist相比ASM的性能比他们高很多。但是由于过于底层，所以复杂度还是有的。因此开发人员一般是不直接便携ASM对class文件进行编写的。

## ClassLoader

### Class的加载流程
系统装载Class类型可以分为加载，连接，初始化三个步骤。其中，连接又可以分为验证，准备，解析三步。
![Class Init](https://i.imgsafe.org/6bfdd63aa4.png)

#### 类加载的条件
类不会全部加载，只有在`主动`使用的时候才会被加载，下面为主动加载的情况：
* 使用new关键字，或者使用反射，克隆，反序列化。
* 当调用了static方法时候，即使用了字节码invokestatic指令
* 使用了类或者接口的静态字段（不是final的）时候。
* 使用了反射包下面的方法时候。
* 初始化子类时候，父类会首先初始化。
* 作为启动虚拟机，含有main方法的类。
```java
public class InitTest{
	
	static class Father{
		protected static int d = 10;
		static{
			System.out.println("this is father !");
		}
	}
	
	static class Child extends Father{
		static{
			System.out.println("this is child !");
		}
	}
	
	public static void main(String[] args) {
		System.out.println(InitTest.Child.d);
	}
	
}
```
```
[Loaded test.InitTest$Father from file:/Users/clark/git/forum_demo/forum_demo/target/test-classes/]
[Loaded test.InitTest$Child from file:/Users/clark/git/forum_demo/forum_demo/target/test-classes/]
this is father !
10
```
我们可以看到子类并没有初始化，但是我们可以看到Child类已经被加载入内存。

#### 类的加载
* 通过类的全名，获取类的二进制数据流。
* 解析类的二进制数据流为方法区内的数据结构。
* 创建java.lang.Class类的实例，表示该类型。 （Class对象，是进行反射的前提）

#### 连接之验证类
![验证类](https://i.imgsafe.org/6c463e3881.png)

#### 连接之准备
这个阶段会为类分配响应的内存空间。并且设置初始值。
![各种类型的初始值](https://i.imgsafe.org/6c5309019a.png)

#### 连接之解析类
这个阶段将类，接口，字段，方法的符号引用转化为指针指向具体的地址。

#### 初始化
初始化阶段执行的方法是`<clinit>` 包括一些静态字段以及一些静态方法的赋值。 注意类的初始化并不等同与类的对象的初始化。类的对象的初始化，会初始化一些成员属性。但是在类的初始化中，并不会。最后需要说的是，`<clinit>`函数会通过加锁保证线程安全性。那么这个时候，如果线程1初始化类A，这个时候在类A的初始化的过程中，会需要获取类B的锁。而这个时候线程B已经拥有初始化了，并且它需要类A的锁完成初始化。这个时候，线程1，和线程2就成了一个闭环的死锁。
```java
public class ClInitDeadLock extends Thread{
	
	private String className ;
	
	static class A{
		static {
			try {
				Class.forName("test.ClInitDeadLock$B");
			} catch (ClassNotFoundException e) {
				e.printStackTrace();
			}
		}
	}
	
	static class B{
		static {
			try {
				Class.forName("test.ClInitDeadLock$A");
			} catch (ClassNotFoundException e) {
				e.printStackTrace();
			}
		}
	}
	
	public ClInitDeadLock(String className) {
		this.className = className;
	}
	
	@Override
	public void run() {
		try {
			Class.forName(className);
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		}
	}
	
	public static void main(String[] args) {
		
		new ClInitDeadLock("test.ClInitDeadLock$A").start();
		new ClInitDeadLock("test.ClInitDeadLock$B").start();
	}
	
}
```

### ClassLoader
ClassLoader主要用于类初始化的加载阶段。主要用于从系统外部获得class二进制数据流。ClassLoader有一个parent字段指向他的双亲类加载器。

#### ClassLoader的类别
1. ClassLoader的分类，在标准的Java程序中，Java虚拟机会创建三类ClassLoader为整个应用程序服务。他们分别称为Bootstrap ClassLoader (启动类加载器)，Extension ClassLoader(扩展类加载器)，App ClassLoader（应用类加载器）。除此之外，我们还可以自定义类加载器,扩展Java虚拟机获取Class数据的能力。![类加载器](https://i.imgsafe.org/6f625e9810.png)在这些类加载器中启动类加载器完全是由C实现的，它启动的一些类也全都是C实现的代码，所以这个扩展器并没有任何对应的java对象，显示就是null。在虚拟机设计中，使用这种分散的ClassLoader去装载类是有好处的，不同层次的类可以由不同的类加载器加载。从而进行划分。这有助于系统的模块化设计。比如启动类加载器用于加载rt.jar中的类。扩展类用于加载ext包下面的类。用户自定义的加载器加载自己的代码。

2. ClassLoader在进行类加载的时候使用的是双亲委托模式，也就是先去问自己的parent，看他们能不能加载，如果他们不能加载，就自己去加载。判断类加载的时候，应用类加载器会顺着双亲路经往上判断，知道找到启动类加载器。但是启动类加载器不会向下询问，这个委托路线是单向的，理解这一点很重要。

3. 双亲委托模式的弊端，在前文中已经提到，检查类是否已经加载的委托过程是单向的，这样导致的问题是位于高层类加载的类，不能访问由底层类加载器加载的类。

4. 双亲委托模式的补充，在Java平台中，把核心类中提供给外部服务给应用层实现的接口，叫做SPI（service provider Interface）。那么现在的一个主要的问题就是如何在bootstrap classLoader中使用由App ClassLoader加载的类？？？  通过`Thread.getContextClassLoader();Thread.setContextClassLoadser()` 进行实现的。通过这两个方法，可以把一个ClassLoader置于一个线程实例之中，使该ClassLoader称为一个相对共享的实例。默认情况下ContextClassLoader() 就是App ClassLoader。这样如果我们在Bootstrap classLoader加载的类中可以，使用ContextClassLoader，然后使用这个类加载器去加载由App ClassLoader加载的类的实例。

5. 突破双亲模式。 如果我们不喜欢双亲模式的类加载方式，可以通过即成ClassLoader重写loadClass方法。

6. 热替换：所谓热替换就是在不重启服务器的情况下，只是修改代码，然后输出结果可以改变。其实java要完成热替换最根本的思想就是在调用方法的地方，每次调用的时候，都重新加载一次类。这样如果有新的类替换了原来的类，那么新的类就可以被重新载入替换原有的代码。
![热替换](https://i.imgsafe.org/70ef8dd9d0.png)
