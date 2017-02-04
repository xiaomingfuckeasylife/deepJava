## Concurrency

####  A brief history of concurrency
  once before we had operation system , we can only execute one program each time which will access the whole resource of a mathine . nowadays , operation system can execute multiple program individually at the sametime using a thing called process . a process is an isolated , independent environment which is allocated by the system. process can comunicated with each other through `sockets` , `shared memory`,`semaphores`,`files`. 
  several motivations factors led to the development of operation system that execute task simultaneously . 
  * Resource utilization : sometimes a program will wait for some extra input or output , in these time , nothing valuable thing has been done . we can put other program to run at this time . 
  * Fairness : there are more than one user using the mathine then it is only fair that they using the resource by time slicing .
  * convinience : it is much more desirable to write multiple program and each program executed one task than one program perform all the task . 
  
#### Benefits of threads 
  threads can reduce development and maintenance costs and improve the performance of complex applications. Threads make it easier to model how humans work and interact , by turning asynchonous workflows into mostly sequential ones . 
  * one of the most obvious benefits is Exploiting the multi-process cpu . 
  * simplicify modeling. sometimes the problem is just too complicated then when you think as a whole problem , you have no idea which way to think , but when you cut the problem into pieces and put every piece task into different thread . it gets easier. 
  * simplified handling of asynchronous events . when we using a single-thread events loop , when one user blocked , then all other user blocked . so the best way to create a usable and stable program is to give each user with their own thread . 
 
#### Rists of threads 
