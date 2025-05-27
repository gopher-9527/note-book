# README

并发编程
 - goroutine 
 - GMP
 - channel 
 - lock sync/mutex
 - context

package命名的原则：

- short 简短
- concise 简洁
- evocative  语义明确       

package name一般是小写的单个单词，不应该是带下划线或者大小写混合的形式


### go的优势
1. goroutine轻量化带来的优势，goroutine相对于传统的线程来说，消耗的资源更少（goroutine默认是2KB，传统的线程一般是几MB的大小，可以通过命令：**ulimit -s**查看），所以在相同的资源消耗下，
go可以创建更多的goroutine来执行任务。由于goroutine的轻量化，其上下文切换的开销也较传统的线程更小。
2. goroutine的调度策略带来的性能提升，go采用了GMP的调度模型来调度运行时的goroutine，其中 Go 调度器实现 Goroutine 到 KSE（内核调度实体）的调度，由内核调度器实现 KSE 到 CPU 上的调度。
M会绑定到KSE，所以不会频繁触发操作系统的线程调度和上下文切换，所有的调度都会发生在用户态，由 Go 语言调度器触发，能够减少很多额外开销。
3. channel带来的开发便利，channel的本质是一个带锁的队列，可用于在不同的goroutine之间通信，这就相当于go内置了消息队列，在一些复杂的场景下，我们可以使用channel的特性，将操作和数据解耦，
从而构建更加清晰的处理流程。
4. 网络轮询器带来的高效I/O操作

