---
title: "仓颉文档阅读-开发指南VI: 枚举类型和模式匹配(II) - 模式的Refutability、match表达式 与 其他使用模式的地方"
published: 2025-11-17 17:53:10
description: '仓颉文档阅读的开发指南部分, 本篇文章介绍仓颉语言的模式匹配相关内容'
image: 'https://humid1ch.oss-cn-shanghai.aliyuncs.com/20250929154944807.webp'
tags: ["开发语言", "仓颉"]
category: Blogs
draft: true
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

### 模式的 `Refutability`

> 模式可以分为两类：`refutable` 模式和 `irrefutable` 模式
> 
> 在**类型匹配的前提下**，当一个模式有可能和待匹配值不匹配时，称此模式为 `refutable` 模式；反之，当一个模式总是可以和待匹配值匹配时，称此模式为 `irrefutable` 模式

`Refutability`翻译为可证伪性

仓颉将模式分为两类: 可证伪(`refutable`) 和 不可证伪(`irrefutable`)

并解释, 在类型匹配的前提下:

1. `refutable`可证伪, 表示 待匹配值**可能存在无法匹配成功**的情况

2. `irrefutable`不可证伪, 表示 待匹配值**总能匹配成功**的情况

> 对于上述介绍的各种模式，规定如下：
> 
> 常量模式是 `refutable` 模式
> 
> 例如，下例中第一个`case`中的`1` 和 第二个`case`中的`2` 都有可能和`x`的值不相等
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

> 通配符模式是 `irrefutable` 模式
> 
> 例如，下例中无论 `x` 的值是多少，`_` 总能和其匹配
> 
> ```cangjie
> func wildcardPat(x: Int64) {
>     match (x) {
>         case _ => "_"
>     }
> }
> ```
> 
> 绑定模式是 `irrefutable` 模式
> 
> 例如，下例中无论 `x` 的值是多少，绑定模式 `a` 总能和其匹配
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

> `Tuple` 模式是 `irrefutable` 模式，当且仅当其包含的每个模式都是 `irrefutable` 模式
> 
> 例如，下例中 `(1, 2)` 和 `(a, 2)` 都有可能和 `x` 的值不匹配，所以它们是 `refutable` 模式，而 `(a, b)` 可以匹配任何 `x` 的值，所以它是 `irrefutable` 模式
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

> 类型模式是 `refutable` 模式
> 
> 例如，下例中（假设 `Base` 是 `Derived` 的父类，并且 `Base` 实现了接口 `I`），`x` 的运行时类型有可能既不是 `Base` 也不是 `Derived`，所以 `a: Derived` 和 `b: Base` 均是 `refutable` 模式
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

> `enum` 模式是 `irrefutable` 模式，当且仅当它对应的 `enum` 类型中只有一个有参构造器，且`enum` 模式中包含的其他模式也是 `irrefutable` 模式
> 
> 例如，对于下例中的 `E1` 和 `E2` 定义，函数 `enumPat1` 中的 `A(1)` 是 `refutable` 模式，`A(a)` 是 `irrefutable` 模式；而函数 `enumPat2` 中的 `B(b)` 和 `C(c)` 均是 `refutable` 模式
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

`enum`模式 与 `Tuple`模式类似, 都 可以是`refutable`可证伪的, 也可以是`irrefutable`不可证伪的, 因为他们都是要依赖其他模式的

但`enum`还有一点不同的是, `match`表达式中`enum`模式通常需要覆盖目标`enum`类型的所有构造器

所以 `enum`如果要是`irrefutable`的 还有其他条件, **当且仅当`enum`类型中只有一个有参构造器, 且包含的每个模式都是`irrefutable`的时, `enum`模式是`irrefutable`的**

### `match`表达式

#### `match` 表达式的定义

仓颉支持两种 `match` 表达式，第一种是包含待匹配值的 `match` 表达式，第二种是不含待匹配值的 `match` 表达式

含匹配值的 match 表达式：

main() {
    let x = 0
    match (x) {
        case 1 => let r1 = "x = 1"
                  print(r1)
        case 0 => let r2 = "x = 0" // Matched.
                  print(r2)
        case _ => let r3 = "x != 1 and x != 0"
                  print(r3)
    }
}

match 表达式以关键字 match 开头，后跟要匹配的值（如上例中的 x，x 可以是任意表达式），接着是定义在一对花括号内的若干 case 分支。

每个 case 分支以关键字 case 开头，case 之后是一个模式或多个由 | 连接的相同种类的模式（如上例中的 1、0、_ 都是模式，详见模式概述章节）；模式之后可以接一个可选的 pattern guard，表示本条 case 匹配成功后额外需要满足的条件，pattern guard 使用 where cond 表示，要求表达式 cond 的类型为 Bool；接着是一个 =>，=> 之后即本条 case 分支匹配成功后需要执行的操作，可以是一系列表达式、变量和函数定义（新定义的变量或函数的作用域从其定义处开始到下一个 case 之前结束），如上例中的变量定义和 print 函数调用。

match 表达式执行时依次将 match 之后的表达式与每个 case 中的模式进行匹配，一旦匹配成功（如果有 pattern guard，也需要 where 之后的表达式的值为 true；如果 case 中有多个由 | 连接的模式，只要待匹配值和其中一个模式匹配则认为匹配成功）则执行 => 之后的代码然后退出 match 表达式的执行（意味着不会再去匹配它之后的 case），如果匹配不成功则继续与它之后的 case 中的模式进行匹配，直到匹配成功（match 表达式可以保证一定存在匹配的 case 分支）。

上例中，因为 x 的值等于 0，所以会和第二条 case 分支匹配（此处使用的是常量模式，匹配的是值是否相等，详见常量模式章节），最后输出 x = 0。

编译并执行上述代码，输出结果为：

x = 0

match 表达式要求所有匹配必须是穷尽（exhaustive）的，意味着待匹配表达式的所有可能取值都应该被考虑到。当 match 表达式非穷尽，或者编译器判断不出是否穷尽时，均会编译报错，换言之，所有 case 分支（包含 pattern guard）所覆盖的取值范围的并集，应该包含待匹配表达式的所有可能取值。常用的确保 match 表达式穷尽的方式是在最后一个 case 分支中使用通配符模式 _，因为 _ 可以匹配任何值。

match 表达式的穷尽性保证了一定存在和待匹配值相匹配的 case 分支。下面的例子将编译报错，因为所有的 case 并没有覆盖 x 的所有可能取值：

func nonExhaustive(x: Int64) {
    match (x) {
        case 0 => print("x = 0")
        case 1 => print("x = 1")
        case 2 => print("x = 2")
    }
}

如果被匹配值的类型包含 enum 类型且该 enum 为 non-exhaustive enum，则其在匹配时需要使用可匹配所有构造器的模式，如通配符模式 _ 和绑定模式。

enum T {
    | Red | Green | Blue | ...
}
func foo(a: T) {
    match (a) {
        case Red => 0
        case Green => 1
        case Blue => 2
        case _ => -1
    }
}

func bar(a: T) {
    match (a) {
        case Red => 0
        case k => -1 // simple binding pattern
    }
}

func baz(a: T) {
    match (a) {
        case Red => 0
        case k: T => -1 // binding pattern with nested type pattern
    }
}

在 case 分支的模式之后，可以使用 pattern guard 进一步对匹配出来的结果进行判断。

在下面的例子中（使用到了 enum 模式，详见 enum 模式章节），当 RGBColor 的构造器的参数值大于等于 0 时，输出它们的值，当参数值小于 0 时，认为它们的值等于 0：

enum RGBColor {
    | Red(Int16) | Green(Int16) | Blue(Int16)
}
main() {
    let c = RGBColor.Green(-100)
    let cs = match (c) {
        case Red(r) where r < 0 => "Red = 0"
        case Red(r) => "Red = ${r}"
        case Green(g) where g < 0 => "Green = 0" // Matched.
        case Green(g) => "Green = ${g}"
        case Blue(b) where b < 0 => "Blue = 0"
        case Blue(b) => "Blue = ${b}"
    }
    print(cs)
}

编译执行上述代码，输出结果为：

Green = 0

没有匹配值的 match 表达式：

main() {
    let x = -1
    match {
        case x > 0 => print("x > 0")
        case x < 0 => print("x < 0") // Matched.
        case _ => print("x = 0")
    }
}

与包含待匹配值的 match 表达式相比，关键字 match 之后并没有待匹配的表达式，并且 case 之后不再是 pattern，而是类型为 Bool 的表达式（上述代码中的 x > 0 和 x < 0）或者 _（表示 true），当然，case 中也不再有 pattern guard。

无匹配值的 match 表达式执行时依次判断 case 之后的表达式的值，直到遇到值为 true 的 case 分支；一旦某个 case 之后的表达式值等于 true，则执行此 case 中 => 之后的代码，然后退出 match 表达式的执行（意味着不会再去判断该 case 之后的其他 case）。

上例中，因为 x 的值等于 -1，所以第二条 case 分支中的表达式（即 x < 0）的值等于 true，执行 print("x < 0")。

编译并执行上述代码，输出结果为：

x < 0

match 表达式的类型

对于 match 表达式（无论是否有匹配值）：

    在上下文有明确的类型要求时，要求每个 case 分支中 => 之后的代码块的类型是上下文所要求的类型的子类型；

    在上下文没有明确的类型要求时，match 表达式的类型是每个 case 分支中 => 之后的代码块的类型的最小公共父类型；

    当 match 表达式的值没有被使用时，其类型为 Unit，不要求各分支的类型有最小公共父类型。

下面分别举例说明。

let x = 2
let s: String = match (x) {
    case 0 => "x = 0"
    case 1 => "x = 1"
    case _ => "x != 0 and x != 1" // Matched.
}

上面的例子中，定义变量 s 时，显式地标注了其类型为 String，属于上下文类型信息明确的情况，因此要求每个 case 的 => 之后的代码块的类型均是 String 的子类型，显然上例中 => 之后的字符串类型的字面量均满足要求。

再来看一个没有上下文类型信息的例子：

let x = 2
let s = match (x) {
    case 0 => "x = 0"
    case 1 => "x = 1"
    case _ => "x != 0 and x != 1" // Matched.
}

上例中，定义变量 s 时，未显式标注其类型，因为每个 case 的 => 之后的代码块的类型均是 String，所以 match 表达式的类型是 String，进而可确定 s 的类型也是 String。
