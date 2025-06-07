# goroutine

## 调度器

Go 语言的调度器通过使用与 CPU 数量相等的线程减少线程频繁切换的内存开销，同时在每一个线程上执行额外开销更低的 Goroutine 来降低操作系统和硬件的负载。

Go最新版本使用的是基于信号的抢占式调度器。

- 基于信号的抢占式调度器 - 1.14 ~ 至今
  - 实现**基于信号的真抢占式调度**；
  - 垃圾回收在扫描栈时会触发抢占调度；
  - 抢占的时间点不够多，还不能覆盖全部的边缘情况；

## GMP模型

G — 表示 Goroutine，它是一个待执行的任务；

M — 表示操作系统的线程，它由操作系统的调度器调度和管理；

P — 表示处理器，它可以被看做运行在线程上的本地调度器；

### G

g的数据结构： https://github.com/golang/go/blob/master/src/runtime/runtime2.go#L396


g的状态：https://github.com/golang/go/blob/master/src/runtime/runtime2.go#L18

g的状态描述：

| 状态        | 描述                                                         |
| ----------- | :----------------------------------------------------------- |
| _Gidle      | 刚被分配但是还没有完成初始化                                 |
| _Grunnable  | 已经在运行队列中，但是还没有执行，也没有栈的所有权           |
| _Grunning   | 正在执行用户代码，拥有栈的所有权，已经被移出运行队列被分配给M或者P |
| _Gsyscall   | 正在执行系统调用，没有执行用户代码，拥有栈的所有权，已经被移出执行队列被分配给M |
| _Gwaiting   | 被阻塞，没有执行用户代码，没有在运行队列，可能存在于channel的等待队列。只有channel在特定的lock状态下可以读、写堆栈，其他情况下，在goroutines进入waiting状态时操作堆栈都是不安全的 |
| _Gdead      | 没有被使用，没有执行代码，可能有分配的栈                     |
| _Gcopystack | 拥有栈的所有权，栈正在被copy，没有执行用户代码也不在运行队列 |
| _Gpreempted | 由于抢占而被阻塞，没有执行用户代码并且不在运行队列上，等待唤醒 |
| _Gscan      | GC正在扫描栈空间，没有执行代码，可以与其他状态同时存在       |

### P

调度器中的处理器 P 是线程和 Goroutine 的中间层，它能提供线程需要的上下文环境，也会负责调度线程上的等待队列。

p的数据结构：https://github.com/golang/go/blob/master/src/runtime/runtime2.go#L632

### M

m的数据结构：https://github.com/golang/go/blob/master/src/runtime/runtime2.go#L528

M表示操作系统线程，go调度器最大可以设置10000，但是最多只会有GOMAXPROCS个线程能够正常运行，其他的线程都不会执行用户代码（如：等待系统调用）。go运行时默认设置GOMAXPROCS为当前的cpu核心数，此时不会发生操作系统的线程调度和上下文切换，所有的调度都在用户态下进行，减少了很多额外的开销。

