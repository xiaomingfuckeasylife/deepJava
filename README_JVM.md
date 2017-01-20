
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

```
java [-options] class [args...]
```
其中 -option是指在这个地方可以设置jvm参数 后面的class是我们带有main的字节码文件，args是这个main函数的参数。例如：
```
public class Main{
  public static void main(String args[]){
    for(int i=0;i<args.length;i++){
      System.out.println("参数 " + (i+1) + " : " + args[i]);
    }
    System.out.println("Xmx" + Runtime.getRuntime().maxMemory()/1000/1000 +"M");
  }
}
```
```
$>java -Xmx32 cn.Main a 
参数 1 ：a
Xmx32M
```
eclipse中jvm参数的设置在run as->run configurations->Arguments中进行配置即可
提醒大家可能很久没有用过java 以及javac命令了，如果大家在用java命令的时候找不到`Error: Could not find or load main class`  出现这种错误首先可以看下classpath有没有将当前目录设置到里面。linux中应该用引号区隔，windows中用分号区隔。如果这也做了还是有问题。请看你的当前位置在哪里，跳出到包所在的目录然后执行命令。因为如果你在包里面的目录，这个时候你执行命令，java命令还是找不到你的class文件的。

## 辨清Java堆
根据回收机制实现的不同java堆也有所不同，常见的java堆可以分为新生代和老年代两个部分，其中新生代用于存放刚生成不久的对象，老年代用于存放年龄较大的对象，其中新生代还可以分为eden区,s0区以及s1区，其中s0和s1大小一样，也被称为from和to区域。大多数情况下对象出世的时候首先被放入eden区域，之后没经过一次新生垃圾回收，所有存活下来的对象年龄就涨了一岁，这个时候就从eden区域转化到，s0或者s1区域。经过一定次数的回收循环，当对象到达一定的年龄以后就会转到老年区域（tenured）。下面的一个实例用于展示方法区，java堆，java栈的区别，
![堆明细图](https://i.imgsafe.org/1c89869282.png) 

```
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
```
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
```
$> java -Xss200k Main 
the bigest depth is 1359
$> java -Xss400k Main
the bigest depth is 6109
```
### `局部变量表` 局部变量表只有在当前函数调用中有效，当前函数调用结束后，随着函数的销毁，局部变量表随之销毁。由于局部变量表在栈桢，所以如果局部变量变量和参数较多，会使得局部变量表膨胀，从而导致栈桢占的内存变大，从而栈能够嵌套的函数变少。

```
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
```
$> java -Xss200k Main 
the bigest depth is 799
```
局部变量表的槽位是可以充用的，如果一个局部变量过了作用域（出了括号）那么再之后，如果有新的局部变量，就很可能复用过期的局部变量的槽位，从而达到节省资源的目的。局部变量表中的变量也是垃圾回收的重要的根结点，只要被局部变量表直接或者间接联系的对象都是不会被回收的。
```
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
```
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
```
-server -Xmx10M -Xms10M -XX:+DoEscapeAnalysis -XX:+PrintGC -XX:-UseTLAB -XX:+EliminateAllocations
```	
-server : 因为必须在server模式下，才可以开启逃逸分析。-XX:DoEscapeAnalysis ： 启用逃逸分析。 -Xmx10M -Xms10M ：执行最大堆内存以及初始化堆内存。-XX:+PrintGC 打印gc日志。-XX:+ElimateAllocations：开启标量替换，允许将对象打散到栈上。这个时候id会被当作一个局部变量-XX:-UseTLAB：关闭了TLAN。
我们可以看到没有答应什么日志，`[Full GC 408K->386K(9856K), 0.0078271 secs]` 基本上没有堆内存的清理。 栈上分配，一般针对小对象，并且依赖于启用逃逸分析，以及开启标量替换。

## 识别方法区
和java堆一样，方法区（永久区）也是所有线程都能访问的区域，方法区域的大小决定了，jvm可以加载的类的数量。如果类太多，也会导致内存溢出。在jdk1.6,1.7中可以通过设置，-XX:PermSize -XX:MaxPermSize指定。如果系统使用了动态代理，那么可能会产生很多的类。就可能导致永久区内存溢出。

```
	public static void main(String[] args) {
		Interceptor c = new Interceptor();
		for(int i=0;i<1000000;i++){
			CglibBean proxy = c.createProxyObject();
		}
	}
```
然后将jvm参数设置：默认的永久区内存大小为64兆
```
-XX:+PrintGCDetails -XX:PermSize=5M -XX:MaxPermSize=5M
```
然后执行代码后发现永久区内存溢出：
```
Exception in thread "Reference Handler" java.lang.OutOfMemoryError: PermGen space
```
在JDK1.8中，彻底废除了方法区（永久区），取而代之的为元数据区，元数据区大小可以通过使用参数-XX:MaxMetaspaceSize进行设置。这是一块堆外的直接内存。与方法区不同，默认情况下，他会一直使用知道系统内存使用完毕为止。如果元数据区内存溢出同样会报错。`Metaspace`

# 常用虚拟机参数
## 跟踪调试参数

##### 1. 最简单的一个GC参数 -XX:+PrintGC 使用这个参数，是要程序调用了GC，则会打印
```
[Full GC 1362K->375K(83008K), 0.0197301 secs]
```
这个表示执行了一次Full垃圾回收，堆内存使用从1362K到回收后的375k，总的堆内存为83M左右，这词gc使用的时间为0.019秒
##### 2. 如果觉得这个显示的不是那么清楚，可以使用 -XX:+PrintGCDetails 
```
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
##### 3. 使用-XX:+PrintHeapAtGC 可以打印GC前后的堆内存状况
```
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
##### 4.由于GC会引起程序停顿，因此我们可以通过-XX:+PrintGCApplicationConcurrentTime -XX:+PrintGCApplicationStoppedTime观察程序的执行时间，和停顿时间
```
Application time: 0.5324587 seconds
Total time for which application threads were stopped: 0.0160692 seconds
Application time: 0.0262697 seconds
```
##### 5. 将GC产生的日志输入到日志文件。 -Xloggc:gc.log

##### 6. 使用-XX:+TraceClassLoading  -XX:+TraceClassUnloading 用于监测加载的类以及卸载的类
```
[Opened /Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Classes/classes.jar]
[Loaded java.lang.Object from /Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Classes/classes.jar]
[Loaded java.io.Serializable from /Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Classes/classes.jar]
[Loaded java.lang.Comparable from /Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Classes/classes.jar]
[Loaded java.lang.CharSequence from /Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Classes/classes.jar]
[Loaded java.lang.String from /Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Classes/classes.jar]
......
```

###### 7. 使用-XX:+PrintVMOptions 打印JVM接受的显示参数。 -XX:+PrintCommandLineFlags 打印JVM接受的显示和隐式参数 :
```
(显示和隐式)
-XX:MaxNewSize=87244800 -XX:MaxTenuringThreshold=4 -XX:NewRatio=7 -XX:NewSize=21811200 -XX:OldPLABSize=16 -XX:OldSize=65433600 -XX:+PrintCommandLineFlags -XX:+PrintGCDetails -XX:+UseCompressedOops -XX:+UseConcMarkSweepGC -XX:+UseParNewGC 
```

##### 8. 最大堆初始堆的设置：-Xmx-Xms 一般来说虚拟机会维持在初始堆空间，但是如果堆空间不足，则会扩展堆空间内容，扩展的上线为最大堆空间 , 在工作中我们可以将初始和最大堆空间设置成一样的这样可以减少GC，提高性能。

##### 9. 使用参数-Xmn 可以设置新生代的内存大小，这个参数对GC行为有很大的影响，新生代的大小一般设置为整个堆空间的1/3到1/4左右。 -XX:SurvivorRatio用来设置新生代中eden和from/to的比例。
```
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
```
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
```
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

##### 10. 堆内存溢出的时候，我们可以使用-XX:HeapDumpOnOutofMemoryError -XX:HeapDumpPath=XX/a.dump 使用这两个参数如果发生堆内存溢出可以将整个堆信息导出到对应的文件中。
```
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

##### 11. 配置方法区（永久区）内存 可以通过 -XX:PermSize -XX:PermMaxSize 配置元数据区内存可以使用 -XX:MaxMetaspaceSize 指定元数据区的大小。

##### 12. 栈内存的配置可以使用 -XX:Xss 

##### 13. 直接内存的配置可以使用 ， -XX:MaxDirectoryMemorySize 如果不设置默认值为最大的堆内存。当直接内存的使用到达最大值的时候，这个时候就会触发垃圾回收机制，如果这个时候仍然不能释放足够的空间就会报出OutOfMemory的异常，一般来说直接内存的读写速度快于java堆，但是申请内存的速度小于java堆。

首先来测试堆内存和直接内存的访问：
```
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
```
direct Access : 177
buffer Access : 529
direct Access : 417
buffer Access : 511
```
再来测试直接内存和java堆的内存申请速度
```
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
```
direct Access : 44
buffer Access : 16
direct Access : 10
buffer Access : 4
```

##### 14. JVM的两种模式，client 模式和server模式 。 其中client模式相对server模式启动块，针对用户界面程序，运行时间不长，追求启动速度的。但是server模式，启动缓慢，但是稳定，执行速度快。这两种模式下的各种参数大小也有很大的不同。

## 垃圾回收概念与算法

### 讨论常用的垃圾回收算法

##### 1. 引用指数法
 * 实现：通过计算对象当前的引用数，如果为0则可以回收，如果不为0则不能回收。
 * 问题：1 无法处理循环的情况，比如，对象A引用对象B，然后对象B引用对象A，这种情况下，A和B都不会被回收。但是A，B已经没有其他的对象引用了，这个已经说明这两个，对象都是有问题的。都应该被回收。2 由于每个对象都有一个计数器跟着，这样每次对象引用的添加以及对象引用的销毁都会计数，这样堆系统的性能来说并不是很好。因此java并没有使用这种垃圾回收方式。
 
##### 2. 标记清除法

 * 实现：将垃圾回收分为两个阶段，标记阶段，清除阶段。标记从根结点（java栈的局部变量表）开始向上寻找对象找到的对象进行标记。在清除阶段，清除掉所有的没有被标记的对象。
 * 问题：容易产生空间碎片（也就是由于被清除的对象不是一整块一整块的，导致被清除的空间也是不连续的）
 
##### 3. 复制算法
 
 * 实现：将内存分为两份，一份使用，一份暂时不使用。创立对象后，将需要回收的对象移到未使用区域，同时清空第一个区域的内存即可。
 * 缺点：内存减半。
 * 扩展：jvm也使用复制算法。在新生代的 eden from to 三个区域中 from和to就是复制区域。存活的放入到它们的内存里面，然后清除eden区域。
 
##### 4 标记压缩法
 * 实现：基本思想和标记清除法一样，只不过多了一个步骤，将标记的内存压缩到内存的一边，然后以此为边界清楚不需要的内存。
 * 扩展：jvm的老年代的垃圾回收使用的就是这种方式，这种方式既避免了将内存减半，有避免了空间碎片。由于老年区的对象一般很稳定所以并不适合复制算法。
 
##### 5 分代算法
 * 实现：由于不同的算法适用于不同的场景，所以，通常情况下，将内存分为新生代和老年代，新生代的对象基本上百分之90的会被销毁所以适用于复制算法，而老年代的对象，基本上很少需要被回收，所以适合使用标记清除法。
 
##### 6 分区算法
 * 实现：由于GC会是程序停顿，所以，我们如果每次都对整个堆内存进行gc的话，可能是程序陷入较长时间的停顿，这个时候，如果我们将内存分为几个不同的块，这个时候，如果我们只是对个别几个小的区块进行gc，就会减少一次gc的时间。

### 判断可触及性
对象的状态可以分为三种：
 * 可触及的，从根结点开始，可以到达的对象。
 * 可复活的，对象的所有引用都被释放，但是对象仍然可能在finalize函数中复活。 
 * 不可触及的，对象的finalize函数被调用，并且没有复活。因为finalize只可能被调用一次。
 
#### 对象的复活
```
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
```
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
```
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
```
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
