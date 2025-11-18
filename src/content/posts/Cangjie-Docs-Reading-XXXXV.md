---
title: "仓颉文档阅读-开发指南VI: 枚举类型和模式匹配(II) - 模式的Refutability、match表达式 与 其他使用模式的地方"
published: 2025-11-17 17:53:10
description: '仓颉文档阅读的开发指南部分, 本篇文章介绍仓颉语言的模式匹配相关内容'
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

## 枚举类型和模式匹配

### 模式的`Refutability`

> 模式可以分为两类: `refutable`模式和`irrefutable`模式
> 
> 在**类型匹配的前提下**, 当一个模式有可能和待匹配值不匹配时, 称此模式为`refutable`模式; 反之, 当一个模式总是可以和待匹配值匹配时, 称此模式为`irrefutable`模式

`Refutability`翻译为可证伪性

仓颉将模式分为两类: 可证伪(`refutable`) 和 不可证伪(`irrefutable`)

并解释, 在类型匹配的前提下:

1. `refutable`可证伪, 表示 待匹配值**可能存在无法匹配成功**的情况

2. `irrefutable`不可证伪, 表示 待匹配值**总能匹配成功**的情况

> 对于上述介绍的各种模式, 规定如下: 
> 
> 常量模式是`refutable`模式
> 
> 例如, 下例中第一个`case`中的`1`和 第二个`case`中的`2`都有可能和`x`的值不相等
> 
> ```cangjie
> func constPat(x: Int64) {
>     match (x) {
>         case 1 => "one"
>         case 2 => "two"
>         case _ => "_"
>     }
> }
> ```

常量模式一定为`refutable`的, 即 可证伪的

因为你无法列举出所有的字面量, 更别说一定匹配成功了

> 通配符模式是`irrefutable`模式
> 
> 例如, 下例中无论`x`的值是多少, `_`总能和其匹配
> 
> ```cangjie
> func wildcardPat(x: Int64) {
>     match (x) {
>         case _ => "_"
>     }
> }
> ```
> 
> 绑定模式是`irrefutable`模式
> 
> 例如, 下例中无论`x`的值是多少, 绑定模式`a`总能和其匹配
> 
> ```cangjie
> func varPat(x: Int64) {
>     match (x) {
>         case a => "x = ${a}"
>     }
> }
> ```

通配符模式和绑定模式, 是`irrefutable`不可证伪的

因为他们总能匹配任意值

> `Tuple`模式是`irrefutable`模式, 当且仅当其包含的每个模式都是`irrefutable`模式
> 
> 例如, 下例中`(1, 2)`和`(a, 2)`都有可能和`x`的值不匹配, 所以它们是`refutable`模式, 而`(a, b)`可以匹配任何`x`的值, 所以它是`irrefutable`模式
> 
> ```cangjie
> func tuplePat(x: (Int64, Int64)) {
>     match (x) {
>         case (1, 2) => "(1, 2)"
>         case (a, 2) => "(${a}, 2)"
>         case (a, b) => "(${a}, ${b})"
>     }
> }
> ```

`Tuple`模式, 可以是`refutable`可证伪的, 也可以是`irrefutable`不可证伪的

因为`Tuple`模式依赖于其他模式, 所以 **当且仅当`Tuple`模式包含的每个模式都是`irrefutable`的时, `Tuple`模式是`irrefutable`的**

> 类型模式是`refutable`模式
> 
> 例如, 下例中(假设`Base`是`Derived`的父类, 并且`Base`实现了接口`I`), `x`的运行时类型有可能既不是`Base`也不是`Derived`, 所以`a: Derived`和`b: Base`均是`refutable`模式
> 
> ```cangjie
> interface I {}
> open class Base <: I {}
> class Derived <: Base {}
> 
> func typePat(x: I) {
>     match (x) {
>         case a: Derived => "Derived"
>         case b: Base => "Base"
>         case _ => "Other"
>     }
> }
> ```

类型模式是`refutable`可证伪的

因为, 可能存在待匹配目标为接口类型, 此时可能存在`match`中无法匹配待匹配目标的类型

> `enum`模式是`irrefutable`模式, 当且仅当它对应的`enum`类型中只有一个有参构造器, 且`enum`模式中包含的其他模式也是`irrefutable`模式
> 
> 例如, 对于下例中的`E1`和`E2`定义, 函数`enumPat1`中的`A(1)`是`refutable`模式, `A(a)`是`irrefutable`模式; 而函数`enumPat2`中的`B(b)`和`C(c)`均是`refutable`模式
> 
> ```cangjie
> enum E1 {
>     A(Int64)
> }
> 
> enum E2 {
>     B(Int64) | C(Int64)
> }
> 
> func enumPat1(x: E1) {
>     match (x) {
>         case A(1) => "A(1)"
>         case A(a) => "A(${a})"
>     }
> }
> 
> func enumPat2(x: E2) {
>     match (x) {
>         case B(b) => "B(${b})"
>         case C(c) => "C(${c})"
>     }
> }
> ```

`enum`模式 与`Tuple`模式类似, 都 可以是`refutable`可证伪的, 也可以是`irrefutable`不可证伪的, 因为他们都是要依赖其他模式的

但`enum`还有一点不同的是, `match`表达式中`enum`模式通常需要覆盖目标`enum`类型的所有构造器

所以`enum`如果要是`irrefutable`的 还有其他条件, **当且仅当`enum`类型中只有一个有参构造器, 且包含的每个模式都是`irrefutable`的时, `enum`模式是`irrefutable`的**

### `match`表达式

#### `match`表达式的定义

> 仓颉支持两种`match`表达式, 第一种是**包含待匹配值**的`match`表达式, 第二种是**不含待匹配值**的`match`表达式

> **含匹配值的`match`表达式**: 
> 
> ```cangjie
> main() {
>     let x = 0
>     match (x) {
>         case 1 => let r1 = "x = 1"
>                   print(r1)
>         case 0 => let r2 = "x = 0"                    // Matched
>                   print(r2)
>         case _ => let r3 = "x != 1 and x != 0"
>                   print(r3)
>     }
> }
> ```
> 
> `match`表达式以关键字`match`开头, 后跟要匹配的值(如上例中的`x`, `x`可以是任意表达式), 接着是定义在一对花括号内的**若干`case`分支**
> 
> 每个`case`分支以关键字`case`开头, `case`之后是一个模式或多个由`|`连接的**相同种类的模式**(如上例中的`1`、`0`、`_`都是模式, 详见 [模式概述]() 章节)

`match`表达式的语法即为:

```cangjie
match (待匹配值) {
    case 模式1 => ""
    case 模式2 | 模式2 => ""
}
```

`match`表达式的`case`分支是用于匹配模式的, `case`之后的模式 可以使用`|`连接同种模式

> 模式之后可以接一个可选的`pattern guard`, 表示本条`case`匹配成功后额外需要满足的条件, `pattern guard`使用`where cond`表示, 要求表达式`cond`的类型为`Bool`
> 
> 接着是一个`=>`, `=>`之后即本条`case`分支匹配成功后需要执行的操作, 可以是一系列表达式、变量和函数定义(新定义的变量或函数的作用域从其定义处开始到下一个`case`之前结束), 如上例中的变量定义和`print`函数调用
> 
> `match`表达式执行时, **依次**将`match`之后的表达式与每个`case`中的模式进行匹配
> 
> **一旦匹配成功**, **则执行`=>`之后的代码, 然后退出`match`表达式的执行(意味着不会再去匹配它之后的`case`**)
> 
> 关于匹配成功, 如果有`pattern guard`, 也需要`where`之后的表达式的值为`true`; 如果`case`中有多个由`|`连接的模式, 只要待匹配值和其中一个模式匹配, 则认为匹配成功
> 
> 如果**匹配不成功, 则继续与它之后的`case`中的模式进行匹配, 直到匹配成功(`match`表达式可以保证一定存在匹配的`case`分支)**
> 
> 上例中, 因为`x`的值等于`0`, 所以会和第二条`case`分支匹配(此处使用的是常量模式, 匹配的是值是否相等, 详见 [常量模式]() 章节), 最后输出`x = 0`
> 
> 编译并执行上述代码, 输出结果为: 
> 
> ```text
> x = 0
> ```

`case`模式之后, 还可以接一个`pattern guard`, 即`where cond`

所以, `match`表达式的语法可以是:

```cangjie
match (待匹配值) {
    case 模式1 where 条件 => ""
}
```

`pattern guard`中的条件, 必须是`Bool`类型

在匹配成功之后, 才会进行`pattern guard`判断, 如果模式是绑定模式, 那么条件中可以使用绑定的变量

> `match`表达式要求 **所有匹配必须是穷尽(`exhaustive`)** 的, 意味着 **待匹配表达式的所有可能取值都应该被考虑到**
> 
> 当`match`表达式非穷尽, 或者编译器判断不出是否穷尽时, 均会编译报错, 换言之, 所有`case`分支(包含`pattern guard`)所覆盖的取值范围的并集, 应该包含待匹配表达式的所有可能取值
> 
> 常用的确保`match`表达式穷尽的方式是在最后一个`case`分支中使用通配符模式`_`, 因为`_`可以匹配任何值
> 
> `match`表达式的穷尽性保证了一定存在 和 待匹配值相匹配 的`case`分支
> 
> 下面的例子将编译报错, 因为所有的`case`并没有覆盖`x`的所有可能取值: 
> 
> ```cangjie
> func nonExhaustive(x: Int64) {
>     match (x) {
>         case 0 => print("x = 0")
>         case 1 => print("x = 1")
>         case 2 => print("x = 2")
>     }
> }
> ```
> 
> 如果被匹配值的类型包含`enum`类型, 且该`enum`为`non-exhaustive enum(包含无名构造器的enum)`, 则其在匹配时 需要使用可匹配所有构造器的模式, 如通配符模式`_`和绑定模式
> 
> ```cangjie
> enum T {
>     | Red | Green | Blue | ...
> }
> func foo(a: T) {
>     match (a) {
>         case Red => 0
>         case Green => 1
>         case Blue => 2
>         case _ => -1
>     }
> }
> 
> func bar(a: T) {
>     match (a) {
>         case Red => 0
>         case k => -1              // 简单绑定模式
>     }
> }
> 
> func baz(a: T) {
>     match (a) {
>         case Red => 0
>         case k: T => -1           // 带嵌套类型模式的绑定模式
>     }
> }
> ```

仓颉`match`表达式, 要求待匹配项必须能够匹配成功, 即 **待匹配表达式的所有可能取值都应该被考虑到**

即 使用`match`表达式进行模式匹配, 必须存在至少一个`case`分支能够匹配成功

如果待匹配项是`enum`类型, 那么同样需要手动匹配所有构造器, 如果`enum`存在`...无名构造器`, 一般就用 通配符`_`模式 或 绑定模式

当然, 其他类型也可以使用通配符`_`模式 和 绑定模式进行 匹配所有可能的取值

> 在`case`分支的模式之后, 可以使用`pattern guard`进一步**对匹配出来的结果进行判断**
> 
> 在下面的例子中(使用到了`enum`模式, 详见 [`enum`模式]() 章节), 当`RGBColor`的构造器的参数值大于等于`0`时, 输出它们的值, 当参数值小于`0`时, 认为它们的值等于`0`: 
> 
> ```cangjie
> enum RGBColor {
>     | Red(Int16) | Green(Int16) | Blue(Int16)
> }
> main() {
>     let c = RGBColor.Green(-100)
>     let cs = match (c) {
>         case Red(r) where r < 0 => "Red = 0"
>         case Red(r)             => "Red = ${r}"
>         case Green(g) where g < 0 => "Green = 0"      // Matched.
>         case Green(g)             => "Green = ${g}"
>         case Blue(b) where b < 0 => "Blue = 0"
>         case Blue(b)             => "Blue = ${b}"
>     }
>     print(cs)
> }
> ```
> 
> 编译执行上述代码, 输出结果为: 
> 
> ```text
> Green = 0
> ```

`pattern guard`是在匹配成功之后才进行判断的

> **没有匹配值的`match`表达式**: 
> 
> ```cangjie
> main() {
>     let x = -1
>     match {
>         case x > 0 => print("x > 0")
>         case x < 0 => print("x < 0")          // Matched.
>         case _ => print("x = 0")
>     }
> }
> ```
> 
> 与包含待匹配值的`match`表达式相比, 关键字`match`之后并**没有待匹配的表达式**
> 
> 并且**`case`之后不再是`pattern`**, 而是类型为`Bool`的表达式(上述代码中的`x > 0`和`x < 0`)或者`_`(表示`true`), 当然, `case`中也不再有`pattern guard`
> 
> 无匹配值的`match`表达式执行时, 依次判断`case`之后的表达式的值, 直到遇到值为`true`的`case`分支:
> 
> 一旦某个`case`之后的表达式值等于`true`, 则执行此`case`中`=>`之后的代码, 然后退出`match`表达式的执行(意味着不会再去判断该`case`之后的其他`case`)
> 
> 上例中, 因为`x`的值等于`-1`, 所以第二条`case`分支中的表达式(即`x < 0`)的值等于`true`, 执行`print("x < 0")`
> 
> 编译并执行上述代码, 输出结果为: 
> 
> ```cangjie
> x < 0
> ```

从文档来看, 无匹配值的`match`表达式, 更大的用处是 针对可见变量的分支判断, 实际类似`if-else`

文档中的例子, 实际可以等价于:

```cangjie
let x = -1 
if (x > 0) {
    print("x > 0")
}
else if (x < 0) {
    print("x < 0")
}
else {
    print("x = 0")
}
```

#### `match`表达式的类型

> 对于`match`表达式(无论是否有匹配值): 
> 
> - 在上下文有明确的类型要求时, 要求每个`case`分支中`=>`之后的代码块的类型是上下文所要求的类型的子类型
> 
> - 在上下文没有明确的类型要求时, `match`表达式的类型是每个`case`分支中`=>`之后的代码块的类型的最小公共父类型
> 
> - 当`match`表达式的值没有被使用时, 其类型为`Unit`, 不要求各分支的类型有最小公共父类型
> 
> 下面分别举例说明
> 
> ```cangjie
> let x = 2
> let s: String = match (x) {
>     case 0 => "x = 0"
>     case 1 => "x = 1"
>     case _ => "x != 0 and x != 1"             // Matched
> }
> ```
> 
> 上面的例子中, 定义变量`s`时, 显式地标注了其类型为`String`, 属于上下文类型信息明确的情况, 因此要求每个`case`的`=>`之后的代码块的类型均是`String`的子类型, 显然上例中`=>`之后的字符串类型的字面量均满足要求
> 
> 再来看一个没有上下文类型信息的例子: 
> 
> ```cangjie
> let x = 2
> let s = match (x) {
>     case 0 => "x = 0"
>     case 1 => "x = 1"
>     case _ => "x != 0 and x != 1"             // Matched
> }
> ```
> 
> 上例中, 定义变量`s`时, 未显式标注其类型, 因为每个`case`的`=>`之后的代码块的类型均是`String`, 所以`match`表达式的类型是`String`, 进而可确定`s`的类型也是`String`

`match`表达式的类型, 与仓颉其他表达式的类型是一致的

如果`match`表达式的值被使用了:

1. 如果上下文有要求, 每个`case`分支 的最终类型都要是上下文要求的子类型

2. 如果没有要求, 那么 就是每个`case`分支的最小公共父类型

如果没有被使用, 就没有任何要求, 类型就是`Unit`

### 其他使用模式的地方

> 模式除了可以在`match`表达式中使用外, 还可以使用在**变量定义**(等号左侧是一个模式)和 **`for in`表达式**(`for`关键字和`in`关键字之间是一个模式)中
> 
> 但是, 并不是所有的模式都能使用在变量定义和`for in`表达式中, 只有`irrefutable`的模式才能在这两处被使用, 所以只有通配符模式、绑定模式、`irrefutable tuple`模式和`irrefutable enum`模式是允许的

仓颉的模式, 并不是只能在`match`表达式中使用, 还可以使用在变量定义和`for-in`表达式

但只有不可证伪的模式才能在其他地方使用, 即 通配符`_`模式, 绑定模式 以及 不可证伪的`Tuple`以及`enum`模式

定义变量时, `=`左边是一个模式(变量名)

`for-in`表达式中, `for`和`in`之间是一个模式

> 1. 变量定义和`for in`表达式中使用通配符模式的例子如下: 
> 
>     ```cangjie
>     main() {
>         let _ = 100
>         for (_ in 1..5) {
>             println("0")
>         }
>     }
>     ```
>     
>     上例中, 变量定义时使用了通配符模式, 表示定义了一个没有名字的变量(当然此后也就没办法对其进行访问)
> 
>     `for in`表达式中使用了通配符模式, 表示不会将`1..5`中的元素与某个变量绑定(当然循环体中就无法访问`1..5`中元素值)
>     
>     编译执行上述代码, 输出结果为: 
>     
>     ```text
>     0
>     0
>     0
>     0
>     ```

定义变量, 可以使用通配符替代普通变量, 表示忽略目标

`for-in`表达式, `for`和`in`之间也可以使用通配符, 也表示忽略目标

> 2. 变量定义和`for in`表达式中使用绑定模式的例子如下: 
> 
>     ```cangjie
>     main() {
>         let x = 100
>         println("x = ${x}")
>         for (i in 1..5) {
>             println(i)
>         }
>     }
>     ```
>     
>     上例中, 变量定义中的`x`以及`for in`表达式中的`i`都是绑定模式
>     
>     编译执行上述代码, 输出结果为: 
>     
>     ```text
>     x = 100
>     1
>     2
>     3
>     4
>     ```

仓颉的变量定义, 本身就是一个绑定模式

`for-in`表达式最基础的使用方式, 也是一个绑定模式

> 3. 变量定义和`for in`表达式中使用`irrefutable tuple`模式的例子如下: 
> 
>     ```cangjie
>     main() {
>         let (x, y) = (100, 200)
>         println("x = ${x}")
>         println("y = ${y}")
>         for ((i, j) in [(1, 2), (3, 4), (5, 6)]) {
>             println("Sum = ${i + j}")
>         }
>     }
>     ```
>     
>     上例中, 变量定义时使用了`tuple`模式, 表示对`(100, 200)`进行解构并分别和`x`与`y`进行绑定, 效果上相当于定义了两个变量`x`和`y`
>     
>     `for in`表达式中使用了`tuple`模式, 表示依次将`[(1, 2), (3, 4), (5, 6)]`中的`tuple`类型的元素取出, 然后解构并分别和`i`与`j`进行绑定, 循环体中输出`i + j`的值
>     
>     编译执行上述代码, 输出结果为: 
>     
>     ```cangjie
>     x = 100
>     y = 200
>     Sum = 3
>     Sum = 7
>     Sum = 11
>     ```

`irrefutable Tuple`模式, 就是只包含通配符`_`模式和绑定模式的`Tuple`模式

定义变量时, 可以使用`irrefutable Tuple`模式, 但此时并不是简单的通配符模式或绑定模式, 同时包含**元组的解构**

```cangjie
let (x, y) = (100, 200)
println("x = ${x}")
println("y = ${y}")
```

并不是简单定义了一个元组, 而是 在定义变量时 通过`irrefutable Tuple`模式, 元组模式匹配 并 解构元组, 定义了两个不同的变量

```cangjie
for ((i, j) in [(1, 2), (3, 4), (5, 6)]) {
    println("Sum = ${i + j}")
}
```

`for-in`表达式中也是同样的概念

> 4. 变量定义和`for in`表达式中使用`irrefutable enum`模式的例子如下: 
> 
>     ```cangjie
>     enum RedColor {
>         Red(Int64)
>     }
>     main() {
>         let Red(red) = Red(0)
>         println("red = ${red}")
>         for (Red(r) in [Red(10), Red(20), Red(30)]) {
>             println("r = ${r}")
>         }
>     }
>     ```
>     
>     上例中, 变量定义时使用了`enum`模式, 表示对`Red(0)`进行解构并将构造器的参数值(即 0)与`red`进行绑定
>     
>     `for in`表达式中使用了`enum`模式, 表示依次将`[Red(10), Red(20), Red(30)]`中的元素取出, 然后解构并将构造器的参数值与`r`进行绑定, 循环体中输出`r`的值
>     
>     编译执行上述代码, 输出结果为: 
>     
>     ```text
>     red = 0
>     r = 10
>     r = 20
>     r = 30
>     ```

`irrefutable enum`模式与`irrefutable Tuple`是类似的, 包含模式匹配 与 解构
