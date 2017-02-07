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

