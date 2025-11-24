---
title: "仓颉文档阅读-开发指南VIII: 泛型(I) - 泛型与泛型函数"
published: 2025-11-24 15:20:42
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

### 泛型概述

> 在仓颉编程语言中，泛型指的是参数化类型，参数化类型是 **一个在声明时未知 并且需要在使用时指定的类型**
> 
> 类型声明与函数声明可以是泛型的
> 
> 最为常见的例子就是`Array<T>`、`Set<T>` 等容器类型
> 
> 在仓颉中，`function`、`class`、`interface`、`struct` 与 `enum` 的声明都可以声明类型形参，也就是说它们都可以是泛型的
> 
> 为了方便讨论，定义如下几个常用的术语：
> 
> - 类型形参：一个类型或者函数声明可能有一个或者多个需要在使用处被指定的类型，这些类型就被称为类型形参
> 
>     在声明形参时，需要给定一个标识符，以便在声明体中引用
> 
> - 类型变元：在声明类型形参后，当通过标识符来引用这些类型时，这些标识符被称为类型变元
> 
> - 类型实参：当在使用泛型声明的类型或函数时指定了泛型参数，这些参数被称为类型实参
> 
> - 类型构造器：一个需要零个、一个或者多个类型作为实参的类型称为类型构造器
> 
> 类型形参在声明时一般在类型名称的声明或者函数名称的声明后，使用尖括号 `<...>` 括起来
> 
> 例如一个泛型的列表可声明为：
> 
> ```cangjie
> class List<T> {
>     var elem: Option<T> = None
>     var tail: Option<List<T>> = None
> }
> 
> func sumInt(a: List<Int64>) {  }
> ```
> 
> 其中 `List<T>` 中的 `T` 被称为类型形参
> 
> 对于 `elem: Option<T>` 中对 `T` 的引用称为类型变元，同理 `tail: Option<List<T>>` 中的 `T` 也称为类型变元
> 
> 函数 `sumInt` 的参数中 `List<Int64>` 的 `Int64` 被称为 `List` 的类型实参
> 
> `List` 就是类型构造器，`List<Int64>` 通过 `Int64` 类型实参构造出了一个类型 `Int64` 的列表类型

关于泛型, 绝大多数的高级语言, 又存在相关语法

文档中介绍, 泛型 指参数化类型, 一个在声明时未知 并且需要在使用时指定的类型

简单理解, 就是 声明使用任意标识符作为形参 代指未知类型, 使用时传入实参指定实际类型

比如泛型函数: 

```cangjie
func hash<T>(a: T) : Int64 where T <: Hashable {
    return a.hashCode()
}
// 声明 <T>代指未知类型, 可以在参数列表中 作为类型使用

// 使用
main() {
    print(hash<Bool>(false))

    return 0
}
```

泛型内容, 声明时 使用任意标识符代指未知类型, 使用时传入类型指定具体类型

一个泛型存在许多属于:

1. 泛型声明时 存在泛型参数列表`<T1, T2, ...>`

    泛型参数列表中的`T1`、`T2`或其他标识符, 在此处 被称为 **类型形参**

2. 声明时 存在对类型形参的使用, 比如泛型函数的参数列表中`(a: T)`

    在泛型声明中, 除泛型参数列表之外的地方 通过标识符使用类型参数, 被称为 **类型变元**

3. 在使用泛型时, 需要传入具体类型 作为类型形参的实参, 此时的具体类型叫作 **类型形参**

4. 泛型的具体标识符, 被称为类型构造器, 泛型的使用方法 一般是`类型构造器<类型实参>`

    此时, 会构造出一个 拥有具体类型的目标泛型

仓颉中泛型的标识就是 **尖括号包裹的类型形参列表: `<...>`**

在使用泛型, 传入类型实参时, 定义泛型时 使用的类型变元 都会被替换为类型实参

### 泛型函数

> 如果一个函数 声明了一个或多个类型形参，则将其称为泛型函数
> 
> 语法上，类型形参紧跟在函数名后，并用 `<>` 括起，如果有多个类型形参，则用 `,` 分离

泛型函数的语法为:

```cangjie
func ident<T>() {}
```

#### 全局泛型函数

> 在声明全局泛型函数时，只需要在函数名后使用尖括号声明类型形参，然后就可以在函数形参、返回类型及函数体中对这一类型形参进行引用
> 
> 例如 `id` 函数定义为：
> 
> ```cangjie
> func id<T>(a: T): T {
>     return a
> }
> ```
> 
> 其中 `(a: T)` 是函数声明的形参，其中使用到了 `id` 函数声明的类型形参 `T`，并且在 `id` 函数的返回类型使用
> 
> 再比如另一个复杂的例子，定义如下一个泛型函数 `composition`，该函数声明了 `3` 个类型形参，分别是 `T1`, `T2`, `T3`，其功能是把两个函数 `f: (T1) -> T2`, `g: (T2) -> T3` 复合成类型为 `(T1) -> T3` 的函数
> 
> ```cangjie
> func composition<T1, T2, T3>(f: (T1) -> T2, g: (T2) -> T3): (T1) -> T3 {
>     return {x: T1 => g(f(x))}
> }
> ```
> 
> 因为可以被用来复合的函数是任意类型，例如可以是 `(Int32) -> Bool`, `(Bool) -> Int64` 的复合，也可以是 `(Int64) -> Rune`, `(Rune) -> Int8` 的复合，所以才需要使用泛型函数
> 
> ```cangjie
> func times2(a: Int64): Int64 {
>     return a * 2
> }
> 
> func plus10(a: Int64): Int64 {
>     return a + 10
> }
> 
> func times2plus10(a: Int64) {
>     return composition<Int64, Int64, Int64>(times2, plus10)(a)
> }
> 
> main() {
>   println(times2plus10(9))
>   return 0
> }
> ```
> 
> 这里，复合两个 `(Int64) -> Int64`的函数，将 `9` 先乘以 `2`，再加 `10`，结果会是 `28`
> 
> ```text
> 28
> ```

#### 局部泛型函数

> 局部函数也可以是泛型函数
> 
> 例如泛型函数 `id` 可以嵌套定义在其他函数中：
> 
> ```cangjie
> func foo(a: Int64) {
>     func id<T>(a: T): T { a }
> 
>     func double(a: Int64): Int64 { a + a }
> 
>     return (id<Int64> ~> double)(a) == (double ~> id<Int64>)(a)
> }
> 
> main() {
>     println(foo(1))
>     return 0
> }
> ```
> 
> 这里由于 `id` 的单位元性质，函数 `id<Int64> ~> double`和 `double ~> id<Int64>` 是等价的，结果是 `true`
> 
> ```text
> true
> ```

#### 泛型成员函数

> `class`、`struct`、`enum` 与 `interface` 的成员函数可以是泛型的
> 
> 例如：
> 
> ```cangjie
> class A {
>     func foo<T>(a: T): Unit where T <: ToString {
>         println("${a}")
>     }
> }
> 
> struct B {
>     func bar<T>(a: T): Unit where T <: ToString {
>         println("${a}")
>     }
> }
> 
> enum C {
>     | X | Y
> 
>     func coo<T>(a: T): Unit where T <: ToString {
>         println("${a}")
>     }
> }
> 
> interface I {
>     func doo<T>(a: T): Unit where T <: ToString
> }
> 
> class D <: I {
>     public func doo<T>(a: T): Unit where T <: ToString {
>         println("${a}")
>     }
> }
> 
> abstract class E {
>     public func eoo1<T>(a: T): Unit where T <: ToString
>     public open func eoo2<T>(a: T): Unit where T <: ToString
> }
> 
> class F <: E {
>     public func eoo1<T>(a: T): Unit where T <: ToString {
>         println("${a}")
>     }
>     public func eoo2<T>(a: T): Unit where T <: ToString {
>         println("${a}")
>     }
> }
> 
> main() {
>     var a = A()
>     var b = B()
>     var c = C.X
>     var d = D()
>     var f = F()
>     a.foo<Int64>(10)
>     b.bar<String>("abc")
>     c.coo<Bool>(false)
>     d.doo<String>("doo")
>     f.eoo1<String>("eoo1")
>     f.eoo2<String>("eoo2")
>     return 0
> }
> ```
> 
> 程序输出的结果为：
> 
> ```text
> 10
> abc
> false
> doo
> eoo1
> eoo2
> ```
> 
> 在为类型使用 `extend` 声明进行扩展时，扩展中的函数也可以是泛型的，例如可以为 `Int64` 类型增加一个泛型成员函数：
> 
> ```cangjie
> extend Int64 {
>     func printIntAndArg<T>(a: T) where T <: ToString {
>         println(this)
>         println("${a}")
>     }
> }
> 
> main() {
>     var a: Int64 = 12
>     a.printIntAndArg<String>("twelve")
> }
> ```
> 
> 程序输出的结果将为：
> 
> ```text
> 12
> twelve
> ```

这个章节使用了泛型约束, 即 使用`where T <: OtherType` 语法, 约束类型形参`T`需要属于`OtherType`的子类型

然后属于类型变元类型的变量可以访问`OtherType`的成员

#### 静态泛型函数

> `interface`、`class`、`struct`、`enum` 与 `extend` 中可以定义静态泛型函数，例如下例 `ToPair class` 中从 `ArrayList` 中返回一个元组：
> 
> ```cangjie
> import std.collection.*
> 
> class ToPair {
>     public static func fromArray<T>(l: ArrayList<T>): (T, T) {
>         return (l[0], l[1])
>     }
> }
> 
> main() {
>     var res: ArrayList<Int64> = ArrayList([1,2,3,4])
>     var a: (Int64, Int64) = ToPair.fromArray<Int64>(res)
>     return 0
> }
> ```

---

泛型函数其实没有什么需要特别注意的地方

只不过是由本身拥有具体类型的函数, 可能变成了需要手动指定类型使用的函数

在定义上, 只不过是 将具体类型变成了 类型参数, 其他只要符合仓颉语法都没有问题

在调用上, 大多情况甚至不用手动指定类型实参, 可以由编译器进行推导
