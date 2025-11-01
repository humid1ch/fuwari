---
title: "仓颉文档阅读-语言规约IV: 表达式(III)"
published: 2025-09-29 09:42:13
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

## 表达式

> 表达式通常由一个或多个操作数(`operand`)构成, 多个操作数之间由操作符(`operator`)连接, 每个表达式都有一个类型, 计算表达式值的过程称为对表达式的求值(`evaluation`)
>
> 在仓颉编程语言中, 表达式几乎无处不在, 有表示各种计算的表达式(如算术表达式、逻辑表达式等), 也有表示分支和循环的表达式(如`if`表达式、循环表达式等)
>
> 对于包含多个操作符的表达式, 必须明确每个操作符的优先级、结合性以及操作数的求值顺序
>
> 优先级和结合性规定了操作数与操作符的结合方式, 操作数的求值顺序规定了二元和三元操作符的操作数求值顺序, 它们都会对表达式的值产生影响
>
> **注: 本章中对于各操作符的操作数类型的规定, 均建立在操作符没有被重载的前提下**

### 自增自减表达式

> 自增自减表达式是包含自增操作符(`++`)或自减操作符(`--`)的表达式
>
> **`a++`与`a--`分别是`a+=1`与`a-=1`的语法糖**
>
> 自增和自减操作符实现对值的加`1`和减`1`操作, 且**只能作为后缀操作符**使用
>
> `++`和`--`是非结合的, 所以类似于`a++--`这样的在同一个表达式中同时包含两个及以上`++/--`但未使用圆括号规定计算顺序的表达式是被(从语法上)禁止的
>
> 自增(自减)表达式的语法定义为:
>
> ```ANTLR
> incAndDecExpression
>     : postfixExpression ('++' | '--' )
>     ;
> ```
>
> 对于表达式`expr++`(或`expr--`), 规定如下:
>
> 1. `expr`的类型必须是整数类型
> 2. 因为`expr++`(或`expr--`)是`expr += 1`(或`expr -= 1`)的语法糖, 所以此`expr`同时必须也是可被赋值的(见赋值表达式)
> 3. `expr++`(或`expr--`)的类型为`Unit`
>
> 自增(自减)表达式举例:
>
> ```cangjie
> var i: Int32 = 5
> i++         // i = 6
> i--         // i = 5
> i--++       // syntax error
> var j = 0
> j = i--     // semantics error
> ```

仓颉规定`++`和`--`只能后置, 且只能用于整型, 且变量必须可变

这点和C/C++很不一样, C++是可以重载`++`和`--`运算符的, 且区分前置和后置

### 算术表达式

> 算术表达式是包含算术操作符的表达式
>
> 仓颉编程语言支持的算术操作符包括: 一元负号(`-`)、加(`+`)、减(`-`)、乘(`*`)、除(`/`)、取余(`%`)、求幂(`**`)
>
> 除了一元负号是一元前缀操作符, 其他操作符均是二元中缀操作符
>
> 它们的优先级和结合性参见下文
>
> 算术表达式的语法定义为:
>
> ```
> prefixUnaryExpression
>     : prefixUnaryOperator* incAndDecExpression
>     ;
>
> prefixUnaryOperator
>     : '-'
>     | ...
>     ;
>
> additiveExpression
>     : multiplicativeExpression (additiveOperator multiplicativeExpression)*
>     ;
>
> multiplicativeExpression
>     : exponentExpression (multiplicativeOperator exponentExpression)*
>     ;
>
> exponentExpression
>     : prefixUnaryExpression (exponentOperator prefixUnaryExpression)*
>     ;
>
> additiveOperator
>     : '+' | '-'
>     ;
>
> multiplicativeOperator
>     : '*' | '/' | '%'
>     ;
>
> exponentOperator
>     : '**'
>     ;
> ```

> 一元负号(`-`)的操作数只能是数值类型的表达式
>
> 一元前缀负号表达式的值等于操作数取负的值, 类型和操作数的类型相同:
>
> ```cangjie
> let num1: Int64 = 8
> let num2 = -num1      // num2 = -8, with 'Int64' type
> let num3 = -(-num1)   // num3 =  8, with 'Int64' type
> ```
>
> 对于二元操作符`*`, `/`, `%`, `+`和`-`, 要求两个操作数的类型相同
>
> 其中`%`的操作数只支持整数类型, `*`, `/`, `+`和`-`的操作数可以是任意数值类型
>
> ```cangjie
> let a = 2 + 3       // add:       5
> let b = 3 - 1       // sub:       2
> let c = 3 * 4       // multi:    12
> let d = 6.6 / 1.1   // division:  6
> let e = 4 % 3       // mod:       1
> ```
>
> 特别地, 除法(`/`)的操作数为整数时, 将非整数值向 0 的方向舍入为整数
>
> **整数取余运算`a % b`的值定义为`a - b * (a / b)`**
>
> ```cangjie
> let q1 =  7 /  3    // integer division:   2
> let q2 = -7 /  3    // integer division:  -2
> let q3 =  7 / -3    // integer division:  -2
> let q4 = -7 / -3    // integer division:   2
> let r1 =  7 %  3    // integer remainder:  1
> let r2 = -7 %  3    // integer remainder: -1
> let r3 =  7 % -3    // integer remainder:  1
> let r4 = -7 % -3    // integer remainder: -1
> ```

如果不确定负数之间的取余结果, 仓颉给定了取余运算的定义, 可以按照此进行计算

> `**`表示求幂运算(如`x**y`表示计算底数 x 的 y 次幂)
>
> `**`的左操作数只能为`Int64`类型或`Float64`类型, 并且:
>
> 1. 当左操作类型为`Int64`时, 右操作数只能为`UInt64`类型, 表达式的类型为`Int64`
> 2. 当左操作类型为`Float64`时, 右操作数只能为`Int64`类型或`Float64`类型, 表达式的类型为`Float64`
>
> ```cangjie
> let p1 = 2 ** 3                   // p1 = 8
> let p2 = 2 ** UInt64(3 ** 2)      // p2 = 512
> let p3 = 2.0 ** 3.0               // p3 = 8.0
> let p4 = 2.0 ** 3 ** 2            // p4 = 512.0
> let p5 = 2.0 ** 3.0               // p5 = 8.0
> let p6 = 2.0 ** 3.0 ** 2.0        // p6 = 512.0
> ```
>
> 当左操作类型为`Float64`, 右操作数类型为`Int64`时, 存在一些特殊情况需要明确求幂表达式的值
>
> 具体罗列为:
>
> ```cangjie
> x ** 0 = 1.0                                   // 对 任意 x
> 0.0 ** n = POSITIVE_INFINITY                   // 对 n < 0 的奇数
> -0.0 ** n = NEGATIVE_INFINITY                  // 对 n < 0 的奇数
> 0.0 ** n = POSITIVE_INFINITY                   // 对 n < 0 的偶数
> -0.0 ** n = POSITIVE_INFINITY                  // 对 n < 0 的偶数
> 0.0 ** n = 0.0                                 // 对 n > 0 的偶数
> -0.0 ** n = 0.0                                // 对 n > 0 的偶数
> 0.0 ** n = 0.0                                 // 对 n > 0 的奇数
> -0.0 ** n = -0.0                               // 对 n > 0 的奇数
> POSITIVE_INFINITY ** n = POSITIVE_INFINITY     // 对 n > 0
> NEGATIVE_INFINITY ** n = NEGATIVE_INFINITY     // 对 n > 0 的奇数
> NEGATIVE_INFINITY ** n = POSITIVE_INFINITY     // 对 n > 0 的偶数
> POSITIVE_INFINITY ** n = 0.0                   // 对 n < 0
> NEGATIVE_INFINITY ** n = -0.0                  // 对 n < 0 的奇数
> NEGATIVE_INFINITY ** n = 0.0                   // 对 n < 0 的偶数
> ```
>
> > 注: 在上述所列特殊情况之外, 当左操作数的值为`NaN`时, 无论右操作数取何值, 求幂表达式的值均等于`NaN`
>
> 当左操作类型为`Float64`, 右操作数类型为`Float64`时, 同样存在一些特殊情况需要明确求幂表达式的值
>
> 具体罗列为:
>
> ```cangjie
> x ** 0.0 = 1.0                              // 对 任意 x
> x ** -0.0 = 1.0                             // 对 任意 x
> 0.0 ** y = POSITIVE_INFINITY                // 对于 y 的值等于 < 0 的奇数整数
> -0.0 ** y = NEGATIVE_INFINITY               // 对于 y 的值等于 < 0 的奇数整数
> 0.0 ** NEGATIVE_INFINITY = POSITIVE_INFINITY
> -0.0 ** NEGATIVE_INFINITY = POSITIVE_INFINITY
> 0.0 ** POSITIVE_INFINITY = 0.0
> -0.0 ** POSITIVE_INFINITY = 0.0
> 0.0 ** y = 0.0                                 // 对 有限的 y > 0.0 且它的值等于奇数整数时
> -0.0 ** y = -0.0                               // 对 有限的 y > 0.0 且它的值等于奇数整数时
> -1.0 ** POSITIVE_INFINITY = 1.0
> -1.0 ** NEGATIVE_INFINITY = 1.0
> 1.0 ** y = 1.0                                 // 对 任意 y
> x ** POSITIVE_INFINITY = 0.0                   // 对 -1.0 < x < 1.0
> x ** POSITIVE_INFINITY = POSITIVE_INFINITY     // 对 任意 < -1.0 的 x 或 任意 > 1.0 的 x
> x ** NEGATIVE_INFINITY = POSITIVE_INFINITY     // 对 -1.0 < x < 1.0
> x ** NEGATIVE_INFINITY = 0.0                   // 对 任意 x < -1.0 或 任意 > 1.0 的 x
> POSITIVE_INFINITY ** y = 0.0                   // 对 y < 0.0
> POSITIVE_INFINITY ** y = POSITIVE_INFINITY     // 对 y > 0.0
> NEGATIVE_INFINITY ** y = -0.0                  // 对 有限的 y < 0.0 且它的值等于奇数整数时
> NEGATIVE_INFINITY ** y = NEGATIVE_INFINITY     // 对 有限的 y > 0.0 且它的值等于奇数整数时
> NEGATIVE_INFINITY ** y = 0.0                   // 对 有限的 y < 0.0 且它的值不为奇数整数时
> NEGATIVE_INFINITY ** y = POSITIVE_INFINITY     // 对 有限的 y > 0.0 且它的值不为奇数整数时
> 0.0 ** y = POSITIVE_INFINITY                   // 对 有限的 y < 0.0 且它的值不为奇数整数时
> -0.0 ** y = POSITIVE_INFINITY                  // 对 有限的 y < 0.0 且它的值不为奇数整数时
> 0.0 ** y = 0.0                                 // 对 有限的 y > 0.0 且它的值不为奇数整数时
> -0.0 ** y = 0.0                                // 对 有限的 y > 0.0 且它的值不为奇数整数时
> x ** y = NaN                                   // 对 有限的 x < 0.0 且 有限的 y 的值不为整数时
> ```
>
>> 注: 在上述所列特殊情况之外, 一旦有操作数的值为`NaN`, 则求幂表达式的值等于`NaN`

`**`幂运算符, 是C/C++中不存在的一个东西

仓颉中`**`运算符的左、右操作数都是只能为`Int64`和`Float64`类型

整型还好, 浮点型有些特殊结果的运算, 比较繁琐, 需要记

> 算术表达式结果为整数类型时存在整数溢出的可能
>
> 仓颉提供了**三种属性宏及编译选项来控制整数溢出的处理行为**(以下简称为行为)
>
> 如下表格所示:
>
> | attributes            | Options                     | behavior     | explaining for behavior                 |
> | --------------------- | --------------------------- | ------------ | --------------------------------------- |
> | `@OverflowThrowing`   | `--int-overflow throwing`   | `throwing`   | 抛出异常 (默认行为)                     |
> | `@OverflowWrapping`   | `--int-overflow wrapping`   | `wrapping`   | 数值回绕 (达到最大值后从最小值重新开始) |
> | `@OverflowSaturating` | `--int-overflow saturating` | `saturating` | 饱和处理 (达到最大值后保持最大值不变)   |
>
> > 注:
> >
> > 1. 默认的行为为 throwing
> > 1. 假设属性宏与编译选项同时作用在某一范围, 且代表的行为不相同, 则该范围以属性宏代表的行为为准
>
> 行为示例:
>
> ```cangjie
> @OverflowThrowing
> func test1(x: Int8, y: Int8) { // if x equals to 127 and y equals to 3
>     let z = x + y // throwing OverflowException
> }
>
> @OverflowWrapping
> func test2(x: Int8, y: Int8) { // if x equals to 127 and y equals to 3
>     let z = x + y // z equals to -126
> }
>
> @OverflowSaturating
> func test3(x: Int8, y: Int8) { // if x equals to 127 and y equals to 3
>     let z = x + y // z equals to 127
> }
> ```

仓颉对整型计算时可能出现的溢出, 定义了三种处理方法: 抛异常、数值回绕以及饱和处理

> 属性宏与编译选项作用范围冲突示例:
>
> ```cangjie
> // 编译: cjc --int-overflow saturating test.cj
>
> @OverflowWrapping
> func test2(x: Int8, y: Int8) {
>     let z = x + y // the behavior is wrapping
> }
>
> func test3(x: Int8, y: Int8) {
>     let z = x + y // the behavior is saturating
> }
> ```
>
> 特别地, 对于`INT_MIN * -1`, `INT_MIN / -1`和`INT_MIN % -1`, 规定的行为如下:
>
> | Expression                     | Throwing                     | Wrapping  | Saturating |
> | ------------------------------ | ---------------------------- | --------- | ---------- |
> | `INT_MIN * -1 or -1 * INT_MIN` | `throwing OverflowException` | `INT_MIN` | `INT_MAX`  |
> | `INT_MIN / -1`                 | `throwing OverflowException` | `INT_MIN` | `INT_MAX`  |
> | `INT_MIN % -1`                 | `0`                          | `0`       | `0`        |
>
> 需要注意的是, 对于整数溢出行为是 throwing 的场景, 若整数溢出可提前在编译期检测出来, 则编译器会直接给出报错

如果代码中存在属性宏, 但编译时制定了编译选项

那么, 属性宏优先, 编译选项其次

### 关系表达式

> 关系表达式是包含关系操作符的表达式
>
> 关系操作符包括`6`种: 相等(`==`)、不等(`!=`)、小于(`<`)、小于等于(`<=`)、大于(`>`)、大于等于(`>=`)
>
> 关系操作符都是二元操作符, 并且要求两个操作数的类型是一样的
>
> 关系表达式的类型是`Bool`类型, 即**值只可能是`true`或`false`**
>
> 关系操作符的优先级和结合性见下文
>
> 关系表达式的语法定义为:
>
> ```
> equalityComparisonExpression
>     : comparisonOrTypeExpression (equalityOperator comparisonOrTypeExpression)?
>     ;
>
> comparisonOrTypeExpression
>     : shiftingExpression (comparisonOperator shiftingExpression)?
>     | ...
>     ;
>
> equalityOperator
>     : '!=' | '=='
>     ;
>
> comparisonOperator
>     : '<' | '>' | '<=' | '>='
>     ;
> ```
>
> 关系表达式举例:
>
> ```cangjie
> main(): Int64 {
>     3 < 4         // return true
>     3 <= 3        // return true
>     3 > 4         // return false
>     3 >= 3        // return true
>     3.14 == 3.15  // return false
>     3.14 != 3.15  // return true
>     return 0
> }
> ```
>
> 需要注意的是, 关系运算符是非结合(non-associative)运算符, 即无法写出类似于`a < b < c`这样的表达式
>
> ```cangjie
> main(): Int64 {
>     3 < 4 < 5      // error:`<`is non-associative
>     3 == 3 != 4    // error:`==`and`!=`are non-associative
>     return 0
> }
> ```

关系运算符是非结合运算符, 这一点与C/C++一致, 但仓颉中强制关系表达式为`Bool`类型且不可类型转换

### `type test`和`type cast`表达式

> `type test`表达式是包含操作符`is`的表达式, `type cast`表达式是包含操作符`as`的表达式
>
> `is`和`as`的优先级和结合性参见下文
>
> `type test`和`type cast`表达式的语法定义为:
>
> ```
> comparisonOrTypeExpression
>     : ...
>     | shiftingExpression ('is' type)?
>     | shiftingExpression ('as' userType)?
>     ;
> ```

#### `is`操作符

> `e is T`是一个用于类型检查的表达式, `e is T`的类型是`Bool`
>
> 其中`e`可以是任何类型的表达式, `T`可以是任何类型
>
> 当`e`的运行时类型`R`是`T`的子类型时, `e is T`的值为`true`, 否则值为`false`
>
> `is`操作符举例:
>
> ```cangjie
> open class Base {
>     var name: String = "Alice"
> }
> class Derived1 <: Base {
>     var age: UInt8 = 18
> }
> class Derived2 <: Base {
>     var gender: String = "female"
> }
>
> main(): Int64 {
>     var testVT = 1 is Int64            // testVT = true
>     testVT = 1 is String               // testVT = false
>     testVT = true is Int64             // testVT = false
>     testVT = [1, 2, 3] is Array<Int64> // testVT = true
>
>     let base1: Base = Base()
>     let base2: Base = Derived1()
>     let base3: Base = Derived2()
>
>     let derived1: Derived1 = Derived1()
>     let derived2: Derived2 = Derived2()
>
>     var test = base1 is Base          // test = true
>     test = base1 is Derived1          // test = false
>     test = base1 is Derived2          // test = false
>     test = base2 is Base              // test = true
>     test = base2 is Derived1          // test = true
>     test = base2 is Derived2          // test = false
>     test = base3 is Base              // test = true
>     test = base3 is Derived1          // test = false
>     test = base3 is Derived2          // test = true
>
>     test = derived1 is Base           // test = true
>     test = derived1 is Derived1       // test = true
>     test = derived1 is Derived2       // test = false
>     test = derived2 is Base           // test = true
>     test = derived2 is Derived1       // test = false
>     test = derived2 is Derived2       // test = true
>
>     return 0
> }
> ```

仓颉提供了`is`操作符, 可以用于判断变量是否是某个类型:`value is Type`, 如果`value`是`Type`的子类型, 表达式值也会为`true`

此为`type test`表达式

#### `as`操作符

> `e as T`是一个用于类型转换的表达式, `e as T`的类型是`Option<T>`
>
> 其中`e`可以是任何类型的表达式, `T`可以是任何具体类型
>
> 当`e`的运行时类型`R`是`T`的子类型时, `e as T`的值为`Some(e)`, 否则值为`None`
>
> `as`操作符举例:
>
> ```cangjie
> open class Base {
>     var name: String = "Alice"
> }
> class Derived1 <: Base {
>     var age: UInt8 = 18
> }
> class Derived2 <: Base {
>     var gender: String = "female"
> }
>
> main(): Int64 {
>     let base1: Base = Base()
>     let base2: Base = Derived1()
>     let base3: Base = Derived2()
>
>     let derived1: Derived1 = Derived1()
>     let derived2: Derived2 = Derived2()
>
>     let castOP1 = base1 as Base                // castOP = Option<Base>.Some(base1)
>     let castOP2 = base1 as Derived1            // castOP = Option<Derived1>.None
>     let castOP3 = base1 as Derived2            // castOP = Option<Derived2>.None
>     let castOP4 = base2 as Base                // castOP = Option<Base>.Some(base2)
>     let castOP5 = base2 as Derived1            // castOP = Option<Derived1>.Some(base2)
>     let castOP6 = base2 as Derived2            // castOP = Option<Derived2>.None
>     let castOP7 = base3 as Base                // castOP = Option<Base>.Some(base3)
>     let castOP8 = base3 as Derived1            // castOP = Option<Derived1>.None
>     let castOP9 = base3 as Derived2            // castOP = Option<Derived2>.Some(base3)
>
>     let castOP10 = derived1 as Base            // castOP = Option<Base>.Some(derived1)
>     let castOP11 = derived1 as Derived1        // castOP = Option<Derived1>.Some(derived1)
>     let castOP12 = derived1 as Derived2        // castOP = Option<Derived2>.None
>     let castOP13 = derived2 as Base            // castOP = Option<Base>.Some(derived2)
>     let castOP14 = derived2 as Derived1        // castOP = Option<Derived1>.None
>     let castOP15 = derived2 as Derived2        // castOP = Option<Derived2>.Some(derived2)
>
>     return 0
> }
> ```

`as`是用于类型转换的, 不过转换的结果的类型不是原始类型, 而是`Option<T>`类型

`value as Type`, 如果`value`的类型是`Type`的子类型, 则表达式结果为`Option<Type>.Some(value)`

此为`type cast`

### 位运算表达式

> 位运算表达式是包含位运算操作符的表达式
>
> 仓颉编程语言支持 1 种一元前缀位运算操作符: 按位求反(`!`)
>
> 以及 5 种二元中缀位运算操作符: 左移(`<<`)、右移(`>>`)、按位与(`&`)、按位异或(`^`)和按位或(`|`)
>
> 位运算操作符的操作数**只能为整数类型**, 通过将操作数视为二进制序列, 然后在每一位上进行逻辑运算(`0`视为`false`, `1`视为`true`)或移位操作来实现位运算
>
> `&`、`^`和`|`的操作中, 位与位之间执行的是逻辑操作(参见 [逻辑表达式])
>
> 位运算操作符的优先级和结合性参见下文
>
> 位运算表达式的语法定义为:
>
> ```
> prefixUnaryExpression
>     : prefixUnaryOperator* incAndDecExpression
>     ;
>
> prefixUnaryOperator
>     : '!'
>     | ...
>     ;
>
> bitwiseDisjunctionExpression
>     : bitwiseXorExpression ( '|' bitwiseXorExpression)*
>     ;
>
> bitwiseXorExpression
>     : bitwiseConjunctionExpression ( '^' bitwiseConjunctionExpression)*
>     ;
>
> bitwiseConjunctionExpression
>     : equalityComparisonExpression ( '&' equalityComparisonExpression)*
>     ;
>
> shiftingExpression
>     : additiveExpression (shiftingOperator additiveExpression)*
>     ;
>
> shiftingOperator
>     : '<<' | '>>'
>     ;
> ```
>
> 位运算表达式举例:
>
> ```cangjie
> func foo(): Unit {
>     !10                 // The result is -11
>     !20                 // The result is -21
>     10 << 1             // The result is 20
>     10 << 1 << 1        // The result is 40
>     10 >> 1             // The result is 5
>     10 & 15             // The result is 10
>     10 ^ 15             // The result is 5
>     10 | 15             // The result is 15
>     1 ^ 8 & 15 | 24     // The result is 25
> }
> ```

如果已经熟悉C/C++中的位运算, 仓颉的位运算应该也不会陌生

> 对于移位操作符, 要求其操作数必须是整数类型(但两个操作数的类型可以不一样), 并且无论左移还是右移, **右操作数都不允许为负数**(对于编译时可检查出的此类错误, 编译报错, 如果运行时发生此错误, 则抛出异常)
>
> 对于**无符号数**的移位操作, 移位和补齐规则是: 左移低位补 0 高位丢弃, 右移高位补 0 低位丢弃
>
> 对于**有符号数**的移位操作, 移位和补齐规则是:
>
> 1. 正数和无符号数的移位补齐规则一致
> 2. 负数左移低位补 0 高位丢弃
> 3. 负数右移高位补 1 低位丢弃
>
> ```cangjie
> let p: Int8 = -30
> let q = p << 2      // q = -120
> let r = p >> 2      // r = -8
> let r = p >> -2     // error
> let x: UInt8 = 30
> let b = x << 3      // b = 240
> let b = x >> 1      // b = 15
> ```
>
> 另外, **如果右移或左移的位数(右操作数)等于或者大于操作数的宽度, 则为`overshift`, 如果编译时可以检测到则报错, 否则运行时抛出异常**
>
> ```cangjie
> let x1 : UInt8 = 30     // 0b00011110
> let y1 = x1 >> 11       // compilation error
> ```

仓颉的位运算, 除了移位时超出位数的强制安全措施, 其他运算方式基本与与C/C++中的一致

### 区间表达式

> 区间表达式是包含区间操作符的表达式
>
> 区间表达式用于创建`Range`实例
>
> 区间表达式的语法定义为:
>
> ```
> rangeExpression
>     : bitwiseDisjunctionExpression ('..=' | '..') bitwiseDisjunctionExpression (':' bitwiseDisjunctionExpression)?
>     | bitwiseDisjunctionExpression
>     ;
> ```
>
> 区间操作符有两种: `..`和`..=`, 分别用于创建"左闭右开"和"左闭右闭"的`Range`实例
>
> 关于它们的介绍, 请参见 [Range 类型](https://www.humid1ch.cn/blog/cangjie-docs-reading-ii#heading-18)

### 逻辑表达式

> 逻辑表达式是包含逻辑操作符的表达式
>
> 逻辑操作符的操作数只能为`Bool`类型的表达式
>
> 仓颉编程语言支持 3 种逻辑操作符: 逻辑非(`!`)、逻辑与(`&&`)、逻辑或(`||`)
>
> 它们的优先级和结合性参见下文
>
> 逻辑表达式的语法定义为:
>
> ```
> prefixUnaryExpression
>     : prefixUnaryOperator* incAndDecExpression
>     ;
>
> prefixUnaryOperator
>     : '!'
>     | ...
>     ;
>
> logicDisjunctionExpression
>     : logicConjunctionExpression ( '||' logicConjunctionExpression)*
>     ;
>
> logicConjunctionExpression
>     : rangeExpression ( '&&' rangeExpression)*
>     ;
> ```
>
> **逻辑非(`!`)**是一元操作符, 它的作用是对其操作数的布尔值取反: `!false`的值等于`true`, `!true`的值等于`false`
>
> **逻辑与(`&&`)和逻辑或(`||`)**均是二元操作符
>
> 对于表达式`expr1 && expr2`, 只有当`expr1`和`expr2`的值均等于`true`时, 它的值才等于`true`
>
> 对于表达式`expr1 || expr2`, 只有当`expr1`和`expr2`的值均等于`false`时, 它的值才等于`false`
>
> `&&`和`||`采用短路求值策略:
>
> 计算`expr1 && expr2`时, 当`expr1=false`则无需对`expr2`求值, 整个表达式的值为`false`
>
> 计算`expr1 || expr2`时, 当`expr1=true`则无需对`expr2`求值, 整个表达式的值为`true`
>
> ```cangjie
> main(): Int64 {
>     let expr1 = false
>     let expr2 = true
>     !true                           // Logical NOT, return false.
>     1 > 2 && expr1                  // Logical AND, return false without computing the value of expr1.
>     1 < 2 || expr2                  // Logical OR, return true without computing the value of expr2.
>     return 0
> }
> ```

仓颉中的逻辑操作符, 与C/C++中的也一致, 包括**短路求值策略**

### `coalescing`表达式 **

> `coalescing`表达式是包含`coalescing`操作符的表达式
>
> `coalescing`操作符使用`??`表示, `??`是二元中缀操作符
>
> 其优先级和结合性参见下文
>
> `coalescing`表达式的语法定义为:
>
> ```
> coalescingExpression
>     : logicDisjunctionExpression ('??' logicDisjunctionExpression)*
>     ;
> ```
>
> `coalescing`操作符用于`Option`类型的解构
>
> 假设表达式`e1`的类型是`Option<T>`, 对于表达式`e1 ?? e2`, 规定:
>
> 1. 表达式`e2`具有类型`T`
>
> 2. 表达式`e1 ?? e2`具有类型`T`
>
> 3. 当`e1`的值等于`Option<T>.Some(v)`时, `e1 ?? e2`的值等于`v`的值(此时, 不会再去对`e2`求值, 即满足"短路求值")
>
>     当`e1`的值等于`Option<T>.None`时, `e1 ?? e2`的值等于`e2`的值
>
> 表达式`e1 ?? e2`是如下`match`表达式的语法糖:
>
> ```cangjie
> // when e1 is Option<T>
> match (e1) {
>     case Some(v) => v
>     case None => e2
> }
> ```
>
> `coalescing`表达式使用举例:
>
> ```cangjie
> main(): Int64 {
>     let v1 = Option<Int64>.Some(100)
>     let v2 = Option<Int64>.None
>     let r1 = v1 ?? 0
>     let r2 = v2 ?? 0
>     print("${r1}") // output: 100
>     print("${r2}") // output: 0
>
>     return 0
> }
> ```

`coalescing`也就是`??`是`Option`类型模式匹配的语法糖

是用来解构`Option`类型数据的

如果存在`e1`为`Option<T>`, 则`e1 ?? e2`中, `e2`类型需要为`T`, 此时:

1. 如果`e1`是`Some(value)`, 则此表达式值为`value`, 且不再计算`e2`
2. 如果`e1`是`None`, 则此表达式值为`e2`

也就是说, 对于`Option<T>`类型的变量, 如果比较简单就不用再写`match`匹配模式, 可以直接`??`进行解构
