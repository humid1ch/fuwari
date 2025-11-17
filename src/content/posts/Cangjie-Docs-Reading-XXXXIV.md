---
title: "仓颉文档阅读-开发指南VI: 枚举类型和模式匹配(II) - 模式匹配概述"
published: 2025-11-17 15:34:12
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

### 模式概述

> 对于包含匹配值的 `match` 表达式，`case` 之后支持哪些模式决定了 `match` 表达式的表达能力
> 
> 本节中将依次介绍仓颉支持的模式，包括：常量模式、通配符模式、绑定模式、`tuple` 模式、类型模式和 `enum` 模式

`match`表达式, 还未阅读到开发指南文档中的相关部分

但, 从已阅读的语法规约中, `match`是一种模式匹配的表达式

如果非要类比的话, 与C/C++中与之最相似的语法应该是`switch-case`, 但要复杂的多

而`match`表达式, `case`之后支持的模式决定了表达式的表达能力

**什么是模式?**

模式, 是一种匹配目标值的结构, 通常用在`match`表达式、定义变量等场景

可以类比C/C++中的`switch-case`来理解, `switch`是只能匹配整型的一种结构

而仓颉中, 不同模式对应不同的目标类型, 不过像`switch`这样匹配整型的结构, 在仓颉中 属于 **常量模式**

**模式并不是一种单独的语法, 而是与特定的匹配结构结合使用**

#### 常量模式

> 常量模式可以是整数字面量、浮点数字面量、字符字面量、布尔字面量、字符串字面量（不支持字符串插值）、`Unit` 字面量
> 
> 在包含匹配值的 `match` 表达式（参见[`match`表达式]）中使用常量模式时，要求常量模式表示的值的类型与待匹配值的类型相同，匹配成功的条件是待匹配的值与常量模式表示的值相等
> 
> 下面的例子中，根据 `score` 的值（假设 `score` 只能取 `0` 到 `100` 间被 `10` 整除的值），输出考试成绩的等级：
> 
> ```cangjie
> main() {
>     let score = 90
>     let level = match (score) {
>         case 0 | 10 | 20 | 30 | 40 | 50 => "D"
>         case 60 => "C"
>         case 70 | 80 => "B"
>         case 90 | 100 => "A"                      // Matched
>         case _ => "Not a valid score"
>     }
>     println(level)
> }
> ```
> 
> 编译执行上述代码，输出结果为：
> 
> ```text
> A
> ```
> 
> - 在模式匹配的目标是静态类型为 `Rune` 的值时，`Rune` 字面量和单字符字符串字面量都可用于表示 `Rune` 类型字面量的常量 `pattern`
> 
>     ```cangjie
>     func translate(n: Rune) {
>         match (n) {
>             case "A" => 1
>             case "B" => 2
>             case "C" => 3
>             case _ => -1
>         }
>     }
>     
>     main() {
>         println(translate(r"C"))
>     }
>     ```
>     
>     编译执行上述代码，输出结果为：
>     
>     ```text
>     3
>     ```
> 
> - 在模式匹配的目标是静态类型为 `Byte` 的值时，一个表示 `ASCII` 字符的字符串字面量可用于表示 `Byte` 类型字面量的常量 `pattern`
> 
>     ```cangjie
>     func translate(n: Byte) {
>         match (n) {
>             case "1" => 1
>             case "2" => 2
>             case "3" => 3
>             case _ => -1
>         }
>     }
>     
>     main() {
>         println(translate(51))            // UInt32(r'3') == 51
>     }
>     ```
>     
>     编译执行上述代码，输出结果为：
>     
>     ```text
>     3
>     ```

常量模式, 只会出现在`match`表达式中

它用于对比 变量 是否与 目标字面量 匹配, 可用字面量有: 整数字面量、浮点数字面量、字符字面量、布尔字面量、字符串字面量（不支持字符串插值）、`Unit` 字面量

使用语法为:

```cangjie
match (value) {
    case 目标字面量1 => 匹配成功执行语句
    case 目标字面量2 => 匹配成功执行语句
    case 目标字面量3 | 目标字面量4 => 匹配成功执行语句
    ...
}
```

匹配的字面量之间, 可以使用`|`进行连接, 表示 多目标匹配

被`|`连接的, `value`只要与其中一个相等就算匹配成功

#### 通配符模式

> 通配符模式使用下划线 `_` 表示，可以**匹配任意值**
> 
> 通配符模式通常作为最后一个 `case` 中的模式，用来匹配其他 `case` 未覆盖到的情况，如常量模式中匹配 `score` 值的示例中，最后一个 `case` 中使用 `_` 来匹配无效的 `score` 值

通配符模式, 不止会出现在`match`表达式中

通配符`_`可以匹配任意值, 如果`match`中存在`_`就不会出现匹配不不到的情况: 如果`match`前面都没有匹配到, 绝对会匹配到`_`

上面已经有了通配符模式的例子:

```cangjie
match (value) {
    case _ => ""
}
```

只要走到`case _`, 就会匹配成功

#### 绑定模式

> 绑定模式使用 `id` 表示，`id` 是一个合法的标识符
> 
> 与通配符模式相比，绑定模式同样可以匹配任意值，但绑定模式会将匹配到的值与 `id` 进行绑定，在 `=>` 之后可以通过 `id` 访问其绑定的值
> 
> 下面的例子中，最后一个 `case` 中使用了绑定模式，用于绑定非 `0` 值：
> 
> ```cangjie
> main() {
>     let x = -10
>     let y = match (x) {
>         case 0 => "zero"
>         case n => "x is not zero and x = ${n}"          // Matched
>     }
>     println(y)
> }
> ```
> 
> 编译执行上述代码，输出结果为：
> 
> ```text
> x is not zero and x = -10
> ```

绑定模式, 同样不止出现在`match`表达式中

绑定模式与通配符模式类似, 能**匹配到任意值**, 即 只要走到这里, 就能匹配成功

但通配符使用`_`, 而绑定模式使用一个合法的标识符

通配符模式, 匹配成功之后, 在`=>`之后不能尝试访问匹配成功的值, 因为没有绑定到标识符

但 绑定模式不同, 绑定模式存在标识符, 所以在`=>`之后, 可以通过标识符 访问匹配的值:

```cangjie
match (value) {
    case num => "the value is ${num}"
}
```

只要走到`case num`, 就能匹配成功, 且`value`绑定到`num`, 即 `num`就是`value`

> [!TIP]
> 
> 绑定模式, 绑定行为与 值类型或引用类型的赋值、传参行为保持一致

> 使用 `|` 连接多个模式时不能使用绑定模式，也不可嵌套出现在其他模式中，否则会报错：
> 
> ```cangjie
> main() {
>     let opt = Some(0)
>     match (opt) {
>         case x | x => {}                          // Error, 在由 '|' 连接的模式中不能引入绑定模式
>         case Some(x) | Some(x) => {}              // Error, 在由 '|' 连接的模式中不能引入绑定模式
>         case x: Int64 | x: String => {}           // Error, 在由 '|' 连接的模式中不能引入绑定模式
>     }
> }
> ```
> 
> 绑定模式 `id` 相当于新定义了一个名为 `id` 的**不可变变量**（其作用域从引入处开始到该 `case` 结尾处），因此在 `=>` 之后无法对 `id` 进行修改
> 
> 例如，下例中最后一个 `case` 中对 `n` 的修改是不允许的
> 
> ```cangjie
> main() {
>     let x = -10
>     let y = match (x) {
>         case 0 => "zero"
>         case n => n = n + 0                       // Error, 'n' 不能被修改
>                   "x is not zero"
>     }
>     println(y)
> }
> ```
> 
> 对于每个 `case` 分支，**`=>`之后变量作用域级别 与 `case`后`=>`前引入的变量 作用域级别相同, 在`=>`之后再次引入相同名字会触发重定义错误**
> 
> 例如：
> 
> ```cangjie
> main() {
>     let x = -10
>     let y = match (x) {
>         case 0 => "zero"
>         case n => let n = 0                       // Error, 重定义
>                   println(n)
>                   "x is not zero"
>     }
>     println(y)
> }
> ```
> 
> > 注意：
> > 
> > 当模式的 `identifier` 为 `enum` 构造器时，该模式会被当成 `enum` 模式进行匹配，而不是绑定模式（关于 `enum`模式，详见 [`enum`模式]()章节）
> 
> ```cangjie
> enum RGBColor {
>     | Red | Green | Blue
> }
> 
> main() {
>     let x = Red
>     let y = match (x) {
>         case Red => "red"                         // 'Red' 在这是 enum 模式
>         case _ => "not red"
>     }
>     println(y)
> }
> ```
> 
> 编译执行上述代码，输出结果为：
> 
> ```text
> red
> ```

绑定模式不能使用`|`连接, 因为`|`连接 只要匹配其中一个, 就能执行`=>`后面的语句

如果是绑定模式, 则后面应该是可以访问绑定标识符的, 但如果`|`连接了, 绑定标识符可能就无效了, 所以应该被禁止

绑定模式的标识符, 绑定匹配成功之后 相当于定义了一个**不可变变量**

且 `=>`之后执行语句的作用域 与 `case`后的作用域是同等级的, 所以 **`=>`之后不允许再定义 与绑定标识符同名的变量**

如果绑定模式的标识符是`enum`构造器时, 会被当作`enum`模式, 而不是绑定模式

#### `Tuple` 模式

> `Tuple` 模式用于 **`tuple`值的匹配**，它的定义**和`tuple`字面量类似**：`(p_1, p_2, ..., p_n)`，区别在于这里的 `p_1` 到 `p_n`（`n` 大于等于 `2`）是模式（**可以是本章节中介绍的任何模式**，多个模式间使用逗号分隔）而不是表达式
> 
> 例如，`(1, 2, 3)` 是一个包含三个常量模式的 `tuple` 模式，`(x, y, _)` 是一个包含两个绑定模式，一个通配符模式的 `tuple` 模式
> 
> 给定一个 `tuple` 值 `tv` 和一个 `tuple` 模式 `tp`，当且仅当 `tv` 每个位置处的值均能与 `tp` 中对应位置处的模式相匹配，才称 `tp` 能匹配 `tv`
> 
> 例如，`(1, 2, 3)` 仅可以匹配 `tuple` 值 `(1, 2, 3)`，`(x, y, _)` 可以匹配任何三元 `tuple` 值
> 
> 下面的例子中，展示了 `tuple` 模式的使用：
> 
> ```cangjie
> main() {
>     let tv = ("Alice", 24)
>     let s = match (tv) {
>         case ("Bob", age) => "Bob is ${age} years old"
>         case ("Alice", age) => "Alice is ${age} years old"        // Matched, "Alice"是一个常量模式，而'age'是一个绑定模式
>         case (name, 100) => "${name} is 100 years old"
>         case (_, _) => "someone"
>     }
>     println(s)
> }
> ```
> 
> 编译执行上述代码，输出结果为：
> 
> ```text
> Alice is 24 years old
> ```
> 
> 同一个 `tuple` 模式中**不允许引入多个名称相同的绑定模式**
> 
> 例如，下例中最后一个 `case` 中的 `case (x, x)` 是不合法的
> 
> ```cangjie
> main() {
>     let tv = ("Alice", 24)
>     let s = match (tv) {
>         case ("Bob", age) => "Bob is ${age} years old"
>         case ("Alice", age) => "Alice is ${age} years old"
>         case (name, 100) => "${name} is 100 years old"
>         case (x, x) => "someone"                                  // Error, 不能引入同名的变量模式，将重定义错误
>     }
>     println(s)
> }
> ```

`Tuple`模式, 不止出现在`match`表达式中

了解了上面几个模式之后, `Tuple`模式其实很好理解

`Tuple`就是元组, 普通元组字面量可以存储数据

而`Tuple`模式, 与元组字面量语法类似, 只不过`Tuple`模式的`()`中, 应该是其他模式: 常量模式、通配符模式、绑定模式、类型模式等等

即, `Tuple`模式里可以是其他任意模式

但, **`Tuple`模式, 必须匹配到`()`内的所有模式, 才算匹配成功**

相关的语法为

```cangjie
match (value) {
    case (mode, mode, mode) => ""
}
```

当`value`是一个元组, 且能够与目标模式匹配时, 匹配成功

#### 类型模式

> 类型模式用于 **判断一个值的运行时类型是否是某个类型的子类型**
> 
> 类型模式有两种形式：**`_: Type`（嵌套一个通配符模式 `_`）和 `id: Type`（嵌套一个绑定模式 `id`）**，它们的区别是**后者会发生变量绑定，而前者不会**
> 
> 对于待匹配值 `v` 和类型模式 `id: Type`（或 `_: Type`）
> 
> 首先 **判断`v`的运行时类型是否是`Type`的子类型**，若 成立则视为匹配成功，否则视为匹配失败
> 
> 如匹配成功，则将 `v` 的类型转换为 `Type` 并与 `id` 进行绑定（对于 `_: Type`，不存在绑定这一操作）
> 
> 假设有如下两个类，`Base` 和 `Derived`，并且 `Derived` 是 `Base` 的子类，`Base` 的无参构造函数中将 `a` 的值设置为 `10`，`Derived` 的无参构造函数中将 `a` 的值设置为 `20`：
> 
> ```cangjie
> open class Base {
>     var a: Int64
>     public init() {
>         a = 10
>     }
> }
> 
> class Derived <: Base {
>     public init() {
>         a = 20
>     }
> }
> ```
> 
> 下面的代码展示了使用类型模式并匹配成功的例子：
> 
> ```cangjie
> main() {
>     var d = Derived()
>     var r = match (d) {
>         case b: Base => b.a // Matched.
>         case _ => 0
>     }
>     println("r = ${r}")
> }
> ```
> 
> 编译执行上述代码，输出结果为：
> 
> ```text
> r = 20
> ```
> 
> 下面的代码展示了使用类型模式但类型模式匹配失败的例子：
> 
> ```cangjie
> open class Base {
>     var a: Int64
>     public init() {
>         a = 10
>     }
> }
> 
> class Derived <: Base {
>     public init() {
>         a = 20
>     }
> }
> 
> main() {
>     var b = Base()
>     var r = match (b) {
>         case d: Derived => d.a // Type pattern match failed.
>         case _ => 0 // Matched.
>     }
>     println("r = ${r}")
> }
> ```
> 
> 编译执行上述代码，输出结果为：
> 
> ```cangjie
> r = 0
> ```

类型模式, 只会出现在`match`表达式中

类型模式是为了匹配目标的运行时类型, 是否为目标类型的子类型

语法为:

```cangjie
match (value) {
    case ident: Type => "${ident}"
    case _: Type => "${ident}"
}
```

当`value`运行时类型为`Type`的子类型时, 匹配成功

#### `enum` 模式

> `enum` 模式用于匹配 `enum` 类型的实例，它的定义和 `enum` 的构造器类似：无参构造器 `C` 或有参构造器 `C(p_1, p_2, ..., p_n)`，构造器的类型前缀可以省略，区别在于这里的 `p_1` 到 `p_n`（`n` 大于等于 `1`）是模式
> 
> 例如，`Some(1)` 是一个包含一个常量模式的 `enum` 模式，`Some(x)` 是一个包含一个绑定模式的 `enum` 模式
> 
> 给定一个 `enum` 实例 `ev` 和一个 `enum` 模式 `ep`，当且仅当`ev` 的构造器名字和 `ep` 的构造器名字相同，且 `ev` 参数列表中每个位置处的值均能与 `ep` 中对应位置处的模式相匹配，才称 `ep` 能匹配 `ev`
> 
> 例如，`Some("one")` 仅可以匹配 `Option<String>` 类型的`Some`构造器 `Option<String>.Some("one")`，`Some(x)` 可以匹配任何 `Option` 类型的 `Some` 构造器
> 
> 下面的例子中，展示了 `enum` 模式的使用，因为 `x` 的构造器是 `Year`，所以会和第一个 `case` 匹配：
> 
> ```cangjie
> enum TimeUnit {
>     | Year(UInt64)
>     | Month(UInt64)
> }
> 
> main() {
>     let x = Year(2)
>     let s = match (x) {
>         case Year(n) => "x has ${n * 12} months" // Matched.
>         case TimeUnit.Month(n) => "x has ${n} months"
>     }
>     println(s)
> }
> ```
> 
> 编译执行上述代码，输出结果为：
> 
> ```text
> x has 24 months
> ```
> 
> 使用 `|` 连接多个 `enum` 模式：
> 
> ```cangjie
> enum TimeUnit {
>     | Year(UInt64)
>     | Month(UInt64)
> }
> 
> main() {
>     let x = Year(2)
>     let s = match (x) {
>         case Year(0) | Year(1) | Month(_) => "Ok" // Ok
>         case Year(2) | Month(m) => "invalid" // Error, Variable cannot be introduced in patterns connected by '|'
>         case Year(n: UInt64) | Month(n: UInt64) => "invalid" // Error, Variable cannot be introduced in patterns connected by '|'
>     }
>     println(s)
> }
> ```

`enum`模式, 虽然文档中说明很多, 但是并不复杂

其实就是尝试匹配 目标`enum`类型的构造器, 要求与构造器名和参数列表保持一致

但, `enum`模式里的参数列表就和`Tuple`模式的`()`一样, 是其他模式了

`enum`也可以使用`|`连接, 同样此时`enum`模式构造器的参数中不能存在匹配模式和类型模式

`enum`模式的语法在`match`里为:

```cangjie
match (value) {
    case EnumConstructor => ""
    case EnumConstructor(mode, mode) => ""
}
```

`value`需要是目标`enum`类型的值, 匹配时要与目标构造器进行匹配

> 使用 `match` 表达式匹配 `enum` 值时，要求`case` 之后的模式要**覆盖待匹配 `enum` 类型中的所有构造器**，如果未做到完全覆盖，编译器将报错：
> 
> ```cangjie
> enum RGBColor {
>     | Red | Green | Blue
> }
> 
> main() {
>     let c = Green
>     let cs = match (c) { // Error, Not all constructors of RGBColor are covered.
>         case Red => "Red"
>         case Green => "Green"
>     }
>     println(cs)
> }
> ```
> 
> 可以通过加上 `case Blue` 来实现完全覆盖，也可以在 `match` 表达式的最后通过使用 `case _` 来覆盖其他 `case` 未覆盖的到的情况，如：
> 
> ```cangjie
> enum RGBColor {
>     | Red | Green | Blue
> }
> 
> main() {
>     let c = Blue
>     let cs = match (c) {
>         case Red => "Red"
>         case Green => "Green"
>         case _ => "Other" // Matched.
>     }
>     println(cs)
> }
> ```
> 
> 上述代码的执行结果为：
> 
> ```text
> Other
> ```

`enum`模式在`match`表达式中使用时, 要保证`match`表达式的`case`匹配, 能够**覆盖目标`enum`类型的所有构造器**

可以使用通配符模式实现, 也可以手动`case`所有构造器实现

#### 模式的嵌套组合

> `Tuple` 模式和 `enum` 模式可以嵌套任意模式
> 
> 下面的代码展示了不同模式嵌套组合使用：
> 
> ```cangjie
> enum TimeUnit {
>     | Year(UInt64)
>     | Month(UInt64)
> }
> 
> enum Command {
>     | SetTimeUnit(TimeUnit)
>     | GetTimeUnit
>     | Quit
> }
> 
> main() {
>     let command = (SetTimeUnit(Year(2022)), SetTimeUnit(Year(2024)))
>     match (command) {
>         case (SetTimeUnit(Year(year)), _) => println("Set year ${year}")
>         case (_, SetTimeUnit(Month(month))) => println("Set month ${month}")
>         case _ => ()
>     }
> }
> ```
> 
> 编译并执行上述代码，输出结果为：
> 
> ```cangjie
> Set year 2022
> ```

`Tuple`模式和`enum`模式, 是可以嵌套使用其他所有模式的

或者说, `Tuple`模式和`enum`模式, 本身就是结合其他模式一起使用的
