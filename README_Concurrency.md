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
this is not a thread safe program.  as value++ has three steps ( read , add , write )
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

