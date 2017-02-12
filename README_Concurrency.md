## Concurrency

###  A brief history of concurrency
  once before we had operation system , we can only execute one program each time which will access the whole resource of a mathine . nowadays , operation system can execute multiple program individually at the sametime using a thing called process . a process is an isolated , independent environment which is allocated by the system. process can comunicated with each other through `sockets` , `shared memory`,`semaphores`,`files`. 
  several motivations factors led to the development of operation system that execute task simultaneously . 
  * Resource utilization : sometimes a program will wait for some extra input or output , in these time , nothing valuable thing has been done . we can put other program to run at this time . 
  * Fairness : there are more than one user using the mathine then it is only fair that they using the resource by time slicing .
  * convinience : it is much more desirable to write multiple program and each program executed one task than one program perform all the task . 
  
### Benefits of threads 
  threads can reduce development and maintenance costs and improve the performance of complex applications. Threads make it easier to model how humans work and interact , by turning asynchonous workflows into mostly sequential ones . 
  * one of the most obvious benefits is Exploiting `the multi-process cpu` . 
  * `simplicify modeling`. sometimes the problem is just too complicated then when you think as a whole problem , you have no idea which way to think , but when you cut the problem into pieces and put every piece task into different thread . it gets easier. 
  * `simplified handling of asynchronous events` . when we using a single-thread events loop , when one user blocked , then all other user blocked . so the best way to create a usable and stable program is to give each user with their own thread . 
 
### Rists of threads 
  threads safe issues.
#### safety hazards 
 Thread safety can be unexpectedly subtle because in the absence of sufficient synchronization. the ordering of operations in multiple threads is unpredictable and sometimes surprising.  
```java
public class UnsafeSequence {
  private int value ; 
  public int getNext(){
    return value++;
  }
}
```
this is not a thread safe program.  as value++ has three steps ( read , modify , write )
![unsafeSequence](https://i.imgsafe.org/58a3c9c6d1.png)
UnsafeSequence illustrates a common concurrency hazard called a race condition. we can using synchronized machanism to  coordinate the access order . 
```java
public class UnsafeSequence {
  private int value ; 
  public synchronized int getNext(){
    return value++;
  }
}
```
In the absence of synchronization, the compiler, hardware, and runtime are allowed to take substantial liberties with the timing and ordering of actions, such as caching variables in registers or processor-local caches where they are temporarily
(or even permanently) invisible to other threads. These tricks are in aid of better performance and are generally desirable, but they place a burden on the developer to clearly identify where data is being shared across threads so that these optimizations do not undermine safety.
 
#### liveness hazards 
safety means “nothing bad ever happens”, liveness concerns the complementary goal that “something good eventually happens”. A liveness failure occurs when an activity gets into a state such that it is permanently unable to make forward progress.

#### performance hazards 
In well designed concurrent applications the use of threads is a net performance gain, but threads nevertheless carry some degree of runtime overhead.Context switches—when the scheduler suspends the active thread temporarily so another thread can run—are more frequent in applications with many threads,and have significant costs: saving and restoring execution context, loss of locality,and CPU time spent scheduling threads instead of running them. When threads share data, they must use synchronization mechanisms that can inhibit compiler optimizations, flush or invalidate memory caches, and create synchronization traffic on the shared memory bus. All these factors introduce additional performance costs;

#### Threads are everywhere 
Every Java application uses threads. When the JVM starts, it creates threads for JVM housekeeping tasks (garbage collection, finalization) and a main thread for running the main method. The AWT (Abstract Window Toolkit) and Swing user interface frameworks create threads for managing user interface events. Timer creates threads for executing deferred tasks. Component frameworks, such as servlets and RMI create pools of threads and invoke component methods in these threads.

Frameworks introduce concurrency into applications by calling application components from framework threads. Components invariably access application state, thus requiring that all code paths accessing that state be thread-safe.

* Timer. Timer is a convenience mechanism for scheduling tasks to run at a later time, either once or periodically. The introduction of a Timer can complicate an otherwise sequential program, because TimerTasks are executed in a thread managed by the Timer, not the application. If a TimerTask accesses data that is also accessed by other application threads, then not only must the TimerTask do so in a thread-safe manner, but so must any other classes that access that data. Often the easiest way to achieve this is to ensure that objects accessed by the Timer Task are themselves thread-safe, thus encapsulating the thread safety within the shared objects.

* Servlets and JavaServer Pages (JSPs). The servlets framework is designed to handle all the infrastructure of deploying a web application and dispatching requests from remote HTTP clients. A request arriving at the server is dispatched, perhaps through a chain of filters, to the appropriate servlet or JSP. Each servlet represents a component of application logic, and in high-volume web sites, multiple clients may require the services of the same servlet at once. The servlets specification requires that a servlet be prepared to be called simultaneously from multiple threads. In other words, servlets need to be thread-safe.Even if you could guarantee that a servlet was only called from one thread at a time, you would still have to pay attention to thread safety when building a web application. Servlets often access state information shared with other servlets, such as application-scoped objects (those stored in the ServletContext) or session-scoped objects (those stored in the per-client HttpSession). When a servlet accesses objects shared across servlets or requests, it must coordinate access to these objects properly, since multiple requests could be accessing them simultaneously from separate threads. Servlets and JSPs, as well as servlet filters and objects stored in scoped containers like ServletContext and HttpSession, `simply have to be thread-safe`.

* Remote Method Invocation. RMI lets you invoke methods on objects running in another JVM. When you call a remote method with RMI, the method arguments are packaged (marshaled) into a byte stream and shipped over the network to the remote JVM, where they are unpacked (unmarshaled) and passed to the remote method. When the RMI code calls your remote object, in what thread does that call happen? You don’t know, but it’s definitely not in a thread you created—your object gets called in a thread managed by RMI. How many threads does RMI create? Could the same remote method on the same remote object be called simultaneously in multiple RMI threads?4 A remote object must guard against two thread safety hazards: properly coordinating access to state that may be shared with other objects, and properly coordinating access to the state of the remote object itself (since the same object may be called in multiple threads simultaneously). Like servlets, RMI objects should be prepared for multiple simultaneous calls and must provide their own thread safety.

* Swing and AWT. GUI applications are inherently asynchronous. Users may select a menu item or press a button at any time, and they expect that the application will respond promptly even if it is in the middle of doing something else. Swing and AWT address this problem by creating a separate thread for handling user-initiated events and updating the graphical view presented to the user.Swing components, such as JTable, are not thread-safe. Instead, Swing programs achieve their thread safety by confining all access to GUI components to the event thread. If an application wants to manipulate the GUI from outside the event thread, it must cause the code that will manipulate the GUI to run in the event thread instead. When the user performs a UI action, an event handler is called in the event thread to perform whatever operation the user requested. If the handler needs to access application state that is also accessed from other threads (such as a document being edited), then the event handler, along with any other code that accesses that state, must do so in a thread-safe manner.

### Thread Safety 

Whenever more than one thread accesses a given state variable, and one of them might write to it, they all must coordinate their access to it using synchronization.
If multiple threads access the same mutable state variable without appropriate synchronization, your program is broken. There are three ways to fix it:
* Don’t share the state variable across threads;
* Make the state variable immutable; or
* Use synchronization whenever accessing the state variable.

It is far easier to design a class to be thread-safe than to retrofit it for thread safety later. When designing thread-safe classes, good object-oriented techniques encapsulation, immutability, and clear specification of invariants are your best friends.
Is a thread-safe program one that is constructed entirely of thread-safe classes? Not necessarily a program that consists entirely of thread-safe classes may not be thread-safe, and a thread-safe program may contain classes that are not thread-safe.In any case, the concept of a thread-safe class makes sense only if the class encapsulates its own state. Thread safety may be a term that is applied to code, but it is about state, and it can only be applied to the entire body of code that encapsulates its state, which may be an object or an entire program.

```java
// SimpleDateFormat is not a thread safe class but this is a thread safe program.
public Date parse(String dateStr , String format) throws Exception{
  SimpleDateFormat sdf = new SimpleDateFormat(format);
  return sdf.parse(dateStr);
}
// the program is used safe class AtomicLong but still this is not a thread safe class . it contains race condition  problem.
public class Unsafe{
  private AtomicLong count = new AtomicLong();
  public void service() {
		if (count.get() % 2 == 0) {
			count.get();
		} else {
			count.addAndGet(1);
		}
	}
}
```

#### what is thread safety ?
A class is thread-safe if it behaves correctly when accessed from multiple threads, regardless of the scheduling or interleaving of the execution of those threads by the runtime environment, and with no additional synchronization or other coordination on the part of the calling code.Thread-safe classes encapsulate any needed synchronization so that clients need not provide their own.
Stateless objects are always thread-safe. it has no fields and references no fields from other classes.
```java
public class StatelessFactorizer implements Servlet {
  public void service(ServletRequest req, ServletResponse resp) {
    BigInteger i = extractFromRequest(req);
    BigInteger[] factors = factor(i);
    encodeIntoResponse(resp, factors);
  }
}
```

#### Atomicity
`Race conditions` : The most common type of race condition is check-then-act, where a potentially stale observation is used to make a decision on what to do next. example : Let’s say you planned to meet a friend at noon at the Starbucks on University Avenue. But when you get there, you realize there are two Starbucks on University Avenue, and you’re not sure which one you agreed to meet at. At 12:10, you don’t see your friend at Starbucks A, so you walk over to Starbucks B to see if he’s there, but he isn’t there either. There are a few possibilities: your friend is late and not at either Starbucks; your friend arrived at Starbucks A after you left; or your friend was at Starbucks B, but went to look for you, and is now en route to Starbucks A. another example of race condition is lazy initialization . as first you will check if the instance is created then you will decide if you will create an instance . 

#### Compound actions 
Operations A and B are atomic with respect to each other if, from the perspective of a thread executing A, when another thread executes B, either all of B has executed or none of it has. An atomic operation is one that is atomic with respect to all operations, including itself, that operate on the same state.  Where practical, use existing thread-safe objects, like AtomicLong, to manage your class’s state. It is simpler to reason about the possible states and state transitions for existing thread-safe objects than it is for arbitrary state variables, and this makes it easier to maintain and verify thread safety.

### Lock 
To preserve state consistency, update related state variables in a single atomic operation.

#### Intrinsic locks
Java provides a built-in locking mechanism for enforcing atomicity: the synchronized block. A synchronized block has two parts: a reference to an object that will serve as the lock, and a block of code to be guarded by that lock. Every Java object can implicitly act as a lock for purposes of synchronization; these built-in locks are called `intrinsic locks or monitor locks`. The lock is automatically acquired by the executing thread before entering a synchronized block and automatically released when control exits the synchronized block, whether by the normal control path or by throwing an exception out of the block. The only way to acquire an intrinsic lock is to enter a synchronized block or method guarded by that lock. Intrinsic locks in Java act as mutexes (or mutual exclusion locks), which means that at most one thread may own the lock. When thread A attempts to acquire a lock held by thread B, A must wait, or block, until B releases it. If B never releases the lock, A waits forever.

#### Reentrancy
When a thread requests a lock that is already held by another thread, the requesting thread blocks. But because `intrinsic locks are reentrant`, if a thread tries to acquire a lock that it already holds, the request succeeds. Reentrancy means that locks are acquired on a per-thread rather than per-invocation basis.7 Reentrancy is implemented by associating with each lock an acquisition count and an owning thread. When the count is zero, the lock is considered unheld. When a thread acquires a previously unheld lock, the JVM records the owner and sets the acquisition count to one. If that same thread acquires the lock again, the count is incremented, and when the owning thread exits the synchronized block, the count is decremented. When the count reaches zero, the lock is released.
Reentrancy facilitates encapsulation of locking behavior, and thus simplifies the development of object-oriented concurrent code. Without reentrant locks, the very natural-looking code in the example below, in which a subclass overrides a synchronized method and then calls the superclass method, would deadlock. Because the work methods in Father and Son are both synchronized, each tries to acquire the lock on the Father before proceeding. But if intrinsic locks were not reentrant, the call to super.doSomething would never be able to acquire the lock because it would be considered already held, and the thread would permanently stall waiting for a lock it can never acquire. Reentrancy savesus from deadlock in situations like this.
```java
public class Father {
  public synchronized void work(){
    ...
  }
}
public class son extends Father{
  public synchroniezd void work(){
    super.work();
  }
}
```
### Guarding state with locks 
Compound actions on shared state, such as incrementing a hit counter (readmodifywrite) or lazy initialization (check-then-act), must be made atomic to avoid race conditions. Holding a lock for the entire duration of a compound action can make that compound action atomic. However, just wrapping the compound action with a synchronized block is not sufficient; if synchronization is used to coordinate access to a variable, it is needed everywhere that variable is accessed. Further, when using locks to coordinate access to a variable, the same lock must be used wherever that variable is accessed. For each mutable state variable that may be accessed by more than one thread, all accesses to that variable must be performed with the same lock held. In this case, we say that the variable is guarded by that lock.`Every shared, mutable variable should be guarded by exactly one lock.Make it clear to maintainers which lock that is. For every invariant that involves more than one variable, all the variables involved in that invariant must be guarded by the same lock.`

### Liveness and performance
```java
@ThreadSafe
public class CachedFactorizer implements Servlet {
	@GuardedBy("this") private BigInteger lastNumber;
	@GuardedBy("this") private BigInteger[] lastFactors;
	@GuardedBy("this") private long hits;
	@GuardedBy("this") private long cacheHits;
	public synchronized long getHits() { return hits; }
	public synchronized double getCacheHitRatio() {
		return (double) cacheHits / (double) hits;
	}
	public void service(ServletRequest req, ServletResponse resp) {
		BigInteger i = extractFromRequest(req);
		BigInteger[] factors = null;
		synchronized (this) {
		++hits;
		if (i.equals(lastNumber)) {
			++cacheHits;
			factors = lastFactors.clone();
		}
		}
		if (factors == null) {
			factors = factor(i);
			synchronized (this) {
			lastNumber = i;
			lastFactors = factors.clone();
		}
		}
		encodeIntoResponse(resp, factors);
	}
}
```
CachedFactorizer no longer uses AtomicLong for the hit counter, instead reverting to using a long field. It would be safe to use AtomicLong here, but there is less benefit than there was in CountingFactorizer. Atomic variables are useful for effecting atomic operations on a single variable, but since we are already using synchronized blocks to construct atomic operations, using two different synchronization mechanisms would be confusing and would offer no performance or safety benefit.
The restructuring of CachedFactorizer provides a balance between simplicity(synchronizing the entire method) and concurrency (synchronizing the shortest possible code paths).
Avoid holding locks during lengthy computations or operations at risk of not completing quickly such as network or console I/O.

### Sharing Objects 
We have seen how synchronized blocks and methods can ensure that operations execute atomically, but it is a common misconception that synchronized is only about atomicity or demarcating “critical sections”. Synchronization also has another significant, and subtle, aspect: `memory visibility`. We want not only to prevent one thread from modifying the state of an object when another is using it, but also to ensure that when a thread modifies the state of an object, other threads can actually see the changes that were made. But without synchronization, this may not happen. You can ensure that objects are published safely either by using explicit synchronization or by taking advantage of the synchronization built into library classes.

#### Visibility
In a single-threaded environment, if you write a value to a variable and later read that variable with no intervening writes, you can expect to get the same value back. This seems only natural. It may be hard to accept at first, but when the reads and writes occur in different threads, this is simply not the case. In general, there is no guarantee that the reading thread will see a value written by another thread on a timely basis, or even at all. In order to ensure visibility of memory writes across threads, you must use synchronization.
```java
public class NoVisibility{
  private static boolean ready ;
  private static int num;
  static class Reader extends Thread{
    public void run() {
      try {
	while (!ready) {
	  Thread.yield();
	}
	System.out.println(num);
      } catch (Exception ex) {
	ex.printStackTrace();
      }
    }
  }
  public static void main(String[] args){
    new Reader().start();
    num = 41;
    ready = true;
  }
}
```
the result may be 0 , 41 or even never stop spining . `This may seem like a broken design, but it is meant to allow JVMs to take full advantage of the performance of modern multiprocessor hardware. For example, in the absence of synchronization,the Java Memory Model permits the compiler to reorder operations and cache values in registers, and permits CPUs to reorder operations and cache values in processor-specific caches.`
```java
@NotThreadSafe
public class MutableInteger{
  private int value;
  public int get(){return value;}
  public void set(int value){this.value = value;}
}
```
We can make MutableInteger thread safe by synchronizing the getter and setter. `Synchronizing only the setter would not be sufficient: threads calling get would still be able to see stale values`.

#### Nonatomic 64-bit operations
stale values are as we know from some thread . but if our value is a 64-bit type value , then the value may be strange , for the JVM is permitted to treat a 64-bit read or write as two seperate 32-bit operation . so if the read or write accour in two different threads then bad things happends. Thus, even if you don’t care about stale values, it is not safe to use shared mutable long and double variables in multithreaded programs unless they are declared volatile or guarded by a lock.

#### Volatile variables
The Java language also provides an alternative, weaker form of synchronization,volatile variables, to ensure that updates to a variable are propagated predictably to other threads. When a field is declared volatile, the compiler and runtime are put on notice that this variable is shared and that operations on it should not be reordered with other memory operations. Volatile variables are not cached in registers or in caches where they are hidden from other processors, so a read of a volatile variable always returns the most recent write by any thread.accessing a volatile variable performs no locking and so cannot cause the executing thread to block, making volatile variables a lighter-weight synchronization mechanism than synchronized.

When thread A writes to a volatile variable and subsequently thread B reads that same variable, the values of all variables that were visible to A prior to writing to the volatile variable become visible to B after reading the volatile variable. So from a memory visibility perspective, writing a volatile variable is like exiting a synchronized block and reading a volatile
variable is like entering a synchronized block. However, we do not recommend relying too heavily on volatile variables for visibility; code that relies on volatile variables for visibility of arbitrary state is more fragile and harder to understand
than code that uses locking.

Use volatile variables only when they simplify implementing and verifying your synchronization policy; avoid using volatile variables when veryfing correctness would require subtle reasoning about visibility. Good uses of volatile variables include ensuring the visibility of their own state, that of the object they refer to, or indicating that an important lifecycle event (such as initialization or shutdown) has occurred.Volatile variables are convenient, but they have limitations. The most common use for volatile variables is as a completion, interruption, or status flag, such as the asleep flag.

`Locking can guarantee both visibility and atomicity; volatile variables can only guarantee visibility.`

You can use volatile variables only when all the following criteria are met:
* Writes to the variable do not depend on its current value, or you can ensure that only a single thread ever updates the value;
* The variable does not participate in invariants with other state variables;
* Locking is not required for any other reason while the variable is being accessed.

### Publication and escape 
Publishing an object means making it available to code outside of its current scope.An object that is published when it should not have been is said to have escaped.
```java
class UnsafeStates{
  private String[] states = new String[]{...};
  public String[] getStates(){status;}
}
```
Publishing states in this way is problematic because any caller can modify its contents. In this case, the states array has escaped its intended scope, because what was supposed to be private state has been effectively made public.
```java
public class ThisEscape{
  // constructor 
  public ThisEscape(EventSource source){
    source.registerListener(
      new EventListener(){
        public void onEvent(Event e){
	  doSomething(e); // escape this 
	}
      }
    );
  }
  public void doSomething(Event e){}
}
```
When ThisEscape publishes the EventListener, it implicitly publishes the enclosing ThisEscape instance as well, because inner class instances contain a hidden reference to the enclosing instance.`Do not allow the this reference to escape during construction.` A common mistake that can let the this reference escape during construction is to start a thread from a constructor. for the class may not fully initialized . 

### Thread Confinement
If data is only accessed from a single thread, no synchronization is needed. This technique, thread confinement, is one of the
simplest ways to achieve thread safety.

#### Stack confinement
Stack confinement is a special case of thread confinement in which an object can only be reached through local variables.

#### ThreadLocal
ThreadLocal, which allows you to associate a per-thread value with a value-holding object. Thread Local provides get and set accessor methods that maintain a separate copy of the value for each thread that uses it, so a get returns the most recent value passed to set from the currently executing thread.
If you are porting a single-threaded application to a multithreaded environment,you can preserve thread safety by converting shared global variables into ThreadLocals, if the semantics of the shared globals permits this; an applicationwide cache would not be as useful if it were turned into a number of thread-local caches.
ThreadLocal is widely used in implementing application frameworks. For example, J2EE containers associate a transaction context with an executing thread for the duration of an EJB call. This is easily implemented using a static Thread-
Local holding the transaction context: when framework code needs to determine what transaction is currently running, it fetches the transaction context from this is convenient in that it reduces the need to pass execution context information into every method, but couples any code that uses this mechanism to the framework.

### Immutability
Immutable objects are always thread-safe.
An object is immutable if:
* Its state cannot be modified after construction
* All its fields are final
* It is properly constructed (the this reference does not escape during construction).

### Safe Publication 
simply storing a reference to an object into a public field like the code below is not enough to publish that object safely .
```java
@NotSafety
public Holder holder;

public void initial(){
  holder = new Holder(10);
}
```
for visibility problems , the holder could appear to another thread to be in a inconsistent state , this improper publication could allow another thread to observe `a partially constructed object`.
to publish an object safety , both the reference to the object and the object's state must be made visible to other threads at the same time . A properly constructed object can be safely published by (not consider visibility):
* initialzing an Object reference from a static initialzer ; 
* Storing a reference to it into a volatile field or AtomicReference ;
* Storing a reference to it into a final field of a properly constructed object;
* Storing a reference to it into a field that is properly guarded by a lock;
using a static initializer is often the easiest and safest way to publish objects that can be statically constructed ; 
```java
public static Holder holder = new Holder(32);
```
Static Initializers are executed by the JVM at class initialization time ;because of internal synchronization in the JVM, this mechanism is guaranteed to safely publish any objects initiailzed in this way . 

### Effectively immutable objects 
Objects that are not technically immutable , but whose state will not be modified after publication , are called effectively immutable.they are treated like an immutable object . using effectively immutable objects can simply development and improve performance by reducing the need for synchronization . 

The publication requirement for an object depend on its mutability:
* Immutable objects can be published through any mechanism;
* Effectively immutable objects must be safely published ;
* Mutable Objects must be safely published , and must be either thread safe or guared by a lock .

The Most useful policies for using and sharing objects in a concurrency program are : 
* Thread-confined: object that exclusive belong to current thread . 
* Share read-only : a read only object can be access concurrently by multiply thread without synchronization . 
* Share thread-safe : a published object that is synchronized internally so multiple can access it without synchronization.so  threads using them without need further snchronization . 
* Guard : a guared object can only be accessed by owning specific lock held.

### Composing Objects 
we want to be able to take thread-safe components and safely compose them into larger components or programs .

#### Designing a thread-safe class
The design process for a thread-safe class should include these three basic element: 
* Identify the variables that form the object's state . 
* Identify the invariables that constrain the state variables . 
* Establish a policy for managing concurrency access to the object's state . 

1. you cannot ensure thread safety without understanding an object's invariants and postconditions . Constraints on the valid values or state transitions for state variables can create atomicity and encapulation requirements.
2. Some Objects also have methods with state-based `precondition`. for example , you cannot remove an item from an empty queue;a queue must be in the "nonempty" state before you can remove an element . Operations with state-based preconditions are called `state dependent`. 
3. state Ownership , for better or worse , garbage collection lets us avoid thinking carefully about `ownership` . when passing an object to a method in C++,you have to think fairly carefully about whether you are transforing ownership . engaging in a short-term loan, or envisioning long-term joint ownership .Ownership implies control,but once you publish a reference to a mutable object, you no longer have exclusive control, at best you might have shared ownership .

#### Instance confinement -- Java monitor pattern 
An object following the Java monitor pattern encapsulates all its mutable state and guards it with the object's own intrisic lock. 
```java
@ThreadSafe
public final class Counter {
	@GuardedBy("this") private long value = 0;
	public synchronized long getValue() {
		return value;
	}
	public synchronized long increment() {
		if (value == Long.MAX_VALUE)
		throw new IllegalStateException("counter overflow");
		return ++value;
	}
}
```
#### Instance confinement -- private lock .
```java
public class PrivateLock {
  private final Object lock = new Object();
  @GuardedBy("lock") Widget widget ;
  void dosomething(){
    synchronized(lock){
      // ... access widget 
    }
  }
}
```
There are advantages to using a private lock object instead of an object’s intrinsic lock (or any other publicly accessible lock). Making the lock object private encapsulates the lock so that client code cannot acquire it, whereas a publicly accessible lock allows client code to participate in its synchronization policy correctly or incorrectly.

#### examples 
```java
@ThreadSafe
public class MonitorVehicleTracker {
	@GuardedBy("this")
	private final Map<String, MutablePoint> locations;

	public MonitorVehicleTracker(Map<String, MutablePoint> locations) {
		this.locations = deepCopy(locations);
	}

	public synchronized Map<String, MutablePoint> getLocations() {
		return deepCopy(locations);
	}

	public synchronized MutablePoint getLocation(String id) {
		MutablePoint loc = locations.get(id);
		return loc == null ? null : new MutablePoint(loc);
	}

	public synchronized void setLocation(String id, int x, int y) {
		MutablePoint loc = locations.get(id);
		if (loc == null)
			throw new IllegalArgumentException("No such ID: " + id);
		loc.x = x;
		loc.y = y;
	}

	private static Map<String, MutablePoint> deepCopy(Map<String, MutablePoint> m) {
		Map<String, MutablePoint> result = new HashMap<String, MutablePoint>();
		for (String id : m.keySet())
			result.put(id, new MutablePoint(m.get(id)));
		return Collections.unmodifiableMap(result);
	}
}
@NotThreadSafe
public class MutablePoint {
	public int x, y;

	public MutablePoint() {
		x = 0;
		y = 0;
	}

	public MutablePoint(MutablePoint p) {
		this.x = p.x;
		this.y = p.y;
	}
}
```
this example copy the passing location for the location is changable so we can not trust it . we can make a copy of it .so if the original changed there will not have any bad influence on this class . 
```java
@Immutable
public class Point {
	public final int x, y;

	public Point(int x, int y) {
		this.x = x;
		this.y = y;
	}
}
```
instead we can use an Immutable class then we do not need to deepCopy .

#### Adding functionality to existing thread-safe classes
sometimes we need to add some operation to a class ? here is how we can do it.

1. If you can modify the original class, you need to understand the implementation’s synchronization policy(which means how this class maintains its thread safety . synchronization ? lock ? confinement ? valotile ?) so that you can enhance it in a manner consistent with its original design. Adding the new method directly to the class means that all the code that implements the synchronization policy for that class is still contained in one source file, facilitating easier comprehension and maintenance.
2. another way is to extend the class, assuming it was designed for extension . this approach is more fragile than adding code directly to a class . for the implementation of the synchronization policy is now distributed over multiple seperately maintained source files. if the underly class changed its synchronation policy then the class state break . 
3. a third stratege is to extend the functionality of the class without extending the class itself by placing extension code in a "helper" class . 
```java
@NotThreadSafe
public class ListHelper<E>{
	public List<E> list = Collections.synchronizedList(new ArrayList<E>());
	//...
	public synchronized boolean putIfAbsence(E x){
		boolean absent = contains(x);
		if(!absent)
			add(x);
		return absent;	
	}
}
```
this would not be thread safe because it synchronized the wrong lock . Whatever lock the List uses to guard its state, it sure isn’t the lock on the ListHelper. we can change the code by add a client side lock . 
```java
@NotThreadSafe
public class ListHelper<E>{
	public List<E> list = Collections.synchronizedList(new ArrayList<E>());
	//...
	public  boolean putIfAbsence(E x){
		synchronized(list){
			boolean absent = list.contains(x);
			if(!absent)
				list.add(x);
		}
		return absent;
	}
}
```
this kind of lock is also fragile for it is very like the second approach. and you add a new lock out of nowhere clearly not very concern the orginal's synchronization policy .

#### Documentation synchronization policies 
Document a class’s thread safety guarantees for its clients; document its synchronization policy for its maintainers.
Crafting a synchronization policy requires a number of decisions: which variables to make volatile, which variables to guard with locks, which lock(s) guard which variables, which variables to make immutable or confine to a thread, which operations must be atomic, etc. Some of these are strictly implementation details and should be documented for the sake of future maintainers, but some affect the publicly observable locking behavior of your class and should be documented as part of its specification.
At the very least, document the thread safety guarantees made by a class. Is it thread-safe? Does it make callbacks with a lock held? Are there any specific locks that affect its behavior? Don’t force clients to make risky guesses. If you don’t want to commit to supporting client-side locking, that’s fine, but say so. If you want clients to be able to create new atomic operations on your class, you need to document which locks they should acquire to do so safely. If you use locks to guard state, document this for future maintainers, because it’s so easy—the @GuardedBy annotation will do the trick. If you use more
subtle means to maintain thread safety, document them because they may not be obvious to maintainers.

without the specific documentation about concurrency policy how we assuming the API class safety issue . `By semantic And Google`

### Building Blocks
#### Synchronization Collections 
1. The synchronized collection classes include Vector and Hashtable, part of the original JDK, as well as their cousins added in JDK 1.2, the synchronized wrapper classes created by the Collections.synchronizedXxx factory methods. These classes achieve thread safety by encapsulating their state and synchronizing every public method so that only one thread at a time can access the collection state.
2. The synchronized collections are thread-safe, but you may sometimes need to use additional client-side locking to guard compound actions. Common compound actions on collections include iteration (repeatedly fetch elements until the collection is exhausted), navigation (find the next element after this one according to some order), and conditional operations such as put-if-absent (check if a Map has a mapping for key K, and if not, add the mapping (K,V)). With a synchronized collection, these compound actions are still technically thread-safe even without client-side locking, but they may not behave as you might expect when other threads can concurrently modify the collection. or will throw ConcurrentModifyException if the value has been changed during iteration . 

#### Concurrent Collections 
Replacing synchronized collections with concurrent collections can offer dramatic scalability improvements with little risk. such as ConcurrentHashMap , Multiple thread can manipulate this map simultaneously , without worry about thread safety . they use different synchronized policy compare to synchronized collection . 

Queue and Blocking Queue is intented to hold a set of elements temporarily while they wait processing . Queue's operation is not blocked , while BLocking Queue does . if Queue is empty , the retrieval operation return null . if it is a block queue then it will block until it is not empty . if it is full when some thread want to add element in it will block too until the queue is not full . 

* ConcurrentLinkedQueue : a traditinoal Concurrent FIFO queue. 
* Priority Queue : a none concurrent priority ordered queue.
* ConcurrentSkipListMap is a concurrent replacement of synchronized SortedMap.
* ConcurrentSkitListSet is a concurrent replacement of synchronized SortedSet.
* CopyOnWriteArrayList is a concurrent replacement of synchronized List.
* copyOnWriteArratySet is a concurrent replacement of synchronized set. 

The copy-on-write collections derive their thread safety from the fact that as long as an effectively immutable object is properly published, no further synchronization is required when accessing it. They implement mutability by creating and republishing a new copy of the collection every time it is modified. Iterators for the copy-on-write collections retain a reference to the backing array that was current at the start of iteration, and since this will never change.`the copy-on-write collections are reasonable to use only when iteration is far more common than modification`.

### Blocking queues and the producer-consumer pattern
In a producer-consumer design built around a blocking queue, producers place data onto the queue as it becomes available, and consumers retrieve data from the queue when they are ready to take the appropriate action. Producers don’t need to know anything about the identity or number of consumers, or even whether they are the only producer—all they have to do is place data items on the queue. Similarly, consumers need not know who the producers are or where the work came from. BlockingQueue simplifies the implementation of producerconsumer designs with any number of producers and consumers.
this kind of machanizm can be used in a lot of scenes . such as dish washing , webcrawing and so on . as long as the manipulation can be seperated into two threads and may be one thread never need to be shutdown and it depended on the next thread.  we can try producer and consumer pattern . 

### Deques and work stealing
Java 6 also adds another two collection types, Deque (pronounced “deck”) and BlockingDeque, that extend Queue and BlockingQueue. A Deque is a doubleended queue that allows efficient insertion and removal from both the head and the tail. Implementations include ArrayDeque and LinkedBlockingDeque. Just as blocking queues lend themselves to the producer-consumer pattern,deques lend themselves to a related pattern called work stealing. A producerconsumer design has one shared work queue for all consumers; in a work stealing design, every consumer has its own deque. If a consumer exhausts the work in its own deque, it can steal work from the tail of someone else’s deque. Work stealing can be more scalable than a traditional producer-consumer design because workers don’t contend for a shared work queue; most of the time they access only their
own deque, reducing contention. When a worker has to access another’s queue, it does so from the tail rather than the head, further reducing contention. Work stealing is well suited to problems in which consumers are also producers when performing a unit of work is likely to result in the identification of more work. For example, processing a page in a web crawler usually results in the identification of new pages to be crawled. Similarly, many graph-exploring algorithms, such as marking the heap during garbage collection, can be efficiently parallelized using work stealing. When a worker identifies a new unit of work, it places it at the end of its own deque (or alternatively, in a work sharing design, on that of another worker); when its deque is empty, it looks for work at the end of someone else’s deque, ensuring that each worker stays busy.

### Blocking and interruptible methods
When your code calls a method that throws InterruptedException, then your method is a blocking method too, and must have a plan for responding to interruption. For library code, there are basically two choices:
* Propagate the InterruptedException.  Throw the exception out . 
* Restore the interrupt. catch the exception . restore the interupted status . 
```java
public class TaskRunnable implements Runnable {
BlockingQueue<Task> queue;
...java
public void run() {
		try {
			processTask(queue.take());
		} catch (InterruptedException e) {
			// restore interrupted status
			Thread.currentThread().interrupt();
		}
	}
}
```

### Synchronizers 
A synchronizer is any object that coordinates the control flow of threads based on its state. Blocking queues can act as synchronizers; other types of synchronizers include semaphores, barriers, and latches.
All synchronizers share certain structural properties: they encapsulate state that determines whether threads arriving at the synchronizer should be allowed to pass or forced to wait, provide methods to manipulate that state, and provide methods to wait efficiently for the synchronizer to enter the desired state.

#### Latches
A latch acts as a gate: until the latch reaches the terminal state the gate is closed and no thread can pass, and in the terminal state the gate opens, allowing all threads to pass. Once the latch reaches the terminal state, it cannot change state again, so it remains open forever. Latches can be used to ensure that certain activities do not proceed until other one-time
activities complete, such as:
* Ensuring that a computation does not proceed until resources it needs have been initialized. A simple binary (two-state) latch could be used to indicate “Resource R has been initialized”, and any activity that requires R would wait first on this latch.
* Ensuring that a service does not start until other services on which it depends have started. Each service would have an associated binary latch; starting service S would involve first waiting on the latches for other services on which S depends, and then releasing the S latch after startup completes so any services that depend on S can then proceed.
* Waiting until all the parties involved in an activity, for instance the players in a multi-player game, are ready to proceed. In this case, the latch reaches the terminal state after all the players are ready.
```java
public static void main(String[] args) {
	final CountDownLatch startLatch = new CountDownLatch(1);
	final CountDownLatch endLatch = new CountDownLatch(1);
	new Thread(new Runnable() {
		public void run() {
			try {
				startLatch.await();
				for(int i=0;i<100;i++){
					Thread.sleep(10);
				}
				endLatch.countDown();
			} catch (InterruptedException e) {
				e.printStackTrace();
				Thread.currentThread().interrupt();
			}
		}
	}).start();;

	try {
		Thread.sleep(1000);
		long start = System.currentTimeMillis();
		startLatch.countDown();
		endLatch.await();
		long end = System.currentTimeMillis();
		System.out.println(end - start);
	} catch (InterruptedException e) {
		e.printStackTrace();
	}
}
```

#### FutureTask
A computation represented by a FutureTask is implemented with a Callable, the result-bearing equivalent of Runnable, and can be in one of three states: waiting to run, running, or completed. Completion subsumes all the ways a computation can complete,
including normal completion, cancellation, and exception. Once a FutureTask enters the completed state, it stays in that state forever.The behavior of Future.get depends on the state of the task. If it is completed, get returns the result immediately, and otherwise blocks until the task transitions to the completed state and then returns the result or throws an exception.
```java
private static FutureTask<String> future = new FutureTask<String>(new Callable<String>() {
	@Override
	public String call() throws Exception {
		return "A string for future task";
	}
});

private final static Thread t = new Thread(future);

public static void start(){
	t.start();
}

public static void main(String[] args) throws InterruptedException{
	try {
		new Thread(new Runnable() {
			public void run() {
				try {
					Thread.sleep(1000);
					start();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}).start();
		System.out.println(future.get());
	} catch (ExecutionException e) {
		e.printStackTrace();
	}
}
```
#### Semaphores
A semaphores manages a set of virtual permits . the initial number of permits is passed to semaphores through the constructor .Activities can aquire permits(as long as they are some left), and release them when they are done with it . If no permit is available , acquire blocks until one is release the permit or interupted , or timeout . 
Semaphores are useful for implementing resource pools such as database connection pools. While it is easy to construct a fixed-sized pool that fails if you request a resource from an empty pool, what you really want is to block if the pool is empty and unblock when it becomes nonempty again.
Similar you can use a semaphores turn any kind of collection  into a blocking bounded collection . 
```java
public class BoundedHashSet<T> {
	private final Set<T> set;
	private final Semaphore sem;

	public BoundedHashSet(int bound) {
		this.set = Collections.synchronizedSet(new HashSet<T>());
		sem = new Semaphore(bound);
	}

	public boolean add(T o) throws InterruptedException {
		sem.acquire();
		boolean wasAdded = false;
		try {
			wasAdded = set.add(o);
			return wasAdded;
		} finally {
			if (!wasAdded)
				sem.release();
		}
	}

	public boolean remove(Object o) {
		boolean wasRemoved = set.remove(o);
		if (wasRemoved)
			sem.release();
		return wasRemoved;
	}
}
```
#### Barriers 
Barriers are similar to latches in that they block a group of threads until some event has occurred . The key difference is that with a barrier, all the threads must come together at a barrier point at the same time in order to proceed. Latches are for waiting for events; barriers are for waiting for other threads.

CyclicBarrier allows a fixed number of parties to rendezvous repeatedly at a barrier point and is useful in parallel iterative algorithms that break down a problem into a fixed number of independent subproblems. Threads call await when they reach the barrier point, and await blocks until all the threads have reached the barrier point. If all threads meet at the barrier point, the barrier has been successfully passed, in which case all threads are released and the barrier is reset so it can be used again. If a call to await times out or a thread blocked in await is interrupted, then the barrier is considered broken and all outstanding calls to await terminate with BrokenBarrierException.

Another form of barrier is Exchanger, a two-party barrier in which the parties exchange data at the barrier point Exchangers are useful when the parties perform asymmetric activities, for example when one thread fills a buffer with data and the other thread consumes the data from the buffer; these threads could use an Exchanger to meet and exchange a full buffer for an empty one. When two threads exchange objects via an Exchanger, the exchange constitutes a safe publication of both objects to the other party . 

### Build a perfect cache 
```java
public class Memoizer<A, V> implements Computable<A, V> {
	private final ConcurrentMap<A, Future<V>> cache = new ConcurrentHashMap<A, Future<V>>();
	private final Computable<A, V> c;

	public Memoizer(Computable<A, V> c) {
		this.c = c;
	}

	public V compute(final A arg) throws InterruptedException {
		while (true) {
			Future<V> f = cache.get(arg);
			if (f == null) {
				Callable<V> eval = new Callable<V>() {
					public V call() throws InterruptedException {
						return c.compute(arg);
					}
				};
				FutureTask<V> ft = new FutureTask<V>(eval);
				f = cache.putIfAbsent(arg, ft);
				if (f == null) {
					f = ft;
					ft.run();
				}
			}
			try {
				return f.get();
			} catch (CancellationException e) {
				cache.remove(arg, f);
			} catch (ExecutionException e) {
				throw launderThrowable(e.getCause());
			}
		}
	}
}
```
using a Future to put the long computation in the back ground. 
