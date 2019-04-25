# 并发、进程与线程初探

### 并发

两个或者更多的任务（独立的活动）同时发生（进行）。计算机中指一个程序同时执行多个独立的任务。

单核CPU计算机中，某一个时刻只能执行单个任务，由操作系统调度进行多次任务切换来实现并发。这种切换也叫做上下文切换，需要一定的时间开销，切换越多，开销越大。这种开销主要发生在操作系统切换时保存程序的运行状态上。

多核（多个）CPU计算机中，通过硬件并发可以实现真正意义上的并行执行多个任务。

使用并发的目的在于可以同时完成多个任务，提升程序的性能。

### 进程

将一个可执行程序运行起来就创建了一个进程，通俗地讲：进程就是运行起来了的可执行程序。

### 线程

每个进程都有一个主线程，并且这个主线程是唯一的。

执行一个可执行程序时，一个进程就被创建了，同时，对应着的主线程也自动启动起来。

运行程序时，是进程中的线程来执行程序中的代码。主线程结束时，对应的进程也就结束了。

每个进程中至少包含一个线程（主线程）。

线程：用来执行程序代码的。

进程中，除了主线程，还可以使用程序代码来创建新的线程（非主线程）用于执行与主线程不同的其他代码。每创建一个线程，相当于就可以在同一时间多执行一个任务。

线程并非越多越好，每个线程都需要一个独立的堆栈空间，线程之间的切换同样需要保存很多中间状态，切换过程会耗费本该程序运行的时间。

### 并发的实现

- 多进程并发：通过多个进程实现并发。涉及到进程间通信的知识（管道、文件、消息队列、共享内存、socket通讯）。
- 多线程并发：在单独的进程中，创建多个线程来实现并发。同一进程中的所有线程共享进程的地址空间。

因为共享内存的存在，使用多线程并发的开销远远小于使用多进程并发，但是多线程并发时需要考虑到内存数据的一致性问题。