---
title: GCD
date: 2018-12-04 11:19:23
categorie: Object-C
---
**Dispatch Queue（队列）**是一个对象，类似队列的数据结构，而且是 FIFO(First In, First Out)队列，因此任务开始执行的顺序，就是你把它们放到 queue 中的顺序。

## GCD 中的队列类型可分为三种：

1. **Serial Queue（串行队列）** 串行队列中任务会按照添加到 queue 中的顺序一个一个执行。串行队列在前一个任务执行之前，后一个任务是被阻塞的，正因为如此，它们可以用来完成同步机制。
2. **Concurrent Queue(并行队列):** 并行队列，可以并发地执行多个任务，但是任务开始的顺序仍然是按照被添加到队列中的顺序。具体任务执行的线程和任务执行的并发数，都是由 GCD 进行管理的
3. **Main Dispatch Queue（主队列）：** 主队列是一个全局可见的串行队列，其中的任务会在主线程中执行。主队列通过与应用程序的 runloop 交互，把任务安插到 runloop 当中执行。因为主队列比较特殊，其中的任务确定会在主线程中执行，通常主队列会被用作同步的作用<!--more-->
	
## 获取队列
### 1.系统默认不提供串行队列，需要我们手动创建（**Main Dispatch Queue**是特殊串行队列除外）

~~~objc
dispatch_queue_t queue;
queue = dispatch_queue_create("com.sepeak.serialQueue", NULL); // OS X 10.7 和 iOS 4.3 之前
queue = dispatch_queue_create("com.sepeak.serialQueue",  DISPATCH_QUEUE_SERIAL); // 之后
~~~
 
#### 2. 获取并行队列(可获取系统提供全局并行队列或者手动创建)

~~~objc
    /**
     获取全局并行队列：
     系统默认提供了四个全局可用的并行队列，其优先级不同，分别为
     QOS_CLASS_USER_INTERACTIVE
     QOS_CLASS_USER_INITIATED
     QOS_CLASS_UTILITY
     QOS_CLASS_BACKGROUND
     优先级依次降低。优先级越高的队列中的任务会更早执行
     */
    /**
     DISPATCH_QUEUE_PRIORITY_HIGH
     DISPATCH_QUEUE_PRIORITY_DEFAULT
     DISPATCH_QUEUE_PRIORITY_LOW
     DISPATCH_QUEUE_PRIORITY_BACKGROUND
     这些值映射为一个适合QOS的级别
     */
    dispatch_queue_t globalQueue = dispatch_get_global_queue(QOS_CLASS_USER_INTERACTIVE, 0);
    
    /** 手动创建队列 */
    dispatch_queue_t concurrentQueue = dispatch_queue_create("com.sepeak.concurrentQueue", DISPATCH_QUEUE_CONCURRENT);
    
    /** 设置concurrentQueue和globalQueue的优先级一样 */
    dispatch_set_target_queue(concurrentQueue, globalQueue);
~~~
### 获取主线程队列dispatch_get_main_queue()
~~~objc
dispatch_queue_t mianQueue = dispatch_get_main_queue();
~~~
### 手动创建队列和系统创建队列区别：
事实上，我们自己创建的队列，最终会把任务分配到系统提供的主队列和四个全局的并行队列上，这种操作叫做 Target queues。具体来说，我们创建的串行队列的 target queue 就是系统的主队列，我们创建的并行队列的 target queue 默认是系统 default 优先级的全局并行队列。所有放在我们创建的队列中的任务，最终都会到 target queue 中完成真正的执行

通常情况下，对于串行队列，我们应该自己创建，对于并行队列，就直接使用系统提供的 Default 优先级的 queue。

### 创建的 Queue 释放问题
在 iOS 6 系统把`dispatch queue`纳入了 ARC 管理的范围，就不需要我们进行手动管理了。把`dispatch queue`从原来的非 OC 对象（原生 C 指针），变成了 OC 对象,就需要使用`strong`或者`weak`来修饰

~~~objc
@property (nonatomic, strong) dispatch_queue_t queue;
~~~
### 执行方式
1. 同步执行（sync）
	* 同步添加任务到指定的队列中，在添加的任务执行结束之前，会一直等待，直到队列里面的任务完成之后再继续执行。
	
	* 只能在当前线程中执行任务，不具备开启新线程的能力。

2. 异步执行（async）
	* 异步添加任务到指定的队列中，它不会做任何等待，可以继续执行任务。
	
	* 可以在新的线程中执行任务，具备开启新线程的能力。

### 执行任务
给 queue 添加任务有两种方式，同步和异步。同步方式会阻塞当前线程的执行，等待添加的任务执行完毕之后，才继续向下执行。异步方式不会阻塞当前线程的执行。

~~~objc
    dispatch_queue_t queue;
    queue = dispatch_queue_create("com.sepeak.serialQueue",  DISPATCH_QUEUE_SERIAL);
    
    // 异步添加
    /** 不会阻塞线程，将任务添加到queue队列中，直接继续执行主线程 */
    dispatch_async(queue, ^{
        NSLog(@"异步添加任务--%@\n",[NSThread currentThread]);
    });
    
    NSLog(@"第一个 block 可能还没有执行--%@\n",[NSThread currentThread]);
    // 同步添加
    /** 会阻塞线程，将任务添加到queue队列中，等待任务在queue队列完成后继续执行主线程 */
    dispatch_sync(queue, ^{
        NSLog(@"同步添加任务--%@\n",[NSThread currentThread]);
    });
    
    NSLog(@"两个 block 都已经执行完毕--%@\n",[NSThread currentThread]);
~~~
##### 注意事项
* 同步和异步添加，与队列是串行队列和并行队列没有关系。可以同步地给并行队列添加任务，也可以异步地给串行队列添加任务。同步和异步添加只影响是不是阻塞当前线程，和任务的串行或并行执行没有关系
* 如果在任务 block 中创建了大量对象，可以考虑在 block 中添加 autorelease pool。尽管每个 queue 自身都会有 autorelease pool 来管理内存，但是 pool 进行 drain 的具体时间是没办法确定的。如果应用对于内存占用比较敏感，可以自己创建 autorelease pool 来进行内存管理。

