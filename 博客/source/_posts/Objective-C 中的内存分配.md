---
title: Objective-C 中的内存分配
date: 2018-10-05 11:19:23
tags: 
- Object-C
- 内存
categories: Object-C
---
在 Objective-C 中，对象通常是使用 alloc 方法在堆上创建的。 [NSObject alloc] 方法会在对堆上分配一块内存，按照NSObject的内部结构填充这块儿内存区域。<!--more-->

一旦对象创建完成，就不可能再移动它了。因为很可能有很多指针都指向这个对象，这些指针并没有被追踪。因此没有办法在移动对象的位置之后更新全部的这些指针。
## MRC与ARC
Objective-C中提供了两种内存管理机制：MRC（MannulReference Counting）和 ARC(Automatic Reference Counting)，分别提供对内存的手动和自动管理，来满足不同的需求。现在苹果推荐使用 ARC 来进行内存管理。

## MRC

| 对象操作        | OC中对应的方法                | 对应的 retainCount 变化 |
|:-------------: |:---------------:            | :-------------------: |
| 生成并持有对象   | alloc/new/copy/mutableCopy等 |         +1            |
| 持有对象        | retain                       |         +1            |
| 释放对象        | release                      |         -1           |
| 废弃对象        | dealloc                      |          -          |

### 四个法则
* 自己生成的对象，自己持有。
* 非自己生成的对象，自己也能持有。
* 不在需要自己持有对象的时候，释放。
* 非自己持有的对象无需释放。

### Autorelease
`autorelease`使得对象在超出生命周期后能正确的被释放(通过调用`release`方法)。在调用`release`后，对象会被立即释放，而调用`autorelease`后，对象不会被立即释放，而是注册到`autoreleasepool`中，经过一段时间后`pool`结束，此时调用`release`方法，对象被释放。

### MRC变量管理方法
在MRC的内存管理模式下，与对变量的管理相关的方法有：`retain`,`release`和`autorelease`。`retain`和`release`方法操作的是引用记数，当引用记数为零时，便自动释放内存。并且可以用 NSAutoreleasePool 对象，对加入自动释放池（autorelease 调用）的变量进行管理，当 drain 时回收内存。

## ARC
ARC 是苹果引入的一种自动内存管理机制，会根据引用计数自动监视对象的生存周期，实现方式是在编译时期自动在已有代码中插入合适的内存管理代码以及在 Runtime 做一些优化。
### ARC变量标识符
* `__strong`
* `__weak`
* `__unsafe_unretained`
* `__autoreleasing`

`__strong` 是默认使用的标识符。只有还有一个强指针指向某个对象，这个对象就会一直存活。

`__weak`声明这个引用不会保持被引用对象的存活，如果对象没有强引用了，弱引用会被置为`nil`

`__unsafe_unretained`声明这个引用不会保持被引用对象的存活，如果对象没有强引用了，它不会被置为 `nil`。如果它引用的对象被回收掉了，该指针就变成了野指针。

`__autoreleasing`用于标示使用引用传值的参数`（id *）`，在函数返回时会被自动释放掉。

~~~objc
Number* __strong num = [[Number alloc] init];
~~~
注意 __strong 的位置应该放到 * 和变量名中间，放到其他的位置严格意义上说是不正确的，只不过编译器不会报错。

### ARC属性标识符
类中的属性也可以加上标志符：

~~~objc
@property (assign/retain/strong/weak/unsafe_unretained/copy) Number* num
~~~

`assign`表明`setter`仅仅是一个简单的赋值操作，通常用于基本的数值类型，例如`CGFloat`和`NSInteger`。

`strong`表明属性定义一个拥有者关系。当给属性设定一个新值的时候，首先这个值进行`retain`，旧值进行`release`，然后进行赋值操作。

`weak`表明属性定义了一个非拥有者关系。当给属性设定一个新值的时候，这个值不会进行`retain`，旧值也不会进行`release`， 而是进行类似`assign`的操作。不过当属性指向的对象被销毁时，该属性会被置为`nil`。

`unsafe_unretained`的语义和`assign`类似，不过是用于对象类型的，表示一个非拥有(unretained)的，同时也不会在对象被销毁时置为`nil`的(unsafe)关系。

`copy`类似于`strong`，不过在赋值时进行`copy`操作而不是`retain`操作。通常在需要保留某个不可变对象（`NSString`最常见），并且防止它被意外改变时使用。