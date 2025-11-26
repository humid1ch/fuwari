---
title: "仓颉文档阅读-开发指南VIV: 扩展(I) - 扩展概述、直接扩展与接口扩展"
published: 2025-11-25 16:48:33
description: '仓颉文档阅读的开发指南部分, 本篇文章介绍仓颉语言的 扩展 的相关内容'
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

## 扩展

### 扩展概述

> 扩展可以为在当前`package`中可见的类型（除函数、元组、接口）添加新功能
> 
> 当不能破坏被扩展类型的封装性，但希望添加额外的功能时，可以使用扩展
> 
> 可以添加的功能包括：
> 
> - 添加成员函数
> 
> - 添加操作符重载函数
> 
> - 添加成员属性
> 
> - 实现接口
> 
> 扩展虽然可以添加额外的功能，但不能变更被扩展类型的封装性，因此扩展不支持以下功能：
> 
> - 扩展不能增加成员变量
> 
> - 扩展的函数和属性必须拥有实现
> 
> - 扩展的函数和属性不能使用 `open`、`override`、`redef`修饰
> 
> - 扩展不能访问被扩展类型中 `private` 修饰的成员
> 
> 根据扩展有没有实现新的接口，扩展可以分为 **直接扩展** 和 **接口扩展** 两种用法
> 
> 直接扩展即不包含额外接口的扩展
> 
> 接口扩展即包含接口的扩展，接口扩展可以用来为现有的类型添加新功能并实现接口，增强抽象灵活性

扩展就是**为已经定义好的类型**, 新增 成员函数、操作符重载、成员属性 或 扩展接口实现

只要类型可见, 都可以进行扩展, 即 即使是第三方库, 也可以实现扩展

但, 扩展存在一些限制:

1. 禁止 新增成员**变量**

2. 扩展函数和属性时, 必须实现, 不能只声明

3. 扩展的成员, 不能是`open`、`override`、`redef`修饰的

4. 扩展 **不能访问原`private`成员**

### 直接扩展

> 一个简单的扩展语法结构示例如下：
> 
> ```cangjie
> extend String {
>     public func printSize() {
>         println("the size is ${this.size}")
>     }
> }
> ```
> 
> 如上例所示，扩展使用 `extend` 关键字声明，其后跟着被扩展的类型 `String` 和扩展的功能
> 
> 当为 `String` 扩展了 `printSize` 函数之后，就能在当前 `package` 内对 `String` 的实例访问该函数，就像是 `String` 本身具备该函数
> 
> ```cangjie
> main() {
>     let a = "123"
>     a.printSize()         // the size is 3
> }
> ```
> 
> 编译执行上述代码，输出结果为：
> 
> ```text
> the size is 3
> ```

扩展使用`extend`关键字

基本语法为:

```cangjie
extend 目标类型名 {
    // 要扩展的成员
}
```

扩展之后, 就能在当前`package`通过类型示例访问扩展的成员

> 被扩展类型是泛型类型时，有两种扩展语法可以对泛型类型扩展功能
> 
> 一种是 **针对特定泛型实例化类型** 进行扩展，关键字`extend`后允许带一个任意实例化完全的泛型类型
> 
> 为这些类型增加的功能只有在类型完全匹配时才能使用，且泛型类型的类型实参必须符合泛型类型定义处的约束要求
> 
> 例如下面所示的 `Foo<T>`
> 
> ```cangjie
> class Foo<T> where T <: ToString {}
> 
> extend Foo<Int64> {}      // Ok
> 
> class Bar {}
> extend Foo<Bar> {}        // Error
> ```

扩展泛型类型, 可以针对 **特定实例化完全的泛型类型** 进行扩展

即, 针对 **已经指定了泛型实参的泛型类型** 进行扩展

**指定泛型实参要满足泛型约束**

> 另一种是在 **`extend`后面引入泛型形参的泛型扩展**
> 
> 泛型扩展可以用来扩展未实例化或未完全实例化的泛型类型
> 
> 在`extend`后声明的泛型形参必须被直接或间接使用在被扩展的泛型类型上
> 
> 为这些类型增加的功能只有在类型和约束完全匹配时才能使用
> 
> 例如下面所示的 `MyList<T>`
> 
> ```cangjie
> class MyList<T> {
>     public let data: Array<T> = Array<T>()
> }
> 
> extend<T> MyList<T> {}                // OK
> extend<R> MyList<R> {}                // OK
> extend<T, R> MyList<(T, R)> {}        // OK
> extend MyList {}                      // Error
> extend<T, R> MyList<T> {}             // Error
> extend<T, R> MyList<T, R> {}          // Error
> ```

扩展泛型类型, 还可以针对 **未实例化 或 未实例化完全 的泛型类型** 进行扩展

从文档来看, 扩展未实例化或未实例化完全的泛型类型, 我个人理解 语法为:

```cangjie
extend<扩展 类型形参列表> 扩展类型<类型实参列表> {}
```

要注意的是 这里扩展类型的类型实参, 并不是具体的类型, 而是 **使用 扩展类型形参列表中的形参 构建的类型**

且, 类型实参列表中, 要包含所有的扩展类型形参

有点不好理解, 根据例子具体分析一下:

```cangjie
class MyList<T> {
    public let data: Array<T> = Array<T>()
}
```

`MyList`拥有一个类型形参

那么针对`MyList`未实例化的类型 进行扩展, 扩展类型形参 不需要与 原泛型类型的类型形参 严格保持一致:

1. 可以按照原类型形参扩展:
    
    ```cangjie
    extend<T> MyList<T> {}
    ```

2. 也可以声明多个扩展类型形参:

    ```cangjie
    extend<T1, T2> MyList<(T1, T2)> {}
    extend<T1, T2, T3> MyList<(T1, T2, T3)> {}
    extend<T1, T2, T3, T4> MyList<(T1, T2, T3, T4)> {}
    extend<T1, T2, T3, T4, T5> MyList<(T1, T2, T3, T4, T5)> {}
    ```

    这几种方式都可以

    但是, **扩展类型的类型实参列表中, 实参个数必须与原类型的类型形参个数保持一致**

    当然**不是只能传入构成的元组类型, 还可以构成其他单个类型传入**

> 对于泛型类型的扩展，**可以在其中声明额外的泛型约束，来实现一些有限情况下才能使用的函数**
> 
> 例如可以定义一个叫 `Pair` 的类型，这个类型可以方便地存储两个元素（类似于 `Tuple`）
> 
> 希望 `Pair` 类型可以容纳任何类型，因此两个泛型变元不应该有任何约束，这样才能保证 `Pair` 能容纳所有类型
> 
> 但同时又希望当两个元素可以判等的时候，让 `Pair` 也可以判等，这时就可以用扩展来实现这个功能
> 
> 如下面的代码所示，使用扩展语法，约束了 `T1` 和 `T2` 在支持 `equals` 的情况下，`Pair` 也可以实现 `equals` 函数
> 
> ```cangjie
> class Pair<T1, T2> {
>     var first: T1
>     var second: T2
>     public init(a: T1, b: T2) {
>         first = a
>         second = b
>     }
> }
> 
> interface Eq<T> {
>     func equals(other: T): Bool
> }
> 
> extend<T1, T2> Pair<T1, T2> where T1 <: Eq<T1>, T2 <: Eq<T2> {
>     public func equals(other: Pair<T1, T2>) {
>         first.equals(other.first) && second.equals(other.second)
>     }
> }
> 
> class Foo <: Eq<Foo> {
>     public func equals(other: Foo): Bool {
>         true
>     }
> }
> 
> main() {
>     let a = Pair(Foo(), Foo())
>     let b = Pair(Foo(), Foo())
>     println(a.equals(b))              // true
> }
> ```
> 
> 编译执行上述代码，输出结果为：
> 
> ```text
> true
> ```

扩展泛型类型时, 也可以针对扩展 添加泛型约束

**添加了泛型约束的扩展, 只有在满足约束时 才能访问扩展的成员**

以上的内容都是直接扩展

扩展还可以 实现接口, 被称为 接口扩展

### 接口扩展

> 例如下面的例子，类型 `Array` 本身没有实现接口 `PrintSizeable`，但可以通过扩展的方式为 `Array` 增加额外的成员函数 `printSize`，并实现 `PrintSizeable`
> 
> ```cangjie
> interface PrintSizeable {
>     func printSize(): Unit
> }
> 
> extend<T> Array<T> <: PrintSizeable {
>     public func printSize() {
>         println("The size is ${this.size}")
>     }
> }
> ```
> 
> 当使用扩展为 `Array` 实现 `PrintSizeable` 之后，就相当于在 `Array` 定义时实现接口 `PrintSizeable`
> 
> 因此可以将 `Array` 作为 `PrintSizeable` 的实现类型来使用，代码如下所示
> 
> ```cangjie
> main() {
>     let a: PrintSizeable = Array<Int64>()
>     a.printSize() // 0
> }
> ```
> 
> 编译执行上述代码，输出结果为：
> 
> ```text
> The size is 0
> ```
> 
> 可以在同一个扩展内同时实现多个接口，多个接口之间使用 `&` 分开，接口的顺序没有先后关系
> 
> 如下面代码所示，可以在扩展中为 `Foo` 同时实现 `I1`、`I2`、`I3`
> 
> ```cangjie
> interface I1 {
>     func f1(): Unit
> }
> 
> interface I2 {
>     func f2(): Unit
> }
> 
> interface I3 {
>     func f3(): Unit
> }
> 
> class Foo {}
> 
> extend Foo <: I1 & I2 & I3 {
>     public func f1(): Unit {}
>     public func f2(): Unit {}
>     public func f3(): Unit {}
> }
> ```

接口扩展, 就是在扩展时 使用实现接口的语法 让目标类型实现接口

```cangjie
extend 类型名 <: 接口1 & 接口2 & 接口3 {}
```

与定义类型时 实现接口的语法是一致的

> 也可以在接口扩展中声明额外的泛型约束，来实现一些特定约束下才能满足的接口
> 
> 例如可以让上面的 `Pair` 类型实现 `Eq` 接口，这样 `Pair` 自己也能成为一个符合 `Eq` 约束的类型，如下代码所示
> 
> ```cangjie
> class Pair<T1, T2> {
>     var first: T1
>     var second: T2
>     public init(a: T1, b: T2) {
>         first = a
>         second = b
>     }
> }
> 
> interface Eq<T> {
>     func equals(other: T): Bool
> }
> 
> extend<T1, T2> Pair<T1, T2> <: Eq<Pair<T1, T2>> where T1 <: Eq<T1>, T2 <: Eq<T2> {
>     public func equals(other: Pair<T1, T2>) {
>         first.equals(other.first) && second.equals(other.second)
>     }
> }
> 
> class Foo <: Eq<Foo> {
>     public func equals(other: Foo): Bool {
>         true
>     }
> }
> 
> main() {
>     let a = Pair(Foo(), Foo())
>     let b = Pair(Foo(), Foo())
>     println(a.equals(b)) // true
> }
> ```
> 
> 编译执行上述代码，输出结果为：
> 
> ```text
> true
> ```

接口扩展也可以扩展泛型接口, 同时也可以对泛型形参进行泛型约束

语法上, 与直接扩展存在相同的特点

> **如果被扩展的类型已经包含接口要求的函数或属性，那么在扩展中不需要并且也不能重新实现这些函数或属性**
> 
> 例如下面的例子，定义了一个新接口 `Sizeable`，目的是获取某个类型的 `size`，而已经知道 `Array` 中包含了这个函数，因此就可以通过扩展让 `Array` 实现 `Sizeable`，而不需要添加额外的函数
> 
> ```cangjie
> interface Sizeable {
>     prop size: Int64
> }
> 
> extend<T> Array<T> <: Sizeable {}
> 
> main() {
>     let a: Sizeable = Array<Int64>()
>     println(a.size)
> }
> ```
> 
> 编译执行上述代码，输出结果为：
> 
> ```text
> 0
> ```

接口扩展时, 有可能出现 目标类型已经包含 要扩展实现的接口中的成员函数或属性

此时, **扩展中不能重新实现这部分函数或属性**

> 当**多个接口扩展**实现的**接口存在继承关系**时，扩展将按照“先检查实现父接口的扩展，再检查子接口的扩展”的顺序进行检查
> 
> 例如，接口 `I1` 存在一个子接口 `I2`，且 `I1` 中包含一个默认实现，类型 `A` 的两个扩展分别实现了父子接口，根据以上检查顺序，实现 `I1` 的扩展将会优先检查，然后再检查实现 `I2` 的扩展
> 
> ```cangjie
> interface I1 {
>     func foo(): Unit { println("I1 foo") }
> }
> interface I2 <: I1 {
>     func foo(): Unit { println("I2 foo") }
> }
> 
> class A {}
> 
> extend A <: I1 {}     // 先检查
> extend A <: I2 {}     // 再检查
> 
> main() {
>     A().foo()
> }
> ```
> 
> 编译执行上述代码，输出结果为：
> 
> ```text
> I2 foo
> ```
> 
> 以上例子中，当检查实现 `I1` 的扩展时，会从 `I1` 中继承 `foo` 函数
> 
> 在检查实现 `I2` 的扩展时，由于 `A` 中已存在一个继承的，且签名相同的默认实现 `foo`，此时 `foo` 将被覆盖
> 
> 因此，调用 `A` 的 `foo` 函数时，最终指向 `I2`（子接口）中的实现

仓颉接口继承, 同一个类型 进行多个接口扩展, 且 接口存在继承关系时

编译器将会按照 **先实现父接口扩展, 再实现子接口扩展** 的顺序进行检查

最终, 如果 子接口扩展 实现与父接口的同名成员, **父接口扩展的同名成员将会被覆盖**

这意味着, 如果父子接口扩展 存在实现同名成员时, 最终类型中将会使用子接口中的成员

> 如果同一类型的两个接口扩展实现的接口存在继承冲突，导致无法确定检查顺序时，将会报错
> 
> ```cangjie
> interface I1 {}
> interface I2 <: I1 {}
> interface I3 {}
> interface I4 <: I3 {}
> 
> class A {}
> extend A <: I1 & I4 {}        // error: 无法决定与下面扩展对比, 哪个扩展首先发生
> extend A <: I2 & I3 {}        // error: 无法决定与上面扩展对比, 哪个扩展首先发生
> ```

关于 实现的接口存在继承冲突

并不是接口本身存在继承冲突, 而是 接口扩展时, 不同扩展之间 要扩展实现的接口存在冲突继承关系

比如, 如果存在`I2 <: I1` 和 `I4 <: I3`, 且要 对一个类型 接口扩展实现这四个接口

就禁止以 `<: I2 & I3`和`<: I1 & I4`的方式进行扩展实现, 因为 `I1`和`I3`属于父接口, `I2`和`I4`属于子接口

仓颉检查接口扩展是 **先检查实现父接口, 再检查实现子接口**的

但`I2 & I3`一个是子接口 一个是父接口, `I1 & I4`同样一个是子接口 一个是父接口

这就会导致 不同接口扩展, 检查顺序无法确定

因为一个接口扩展 同时实现子接口`I2`和父接口`I3`, 另一个接口扩展 同时实现子接口`I1`和父接口`I4`, 并且 `I1 <: I2`且`I3 <: I4`

最终接口扩展 无法确定检查顺序, 所以会报错

> 如果同一类型的两个接口扩展实现的接口不存在继承关系，将会被同时检查
> 
> ```cangjie
> interface I1 {
>     func foo() {}
> }
> interface I2 {
>     func foo() {}
> }
> 
> class A {}
> extend A <: I1 {}         // Error, 多个默认实现，需要在'A'中重新实现'foo'
> extend A <: I2 {}         // Error, 多个默认实现，需要在'A'中重新实现'foo'
> ```

仓颉中, 如果 不同接口扩展 要实现的接口不存在继承关系, 不同接口扩展将会同时检查

如果, 此时要实现的接口 又拥有相同的成员, 那么 扩展将会失败, 因为没有先后检查顺序, 所以存在实现冲突

> > 注意：
> > 
> > 当类 `A` 有个泛型基类 `B<T1,...,Tn>`
> > 
> > `B<T1,...,Tn>` 扩展了一个接口 `I<R1,...,Rn>`
> > 
> > `I<R1,...,Rn>` 带有默认实现的实例或者静态函数（比如 `foo`），该函数没有在 `B<T1,...,Tn>` 及其扩展中被重写，且类 `A` 没有直接实现接口 `I<R1,...,Rn>` 时，通过类 `A` 的实例调用函数 `foo` 时会产生非预期行为
> > 
> > **计划在后续版本修复该问题**
> > ```cangjie
> > interface I<N> {
> >     func foo(n: N): N {n}
> > }
> > 
> > open class B<T> {}
> > 
> > extend<T> B<T> <: I<T> {}
> > 
> > class A <: B<Int64>{}
> > 
> > main() {
> >     A().foo(0)              // 这个调用产生非预期行为
> > }
> > ```
