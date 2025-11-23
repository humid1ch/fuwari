---
title: "仓颉文档阅读-开发指南VII: 类和接口(V) - 子类型关系 以及 类型转换"
published: 2025-11-21 18:08:28
description: '仓颉文档阅读的开发指南部分, 本篇文章介绍仓颉语言的 子类型之间的关系以及转换 的相关内容'
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

## 类和接口

### 子类型关系

> 与其他面向对象语言一样，仓颉语言提供子类型关系和子类型多态
> 
> 举例说明（不限于下述用例）：
> 
> - 假设函数的形参是类型`T`，则函数调用时传入的参数的实际类型既可以是`T`也可以是`T`的子类型（严格地说，`T`的子类型已经包括`T`自身，下同）
> 
> - 假设赋值表达式`=`左侧的变量的类型是`T`，则`=`右侧的表达式的实际类型既可以是`T`也可以是`T`的子类型
> 
> - 假设函数定义中用户标注的返回类型是`T`，则函数体的类型（以及函数体内所有`return`表达式的类型）既可以是`T`也可以是`T`的子类型
> 
> 下文将说明两个类型为子类型关系的几种情况

仓颉是强类型语言, 不同类型之间 是不允许存在 隐式类型转换 的

不过, 某些类型之间是可以通过显式类型转换语法进行转换的, 比如`Int64`与`Float64`之间的转换

除此之外, 实际上 **拥有父子类型关系的类型, 仓颉是允许隐式类型转换的**

但, **只允许子类型转换为父类型, 父类型禁止转换为子类型**

#### 继承 `class` 带来的子类型关系

> 继承 `class` 后，子类即为父类的子类型
> 
> 如下代码中，`Sub`即为 `Super` 的子类型
> 
> ```cangjie
> open class Super { }
> class Sub <: Super { }
> ```

存在继承关系的类型, 默认具有父子类型关系, **父类为父类型 子类为子类型**

#### 实现接口带来的子类型关系

> 实现接口（含扩展实现）后，实现接口的类型即为接口的子类型
> 
> 如下代码中，`I3` 是 `I1` 和 `I2` 的子类型，`C` 是 `I1` 的子类型， `Int64` 是 `I2` 的子类型：
> 
> ```cangjie
> interface I1 { }
> interface I2 { }
> 
> interface I3 <: I1 & I2 { }
> 
> class C <: I1 { }
> 
> extend Int64 <: I2 { }
> ```

一个类型实现了一个接口, 则此类型与接口之间默认具有父子类型关系, **接口为父类型 实现接口的类型为子类型**

#### 元组类型的子类型关系

> 仓颉语言中的元组类型也有子类型关系
> 
> 直观的，如果一个元组 `t1` 的每个元素的类型都是另一个元组 `t2` 的对应位置元素类型的子类型，那么元组 `t1` 的类型也是元组 `t2` 的类型的子类型
> 
> 例如下面的代码中，由于 `C2 <: C1` 和 `C4 <: C3`，因此也有 `(C2, C4) <: (C1, C3)` 以及 `(C4, C2) <: (C3, C1)`
> 
> ```cangjie
> open class C1 { }
> class C2 <: C1 { }
> 
> open class C3 { }
> class C4 <: C3 { }
> 
> let t1: (C1, C3) = (C2(), C4())       // OK
> let t2: (C3, C1) = (C4(), C2())       // OK
> ```

元组类型也存在父子类型关系, 因为元组类型是基于其他数据类型的, 所以 **当元组内 元素的类型均具有同"方向"的父子类型关系时, 则 元组类型之间也具有同"方向"的父子类型关系**

什么意思呢?

如果存在元组类型: `(Type1, Type2)` 和 `(Type3, Type4)`, 如果 `Type1 <: Type3`且`Type2 <: Type4`, 则`(Type1, Type2) <: (Type3, Type4)`

元组类型之间如果想要存在父子类型关系, 需要保证 元组的所有元素类型, 按照顺序具有同"方向"的父子类型关系

#### 函数类型的子类型关系

> 仓颉语言中，函数是一等公民，而函数类型亦有子类型关系：给定两个函数类型 `(U1) -> S2` 和 `(U2) -> S1`，如果存在 `(U1) -> S2` 是 `(U2) -> S1`的子类型，当且仅当 `U2` 是 `U1` 的子类型，且 `S2` 是 `S1` 的子类型（注意顺序）
> 
> 例如下面的代码定义了两个函数 `f : (U1) -> S2` 和 `g : (U2) -> S1`，且 `f` 的类型是 `g` 的类型的子类型
> 
> 由于 `f` 的类型是 `g` 的子类型，所以代码中使用到 `g` 的地方都可以换为 `f`
> 
> ```cangjie
> open class U1 { }
> class U2 <: U1 { }
> 
> open class S1 { }
> class S2 <: S1 { }
> 
> 
> func f(a: U1): S2 { S2() }
> func g(a: U2): S1 { S1() }
> 
> func call1() {
>     g(U2())                   // Ok
>     f(U2())                   // Ok
> }
> 
> func h(lam: (U2) -> S1): S1 {
>     lam(U2())
> }
> 
> func call2() {
>     h(g)                      // Ok
>     h(f)                      // Ok
> }
> ```
> 
> 对于上面的规则，`S2 <: S1` 部分很好理解：函数调用产生的结果数据会被后续程序使用，函数 `g` 可以产生 `S1` 类型的结果数据，函数 `f` 可以产生 `S2` 类型的结果，而 `g` 产生的结果数据应当能被 `f` 产生的结果数据替代，因此要求 `S2 <: S1`
> 
> 对于 `U2 <: U1` 的部分，可以这样理解：在函数调用产生结果前，它本身应当能够被调用，函数调用的实参类型固定不变，同时形参类型要求更宽松时，依然可以被调用，而形参类型要求更严格时可能无法被调用——例如给定上述代码中的定义 `g(U2())` 可以被换为 `f(U2())`，正是因为实参类型 `U2` 的要求更严格于形参类型 `U1`

仓颉是强类型语言, 函数是拥有具体类型的, 而函数类型之间同样可以具有父子类型关系

文档中的例子描述的有一些混乱

如果存在两个函数类型: `(U1) -> S1` 和 `(U2) -> S2`

只有在, `U1`为`U2`的子类型, 且`S2`为`S1`的子类型时, `(U2) -> S2`才为`(U1) -> S1`的子类型

即, 函数类型之间如果要存在父子类型关系, 参数列表的每个形参之间必须存在同"方向"的父子类型关系, 且 **返回值类型之间 必须存在与 参数列表父子"方向"相反的父子类型关系**

即, 如果参数列表`U1 <: U2`, 那么 只有`S2 <: S1`, 两个函数类型之间才存在`(U2) -> S2 <: (U1) -> S1`

反之, 则是 如果参数列表`U2 <: U1`, 那么 只有`S1 <: S2`, 才存在`(U1) -> S1 <: (U2) -> S2`

可以发现, **函数类型之间的父子类型 跟随返回值之间的父子类型方向**

总结, **函数类型之间, 只有 两参数列表存在父子类型关系, 且返回值之间存在 与 参数列表相反的父子类型关系时, 函数类型才会存在与返回值类型相同的父子类型关系**

#### 永远成立的子类型关系

> 仓颉语言中，有些预设的子类型关系是永远成立的：
> 
> - 一个类型 `T` 永远是自身的子类型，即 `T <: T`
> 
> - `Nothing` 类型永远是其他任意类型 `T` 的子类型，即 `Nothing <: T`
> 
> - 任意类型 `T` 都是 `Any` 类型的子类型，即 `T <: Any`
> 
> - 任意 `class` 定义的类型都是 `Object` 的子类型，即如果有 `class C {}`，则 `C <: Object`

仓颉中存在两个最基础类型: `Any`和`Object`:

1. `Any`是接口类型, 所有类型都是`Any`的子类型(`Object`除外)

2. `Object`是`class`类型, 所有`class`都是`Object`的子类型(`Any`除外)

其次, 还存在一个`Nothing`类型, 它是所有类型的子类型

#### 传递性带来的子类型关系

> 子类型关系具有传递性
> 
> 如下代码中，虽然只描述了 `I2 <: I1`、`C <: I2` 以及 `Bool <: I2`，但根据子类型的传递性，也隐式存在 `C <: I1` 以及 `Bool <: I1` 这两个子类型关系
> 
> ```cangjie
> interface I1 { }
> interface I2 <: I1 { }
> 
> class C <: I2 { }
> 
> extend Bool <: I2 { }
> ```

仓颉子类型关系具有传递性, 比较容易理解

简单点说, 如果`Type1`与`Type2`之间存在父子类型关系, 且`Type2`与`Type3`之间也存在父子类型关系, 那么`Type1`和`Type3`之间**可能**也存在父子类型关系

具体是否存在, 就看两种父子类型关系的方向了, 并不复杂

#### 泛型类型的子类型关系

> 泛型类型间也有子类型关系，详见 [泛型类型的子类型关系]()

泛型类型之间的父子类型关系, 具体到时候在了解吧

### 类型转换

> 仓颉不支持不同类型之间的隐式转换（子类型天然是父类型，所以子类型到父类型的转换不是隐式类型转换），类型转换必须显式地进行
> 
> 下面将依次介绍数值类型之间的转换，`Rune` 到 `UInt32` 和整数类型到 `Rune` 的转换，以及 `is` 和 `as` 操作符

#### 数值类型之间的转换

> 对于数值类型（包括：`Int8`，`Int16`，`Int32`，`Int64`，`IntNative`，`UInt8`，`UInt16`，`UInt32`，`UInt64`，`UIntNative`，`Float16`，`Float32`，`Float64`），仓颉支持使用 `T(e)` 的方式得到一个值等于 `e`，类型为 `T` 的值
> 
> 其中，表达式`e` 的类型和 `T` 可以是上述任意数值类型
> 
> 下面的例子展示了数值类型之间的类型转换：
> 
> ```cangjie
> main() {
>     let a: Int8 = 10
>     let b: Int16 = 20
>     let r1 = Int16(a)
>     println("The type of r1 is 'Int16', and r1 = ${r1}")
>     let r2 = Int8(b)
>     println("The type of r2 is 'Int8', and r2 = ${r2}")
> 
>     let c: Float32 = 1.0
>     let d: Float64 = 1.123456789
>     let r3 = Float64(c)
>     println("The type of r3 is 'Float64', and r3 = ${r3}")
>     let r4 = Float32(d)
>     println("The type of r4 is 'Float32', and r4 = ${r4}")
> 
>     let e: Int64 = 1024
>     let f: Float64 = 1024.1024
>     let r5 = Float64(e)
>     println("The type of r5 is 'Float64', and r5 = ${r5}")
>     let r6 = Int64(f)
>     println("The type of r6 is 'Int64', and r6 = ${r6}")
> }
> ```
> 
> 上述代码的执行结果为：
> 
> ```text
> The type of r1 is 'Int16', and r1 = 10
> The type of r2 is 'Int8', and r2 = 20
> The type of r3 is 'Float64', and r3 = 1.000000
> The type of r4 is 'Float32', and r4 = 1.123457
> The type of r5 is 'Float64', and r5 = 1024.000000
> The type of r6 is 'Int64', and r6 = 1024
> ```
> 
> > 注意：
> > 
> > 类型转换时可能发生溢出，若溢出可提前在编译器检测出来，则编译器会直接给出报错，否则根据默认的溢出策略将抛出异常。

#### `Rune` 到 `UInt32` 和整数类型到 `Rune` 的转换

> `Rune` 到 `UInt32` 的转换使用 `UInt32(e)` 的方式，其中 `e` 是一个 `Rune`类型的表达式，`UInt32(e)` 的结果是 `e` 的 `Unicode scalar value` 对应的 `UInt32` 类型的整数值
> 
> 整数类型到 `Rune` 的转换使用 `Rune(num)` 的方式，其中 `num` 的类型可以是任意的整数类型，且仅当 `num` 的值落在 `[0x0000, 0xD7FF]` 或 `[0xE000, 0x10FFFF]` （即 `Unicode scalar value`）中时，返回对应的 `Unicode scalar value` 表示的字符，否则，编译报错（编译时可确定 `num` 的值）或运行时抛异常
> 
> 下面的例子展示了 `Rune` 和 `UInt32` 之间的类型转换：
> 
> ```cangjie
> main() {
>     let x: Rune = 'a'
>     let y: UInt32 = 65
>     let r1 = UInt32(x)
>     let r2 = Rune(y)
>     println("The type of r1 is 'UInt32', and r1 = ${r1}")
>     println("The type of r2 is 'Rune', and r2 = ${r2}")
> }
> ```
> 
> 上述代码的执行结果为：
> 
> ```text
> The type of r1 is 'UInt32', and r1 = 97
> The type of r2 is 'Rune', and r2 = A
> ```

#### `is` 和 `as` 操作符

> 仓颉支持使用 `is` 操作符来判断某个表达式的类型是否是指定的类型（或其子类型)
> 
> 具体而言，对于表达式 `e is T`（`e` 可以是任意表达式，`T` 可以是任何类型），当 `e` 的运行时类型是 `T` 的子类型时，`e is T` 的值为 `true`，否则 `e is T` 的值为 `false`
> 
> 下面的例子展示了 `is` 操作符的使用：
> 
> ```cangjie
> open class Base {
>     var name: String = "Alice"
> }
> class Derived <: Base {
>     var age: UInt8 = 18
> }
> 
> main() {
>     let a = 1 is Int64
>     println("Is the type of 1 'Int64'? ${a}")
>     let b = 1 is String
>     println("Is the type of 1 'String'? ${b}")
> 
>     let b1: Base = Base()
>     let b2: Base = Derived()
>     var x = b1 is Base
>     println("Is the type of b1 'Base'? ${x}")
>     x = b1 is Derived
>     println("Is the type of b1 'Derived'? ${x}")
>     x = b2 is Base
>     println("Is the type of b2 'Base'? ${x}")
>     x = b2 is Derived
>     println("Is the type of b2 'Derived'? ${x}")
> }
> ```
> 
> 上述代码的执行结果为：
> 
> ```text
> Is the type of 1 'Int64'? true
> Is the type of 1 'String'? false
> Is the type of b1 'Base'? true
> Is the type of b1 'Derived'? false
> Is the type of b2 'Base'? true
> Is the type of b2 'Derived'? true
> ```
> 
> `as` 操作符可以用于将某个表达式的类型转换为指定的类型
> 
> 因为类型转换有可能会失败，所以 `as` 操作返回的是一个 `Option` 类型
> 
> 具体而言，对于表达式 `e as T`（`e` 可以是任意表达式，`T` 可以是任何类型），当 `e` 的运行时类型是`T` 的子类型时，`e as T` 的值为 `Option<T>.Some(e)`，否则 `e as T` 的值为 `Option<T>.None`
> 
> 
> 下面的例子展示了 `as` 操作符的使用（注释中标明了 `as` 操作的结果）：
> 
> ```cangjie
> open class Base {
>     var name: String = "Alice"
> }
> class Derived <: Base {
>     var age: UInt8 = 18
> }
> 
> let a = 1 as Int64     // a = Option<Int64>.Some(1)
> let b = 1 as String    // b = Option<String>.None
> 
> let b1: Base = Base()
> let b2: Base = Derived()
> let d: Derived = Derived()
> let r1 = b1 as Base    // r1 = Option<Base>.Some(b1)
> let r2 = b1 as Derived // r2 = Option<Derived>.None
> let r3 = b2 as Base    // r3 = Option<Base>.Some(b2)
> let r4 = b2 as Derived // r4 = Option<Derived>.Some(b2)
> let r5 = d as Base     // r5 = Option<Base>.Some(d)
> let r6 = d as Derived  // r6 = Option<Derived>.Some(d)
> ```
