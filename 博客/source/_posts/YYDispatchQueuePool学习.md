---
title: YYDispatchQueuePool阅读
date: 2018-8-16 11:19:23
tags: 
- Object-C
- 多线程
categories: Object-C
---
## 目的：
解决当用**并行队列**来执行大量块时，当某个块所在线程被锁住时，**并列队列**会创建大量线程以至于占用了过多资源而影响到主线程的问题。<!--more-->
## 原理
**YYDispatchQueuePool**用一个全局的**串行队列**来尽量控制全局线程数。
## 分析源码（TODO:待完善）
1、首先了解Qos的优先级

~~~objc
NSQualityOfServiceUserInteractive //与用户交互的任务，这些任务通常跟UI级别的刷新相关，比如动画，这些任务需要在一瞬间完成
NSQualityOfServiceUserInitiated   //由用户发起的并且需要立即得到结果的任务，比如滑动scroll view时去加载数据用于后续cell的显示，这些任务通常跟后续的用户交互相关，在几秒或者更短的时间内完成
NSQualityOfServiceUtility         //一些可能需要花点时间的任务，这些任务不需要马上返回结果，比如下载的任务，这些任务可能花费几秒或者几分钟的时间 
NSQualityOfServiceBackground      //这些任务对用户不可见，比如后台进行备份的操作，这些任务可能需要较长的时间，几分钟甚至几个小时 
NSQualityOfServiceDefault         //优先级介于user-initiated 和 utility，当没有 QoS信息时默认使用，开发者不应该使用这个值来设置自己的任务
~~~

Qos与GCD queue优先级对应关系
![](http://upload-images.jianshu.io/upload_images/458529-74ed8a8b44ba56e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2、了解结构体YYDispatchContext

~~~objc
typedef struct{ 
  const char *name;    //线程名
  void **queues;       //队列数组
  uint32_t queueCount; //队列数目
  int32_t counter;     //自增计数
}YYDispatchContext;
~~~

3、如何获取队列

传入上下文，根据上下文去分配任务，返回**串行队列**

~~~objc
static dispatch_queue_t YYDispatchContextGetQueue(YYDispatchContext *context) {
    uint32_t counter = (uint32_t)OSAtomicIncrement32(&context->counter);
    void *queue = context->queues[counter % context->queueCount];
    return (__bridge dispatch_queue_t)(queue);
}
~~~

**注：** OSAtomicIncrement32是自增函数，原子性属于线程安全

## 调用示例（TODO:待完善）
导入头文件```#import "YYDispatchQueuePool.h"```

~~~objc
- (void)testYYDispatchQueuePool
{
    // 创建YYDispatchQueuePool实例，设置最大并发数为3
    YYDispatchQueuePool *pool = [[YYDispatchQueuePool alloc] initWithName:@"file. testYYDispatchQueuePool" queueCount:3 qos:NSQualityOfServiceBackground];
    // 遍历执行任务
    for (int index = 0; index < 100; index++) {
        // 获取队列
        dispatch_queue_t queue = [pool queue];
        dispatch_async(queue, ^{
            NSLog(@"index:%d",index);
        });
    }
}
~~~

## 源码链接
#### [YYDispatchQueuePool](https://github.com/ibireme/YYDispatchQueuePool)