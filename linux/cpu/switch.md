# cpu 上下文切换

## 概念

* CPU任务运行借助CPU寄存器、程序计数器；
    * 寄存器：cpu的内容
    * 程序计数器：存储正在运行的指令以及下一条指令的位置；

* CPU上下文切换
    * 前一个任务的 CPU 上下文（也就是 CPU 寄存器和程序计数器）保存起来，然后加载新任务的上下文到这些寄存器和程序计数器，最后再跳转到程序计数器所指的新位置，运行新任务；
* 分类：
    * 进程上下文切换
    * 线程上下文切换
    * 中断上下文切换

* 进程既可以在用户空间运行，又可以在内核空间中运行。进程在用户空间运行时，被称为进程的用户态，而陷入内核空间的时候，被称为进程的内核态。

* 系统调用(特权模式切换)：
    * 用户态 --> 内核态；
    * 不会涉及到虚拟内存等进程用户态的资源，也不会切换进程；
    * 始终是一个进程运行；

* 进程上下文切换：
    * 进程切换；

## 进程上下文切换
* 弊端：
    导致 CPU 将大量 时间耗费在寄存器、内核栈以及虚拟内存等资源的保存和恢复上，进而大大缩短了真正运 行进程的时间；
* 进程什么时候被调度？
    * CPU时间片；
    * 系统资源不足；
    * 进程调用sleep函数；
    * 高优先级进程；

## 线程上下文切换
* 进程和线程的区别：
    * 进程 -- 资源管理的基本单位；
    * 线程 -- 调度的基本单位；
    * 内核中的任务调度，实际上的调度对象是线程；而进程只是给线程提供了虚 拟内存、全局变量等资源
    * 当进程只有一个线程时，可以认为进程就等于线程。
    * 当进程拥有多个线程时，这些线程会共享相同的虚拟内存和全局变量等资源。这些资源 在上下文切换时是不需要修改的。
    * 线程也有自己的私有数据，比如栈和寄存器等，这些在上下文切换时也是需要保 存的。

* 切换本质：
    * 不同进程的两个线程切换 == 进程切换
    * 不同 == 虚拟内存共享，只切换线程私有数据；

## 中断上下文切换
> 响应硬件事件；


## 查看系统上下文切换
*  vmstat
> 整体


``` yaml
# vmstat 5w
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu----- -----timestamp-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st                 CST
 1  0      0 138404 137200 840176    0    0    18    24    0    0 15  0 85  0  0 2019-11-25 20:07:24

# System
# in: The number of interrupts per second, including the clock.
# cs: The number of context switches per second.

# procs
# r: The number of processes waiting for run time.
# b: The number of processes in uninterruptible sleep.
```


## pidstat
# pidstat -w 5

``` yaml
Linux 3.10.0-693.17.1.el7.x86_64 (vm172-31-0-21.ksc.com) 	2019年12月11日 	_x86_64_	(2 CPU)

20时45分54秒   UID       PID   cswch/s nvcswch/s  Command
20时45分59秒     0         1      0.80      0.00  systemd
20时45分59秒     0         9     14.77      0.00  rcu_sched
20时45分59秒     0        10      0.40      0.00  watchdog/0
20时45分59秒     0        11      0.40      0.00  watchdog/1
20时45分59秒     0        12      0.20      0.00  migration/1
20时45分59秒     0       133      1.20      0.00  kauditd

# cswch: 每秒自愿上下文切换 （voluntary context switches）的次数 -- 资源问题(I/O、mem)
# nvcswch: 表示每秒非自愿上下文 切换（non voluntary context switches）的次数。(时间片到期、大量进程争抢)
```

## /proc/interrupts
中断次数过多，分析具体类型；