---
title: "仓颉文档阅读-开发指南VIII: 泛型(II) - 泛型接口、类、结构体以及枚举"
published: 2025-11-24 16:47:23
description: '仓颉文档阅读的开发指南部分, 本篇文章介绍仓颉语言的 泛型 的相关内容'
image: 'https://humid1ch.oss-cn-shanghai.aliyuncs.com/20250929154944807.webp'
tags: ["开发语言", "仓颉"]
category: Blogs
---

> [!NOTE]
>
> 阅读文档版本:
>
> 语言规约 [Cangjie-0.53.18-Spec](<https://cangjie-lang.cn/docs?url=/0.53.18/Spec/source_zh_cn/Chapter_01_Lexical_Structure(zh).html>)
>
> 具体开发指南 [Cangjie-LTS-1.0.4](https://cangjie-lang.cn/docs?url=/1.0.4/index.html)
>
> 在阅读 了解仓颉的语言规约时, 难免会涉及到一些仓颉的示例代码, 但 我们对仓颉并不熟悉, 所以可以用 [仓颉在线体验](https://cangjie-lang.cn/playground) 快速验证
>
> 有条件当然可以直接 [配置 Canjie-SDK](https://cangjie-lang.cn/download/1.0.4)

> [!WARNING]
>
> 博主在此之前, 基本只接触过 C/C++语言, 对大多现代语言都没有了解, 所以在阅读过程中遇到相似的概念, 难免会与 C/C++中的相似概念作类比, 见谅
>
> 且, 本系列是文档阅读, 而不是仓颉的零基础教学, 所以如果要跟着阅读的话最好有一门编程语言的开发经验

> [!WARNING]
>
> 在阅读仓颉编程语言的开发指南之前, 已经大概阅读了一遍 仓颉编程语言的语言规约, 已经对仓颉编程语言有了一个大概的了解
>
> 所以在阅读开发指南时, 不会对类似: 类、函数、结构体、接口等解释起来较为复杂名称 做出解释

> 此样式内容, 表示文档原文内容

## 泛型

仓颉泛型, 其实了解了概念之后, 并没有什么其他需要特别注意的复杂情况

### 泛型接口

> 泛型可以用来定义泛型接口，以标准库中定义的 `Iterable` 为例，它的成员函数 `iterator` 需要返回一个 `Iterator` 类型，这一类型是一个容器的遍历器
> 
> `Iterator` 是一个泛型接口，`Iterator` 内部有一个从容器类型中返回下一个元素的 `next` 成员函数，`next` 成员函数返回的类型是一个需要在使用时指定的类型，所以 `Iterator` 需要声明泛型参数
> 
> ```cangjie
> public interface Iterable<E> {
>     func iterator(): Iterator<E>
> }
> 
> public interface Iterator<E> <: Iterable<E> {
>     func next(): Option<E>
> }
> 
> public interface Collection<T> <: Iterable<T> {
>      prop size: Int64
>      func isEmpty(): Bool
> }
> ```

泛型接口的语法是:

```cangjie
interface Inter<T> {}
```

依旧是使用`<...>`声明类型形参列表

要实现泛型接口, 也同样通过`接口名<实参>`构建特定类型的接口类型

### 泛型类

> 泛型接口中介绍了泛型接口的定义和使用，本节介绍泛型类的定义和使用
> 
> 如 `Map` 的键值对就是使用泛型类来定义的
> 
> `Map` 类型中的键值对 `Node` 类型就可以使用泛型类来定义：
> 
> ```cangjie
> public open class Node<K, V> where K <: Hashable & Equatable<K> {
>     public var key: Option<K> = Option<K>.None
>     public var value: Option<V> = Option<V>.None
> 
>     public init() {}
> 
>     public init(key: K, value: V) {
>         this.key = Option<K>.Some(key)
>         this.value = Option<V>.Some(value)
>     }
> }
> ```
> 
> 由于键与值的类型有可能不相同，且可以为任意满足条件的类型，所以 `Node` 需要两个类型形参 `K` 与 `V` ，`K <: Hashable`, `K <: Equatable<K>` 是对于键类型的约束，意为 `K` 要实现 `Hashable` 与 `Equatable<K>` 接口，也就是 `K` 需要满足的条件
> 
> 对于泛型约束，详见 [泛型约束]() 章节
> 
> 由于 **泛型类的静态成员变量的内存是共享的**，因此，**静态成员变量 或 属性的类型声明和表达式中, 不能引用类型参数 或 包含未实例化泛型类型表达式**
> 
> 另外，静态变量或属性初始化表达式中不能调用泛型类的静态成员函数或属性
> 
> ```cangjie
> class A<T> {}
> class B<T> {
>     static func foo() {1}
>     static var err1: A<T> = A<T>()        // Error, 静态成员不能依赖于泛型形参'Generics-T'
>     static prop err2: A<T> {              // Error, 静态成员不能依赖于泛型形参'Generics-T'
>         get() {
>             A<T>()                        // Error, 静态成员不能依赖于泛型形参'Generics-T'
>         }
>     }
>     static var vfoo = foo()               // Error, 等于'static var vfoo = B<T>.foo()'，隐式引用泛型'T'
>     static var ok: Int64 = 1
> }
> 
> main() {
>     B<Int32>.ok = 2
>     println(B<Int64>.ok) // 2
> }
> ```

仓颉泛型类的定义并不复杂

但, 类是可以定义 静态成员变量和静态属性的

在非泛型的类中 静态成员变量是类共享的, 而 **在泛型类中, 静态成员变量也是类共享的**

文档中描述: 泛型类的静态成员变量的内存是共享的

这表示, **在使用泛型类时, 无论类型实参是什么, 静态成员变量都是唯一的**

这就意味着, **泛型类的静态成员变量 必须与泛型形参无关**

所以, **泛型类的静态成员变量和属性, 不允许引用 包含类型变元的任意表达式**

除此之外, 泛型类没有其他需要特别注意的地方

### 泛型结构体

> `struct` 类型的泛型与 `class` 是类似的，下面可以使用 `struct` 定义一个类似于二元元组的类型：
> 
> ```cangjie
> struct Pair<T, U> {
>     let x: T
>     let y: U
>     public init(a: T, b: U) {
>         x = a
>         y = b
>     }
>     public func first(): T {
>         return x
>     }
>     public func second(): U {
>         return y
>     }
> }
> 
> main() {
>     var a: Pair<String, Int64> = Pair<String, Int64>("hello", 0)
>     println(a.first())
>     println(a.second())
> }
> ```
> 
> 程序输出的结果为：
> 
> ```text
> hello
> 0
> ```
> 
> 在`Pair` 中提供了 `first` 与 `second` 两个函数来取得元组的第一个与第二个元素

泛型结构体同样没有其他注意事项, 同样要注意 泛型结构体的静态成员共享的特点

### 泛型枚举

> 在仓颉编程语言的泛型 `enum` 类型设计中，`Option` 类型是一个典型的示例，关于 `Option` 详细描述请参见 [`Option`类型]() 章节
> 
> `Option` 类型用于表示在某一类型上的值可能是个空的值
> 
> 这样，`Option` 就可以用来表示在某种类型上计算的失败
> 
> 这里是何种类型上的失败是不确定的，所以很明显，`Option` 是一个泛型类型，需要声明类型形参
> 
> ```cangjie
> package std.core              // `Option` 定义在 std.core
> 
> public enum Option<T> {
>       Some(T)
>     | None
> 
>     public func getOrThrow(): T {
>         match (this) {
>             case Some(v) => v
>             case None => throw NoneValueException()
>         }
>     }
>     ...
> }
> ```
> 
> 可以看到，`Option<T>` 分成两种情况，一种是 `Some(T)`，用来表示一个有值的返回结果，另一种是 `None` 用来表示一个空的结果
> 
> 其中的 `getOrThrow` 函数会是将 `Some(T)` 内部的值返回出来的函数，返回的结果就是 `T` 类型，而如果参数是 `None`，那么直接抛出异常
> 
> 例如：如果想定义一个安全的除法，因为在除法上的计算是可能失败的
> 
> 如果除数为 `0`，那么返回 `None` ，否则返回一个用 `Some` 包装过的结果：
> 
> ```cangjie
> func safeDiv(a: Int64, b: Int64): Option<Int64> {
>     var res: Option<Int64> = match (b) {
>                 case 0 => None
>                 case _ => Some(a/b)
>             }
>     return res
> }
> ```
> 
> 这样，在除数为 `0` 时，程序运行的过程中不会因除以 `0` 而抛出算术运算异常

仓颉的泛型枚举, 也没有什么需要特别注意的

仓颉内置的`Option`类型是一个很好的例子

