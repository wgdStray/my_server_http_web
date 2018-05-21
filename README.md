# my_server_http_web

using c/c++ to build a program for http server in Linux

2018.5.21

基于Linux下多线程的web服务器

1.使用语言：c/c++

2.项目介绍：

    1）采用典型的c/s服务器框架结构
    
    2）采用非阻塞I/O模型，同时通过erno值判断读取情况
    
    3）采用Reactor事件处理模式：
    
       i）主线程只负责监听文件描述上是否有事件发生，有的话就立即将该事件通知工作线程。除此之外，主线程不做任何其他实质性的工作。工作流程如下（epoll为例）：
       
      ii）主线程往epoll内核事件表注册socket上的读就绪事件。
      
     iii）主线程调用epoll_wait等待socket上有数据可读
     
      iv）当socket上有数据可读，epoll_wait通知主线程。主线程则将socket可读事件放入请求队列。
      
       v）睡眠在请求队列上某个工作线程被唤醒，它从socket读取数据，并处理客户请求，然后往epoll内核事件表中注册该socket上的写就绪事件
       
      vi）主线程调用epoll_wait等待socket可写。
     
     vii）当socket上有数据可写，epoll_wait通知主线程。主线程则将socket可写事件放入请求队列。
     
    viii）睡眠在请求队列上某个工作线程被唤醒，它从socket写入服务器处理客户请求的结果。
    
   4）由（3）可知采用了主流的epoll，其中为了提高效率，采用了ET工作模式，同时由于一个socket事件还是可能被触发多次（并发程序中，多个线程处理同一个socket），对于注册了EPOLLONESHOT事件的文件描述符，操作系统最多触发其上注册的一个可读、可写或异常事件，且只触发一次，除非使用了epoll_ctl函数重置该文件描述符上注册的EPOLLONESHOT事件。
   
   5）为了节约系统开销，采用线程池，因此封装了一个线程池类，主要采用的是半同步/半反应堆的并发模式，因为使用一个工作队列可以完全解除了主线程和工作线程的耦合关系：主线程往工作队列中插入任务，工作线程通过竞争来获取并执行它。（这种方式必须保证请求无状态，因为同一个连接的请求可能会被不同工作线程处理）
   
   6）同时基于oop的设计思想，封装了一个锁类（包含互斥锁、信号量及条件变量）解决线程之间的竞争问题
   
   7）采用状态机的设计思想，按状态转移的方式设计了一个http的连接处理类
   
 3.效果
  
  目前采用了一个通用的基于epoll的多进程程序，创建指定的进程数，模拟指定数的客户端，然后同时连接上所写服务器，不断进行数据的交换，测试其效果，目前已测可以实现上万的并发连接数据交换。

下一步计划：
  针对非活跃连接的处理，由于非活跃连接占用了连接资源，严重影响服务器的性能，因此接下来设计一个服务器定时器，处理这种非活跃连接，释放连接资源。
