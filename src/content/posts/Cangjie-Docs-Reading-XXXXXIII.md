---
title: "仓颉文档阅读-开发指南VIII: 泛型(III) - 泛型的子类型关系、类型别名以及泛型约束"
published: 2025-11-24 17:36:32
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

### 泛型类型的子类型关系 **

> 实例化后的泛型类型间也有子类型关系
> 
> 例如：
> 
> ```cangjie
> interface I<X, Y> { }
> 
> class C<Z> <: I<Z, Z> { }
> ```
> 
> 根据 `class C<Z> <: I<Z, Z> { }`，便知 `C<Bool> <: I<Bool, Bool>` 以及 `C<D> <: I<D, D>` 等
> 
> 可以解读为“于所有的(不含类型变元的)`Z`类型，都有`C<Z> <: I<Z, Z>`成立”
> 
> 但是对于下列代码：
> 
> ```cangjie
> open class C { }
> class D <: C { }
> 
> interface I<X> { }
> ```
> 
> **`I<D> <: I<C>`是不成立的**(即使 `D <: C` 成立)，这是因为在仓颉语言中，用户定义的类型构造器 在其类型参数处是**不型变**的
> 
> **型变的具体定义为**：
> 
> 如果`A`和`B`是(实例化后的)类型，`T`是类型构造器，设有一个类型参数`X`(例如`interface T<X>`)，那么
> 
> - 如果`T(A) <: T(B)`当且仅当`A = B`，则`T`是不型变的
> 
> - 如果`T(A) <: T(B)`当且仅当`A <: B` ，则`T`在`X`处是协变的
> 
> - 如果`T(A) <: T(B)`当且仅当`B <: A` ，则`T`在`X`处是逆变的
> 
> 因为现阶段的仓颉中，**所有 用户自定义的泛型类型 在其所有的类型变元处都是不变的**
> 
> 所以给定`interface I<X>`和类型`A`、`B`，只有`A = B`，才能得到 `I<A> <: I<B>`
> 
> 反过来，如果知道了 `I<A> <: I<B>`，也可推出 `A = B`
> 
> 内建类型除外：**内建的元组类型**对其每个元素类型来说，都是**协变**的；**内建的函数类型**在其入参类型处是**逆变**的，在其返回类型处是**协变**的
> 
> > 注意：
> > 
> > `class` 以外的类型实现接口，该类型和该接口之间的子类型关系不能作为协变和逆变的依据
> 
> 不型变限制了一些语言的表达能力，但也避免了一些安全问题，例如“协变数组运行时抛异常”的问题

仓颉泛型的父子类型关系, 与一个概念密切相关: **型变**

关于**型变** 仓颉规定, 针对自定义类型`Type`, 如果存在类型形参`X`, 以及另外的已实例化类型`A`和`B`:

1. 如果 有且仅有`A = B`时, 才存在`Type(A) <: Type(B)`, 那么 可称`Type`是**不型变**的

2. 如果 有且仅有`A <: B`时, 才存在`Type(A) <: Type(B)`, 那么 可称`Type`是**协变**的

3. 如果 有且仅有`B <: A`时, 才存在`Type(A) <: Type(B)`, 那么 可称`Type`是**逆变**的

同时仓颉规定, **现阶段, 所有用户自定义类型 都是不型变的**

这也就意味着, 如果类型形参不同, 那么泛型自定义类型之间不可能存在父子类型关系

> [!TIP]
> 
> 文档描述, 内建类型除外:
> 
> 1. 内建元组类型 对其每个元素类型, 都是协变的
> 
> 2. 内建函数类型 对其形参列表类型是逆变的, 对其返回类型是协变的
> 
> 这表示, 元组类型, 只要对应位置元素类型具有同向的父子类型关系, 那么元组类型之间就具有相同的父子类型关系
> 
> 而 函数类型, 只要 参数列表每个形参对应位置类型 具有同向的父子类型关系, 且 返回值 具有与参数列表相反的父子类型关系, 那么 函数类型就具有与返回值类型相同的父子类型关系

### 类型别名

当某个类型的名字比较复杂或者在特定场景中不够直观时，可以选择使用类型别名的方式为此类型设置一个别名

```cangjie
type I64 = Int64
```

类型别名的定义以关键字 `type` 开头，接着是类型的别名（如上例中的 `I64`），然后是等号 `=`，最后是原类型（即被取别名的类型，如上例中的 `Int64`）

**只能在源文件顶层定义类型别名，并且原类型必须在别名定义处可见**

例如，下例中 `Int64` 的别名定义在 `main` 中将报错，`LongNameClassB` 类型在为其定义别名时不可见，同样报错

main() {
    type I64 = Int64 // Error, type aliases can only be defined at the top level of the source file
}

class LongNameClassA { }
type B = LongNameClassB // Error, type 'LongNameClassB' is not defined

一个（或多个）类型别名定义中禁止出现（直接或间接的）循环引用。

type A = (Int64, A) // Error, 'A' refered itself
type B = (Int64, C) // Error, 'B' and 'C' are circularly refered
type C = (B, Int64)

类型别名并不会定义一个新的类型，它仅仅是为原类型定义了另外一个名字，它有如下几种使用场景：

    作为类型使用，例如：

type A = B
class B {}
var a: A = B() // Use typealias A as type B

当类型别名实际指向的类型为 class、struct 时，可以作为构造器名称使用：

type A = B
class B {}
func foo() { A() }  // Use type alias A as constructor of B

当类型别名实际指向的类型为 class、interface、struct 时，可以作为访问内部静态成员变量或函数的类型名：

type A = B
class B {
    static var b : Int32 = 0;
    static func foo() {}
}
func foo() {
    A.foo() // Use A to access static method in class B
    A.b
}

当类型别名实际指向的类型为 enum 时，可以作为 enum 声明的构造器的类型名：

    enum TimeUnit {
        Day | Month | Year
    }
    type Time = TimeUnit
    var a = Time.Day  
    var b = Time.Month   // Use type alias Time to access constructors in TimeUnit

需要注意的是，当前用户自定义的类型别名暂不支持在类型转换表达式中使用，参考如下示例：

type MyInt = Int32
MyInt(0)  // Error, no matching function for operator '()' function call

泛型别名

类型别名也是可以声明类型形参的，但是不能对其形参使用 where 声明约束，对于泛型变元的约束会在后面给出解释。

当一个泛型类型的名称过长时，可以使用类型别名来为其声明一个更短的别名。例如，有一个类型为 RecordData ，可以把他用类型别名简写为 RD ：

struct RecordData<T> {
    var a: T
    public init(x: T){
        a = x
    }
}

type RD<T> = RecordData<T>

main(): Int64 {
    var struct1: RD<Int32> = RecordData<Int32>(2)
    return 1
}

在使用时就可以用 RD<Int32> 来代指 RecordData<Int32> 类型。
