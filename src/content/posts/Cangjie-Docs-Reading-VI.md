---
title: "仓颉文档阅读-语言规约IV: 表达式(I)"
published: 2025-09-25
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

### 字面量

> [字面量](https://www.humid1ch.cn/blog/cangjie-docs-reading-i#heading-4)是一种拥有固定语法的表达式
>
> 对于内部不包含其他表达式的字面量, 它的值就是字面量自身的值, 它的类型可由其语法或所在的上下文决定
>
> 当无法确定字面量类型时, 整数字面量具有`Int64`类型, 浮点数字面量具有`Float64`类型
>
> 对于可在内部包含其他表达式的集合类字面量和元组字面量(参见 [值类型]), 它的值等于对其内部所有表达式求值后得到的字面量的值, 它的类型由其语法确定
>
> 字面量举例:
>
> ```cangjie
> main(): Int64 {
>     10u8                          // UInt8 literal
>     10i16                         // Int16 literal
>     1024u32                       // UInt32 literal
>     1024                          // Int64 literal
>     1024.512_f32                  // Float32 literal
>     1024.512                      // Float64 literal
>     'a'                           // Rune literal
>     true                          // Bool literal
>     "Cangjie"                     // String literal
>     ()                            // Unit literal
>     [1, 2, 3]                     // Array<Int64> literal
>     (1, 2, 3)                     // (Int64, Int64, Int64) literal
>     return 0
> }
> ```

### 变量名和函数名

> 变量名和函数名(这里的变量名和函数名也包括通过包名指向的变量或函数)本身也是表达式
>
> 对于变量名, 它的值等于变量求值后的值, 它的类型为变量的类型;
>
> 对于函数名, 它的值是一个闭包, 它的类型为对应的函数类型
>
> 变量名和函数名举例:
>
> ```cangjie
> let intNum: Int64 = 100 // 'intNum' 是这个变量的名字, 值和类型分别是 100 和 Int64
>
> /* 'add' 是一个函数的名称, 其值和类型分别为 (p1: Int64, p2: Int64) => {p1 + p2} 和 (Int64, Int64) -> Int64 */
> func add(p1: Int64, p2: Int64) {
>     p1 + p2
> }
>
> let value = p1.x // x 是一个定义在 package p1中的变量
> ```
>
> 对于变量名, 规定`var`声明的变量始终是**可变**的, `let`声明的变量只可以被赋一次值(声明时或声明之后), 赋值前是**可变**的, 赋值后是**不可变**的

函数名是一个闭包?

仓颉函数可以在非顶层作用域中定义, 可以自然捕捉所在作用域已经定义的变量? 具体到时候再了解

#### 泛型函数名作为表达式

> 仓颉语言中支持函数做为第一成员, 同时也支持声明类型参数的泛型函数
>
> 当函数为泛型函数时, 函数名作为表达式使用时必须给出泛型函数的类型实参
>
> 例如:
>
> ```cangjie
> func identity<T>(a: T) { // identity 是一个泛型函数
>     return a
> }
>
> var b = identity // error: 泛型函数identity的调用需要指明类型参数
>
> var c = identity<Int32> // ok: Int32 作为类型参数
> ```
>
> `identity`是一个泛型函数, 所以`identity`不是合法的表达式, 只有给出了类型实参的`identity<Int32>`才是一个合法的表达式
>
> 若一个函数在当前作用域中被重载了, 当重载函数中存在多个类型完备的可选函数, 那么直接使用该类型名作为表达式是有歧义错误的, 例如:
>
> ```cangjie
> interface Add<T> {
>     operator func +(a: T): T
> }
>
> func add<T>(i: T, j: Int64): Int64 where T <: Add<Int64> { // f1
>     return i + j;
> }
>
> func add<T>(i: T, j: T): T where T <: Add<T> { // f2
>     return i + j;
> }
>
> main(): Int64 {
>     var plus = add<Int64> // error: 'add' 用法不明确
>
>     return 0
> }
> ```

上面的代码例子中, 先定义一个泛型接口`Add<T>`

重载了两个泛型函数`add<T>`, 且规定类型参数要实现了接口`Add<T>`

但如果泛型参数填入`Int64`, 这样调用就出现了歧义, 因为两个重载的函数如果传入`Int64`, 就长得一样了

### 条件表达式

> 条件表达式即`if`表达式, 可以根据判定条件是否成立来决定执行哪条代码分支, 实现分支控制逻辑
>
> `if`表达式的语法定义为:
>
> ```
> ifExpression
>     : 'if' '(' ('let' deconstructPattern '<-')? expression ')' block ('else' ( ifExpression | block))?
>     ;
> ```
>
> 其中`if`是关键字, `if`之后是一个包围在小括号内的表达式, 接着是一个块, 块之后是可选的`else`分支
>
> `else`分支以`else`关键字开始, 后接新的`if`表达式或一个块

仓颉中条件表达式的`()`中, 可以使用`let 解构模式`作为条件

只熟悉C/C++的我, 没有见过相关用法, 具体后面了解

> `if`表达式举例:
>
> ```
> main(): Int64 {
>     let x = 100
>
>     if (x > 0) {
>         print("x is larger than 0")
>     }
>
>     if (x > 0) {
>         print("x is larger than 0")
>     } else {
>         print("x is not larger than 0")
>     }
>
>     if (x > 0) {
>         print("x is larger than 0")
>     } else if (x < 0) {
>         print("x is lesser than 0")
>     } else {
>         print("x equals to 0")
>     }
>
>     return 0
> }
> ```
>
> `if`表达式首先对`if`之后的表达式进行求值(要求表达式的类型为`Bool`), 如果表达式的值等于`true`, 则执行它之后的块, 否则, 执行`else`之后的`if`表达式或块(如果存在)
>
> `if`表达式的值等于被执行到的分支中的表达式的值
>
> 对于包含`let`的`if`表达式, 我们称之为`if-let`表达式
>
> 我们可以用`if-let`表达式来做一些简单的解构操作
>
> 一个基础的`if-let`表达式举例:
>
> ```
> main(): Int64 {
>     let x: Option<Int64> = Option<Int64>.Some(100)
>
>     if (let Some(v) <- x) {
>         print("x has value")
>     }
>
>     if (let Some(v) <- x) {
>         print("x has value")
>     } else {
>         print("x has not value")
>     }
>
>     return 0
> }
> ```
>
> `if-let`表达式首先对`<-`之后的表达式进行求值(表达式的类型为可以是任意类型), 如果表达式的值能匹配`let`之后的 pattern, 则执行它之后的块, 否则, 执行`else`之后的`if`表达式或块(如果存在)
>
> `if-let`表达式的值等于被执行到的分支中的表达式的值
>
> `let`之后的 pattern 支持常量模式、通配符模式、绑定模式、`Tuple`模式、`enum`模式

普通`if`表达式与C/C++中的基本一样, 不过 仓颉中的普通`if`表达式, `()`内的表达式必须是`Bool`类型的, 且没有隐式类型转换

但仓颉中还有`if-let`表达式, 即`if (let)`, 此`if-let`表达式中`()`内需要是解构模式的用法, 可以是任意类型的

猜测, 用法应该是:

```cangjie
if (let 100 <- x)            // 常量模式
if (let _ <- x)                // 通配符模式, 可能等同于true
if (let v <- x)                // 绑定模式
if (let (v, l) <- (1, 2))    // 元组模式
if (let Some(v) <- x)        // Option 解构, enum 模式
```

用实际的代码测试:

```cangjie
main(): Int64 {
    var x = 100
    if (let 100 <- x) {
        println("常量模式: 匹配到 100")
    }

    if (let 99 <- x) {
        println("常量模式: 匹配到 99")
    }

    if (let _ <- x) {
        println("通配符模式")
    }

    if (let v <- x) {
        println("匹配模式: ${v}")
    }

    if (let (v, l) <- (1, 2)) {
        println("Tuple 解构: ${v} ${l}")
    }

    let y: Option<Int64> = 32
    if (let Some(v) <- y) {
        println("Option 解构: ${v}")
    }

    return 0
}
```

代码的执行结果为:

![](https://humid1ch.oss-cn-shanghai.aliyuncs.com/20250925151945749.webp)

#### `if`表达式的类型

> 对于没有`else`分支的`if`表达式, 它的类型为`Unit`, 它的值等于`()`
>
> 因为`if expr1 {expr2}`是`if expr1 {expr2; ()} else {()}`的语法糖
>
> 对于包含`else`分支的`if`表达式,
>
> - 如果`if`表达式的值没有被读取或者返回, 那么`if`表达式的类型为`Unit`, 两个分支不要求存在公共父类型
>
>     否则, 按如下规则检查
>
> - 在上下文没有明确的类型要求时, 如果`if`的两个分支类型, 设它们为`T1`和`T2`, 则`if`表达式的类型是`T1`和`T2`的最小公共父类型`T`
>
>     如果不存在最小公共父类型`T`, 则编译报错
>
> - 在上下文有明确的类型要求时, 此类型即为`if`表达式的类型
>
>     此时要求`if`的两个分支的类型都是上下文所要求的类型的子类型
>
> 举例如下:
>
> ```cangjie
> struct S1 { }
> struct S2 { }
>
> interface I1 {}
> class C1 <: I1 {}
> class C2 <: I1 {}
>
> interface I2{}
> class D1 <: I1 & I2 {}
> class D2 <: I1 & I2 {}
>
> func test12() {
>     if (true) {  // OK. 这个条件表达式的类型 Unit
>         S1()
>     } else {
>         S2()
>     }
>
>    if (true) {  // OK. 这个条件表达式的类型 Unit
>         C1()
>     } else {
>         C2()
>     }
>
>     return if (true) {  // Error. 返回`if`表达式 但`D1`和`D2`没有最小公共父类型
>         D1()
>     } else {
>         D2()
>     }
> }
> ```
>
> 注意: 为了保持代码格式的规整以及提高代码的可维护性, 同时为了避免悬垂`else`(dangling-else)问题, 仓颉编程语言**要求每个`if`分支和`else`分支中的执行部分(即使只有一条表达式)必须使用一对花括号括起来成为一个块**
>
> (悬垂`else`问题是指无法确定形如`if cond1 if cond2 expr1 else expr2`的代码中的`else expr2`是归属于内层`if`还是外层`if`的问题
>
> 如果其归属于内层`if`, 则代码应解读为`if cond1 (if cond2 expr1 else expr2)`
>
> 如果其归属于外层`if`, 则代码应解读为`if cond1 (if cond2 expr1)expr2`
>
> 但如果强制分支使用大括号, 则无此问题)

仓颉的条件表达式的值, 可以被读取 或 作为返回值被返回

**如果被读取或被作为返回值返回:**

1. **如果上下文没有指定类型(读取类型或返回值类型)**

    **表达式的类型是不同分支的 最小公共夫类型**

2. **如果上下文指定了类型, 那么表达式的每个分支都必须为指定类型或其子类型**

**其他情况下, 条件表达式就恒为`Unit`类型, 值为`()`, 不做任何类型判断**

仓颉中还有一个硬性规定: 条件表达式每个分支, 必须要有`{}`, 否则语法错误

### 模式匹配表达式

> 仓颉编程语言中支持使用模式匹配表达式(`match`表达式)实现模式匹配(pattern matching), 允许开发者使用更精简的代码描述复杂的分支控制逻辑
>
> 直观上看, 模式描述的是一种结构, 这个结构定义了一个与之匹配的实例集合, 模式匹配就是去判断给定的实例是否属于模式定义的实例集合
>
> 显然, 匹配的结果只有两种: 匹配成功和匹配失败
>
> `match`表达式可分为两类: 带 selector 的`match`表达式和不带 selector 的`match`表达式
>
> `match`表达式的语法定义为:
>
> ```
> matchExpression
>     : 'match' '(' expression ')' '{' matchCase+ '}'
>     | 'match' '{' ('case' (expression | '_') '=>' expressionOrDeclaration+)+ '}'
>     ;
> matchCase
>     :  'case' pattern ('|' pattern)* patternGuard? '=>' expressionOrDeclaration+
>     ;
> patternGuard
>     : 'where' expression
>     ;
> ```

C++直到现在也没有完善的模式匹配语法, 只有一个类似的只能匹配整型的`switch`

仓颉语言中, 模式匹配有两种样式的语法: 带 selector 和 不带 selector , 即待匹配的变量

> 对于带 selector 的`match`表达式, 关键字`match`之后的`expression`即为待匹配的 selector
>
>  selector 之后的`{}`内可定义若干`matchCase`
>
> 每个`matchCase`以关键字`case`开头, 后跟一个 pattern 或者多个由`|`分隔的相同种类的 pattern (关于不同 pattern 的详细定义, 见[模式]节)
>
> 一个可选的 pattern guard
>
> 一个胖箭头`=>`和一系列(至少一个)声明或表达式(多个声明或表达式之间使用分号或换行符分隔)
>
> 在执行`match`表达式的过程中, 匹配顺序即`case`定义的顺序,  selector 按照匹配顺序依次和`case`中定义的 pattern 进行匹配, **一旦 selector 和当前 pattern 匹配成功(且满足 pattern guard )**, 则执行`=>`之后的代码, 且**无需再与它之后的 pattern 进行匹配**
>
> 否则(与当前 pattern 不匹配), 继续与下一个 pattern 进行匹配判断, 依次类推
>
> 下面的例子展示了`match`表达式中使用常量模式进行分支控制:
>
> ```cangjie
> let score: Int64 = 90
> var scoreResult: String = match (score) {
>     case 0 => "zero"
>     case 10 | 20 | 30 | 40 | 50 => "fail"
>     case 60 => "pass"
>     case 70 | 80 => "good"
>     case 90 | 100 => "excellent" // matched
>     case _ => "not a valid score"
> }
> ```
>
> 出于安全和完备的考虑, 仓颉编程语言要求`case`表达式中定义的所有 pattern 和其对应的 patternGuard (如果存在)组合起来**要覆盖 selector 的所有可能取值**(即`exhaustive`), 如果编译器判断出未实现完全覆盖, 则会报错
>
> 为实现完全覆盖, 通常可以在最后一个`case`中使用通配符`_`来处理其他`case`未覆盖到的情况
>
> 另外, 不要求每个 pattern 定义的空间是互斥的, 即不同 pattern 之间覆盖的空间可以有重叠

仓颉匹配模式, **只会命中第一个成功匹配的 pattern **, 命中之后就不在进行之后的匹配

模式匹配的 pattern , 需要满足能够匹配到 selector 可能存在的所有值

当然, 可以用通配符`_`实现此情况

> 对于不带 selector 的`match`表达式, 关键字`match`之后没有`expression`, 并且`{}`中的每条`case`中, 关键字`case`和`=>`之间 **只能为一个`Bool`类型的表达式(或者通配符`_`, 表示永远为`true`)**
>
> 在执行的过程中, 依次判断`case`之后的表达式的值, 一旦表达式的值等于`true`, 则执行`=>`之后的代码, 且无需再判断它之后的所有`case`
>
> 事实上, 不带 selector 的`match`表达式其实是一连串嵌套`if-else if`表达式的简洁表达
>
> 同样地, 要求不带 selector 的`match`表达式满足`exhaustive`(即任何情况下至少有一个`case`是满足的)
>
> 编译器会尽量做`exhaustive`检查, 如果无法判断, 则报错并提示添加`case _`
>
> ```cangjie
> let score = 80
> var result: String = match {
>     case score < 60 => "fail"
>     case score < 70 => "pass"
>     case score < 90 => "good" // matched
>     case _ => "excellent"
> }
> ```

不带 selector 的模式匹配, **`case`和`=>`之间只能是`Bool`类型的表达式**

不过, 没有限制可使用的变量, 即带 selector 的模式匹配, 只能匹配特定的一个变量, 而不带 selector 的模式匹配, 可以尝试不同变量的匹配

但, 依旧只能命中一个 pattern

> 类似于`if`表达式, 对于`match`表达式, 无论其是否有 selector , 它的类型遵循如下规则:
>
> - 如果`match`表达式的值没有被读取或者返回, 那么`match`表达式的类型为`Unit`, 所有分支不要求存在公共父类型; 否则, 按如下规则检查
>
> - 在上下文没有明确的类型要求时, 假设`match`的所有分支类型分别为`T1, ..., Tn`, 则`match`表达式的类型是`T1, ..., Tn`的最小公共父类型`T`, 如果不存在最小公共父类型`T`则报错
>
> - 在上下文有明确的类型要求时, 此类型即为`match`表达式的类型
>
>     此时要求每条`case`中`=>`之后的表达式的类型都是上下文所要求的类型的子类型

模式匹配表达式的类型和值, 可以参照条件表达式

只不过模式匹配是根据`=>`之后的内容进行解析的

#### 模式

> 仓颉编程语言提供了丰富的模式种类, 包括:
>
> 1. 常量模式(constant patterns)
> 2. 通配符模式(wildcard patterns)
> 3. 绑定模式(binding patterns)
> 4. `tuple`模式(tuple patterns)
> 5. 类型模式(type patterns)
> 6. `enum`模式(enum patterns)
>
>  pattern 的语法定义为:
>
> ```
> pattern
>     : constantPattern
>     | wildcardPattern
>     | varBindingPattern
>     | tuplePattern
>     | typePattern
>     | enumPattern
>     ;
> ```

##### 常量模式

> 常量模式可以是整数字面量、字节字面量、浮点数字面量、`Rune`字面量、布尔字面量、字符串字面量**(不支持字符串插值)**、Unit 字面量
>
> 常量模式中字面量的类型需要和 selector 的类型一致,  selector 和一个常量模式匹配成功的条件是 selector 与常量模式中的字面量相等(这里指值相等)
>
> 常量模式的语法定义为:
>
> ```
> constantPattern
>     : literalConstant
>     ;
> ```
>
> 使用常量模式的例子如下:
>
> ```cangjie
> func f() {
>     let score: Int64 = 90
>     var scoreResult: String = match (score) {
>         case 0 => "zero"
>         case 10 | 20 | 30 | 40 | 50 => "fail"
>         case 70 | 80 => "good"
>         case 90 => "excellent" // matched
>         case _ => "not a valid score"
>     }
> }
> ```
>
> 需要注意的是, 浮点数字面量匹配在常量模式中遵循浮点数的判等规则, 会和判等存在一样的精度问题

常量模式是最简单基础的用法, 就是判断 selector 与 pattern 是否匹配, 然后执行目标语句

##### 通配符模式

> 通配符模式使用下划线`_`表示, 它可以**匹配任意值**, 常用于部分匹配(例如作为占位符)或作为`match`表达式的最后一个 pattern 来匹配其它`case`未覆盖到的情况
>
> 通配符模式的语法定义为:
>
> ```
> wildcardPattern
>     : '_'
>     ;
> ```
>
> 使用通配符模式的例子如下:
>
> ```cangjie
> let score: Int64 = 90
> var scoreResult: String = match (score) {
>     case 60 => "pass"
>     case 70 | 80 => "good"
>     case 90 | 100 => "excellent" // matched
>     case _ => "fail"             // wildcard pattern: used for default case
> }
> ```

通配符, 匹配任意值

如果作为第一个 pattern , 那就只能命中第一个`case`了

##### 绑定模式

> 绑定模式同样可以匹配任意值, 但与通配符模式不同的是, 绑定模式会将匹配到的值绑定到 binding pattern 中定义的变量, 以便在`=>`之后的表达式中进行访问
>
> 绑定模式的语法定义为:
>
> ```
> varBindingPattern
>     : identifier
>     ;
> ```
>
> **绑定模式中定义的变量是不可变的**
>
> 使用绑定模式的例子如下:
>
> ```cangjie
> let score: Int64 = 90
> var scoreResult: String = match (score) {
>     case 60 => "pass"
>     case 70 | 80 => "good"
>     case 90 | 100 => "excellent" // matched
>     case failScore =>            // binding pattern
>         let passNeed = 60 - failScore
>         "failed with ${failScore}, and ${passNeed} need to pass"
> }
> ```
>
> 绑定模式中定义的变量, 作用域从第一次出现的位置处开始至下一个`case`之前
>
> 需要注意, **使用`|`连接多个模式时不能使用绑定模式, 也不可嵌套出现在其它模式中**, 否则会报错
>
> ```cangjie
> let opt = Some(0)
> match (opt) {
>     case x | x => {}                // error: variable cannot be introduced in patterns connected by '|'
>     case Some(x) | Some(x) => {}    // error: variable cannot be introduced in patterns connected by '|'
>     case x: Int64 | x: String => {} // error: variable cannot be introduced in patterns connected by '|'
> }
> ```

绑定模式, 就是会将 selector 的值, 绑定到定义的变量中

此变量可以在`=>`之后的表达式中使用, 也是会匹配任意值

如果使用绑定模式, 就无法用`|`尝试连接多个匹配

##### `Tuple`模式

>  tuple pattern 用于匹配`Tuple`值, tuple pattern 定义为由圆括号括起来的多个 pattern , 每个 pattern 之间使用逗号分隔: `(pattern_1, pattern_2, … pattern_k)`
>
> 例如, `(x, y, z)`是由三个 binding pattern 组成的一个 tuple pattern , `(1, 0, 0)`是由三个 constant pattern 组成的一个 tuple pattern
>
>  tuple pattern 中的子 pattern 个数需要和 selector 的维度相同, 并且如果子 pattern 是 constant pattern 或 enum pattern 时, 其类型要和 selector 对应维度的类型相同
>
> tuple 模式的语法定义为:
>
> ```
> tuplePattern
>     : '(' pattern (',' pattern)+ ')'
>     ;
> ```
>
> 使用 tuple 模式的例子如下:
>
> ```cangjie
> let scoreTuple = ("Allen", 90)
> var scoreResult: String = match (scoreTuple) {
>     case ("Bob", 90) =>  "Bob got 90"
>     case ("Allen", score) =>  "Allen got ${score}" // matched
>     case ("Allen", 100) | ("Bob", 100) =>  "Allen or Bob got 100"
>     case (_, _) =>  ""
> }
> ```

##### 类型模式

> 使用类型模式可以很方便地实现`type check`和`type cast`
>
> 类型模式的语法定义为:
>
> ```
> typePattern
>   : (wildcardPattern | varBindingPattern) ':' type
>   ;
> ```
>
> 对于类型模式`varBindingPattern : type`(或`wildcardPattern : type`)
>
> 首先判断要匹配的值的运行时类型是否是`:`右侧`type`定义的类型或它的子类, 若类型匹配成功则将值的类型转换为`type`定义的类型, 然后将新类型的值与`:`左侧的`varBindingPattern`进行绑定(对于`wildcardPattern : type`, 不存在绑定)
>
> 只有类型匹配, 才算成功匹配, 否则匹配失败, 因此, `varBindingPattern : type`(或`wildcardPattern : type`)可以同时实现`type test`和`type cast`
>
> 使用类型模式匹配的例子如下:
>
> ```cangjie
> open class Point {
>     var x: Int32 = 1
>     var y: Int32 = 2
>     init(x: Int32, y: Int32) {
>         this.x = x
>         this.y = y
>     }
> }
> class ColoredPoint <: Point {
>     var color: String = "green"
>     init(x: Int32, y: Int32, color: String) {
>         super(x, y)
>         this.color = color
>     }
> }
>
> let normalPt = Point(5,10)
> let colorPt = ColoredPoint(8,24,"red")
> var rectangleArea1: Int32 = match (normalPt) {
>     case _: Point => normalPt.x * normalPt.y  // matched
>     case _ => 0
> }
> var rectangleArea2: Int32 = match (colorPt) {
>     case cpt: Point => cpt.x * cpt.y          // matched
>     case _ => 0
> }
> ```

类型模式, 是根据 selector 的类型进行匹配的, 可以是通配符模式 也可以绑定模式

可以匹配 selector 的类型以及子类型, **如果是子类型且存在绑定, 则绑定的变量会变为父类型变量**

##### `enum`模式

> `enum`模式主要和`enum`类型配合使用
>
>  enum pattern 用于匹配`enum constructor`, 格式是`constructorName`(无参构造器)或`constructorName(pattern_1, pattern_2, ..., pattern_k)`(有参构造器), 圆括号内用逗号分隔的若干 pattern (可以是其它任何类型的 pattern , 并允许嵌套)依次对每个参数进行匹配
>
> `enum`模式的语法定义为:
>
> ```
> enumPattern
>    : (userType '.')? identifier enumPatternParameters?
>    ;
> enumPatternParameters
>    : '(' pattern (',' pattern)* ')'
>    ;
> ```
>
> 使用`enum`模式匹配的例子如下:
>
> ```cangjie
> enum TimeUnit {
>     | Year(Float32)
>     | Month(Float32, Float32)
>     | Day(Float32, Float32, Float32)
>     | Hour(Float32, Float32, Float32, Float32)
> }
> let oneYear = TimeUnit.Year(1.0)
> var howManyHours: Float32 = match (oneYear) {
>     case Year(y) => y * Float32(365 * 24) // matched
>     case Month(y, m) => y * Float32(365 * 24) + m * Float32(30 * 24)
>     case Day(y, m, d) => y * Float32(365 * 24) + m * Float32(30 * 24) + d * Float32(24)
>     case Hour(y, m, d, h) => y * Float32(365 * 24) + m * Float32(30 * 24) + d * Float32(24) + h
> }
>
> let twoYear = TimeUnit.Year(2.0)
> var howManyYears: Float32 = match (twoYear) {
>     case Year(y) | Month(y, m) => y // error: variable cannot be introduced in patterns connected by '|'
>     case Year(y) | Month(x, _) => y // error: variable cannot be introduced in patterns connected by '|'
>     case Year(y) | Month(y, _) => y // error: variable cannot be introduced in patterns connected by '|'
>     ...
> }
> ```
>
> 如果模式匹配所在的作用域中, 某个 pattern 中的`identifier`是`enum`构造器时, 该`identifier`总是会被当成 enum pattern 进行匹配, 否则才会作为 binding pattern 匹配

如果`enum`成员是有参数的, 模式匹配就可以匹配成员以及携带的值

但是不能通过`|`尝试匹配同一个`enum`的不同成员

> 如果模式匹配所在的作用域中, 某个 pattern 中的`identifier`是`enum`构造器时, 该`identifier`总是会被当成 enum pattern 进行匹配, 否则才会作为 binding pattern 匹配
>
> ```cangjie
> enum Foo {
>     A | B | C
> }
>
> func f() {
>     let x = Foo.A
>     match (x) {
>         case A => 0 // enum pattern
>         case B => 1 // enum pattern
>         case C => 2 // enum pattern
>         case D => 3 // binding pattern
>     }
> }
> ```

> 需要注意的是, 对`enum`进行匹配时, 要求 enum pattern 的类型和 selector 的类型相同, 同时编译器会检查`enum`类型的每个`constructor`(包括`constructor`的参数的值)是否被完全覆盖, 如果未做到全覆盖, 则编译器会报错
>
> ```cangjie
> enum TimeUnit {
>     | Year(Float32)
>     | Month(Float32, Float32)
>     | Day(Float32, Float32, Float32)
>     | Hour(Float32, Float32, Float32, Float32)
> }
>
> let oneYear = TimeUnit.Year(1.0)
> var howManyHours: Float32 = match (oneYear) { // error: 匹配必须全覆盖
>     case Year(y) => y * Float32(365 * 24)
>     case Month(y, m) => y * Float32(365 * 24) + m * Float32(30 * 24)
> }
> ```

#### 模式的分类

> 一般地, 在类型匹配的前提下, 当一个 pattern 有可能和它所要匹配的值不匹配时, 称此 pattern 为 *`refutable pattern`*
>
> 反之, 当一个 pattern 总是可以和它所要匹配的值匹配时, 称此 pattern 为 *irrefutable pattern*
>
> 对于上述介绍的各类 pattern , 规定:
>
> - constant pattern 总是 *refutable pattern*;
> - wildcard pattern 总是 *irrefutable pattern*;
> - binding pattern 总是 *irrefutable pattern*;
> - tuple pattern 是 *irrefutable pattern*, 当且仅当其包含的每个 pattern 都是 *irrefutable pattern*;
> - type pattern 总是 *refutable pattern*;
> - enum pattern 是 *irrefutable pattern*, 当且仅当其对应的`enum`类型中只有一个带参`constructor`, 且 enum pattern 中包含的其他 pattern (如果存在)都是 *irrefutable pattern*

通配符和绑定模式, 总是 可以和其所要匹配的值匹配

常量模式和类型模式, 总是 有可能和其所要匹配的值不匹配

`Tuple`模式, 当匹配的 pattern 都是通配符或绑定模式时, 可以说 总是可以和其所要匹配的值匹配

```cangjie
match (value) {
    case (x, y) => println("${x}, ${y}")
    case (x, _) => println("${x}")
    case (_, _) => "Nothing"
}
```

`enum`模式, 如果`enum`本身只有一个带参的`constructor`, 同时 匹配的 pattern 都是通配符或绑定模式时, 可以说 总是可以和其所要匹配的值匹配

```cangjie
enum Single {
    Value(Int64)
}

let value = Single.Value(20)
match (value) {
    case Value(x) => println(value)
    case _ => "Nothing"
}
```

#### 字符串、字节和Rune的匹配规则

> 在模式匹配的目标是静态类型为`Rune`的值时, `Rune`字面量和单字符字符串字面量都可用于表示`Rune`类型字面量的常量 pattern
>
> 在模式匹配的目标是静态类型为`Byte`的值时, 一个表示 ASCII 字符的字符串字面量可用于表示`Byte`类型字面量的常量 pattern

`Rune`和`Byte`的匹配, 一个遵循`Rune`字面量, 一个是`ASCII`字符串

#### `Pattern Guards`

> 为了对匹配出来的值做进一步的判断, 仓颉支持使用 pattern guard
>
>  pattern guard 可以在`match`表达式中使用, 也可以在 for-in 表达式中使用
>
> 本节主要介绍 pattern guard 在`match`表达式中的使用, 关于其在`for in`表达式中的使用请参见[for-in 表达式]
>
> `match`表达式中, 为了提供更加强大和精确的匹配模式, 支持 pattern guard , 即在 pattern 与`=>`之间加上`where boolExpression`(`boolExpression`是值为布尔类型的表达式)
>
> 匹配的过程中, 只有当值与 pattern 匹配并且满足`where`之后的`boolExpression`时, 整个`case`才算匹配成功, 否则匹配失败
>
>  pattern guard 的语法定义为:
>
> ```
> patternGuard
>     : 'where' expression
>     ;
> ```
>
> 使用`pattern guards`的例子如下:
>
> ```cangjie
> let oneYear = Year(1.0)
> var howManyHours: Float32 = match (oneYear) {
>     case Year(y) where y > 0.0f32 => y * Float32(365 * 24) // matched
>     case Year(y) where y <= 0.0f32 => 0.0
>     case Month(y, m) where y > 0.0f32 && m > 0.0f32 => y * Float32(365 * 24) + m * Float32(30 * 24)
>     case Day(y, m, d) where y > 0.0f32 && m > 0.0f32 && d > 0.0f32 => y * Float32(365 * 24) + m * Float32(30 * 24) + d * Float32(24)
>     case Hour(y, m, d, h) where y > 0.0f32 && m > 0.0f32 && d > 0.0f32 && h > 0.0f32 => y * Float32(365 * 24) + m * Float32(30 * 24) + d * Float32(24) + h
>     case _ => 0.0
> }
> ```

 pattern 守卫, 就是在 pattern 匹配成功之后, 又多了一层判断

只有这一个判断为`true`, 才算最终匹配成功
