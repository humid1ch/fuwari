---
title: "仓颉文档阅读-语言规约XIII: 并发"
published: 2025-10-17 09:13:41
description: "一直对仓颉挺感兴趣的, 但是一直没有去读一下文档, 慢慢看一看, 了解一下"
image: 'https://humid1ch.oss-cn-shanghai.aliyuncs.com/20250929154944807.webp'
category: Blogs
tags:
    - 开发语言
    - 仓颉
---

<Info>

阅读文档版本:

语言规约 [Cangjie-0.53.18-Spec](https://cangjie-lang.cn/docs?url=/0.53.18/Spec/source_zh_cn/Chapter_01_Lexical_Structure(zh).html)

具体开发指南 [Cangjie-LTS-1.0.3](https://cangjie-lang.cn/docs?url=/1.0.3/index.html)

在阅读 了解仓颉的语言规约时, 难免会涉及到一些仓颉的示例代码, 但 我们对仓颉并不熟悉, 所以可以用[仓颉在线体验](https://cangjie-lang.cn/playground)快速验证

有条件当然可以直接[配置Canjie-SDK](https://cangjie-lang.cn/download/1.0.1)

</Info>

<Warning>

博主在此之前, 基本就只接触过C/C++语言, 对大多现代语言都没有了解, 所以在阅读过程中遇到相似的概念, 难免会与C/C++中的相似概念作类比, 见谅

</Warning>

> 此样式内容, 表示文档原文内容

<Info>

按照文档中的顺序, 异常与并发之间还存在两个章节: 语言互操作 和 元编程

语言互操作基本上是与C语言的互操作, 元编程则是仓颉宏的一些内容

可以先了解并发的一些内容 再去看这两个章节

</Info>

## 并发

### 仓颉线程

> 仓颉编程语言提供**抢占式**的并发模型, 其中仓颉线程是基本的执行单元
>
> 仓颉线程**由仓颉运行时自行管理**, 并非是底层`native`线程(如操作系统线程)
>
> 在无歧义的情况下, 我们直接使用"线程"一词作为"仓颉线程"的简称
>
> 每个线程都具有如下性质:
>
> - 每个线程在执行的任意时刻都可能被另一个线程抢占
>
> - 多个线程之间并发执行
>
> - 当线程发生阻塞时, 该线程将被挂起
>
> - 不同线程之间可以共享内存(需要显式同步)
>
> 仓颉程序执行从初始化全局变量开始, 然后调用程序入口`main`
>
> 当`main`退出时, 整个程序执行便退出, 而**不会等待其余线程执行完毕**

仓颉线程, 是由仓颉运行时管理的, 而非操作系统, 也不是操作系统底层的原生线程

`main`函数退出时, 不会等待线程执行完毕

仓颉线程使用抢占式并发模型, 每个线程随时都可能被另外线程抢占

#### 创建线程

> 通过`spawn`关键字、一个可缺省的`ThreadContext`类型入参(关于`ThreadContext`的介绍见下方)和一个不包含形参的`lambda`可以创建并启动一个线程, 同时该表达式返回一个`Future<T>`的实例(关于`Future<T>`的介绍见下方)
>
> `spawn`表达式的`ANTLR`如下:
>
> ```
> spawnExpression
>     : 'spawn' ( '(' expression ')' )? trailingLambdaExpression
>     ;
> ```
>
> 其中`spawn`的可选参数的类型为`ThreadContext`
>
> 以下是`spawn`表达式缺省`ThreadContext`类型入参的一个示例
>
> ```cangjie
> func add(a: Int32, b: Int32): Int32 {
>     println("This is a new thread")
>     return a + b
> }
>
> main(): Int64 {
>     println("This is main")
>
>     // 创建一个线程
>     let fut: Future<Int32> = spawn {
>         add(1, 2)
>     }
>
>     println("main waiting...")
>     // 等待线程执行的结果
>     let res: Int32 = fut.get()
>     // 打印结果
>     println("1 + 2 = ${res}")
> }
>
> // OUTPUT maybe:
> // This is main
> // main waiting...
> // This is a new thread
> // 1 + 2 = 3
> ```
>
> 执行`spawn`表达式的过程中, 闭包会被封装成一个运行时任务并提交给调度器, 然后由调度器的策略决定任务的执行时机
>
> 一旦`spawn`表达式执行完成, 任务对应的线程便处于可执行状态
>
> 需要注意的是, `spawn`表达式的闭包中不可捕获`var`声明的局部变量; 关于闭包, 详见[闭包](https://www.humid1ch.cn/blog/cangjie-docs-reading-xi/#heading-2)

使用`spawn`可以创建仓颉线程, 表达式会返回一个`Future<T>`类型变量, 仓颉线程并不是操作系统原生线程

创建线程时, 可以缺省传入一个线程上下文的参数, 但目前并不知道这个参数是什么

#### `Future<T>`泛型类

> `spawn`表达式的返回值是一个`Future<T>`对象可用于获取线程的计算结果
>
> 其中, `T`的类型取决于`spawn`表达式中的**闭包的返回值类型**
>
> ```cangjie
> func foo(): Int64 {...}
> func bar(): String {...}
>
> // foo() 的返回类型是 Int64
> let f1: Future<Int64> = spawn {
>     foo()
> }
>
> // bar() 的返回类型是 String
> let f2: Future<String> = spawn {
>     bar()
> }
>
> // 等待线程的执行结果
> let r1: Int64 = f1.get()
> let r2: String = f2.get()
> ```

创建仓颉线程的表达式, 返回一个可以获取线程结果的`Futrue<T>`类型对象, `T`是闭包的返回值类型

> 一个`Future<T>`对象代表一个未完成的计算或任务
>
> `Future<T>`是一个泛型类, 其定义如下:
>
> ```cangjie
> class Future<T> {
>     ... ...
>
>     /**
>      * 阻塞当前线程,
>      * 等待与此 Future<T> 对象对应的线程的结果
>      * @如果对应线程中发生异常, 则抛出 Exception 或 Error
>      */
>     func get(): T
>
>     /**
>      * 阻塞当前线程,
>      * 等待与此 Future<T> 对象对应的线程的结果
>      * 如果相应线程在`ns`纳秒内未完成执行,
>      * 此方法将会返回一个`Option<T>.None`.
>      * 如果`ns <= 0`, 等价于`get()`.
>      * @如果对应线程中发生异常, 则抛出 Exception 或 Error
>      */
>     func get(ns: Int64): Option<T>
>
>     /**
>      * 非阻塞方法, 如果线程尚未完成执行, 则立即返回 None
>      * 否则返回计算结果
>      * @如果对应线程中发生异常, 则抛出 Exception 或 Error
>      */
>     func tryGet(): Option<T>
>
>     // 获取关联的线程对象
>     prop thread: Thread
> }
> ```
>
> 如果线程由于发生异常而终止, 那么它会将异常传递到`Future<T>`对象, 并在终止前默认打印出异常信息
>
> 当线程对应的`Future<T>`对象调用`get()`时, **异常将被再次抛出**
>
> 例如:
>
> ```cangjie
> main(args:Array<String>) {
>     let fut: Future<Int64> = spawn {
>         if (args.size != -1) {
>             throw IllegalArgumentException()
>         }
>         return 100
>     }
>     try {
>         let res: Int64 = fut.get()
>         print("val = ${res}\n")
>     } catch (e: IllegalArgumentException) {
>         print("Exception occured\n")
>     }
> }
> // OUTPUT:
> // An exception has occurred:
> // IllegalArgumentException
> //     ...
> // Exception occurred
> ```

`Future<T>`类提供了阻塞获取结果和尝试非阻塞获取结果的方法

仓颉线程发生异常, 异常会被传递到`Future`对象, 在`get()`时 异常会被抛出

`Future<T>`类, 拥有一个`Thread`类型的属性

#### `Thread`类

> 每个`Future<T>`对象都有一个关联的线程对象, 其类型为`Thread`, 可通过`Future<T>`的`thread`属性获取
>
> 线程类`Thread`主要用于获取线程的相关信息, 例如线程标识等, 其部分定义如下
>
> ```cangjie
> class Thread {
>     ... ...
>     // 获取当前正在运行的线程
>     static prop currentThread: Thread
>
>     // 获取线程对象的唯一标识符(以Int64表示)
>     prop id: Int64
>
>     // 获取或设置线程的名称
>     mut prop name: String
> }
> ```

`Thread`类是一个仓颉线程相关信息类

> 静态属性`currentThread`可获取当前正在执行的线程对象
>
> 下面的示例代码能够打印出当前执行线程的标识
>
> 注意: **线程在每次执行时可能被赋予不同的线程标识**
>
> ```cangjie
> main() {
>     let fut: Future<Unit> = spawn {
>         let tid = Thread.currentThread.id
>         println("New thread id: ${tid}")
>     }
>
>     fut.get()
> }
> // OUTPUT:
> // New thread id: 2
> ```

#### 线程睡眠


> 在仓颉标准库的`sync`包中提供了`sleep`函数能够让线程睡眠指定时长
>
> 如果睡眠时长为零, 即参数时长为`duration.Zero`, 那么当前线程只会**让出执行资源**并不会进入睡眠
>
> ```cangjie
> func sleep(duration: Duration)
> ```

仓颉中`sleep()`可以让线程睡眠指定时长, 也可以让线程让出资源, 类似C++中的`yield()`

#### 线程终止

> 在正常执行过程中, 若线程对应的闭包执行完成, 则该线程执行结束
>
> 此外, 还可以通过`Future<T>`对象的`cancel()`方法向对应的线程发送终止**请求**, 这一方法**仅发送请求而不会影响线程的执行**
>
> 线程类的`hasPendingCancellation`属性用于检查当前执行的线程是否存在终止请求, 程序员可以通过该检查来**自行决定**是否提前终止线程以及如何终止线程
>
> ```cangjie
> class Future<T> {
>     ... ...
>     // 向其执行线程发送终止请求
>     public func cancel(): Unit
> }
>
> class Thread {
>     ... ...
>     // 检查线程是否有取消请求
>     prop hasPendingCancellation: Bool
> }
> ```
>
> 以下示例展示如何终止线程
>
> ```cangjie
> main(): Unit {
>     let future = spawn { // 创建一个新线程
>         while (true) {
>             //...
>             if (Thread.currentThread.hasPendingCancellation) {
>                 return   // 在收到请求时终止
>             }
>             //...
>         }
>     }
>     //...
>     future.cancel()     // 发送终止请求
>     future.get()        // 等待线程终止
> }
> ```

仓颉线程没有提供直接终止线程的函数, 而是通过向线程发送请求, 设置线程属性

然后在线程代码中, 根据属性设置 自行决定是否终止线程

### 线程上下文

> 线程总是被运行在特定的上下文里, 线程上下文会影响线程运行时的行为
>
> 用户创建线程时, 除了缺省`spawn`表达式入参, 也可以通过传入不同`ThreadContext`类型的实例, 选择不同的线程上下文, 然后一定程度上控制并发行为

`ThreadContext`线程上下文, 但如何使用或许在下文中存在

#### 接口`ThreadContext`

> `ThreadContext`接口类型定义如下:
>
> - 结束方法`end`用于向当前`context`发送结束请求(关于线程上下文结束的介绍见下方`thread context`结束)
>
> - 检查方法`hasEnded`用于检查当前`context`是否已结束
>
> ```cangjie
> interface ThreadContext {
>     func end(): Unit
>     func hasEnded(): Bool
> }
> ```
>
> 目前不允许用户自行实现`ThreadContext`接口, 仓颉语言根据使用场景, 提供了如下线程上下文类型:
>
> - `MainThreadContext`: 用户可以在终端应用开发中使用此线程上下文;
>
> 具体定义可在终端框架库中查阅

找了半天也不知道 终端框架库 在什么地方

`ThreadContext`提供接口向当前上下文发送结束请求, 但 不了解 上下文接收到之后会触发什么

#### 线程上下文关闭

> 当关闭`ThreadContext`对象时, 可以通过`ThreadContext`对象提供的`end`方法向对应的`context`发送关闭请求
>
> 此方法**向运行在此`context`上的所有线程发送终止请求**(参见线程终止), 并马上返回, 运行时待所有绑定至此`context`上的仓颉`thread`运行结束后, 关闭`context`
>
> 同样的, 关闭请求不会影响在此`context`上所有线程的执行, 用户可以通过`hasPendingCancellation`检查来自行决定是否提前终止线程以及如何终止线程

`ThreadContext`的`end()`接口, 是向当前上下文中发送终止请求

上下文收到请求之后, 会向所有运行在此上下文上的线程发送线程终止请求

只有运行在此上下文中的所有线程运行结束之后, 上下文才会被关闭

#### 线程局部变量

> 类型`ThreadLocal<T>`可用于定义线程局部变量
>
> 和普通变量相比, 线程局部变量有不同的访问语义
>
> 当多个线程共享使用同一线程局部变量时, 每个线程都有各自的一份**值拷贝**
>
> 线程对变量的访问会读写线程本地的值, 而不会影响其他线程中变量的值
>
> 因而, `ThreadLocal<T>`类是线程安全的
>
> 类`ThreadLocal<T>`如下定义所示, 其中:
>
> - 构造方法`init`用于构造线程局部变量
>
>     在构造完但未设置变量值时, 线程对变量的读取将得到空值`None`;
>
> - 读写方法`get/set`用于访问线程局部变量的值
>
>     当线程将变量的值设为`None`后, 当前线程中的变量值将被清除
>
> ```cangjie
> class ThreadLocal<T> {
>     // 构建一个包含 None 的线程局部变量
>     public init()
>
>     // 获取当前执行线程的线程局部变量的值
>     public func get(): ?T
>
>     // 为线程局部变量设置值
>     public func set(value: ?T): Unit
> }
> ```
>
> 以下示例展示如何使用线程局部变量
>
> ```cangjie
> let tlv = ThreadLocal<Int64>()                    // 定义线程局部变量
>
> main(): Unit {
>     for (i in 0..3) {                             // 生成三个线程
>         spawn {
>             tlv.set(i)                            // 每个线程设置不同的值
>             // ...
>             println("${tlv.get()}")               // 每个线程打印自己的值
>         }
>     }
>     // ...
>     println("${tlv.get()}")                       // 打印`None`
>                                                   // 由于当前线程没有设置任何值
> }
> ```

线程局部变量的作用, 更像是一个全局变量, 但每个线程拥有自己的独立副本

### 同步机制

#### 原子操作

> 原子操作确保指令被原子地执行, 即执行过程中不会被中断
>
> 对原子变量的一个写操作, 对于后续对同一个原子变量的读操作来说, 总是可见的
>
> 此外, 原子操作是非阻塞的动作, 不会导致线程阻塞
>
> 仓颉语言提供了针对整数类型(包括 Int8、Int16、Int32、Int64、UInt8、UInt16、UInt32、UInt64), 布尔类型(Bool)和引用类型的原子操作

仓颉对整数类型、布尔类型和引用类型提供原子操作

引用类型? 不是自定义`class`吧?

> - 对于整数类型, 我们提供基本的读写、交换以及算术运算的操作:
>
>     - `load`: 读取
>
>     - `store`: 写入
>
>     - `swap`: 交换
>
>     - `compareAndSwap`: 比较再交换
>
>     - `fetchAdd`: 加法
>
>     - `fetchSub`: 减法
>
>     - `fetchAnd`: 与
>
>     - `fetchOr`: 或
>
>     - `fetchXor`: 异或
>
>     ```cangjie
>     // 有符号整型
>     class AtomicInt8 {
>         ... ...
>         init(val: Int8)
>
>         public func load(): Int8
>         public func store(val: Int8): Unit
>         public func swap(val: Int8): Int8
>         public func compareAndSwap(old: Int8, new: Int8): Bool
>
>         public func fetchAdd(val: Int8): Int8
>         public func fetchSub(val: Int8): Int8
>         public func fetchAnd(val: Int8): Int8
>         public func fetchOr(val: Int8): Int8
>         public func fetchXor(val: Int8): Int8
>
>         ... ... // 运算符重载等
>     }
>
>     class AtomicInt16 {...}
>     class AtomicInt32 {...}
>     class AtomicInt64 {...}
>
>     // 无符号整型
>     class AtomicUInt8  {...}
>     class AtomicUInt16 {...}
>     class AtomicUInt32 {...}
>     class AtomicUInt64 {...}
>     ```
>
> - 对于布尔类型, 仅提供基本的读写、交换操作, 不提供算术运算的操作:
>
>     - `load`: 读取
>
>     - `store`: 写入
>
>     - `swap`: 交换
>
>     - `compareAndSwap`: 比较再交换
>
>     ```cangjie
>     // Boolean.
>     class AtomicBool {
>         ... ...
>         init(val: Bool)
>
>         public func load(): Bool
>         public func store(val: Bool): Unit
>         public func swap(val: Bool): Bool
>         public func compareAndSwap(old: Bool, new: Bool): Bool
>
>         ... ... // 运算符重载等
>     }
>     ```
>
> - 对于引用类型, 仅提供基本的读写、交换操作, 不提供算术运算的操作:
>
>     - `load`: 读取
>
>     - `store`: 写入
>
>     - `swap`: 交换
>
>     - `compareAndSwap`: 比较再交换
>
>     ```cangjie
>     class AtomicReference<T> where T <: Object {
>         ... ...
>         init(val: T)
>
>         public func load(): T
>         public func store(val: T): Unit
>         public func swap(val: T): T
>         public func compareAndSwap(old: T, new: T): Bool
>     }
>     ```
>
> - 此外, 对于可空引用类型, 可以通过`AtomicOptionReference`保存"空引用"(以`None`表示)
>
>     ```cangjie
>     class AtomicOptionReference<T> where T <: Object {
>         public init(val: Option(T))
>         public func load(): Option<T>
>         public func store(val: Option<T>): Unit
>         public func swap(val: Option<T>): Option<T>
>         public func compareAndSwap(old: Option<T>, new: Option<T>): Bool
>     }
>   ```

对于比较基础的只有单个数值类型的原子操作, 比较容易理解

但, 关于任意引用类型的原子操作, 暂时未知仓颉的使用和实现细节

#### `IllegalSynchronizationStateException`

> `IllegalSynchronizationStateException`是一个运行时异常, 当出现违规行为, 例如当线程尝试去释放当前线程未获取的互斥量的时候, 内置的并发原语会抛出此异常

这是一个异常, 非法同步状态, 一些线程尝试非法操作时 可能会抛出此异常

#### `IReentrantMutex`

> `IReentrantMutex`是可重入式互斥并发原语的公共接口, 提供如下三个方法, 需要开发者提供具体实现:
>
> - `lock(): Unit`: 获取锁, 如果获取不到, 阻塞当前线程
>
> - `tryLock(): Bool`: 尝试获取锁
>
> - `unlock(): Unit`: 释放锁
>
> ```cangjie
> interface IReentrantMutex {
>     // 锁定互斥锁, 如果互斥锁不可用, 则阻塞当前线程
>     public func lock(): Unit
>
>     // 尝试锁定互斥锁, 如果互斥锁不可用则返回 false, 否则锁定互斥锁并返回 true
>     public func tryLock(): Bool
>
>     // 解锁互斥锁. 如果互斥锁被重复锁定 N 次, 则此方法应被调用 N 次以完全解锁互斥锁
>     // 当互斥锁完全解锁时, 会唤醒等待其 lock 的一个线程(不隐含特定的准入策略)
>     // 如果当前线程没有持有这个互斥锁, 则抛出 ISSE("互斥体未被当前线程锁定")
>     public func unlock(): Unit
> }
> ```
>
> 注意:
>
> - 开发者在实现该接口时, 需要保证 底层互斥锁 确实支持嵌套锁
>
> - 开发者在实现该接口时, 需要保证 在出现锁状态异常时抛出`IllegalSynchronizationStateException`

这是仓颉提供的要实现可重入锁的接口

这里的可重入应该是针对锁的可重入, 而不是与锁无关的可重入函数

锁的可重入, 简单理解 同一执行单元可以多次重复 不先解锁去获取同一个锁

所以, `unlock()`需要实现可以解除全部的重复锁定, 也需要保证实现`lock()/tryLock()`可以支持重复锁定

<Warning>

按照最新版本的Cangjie文档, 此接口在未来将会被废弃, 将使用`Lock`代替

</Warning>

#### `ReentrantMutex`

> `ReentrantMutex`提供如下方法:
>
> - `lock(): Unit`: 获取锁, 如果获取不到, 阻塞当前线程
>
> - `tryLock(): Bool`: 尝试获取锁
>
> - `unlock(): Unit`: 释放锁
>
> `ReentrantMutex`是内置的锁, 在同一时刻最多只能被一个线程持有
>
> 如果给定的`ReentrantMutex`已经被另外一个线程持有, `lock`方法会阻塞当前线程直到该互斥锁被释放, 而`tryLock`方法会立即返回`false`
>
> `ReentrantMutex`是可重入锁, 即如果一个线程尝试去获取一个它已经持有的`ReentrantMutex`锁, 他会立即获取该`ReentrantMutex`
>
> 为了成功释放该锁, `unlock`的调用次数必须与`lock`的调用次数相匹配
>
> 注意:`ReentrantMutex`是内置的互斥锁, **不允许开发者继承**
>
> ```cangjie
> // 内置可重入互斥并发原语的基础类
> sealed class ReentrantMutex <: IReentrantMutex {
>     // 构造函数
>     init()
>
>     // 锁定互斥锁, 如果互斥锁不可用, 则阻塞当前线程
>     public func lock(): Unit
>
>     // 尝试锁定互斥锁, 如果互斥锁不可用则返回 false, 否则锁定互斥锁并返回 true
>     public func tryLock(): Bool
>
>     // 解锁互斥锁. 如果互斥锁被重复锁定 N 次, 则此方法应被调用 N 次以完全解锁互斥锁
>     // 当互斥锁完全解锁时, 会唤醒等待其 lock 的一个线程(不隐含特定的准入策略)
>     // 如果当前线程没有持有这个互斥锁, 则抛出 ISSE("互斥体未被当前线程锁定")
>     public func unlock(): Unit
> }
> ```

`ReentrantMutex`是仓颉提供的, 可重入锁 类, 不允许被继承

<Warning>

按照最新版本的Cangjie文档, 此类在未来将会被废弃, 将使用`Metux`代替

</Warning>

#### `synchronized`

> 通过`synchronized`关键字和一个`ReentrantMutex`对象, 对其后面修饰的代码块进行保护, 使得同一时间只允许一个线程执行里面的代码
>
> `ReentrantMutex`或其派生类的实例可以作为参数传递给`synchronized`关键字, 这将导致如下转换:
>
> ```cangjie
> //=======================================
> //`synchronized`expression
> //=======================================
> let m: ReentrantMutex = ...
> synchronized(m) {
>     foo()
> }
>
> //=======================================
> // is equivalent to the following program.
> //=======================================
> let m: ReentrantMutex = ...
> m.lock()
> try {
>     foo()
> } finally {
>     m.unlock()
> }
> ```
>
> 注意:`synchronized`关键字与`IReentrantMutex`接口不兼容
>
> - 一个线程在进入`synchronized`修饰的代码块之前, 必须先获取该`ReentrantMutex`对象的锁, 如果无法获取该锁, 则当前线程被阻塞;
>
> - 一个线程在退出`synchronized`修饰的代码块之后, 会自动释放该`ReentrantMutex`对象的锁; 只会执行一个`unlock`操作, 因此允许嵌套同一个互斥锁的`synchronized`代码块
>
> - 在到达`synchronized`修饰的代码块中的`return e`表达式后, 会先将`e`求值到`v`, 再调用`m.unlock()`, 最后返回`v`
>
> - 若`synchronized`修饰的代码块内有`break`或`continue`表达式, 并且该表达式的执行会导致程序跳出该代码块, 那么会自动调用`m.unlock()`方法
>
> - 当在`synchronized`修饰的代码块中发生异常并跳出该代码块时, 会自动调用`m.unlock()`方法
>
> `synchronized`表达式的`ANTLR`如下:
>
> ```
> synchronizedExpression
>     : 'synchronized' '(' expression ')' block
>     ;
> ```
>
> 其中`expression`是一个`ReentrantMutex`的对象

`synchronized (mutex) {}`可以先获取目标锁, 再执行目标代码块, 且获取的锁在即将跳出代码块时, 会被自动释放

是一种`RAII`思想使用锁的语法糖

#### `Monitor`

> `Monitor`是一个内置的数据结构, 它绑定了互斥锁和单个与之相关的条件变量(也就是等待队列)
>
> `Monitor`可以使线程阻塞并等待来自另一个线程的信号以恢复执行
>
> 这是一种利用共享变量进行线程同步的机制, 主要提供如下方法:
>
> - `wait(timeout!: Duration = Duration.Max): Bool`: 等待信号, 阻塞当前线程
>
> - `notify(): Unit`: 唤醒一个在`Monitor`上等待的线程(如果有)
>
> - `notifyAll(): Unit`: 唤醒所有在`Monitor`上等待的线程(如果有)
>
> 调用`Monitor`对象的`wait`、`notify`或`notifyAll`方法前, 需要确保当前线程已经持有对应的`Monitor`锁
>
> `wait`方法包含如下动作:
>
> - 添加当前线程到该`Monitor`对应的等待队列中
>
> - 阻塞当前线程, 同时完全释放该`Monitor`锁, 并记录获取次数
>
> - 等待某个其它线程使用同一个`Monitor`实例的`notify`或`notifyAll`方法向该线程发出信号
>
> - 唤醒当前线程并同时获取`Monitor`锁, 恢复第二步记录的获取次数
>
> 注意:`wait`方法接受一个可选参数`timeout`
>
> 需要注意的是, 业界很多常用的常规操作系统不保证调度的实时性, 因此无法保证一个线程会被阻塞"精确时长"——即可能会观察到由系统导致的不精确情况
>
> 此外, 当前语言规范明确允许实现中发生虚假唤醒——在这种情况下, `wait`返回值是由实现决定的——可能为`true`或`false`
>
> 因此鼓励开发者始终将`wait`包在一个循环中:
>
> ```cangjie
> synchronized (obj) {
>     while (<condition is not true>) {
>         obj.wait();
>     }
> }
> ```
>
> `Monitor`定义如下:
>
> ```cangjie
> class Monitor <: ReentrantMutex {
>     // 构造函数
>     init()
>
>     // 阻塞, 直到一个配对的`notify`被调用或`timeout`到期
>     // 如果 Monitor 被其他线程通知, 则返回`true`, 如果在超时情况下则返回`false`. 允许虚假唤醒
>     // 如果当前线程没有持有这个互斥锁, 则抛出 ISSE("互斥体未被当前线程锁定")
>     func wait(timeout!: Duration = Duration.Max): Bool
>
>     // 唤醒在此 Monitor 上等待的一个线程, 如果有的话(不隐含特定的准入策略)
>     // 如果当前线程没有持有这个互斥锁, 则抛出 ISSE("互斥体未被当前线程锁定")
>     func notify(): Unit
>
>     // 唤醒在此监视器上等待的所有线程(如果有, 不隐含特定的准入策略)
>     // 如果当前线程没有持有这个互斥锁, 则抛出 ISSE("互斥体未被当前线程锁定")
>     func notifyAll(): Unit
> }
> ```

`Monitor`类似一个结合了条件变量的锁

`Monitor.wait()`会释放持有的锁+并让线程陷入阻塞, 等待唤醒, 这就是条件变量+锁


<Warning>

按照最新版本的Cangjie文档, 此类在未来将会被废弃, 将使用`Condition`代替

`Condition`是一个接口

</Warning>

#### `MultiConditionMonitor`

> `MultiConditionMonitor`是一个内置的数据结构, 它绑定了互斥锁和**一组**与之相关的动态创建的条件变量
>
> 该类应仅当在`Monitor`类不足以实现高级并发算法时被使用
>
> 提供如下方法:
>
> - `newCondition(): ConditionID`: 创建一个新的等待队列并与当前对象关联, 返回一个特定的`ConditionID`标识符
>
> - `wait(id: ConditionID, timeout!: Duration = Duration.Max): Bool`: 等待信号, 阻塞当前线程
>
> - `notify(id: ConditionID): Unit`: 唤醒一个在`Monitor`上等待的线程(如果有)
>
> - `notifyAll(id: ConditionID): Unit`: 唤醒所有在`Monitor`上等待的线程(如果有)
>
> 初始化时, `MultiConditionMonitor`没有与之相关的`ConditionID`实例
>
> 每次调用`newCondition`都会将创建一个新的等待队列并与当前对象关联, 并返回如下类型作为唯一标识符:
>
> ```cangjie
> external struct ConditionID {
>    private init() { ... } // constructor is intentionally private to prevent
>                           // creation of such structs outside of MultiConditionMonitor
> }
> ```
>
> 请注意使用者不可以将一个`MultiConditionMonitor`实例返回的`ConditionID`传给其它实例, 或者手动创建`ConditionID`(例如使用`unsafe`)
>
> 由于`ConditionID`所包含的数据(例如内部数组的索引, 内部队列的直接地址, 或任何其他类型数据等)和创建它的`MultiConditionMonitor`相关, 所以将"外部"`conditonID`传入`MultiConditionMonitor`中会导致`IllegalSynchronizationStateException`
>
> ```cangjie
> class MultiConditionMonitor <: ReentrantMutex {
>     // 构造函数
>     init()
>
>     // 返回与此 Monitor 关联的新 ConditionID. 可用于实现"单个互斥锁——多个等待队列"并发原语
>     // 如果当前线程没有持有这个互斥锁, 则抛出 ISSE("互斥体未被当前线程锁定")
>     func newCondition(): ConditionID
>
>     // 阻塞, 直到一个配对的`notify`被调用或`timeout`到期
>     // 如果指定条件由其他线程触发, 则返回`true`, 如果在超时情况下则返回`false`
>     // 允许虚假唤醒
>     // 如果当前线程没有持有这个互斥锁, 则抛出 ISSE("互斥体未被当前线程锁定")
>     // 如果`id`未被此 MultiConditionMonitor 实例的`newCondition`方法返回, 则抛出 ISSE("无效条件")
>     func wait(id: ConditionID, timeout!: Duration = Duration.Max): Bool
>
>     // Wakes up a single thread waiting on the specified condition, if any (no particular admission policy implied).
>     // 如果当前线程没有持有这个互斥锁, 则抛出 ISSE("")
>     // 如果`id`未被此 MultiConditionMonitor 实例的`newCondition`方法返回, 则抛出 ISSE("无效条件")
>     func notify(id: ConditionID): Unit
>
>     // Wakes up all threads waiting on the specified condition, if any (no particular admission policy implied).
>     // 如果当前线程没有持有这个互斥锁, 则抛出 ISSE("互斥体未被当前线程锁定")
>     // 如果`id`未被此 MultiConditionMonitor 实例的`newCondition`方法返回, 则抛出 ISSE("无效条件")
>     func notifyAll(id: ConditionID): Unit
> }
> ```

`MultiConditionMonitor`是一个互斥锁 允许绑定 多个条件变量的类

需要调用类的成员`newCondition()`创建属于此`Monitor`的条件变量, 可以创建多个, 此函数会返回`ConditionID`, 表示一个所属的条件变量

创建的条件变量是属于此`MultiConditionMontor`的, 不能给其他对象使用

> **示例: ** 使用`MultiConditionMonitor`去实现一个"长度固定的有界 FIFO 队列", 当队列为空, `get()`会被阻塞; 当队列满了时, `put()`会被阻塞
>
> ```cangjie
> class BoundedQueue {
>     // 创建一个 MultiConditionMonitor, 两个 条件变量
>     let m: MultiConditionMonitor = MultiConditionMonitor()
>     var notFull: ConditionID
>     var notEmpty: ConditionID
>
>     var count: Int64                                                    // 缓冲区中的对象数量
>     var head: Int64                                                     // 写索引
>     var tail: Int64                                                     // 读索引
>
>     // 队列长度为 100
>     let items: Array<Object> = Array<Object>(100, {i => Object()})
>
>     init() {
>         count = 0
>         head = 0
>         tail = 0
>
>         synchronized(m) {
>             notFull  = m.newCondition()
>             notEmpty = m.newCondition()
>         }
>     }
>
>     // 如果队列已满, 插入对象时将阻塞当前线程
>     public func put(x: Object) {
>         // 获取锁
>         synchronized(m) {
>             while (count == 100) {
>                 // 如果队列已满, 请等待"queue notFull"事件
>                 m.wait(notFull)
>             }
>             items[head] = x
>             head++
>             if (head == 100) {
>                 head = 0
>             }
>             count++
>
>             // 一个对象已被插入, 当前队列不再为空
>             // 因此唤醒此前因队列空 而被阻塞在 get() 上的线程
>             m.notify(notEmpty)
>         } // 释放锁
>     }
>
>     // 弹出一个对象, 如果队列是空的, 阻塞当前线程
>     public func get(): Object {
>         // 获取锁
>         synchronized(m) {
>           while (count == 0) {
>             // 如果队列是空的, 等待"队列非空"事件
>             m.wait(notEmpty)
>           }
>           let x: Object = items[tail]
>           tail++
>           if (tail == 100) {
>             tail = 0
>           }
>           count--
>
>           // 一个对象已被弹出, 当前队列不再满
>           // 因此唤醒先前因队列满 而被阻塞在 put() 上的线程
>           m.notify(notFull)
>
>           return x
>         } // 释放锁
>     }
> }
> ```

<Warning>

按照最新版本的Cangjie文档, 此类在未来将会被废弃, 将使用`Mutex`代替

</Warning>

### 内存模型

> 内存模型主要解决并发编程中内存可见性的问题, 它规定了一个线程对一个变量的写操作, 何时一定可以被其他线程对同一变量的读操作观测到
>
> - 如果**存在数据竞争**, 那么行为是未定义的
>
> - 如果**没有数据竞争**, 一个读操作所读到的值是: 在`happens-before`顺序上离它最近的一个写操作所写入的值
>
> > 主要解决并发编程中内存可见性的问题, 即一个线程的写操作何时会对另一个线程可见


#### `Happens-Before`

`happens-before`简单理解为 **先行发生**

> - 程序顺序规则: 同一个线程中的每个操作`happens-before`于该线程中的任意后续操作
>
>     ```cangjie
>     var a: String
>
>     main(): Int64 {
>         a = "hello, world"
>         println(a)
>         return 0
>     }
>
>     // OUTPUT:
>     // hello, world
>     ```
>
> - 线程启动规则: 如果线程`A`通过`spawn`创建线程`B`, 那么线程`A`的`spawn`操作`happens-before`于线程`B`中的任意操作
>
>     ```cangjie
>     var a: String = "123"
>
>     func foo(): Unit {
>         println(a)
>     }
>
>     main(): Int64 {
>         a = "hello, world"
>         let fut: Future<Unit>= spawn {
>             foo()
>         }
>         fut.get()
>         return 0
>     }
>
>     // OUTPUT:
>     // hello, world
>     ```
>
> - 线程终止规则: 如果线程`A`调用`futureB.get()`并成功返回, 那么线程`B`中的任意操作`happens-before`于线程`A`中的`futureB.get()`调用
>
>     如果线程`A`调用`futureB.cancel()`并且线程`B`在此之后访问`hasPendingCancellation`, 那么这两个调用构成`happens-before`关系
>
>     ```cangjie
>     var a: String = "123"
>
>     func foo(): Unit {
>         a = "hello, world"
>     }
>
>     main(): Int64 {
>         let fut: Future<Unit> = spawn {
>             foo()
>         }
>
>         fut.get()
>         println(a)
>         return 0
>     }
>
>     // OUTPUT:
>     // hello, world
>     ```
>
> - 线程同步规则: 在同一个线程同步对象(例如互斥锁、信号量等)上的操作存在一个 **全序**
>
>     一个线程对一个同步对象的操作(例如对互斥锁的解锁操作)`happens-before`于这个全序上后续对这个同步对象的操作(例如对互斥锁的上锁操作)
>
> - 原子变量规则: 对于所有原子变量的操作存在一个 **全序**
>
>     一个线程对一个原子变量的操作`happens-before`于这个全序上后续所有的原子变量的操作
>
> - 传递性规则: 如果`A happens-before B`且`B happens-before C`, 那么`A happens-before C`

全序, 表示不存在不确定的执行顺序

这一小节, 说明了一些 语句执行顺序的先后发生 的一些情况

存在`happens-before`关系, 就说明两个操作在逻辑上存在先后顺序

#### 数据竞争`Data Race`

> 如果两个线程对 **同一个数据** 访问, 其中至少有一个是**写操作**, 而且这两个操作之间没有`happens-before`关系, 那么形成一个数据竞争
>
> > 对"同一个数据访问"的定义:
> >
> > - 对同一个`primitive type`、`enum`、`array`类型的变量或者`struct/class`类型的同一个`field`的访问, 都算作对同一个数据的访问
> >
> > - 对`struct/class`类型的不同`field`的访问, 算作对不同数据的访问

对同一个变量或同一个字段访问, 就是访问同一个数据

如果多线程访问同一个数据, 且存在写操作, 那么就存在数据竞争
