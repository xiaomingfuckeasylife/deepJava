## Concurrency

###  A brief history of concurrency
  once before we had operation system , we can only execute one program each time which will access the whole resource of a mathine . nowadays , operation system can execute multiple program individually at the sametime using a thing called process . a process is an isolated , independent environment which is allocated by the system. process can comunicated with each other through `sockets` , `shared memory`,`semaphores`,`files`. 
  several motivations factors led to the development of operation system that execute task simultaneously . 
  * Resource utilization : sometimes a program will wait for some extra input or output , in these time , nothing valuable thing has been done . we can put other program to run at this time . 
  * Fairness : there are more than one user using the mathine then it is only fair that they using the resource by time slicing .
  * convinience : it is much more desirable to write multiple program and each program executed one task than one program perform all the task . 
  
### Benefits of threads 
  threads can reduce development and maintenance costs and improve the performance of complex applications. Threads make it easier to model how humans work and interact , by turning asynchonous workflows into mostly sequential ones . 
  * one of the most obvious benefits is Exploiting the multi-process cpu . 
  * simplicify modeling. sometimes the problem is just too complicated then when you think as a whole problem , you have no idea which way to think , but when you cut the problem into pieces and put every piece task into different thread . it gets easier. 
  * simplified handling of asynchronous events . when we using a single-thread events loop , when one user blocked , then all other user blocked . so the best way to create a usable and stable program is to give each user with their own thread . 
 
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
this is not a thread safe program.  as value++ has three steps ( read , add , write )
[unsafeSequence](https://i.imgsafe.org/58a3c9c6d1.png)
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
