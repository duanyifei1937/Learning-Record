# memory如何工作?

## 概念
**内存**: 存储系统和应用程序的指令、数据、缓存等;
**物理内存**: (主存)，动态随机访问内存（DRAM), 内核才可以直接访问物理内存;
**虚拟内存**: Linux 内核给每个进程都提供了一个独立的虚拟地址空间，并且这个地址空间是连续的。
    * **内核空间**：
    * **用户空间**：
**内存映射**： 将虚拟内存地址映射到物理内存地址，通过`页表`记录映射关系；
**页表**： 存储在 `MMU`中;
**MMU**: CPU 的内存管理单元; 4KB页大小；
**TLB**: (Translation Lookaside Buffer，转译后备缓冲器),MMU 中页表的高速缓存;

## 分配过程 & 性能问题
当进程通过 malloc() 申请内存后，内存并不会立即分配，而是在首次访问时，才通过`缺页异常`陷入内核中分配内存。
当进程访问的虚拟地址在页表中查不到时，系统会产生一个`缺页异常`，进入内核空间分配物理内存、更新进程页表，最后再返回用户空间，恢复进程的运行;
通过减少进程的上下文切换，减少 TLB 的刷新次数，就可以提高 TLB 缓存的使用率，进而提高 CPU 的内存访问性能；

## 页表项过多问题：
    * 多级页表： Linux 用的正是四级页表来管理内存页
    * 大页： 

## 内存回收机制：
* 回收缓存： LRU
* 回收不常访问的内存，把不常用的内存 --> swap --> disk;(swap 换入问题，会导致内存访问性能下降)
    * 杀死进程(OOM): 

### OOM:
* 其实是内核的一种保护机制。它监控进程的内存使用情况，并且使用 oom_score 为每个进程的内存使用情况进行评分;
* 评分机制：
    *  一个进程消耗的内存越大，oom_score 就越大；
    * 一个进程运行占用的 CPU 越多，oom_score 就越小。
这样，进程的 oom_score 越大，代表消耗的内存越多，也就越容易被 OOM 杀死，从而可以更好保护系统。
    
当然，为了实际工作的需要，管理员可以通过 /proc 文件系统，手动设置进程的 oom_adj ，从而调整进程的 oom_score。oom_adj 的范围是 [-17, 15]，  
数值越大，表示进程越容易被 OOM 杀死；  
数值越小，表示进程越不容易被 OOM 杀死，其中 -17 表示禁止 OOM。  
比如用下面的命令，你就可以把 sshd 进程的 oom_adj 调小为 -16，这样， sshd 进程就不容易被 OOM 杀死。  
`echo -16 > /proc/$(pidof sshd)/oom_adj`

## 如何查看内存使用？
### free

``` yaml
# available 不仅包含未使用内存，还包括了可回收的缓存，所以一般会比未使用内存更大。
$ free
              total        used        free      shared  buff/cache   available
Mem:        8169348      263524     6875352         668     1030472     7611064
Swap:             0           0           0
# buffer: Memory used by kernel buffers (Buffers in /proc/meminfo)
# cache: Memory used by the page cache and slabs (Cached and Slab in /proc/meminfo)
```

### top

# 按下M切换到内存排序

``` yaml
$ top
...
KiB Mem :  8169348 total,  6871440 free,   267096 used,  1030812 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  7607492 avail Mem


  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
  430 root      19  -1  122360  35588  23748 S   0.0  0.4   0:32.17 systemd-journal
 1075 root      20   0  771860  22744  11368 S   0.0  0.3   0:38.89 snapd
 1048 root      20   0  170904  17292   9488 S   0.0  0.2   0:00.24 networkd-dispat
    1 root      20   0   78020   9156   6644 S   0.0  0.1   0:22.92 systemd
12376 azure     20   0   76632   7456   6420 S   0.0  0.1   0:00.01 systemd
12374 root      20   0  107984   7312   6304 S   0.0  0.1   0:00.00 sshd
...

# VIRT 是进程虚拟内存的大小，只要是进程申请过的内存，即便还没有真正分配物理内存，也会计算在内。
# RES 是常驻内存的大小，也就是进程实际使用的物理内存大小，但不包括 Swap 和共享内存。SHR 是共享内存的大小，比如与其他进程共同使用的共享内存、加载的动态链接库以及程序的代码段等。
# %MEM 是进程使用物理内存占系统总内存的百分比。
```