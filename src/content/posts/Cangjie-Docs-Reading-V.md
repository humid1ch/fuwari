---
title: "仓颉文档阅读-语言规约III: 名字、作用域、变量、修饰符"
published: 2025-09-24
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

## 名字、作用域、变量、修饰符

> 本章首先介绍名字、作用域、遮盖, 然后介绍其中一种名字——变量, 包括变量的定义和初始化, 最后是修饰符

这里有一个名词: 遮盖, 猜测 可能是C/C++中嵌套作用域出现同名变量的情况

### 名字

> 仓颉编程语言中, 我们用**名字**(names)标识变量、函数、类型、package、module 等实体(entities)
>
> **名字**必须是一个合法的[标识符](https://www.humid1ch.cn/blog/cangjie-docs-reading-i#heading-1)
>
> 仓颉编程语言的关键字、变量、函数、类型(包括:`class`、`interface`、`struct`、`enum`、`type alias`)、泛型参数、`package`名、`module`名共用同一个**命名空间**, 即, 在同一个`scope`声明或定义的实体, 不允许同名(除了构成重载的名字)
>
> 不同`scope`声明或定义的实体允许同名, 但可能产生`shadow`
>
> ```cangjie
> let f2 = 77
>
> func f2() {                // Error,  function name is the same as the variable f2
>     print("${f2}")
> }
>
> // Variable, function, type names cannot be the same as keywords
> let Int64 = 77            // Error: 'Int64' is a keyword
>
> func class() {            // Error: class is a keyword
>     print("${f2}")
> }
>
> main(): Int64 {
>     print("${f2}")        // Print 77
>
>     let f2 = { =>        // Shadowed the global variable f2
>        print("10")
>     }
>     f2()
>     return 1
> }
> ```

需要注意的就是, 同作用域不能出现同名名字

绝大多数语言应该都是如此吧

### 作用域

> 对一个实体, 在其名字所在的**作用域**(`scope`)中**可以直接使用名字访问**, 不需要通过前缀限定词访问
>
> 作用域是可嵌套的, 一个作用域包含自身以及自身包含的嵌套的作用域
>
> 名字在其嵌套作用域中如果**没有被遮盖或覆盖**, 也可以直接使用名字访问

#### 块

> 在仓颉语言中, 由一对匹配的大括号及其中可选的表达式声明序列组成的结构被称之为**块**(block)
>
> 块在仓颉语言中无处不在, 例如函数定义的函数体、`if`表达式的两个分支、`while`表达式的循环体, 都是块
>
> **块会引出新的作用域**
>
> 块的语法定义为:
>
> ```
> block
>     : '{' expressionOrDeclarations '}'
>     ;
> ```
>
> 块拥有值, 块的值由其中的表达式与声明序列确定
>
> **对块进行求值时, 会按表达式和变量声明在块中的顺序进行**
>
> **若块的最后一项是表达式, 当该表达式求值完毕后, 该表达式的值即为该块的值**:
>
> ```cangjie
> {
>     let a = 1
>     let b = 2
>     a + b
> } // a + b的值就是此块的值
> ```
>
> 若块的最后一项是声明, 当该声明处理完毕后, 该块的值为`()`:
>
> ```cangjie
> {
>     let a = 1
> } // () 是此块的值
> ```
>
> 若块中不含有任何表达式或声明, 该块的值为`()`:
>
> ```cangjie
> { } // () 是此块的值
> ```

**仓颉中块也是存在值的**, 且**与块内的最后一个执行的表达式有关**

类型应该也是吧? 值为`()`的块的类型是`Unit`?

#### 作用域级别

> 如果嵌套的多层作用域中存在相同的名字, **内层作用域引入的名字会遮盖外层作用域的名字**, 我们称**内层作用域级别比外层作用域级别更高**
>
> 在嵌套的作用域中, 作用域级别比较定义如下:
>
> 1. 通过`import`引入的名字, 作用域级别最低
> 2. 包内`top-level`的名字, 其作用域级别比 1 中的名字高
> 3. 类型内部、函数定义或表达式中引入的名字, 其定义通常被包围在一对花括号`{}`(即块)中, 其作用域级别相较于花括号`{}`外层的名字要高
> 4. 对于类和接口, 子类中的名字比父类中的名字的作用域级别更高, 子类中的名字可能会`shadow`或`override`父类中的名字
>
> ```cangjie
> import p1.a   // a 有最低的作用域级别
>
> var x = 1   // x 的作用域级别比a高
>
> open class A {    // A 的作用域级别与 第3行的x相同
>     var x = 3     // 这个x 的作用域级别 比第3行的x高
>     var y = 'a'
>     func f(a: Int32, b: Float64): Unit {
>
>     }
> }
>
> class B <: A {
>     var x = 5   // 这个x 的作用域级别 比第6行的x高
>
>     func f(x!: Int32 = 7) { // 这个x 的作用域级别 比第14行的x高
>
>     }
> }
> ```

#### 作用域类型

> 根据名字引入的位置分为三种:`top-level`, `local-level`, `类型内部`, 这三种类型的`scope`原则各有不同

##### `Top-level`

> `Top-level`引入的名字遵守如下作用域原则:
>
> - `Top-level`函数、类型, 其作用域为整个`package`, 名字对整个`package`可见, 其中类型包括:`class`、`interface`、`enum`、`struct`、`type`引入的名字
>
> - `Top-level`变量, 即由`let`, `var`和`const`引入的名字, 其作用域为从定义(包括赋初值)完成之后开始, 不包括从本文件开头到变量声明之间的区间, 名字对`package`的其它文件可见
>
>     但由于变量的初始化过程可能有副作用, **必须先声明且初始化后再使用**
>
> ```cangjie
> /* 全局变量只能在定义之后再使用 */
> let x = y         //Error: y
> let y = 4
> let a = b         //Error: b is used before being defined
> let b = a
> let c = c         //Error: unresolved identifier 'c' (in the right of '=')
> ```
>
> ```cangjie
> //Function names are visible in the entire package
> func test() { test2() }   // OK
>
> func f() { test() } // OK
>
> func test2(): Int64 {
>     var x = 99
>     return x
> }
> ```

`Top-level`函数和类型部分, 与C/C++有很大的不同

C/C++无论是函数、类型和变量, 都需要"先声明"再使用, 这里的"先声明", 指 在使用之前, 使用所在行之上必须能看到对应的声明

但, 仓颉的意思好像是, 只有变量需要先声明

而函数和类型的定义, 只要定义了就在整个`package`中可见, 不需要关注顺序

##### `Local-level`

> 在函数定义或表达式内部声明或定义的名字具有`local-level`作用域
>
> **定义在块内的变量 比 块外的有更高的作用域级别**
>
> - 局部变量, 其作用域 从声明之后开始到`scope`结束, 必须先定义和初始化后使用
>
>     局部变量的遮盖从引入变量名的声明或定义之后开始
>
> - 局部函数, 其作用域从声明的位置之后到`scope`结束, 支持递归定义, 不支持互递归定义
>
> - 函数的参数和泛型参数 的作用域从参数名声明后开始到函数体结束, 其作用域级别与函数体内定义的变量等同
>
>     - 函数定义`func f(x : Int32, y! : Int32 = x) { }`是合法的
>     - 函数定义`func f(x! : Int32 = x) { }`是不合法的
>
> - 泛型类型声明或者扩展泛型类型时 引入的泛型参数 从参数名声明后开始到类型体或扩展体结束, 其作用域级别与类型内定义的名字等同
>
>     - 泛型类型定义`class C<T> {}`中`T`的作用域从`T`出现到整个`class C`的声明结束
>     - 泛型类型的扩展`extend C<T> {}`中`T`的作用域从`T`出现到整个扩展定义结束
>
> - `lambda`表达式的参数名的作用域与函数的相同, 为`lambda`表达式的函数体部分, 其作用域级别可视为与`lambda`表达式的函数体内定义的变量等同
>
> - `main`、构造函数的形参名字被视为由函数体块引入, 在函数体块中再次引入与形参同名的名字会触发重定义错
>
> - `if-let`表达式条件中引入的名字被视为由`if`块引入, 在`if`块中再次引入相同名字会触发重定义错
>
> - `match`表达式的`match case`中 **`pattern`引入的名字, 其作用域级别比所在的`match`表达式更高**, 从引入处开始到该`match case`结束
>
>     每个`match case`均有独立的作用域
>
>     `match case`绑定模式中引入的名字被视作 由胖箭头`=>`之后的作用域引入, 在`=>`之后再次引入相同名字会触发重定义错
>
> - 对于所有三种循环, **循环条件与循环块的作用域级别相同**, 即其中引入的名字互相不能遮盖
>
>     且额外规定 **循环条件无法引用循环体中定义的变量**
>
>     则有如下推论:
>
>     - 对于`for-in`表达式, 其循环体可以引用循环条件中引入的变量名
>     - 对于`while`和`do-while`表达式, 它们的循环条件都无法引用其循环体中引入的变量名, 即便`do-while`的循环条件在循环体后
>
> - 对于`for-in`循环, 额外规定 **其循环条件处引入的变量不能在`in`关键字之后的表达式中使用**
>
> - 对于`try`异常处理, `try`后面紧跟的块以及每个`catch`块的的作用域互相独立
>
>     `catch pattern`引入的名字被视作由`catch`后紧跟的块引入, 在`catch`块中再次引入相同名字会触发重定义错
>
> - `try-with-resources`表达式, `try`关键字和`{}`之间引入的名字, 被视作由`try`块引入, 在`try`块中再次引入相同名字会触发重定义错
>
> ```cangjie
> // 1. 局部变量的遮盖从引入变量名的声明或定义之后开始
>
> let x = 4
> func f(): Unit {
>     print("${x}") // Print 4
>
>     let x = 99
>     print("${x}") // Print 99
> }
>
> let y = 5
> func g(): Unit {
>     let y = y     // 这两个'y', '='右边的是全局变量'y'
>     print("${y}")
>
>     let z = z     // Error: '='右边的'z' 未定义
> }
> ```
>
> ```cangjie
> // 2. 局部函数的作用域从定义之后开始
> func test1(): Unit {
>     func test2(): Unit {
>         print("test2")
>         test3()  // Error, test3() 的范围在定义之后开始
>     }
>     func test3(): Unit {
>         test2()
>     }
>
>     test2()
> }
> ```
>
> ```cangjie
> let score: Int64 = 90
> let good = 70
> var scoreResult: String = match (score) { // binding pattern
>     case 60 => "pass"     // constant pattern.
>     case 100 => "Full"     // constant pattern.
>     case good => "good" // 这里的good作用域级别高于第 2 行的good, 触发遮盖
> }
> ```

大多都与C/C++的规则类似, 不需要死记硬背, 关于作用域用多了自然有自己的判断

不过, 仓颉可以定义**局部函数**, 局部函数需要先定义再使用, 而**不是在整个局部作用域可见**

其实只需要额外记住这几点:

1. 同级作用域禁止声明或定义重复的名字

2. 不同级作用域允许声明或定义重复的名字, 但触发遮盖现象

    即, 高级遮盖低级

3. `for-in`, 在条件中引入的变量, 无法在`in`之后的表达式中使用

4. 一些拥有代码块`{}`的表达式, 多数情况下, 表达式中引入的变量与块中同级

##### 类型内部引入

> - `class/interface/struct/enum`的成员的`scope`为整个`class/interface/struct/enum`定义内部
> - `enum`定义中的`constructor`的作用域为整个`enum`定义内部, 关于`enum`中`constructor`名字访问的具体规则, 见[enum 类型]

#### 遮盖

> 通常来说, 若两个相互重叠的拥有不同级别的作用域中引入了相同的名字, 则会发生遮盖(shadowing):
>
> 其中作用域级别高的名字会遮盖作用域级别低的名字
>
> 这导致作用域级别低的名字要么需要加上前缀限定词访问, 要么无法访问
>
> 当作用域级别高的名字的作用域结束时, 遮盖消除
>
> 具体说来, 作用域级别不同时:
>
> 1. 若作用域级别高的名字`C`为一个类型, 则直接发生遮盖
>
>     ```cangjie
>     // == 在 package a 中 ==
>     public class C {} // ver 1
>
>     // == 在 package b 中 ==
>     import a.*
>
>     class C {} // ver 2
>
>     let v = C() // will use ver 2
>     ```
>
> 2. 若作用域级别高的名字`x`为一个变量, 则直接发生遮盖(对于成员变量的遮盖规则, 请参见 "类和接口" 章节)
>
>     ```cangjie
>     let x = 1
>
>     func foo() {
>         let x = 2
>         println(x) // will print 2
>     }
>     ```
>
> 3. 若作用域级别高的名字`p`为`package`名字, 则直接发生遮盖
>
>     ```cangjie
>     // == 在 package a 中 ==
>     public class b {
>         public static let c = 1
>     }
>
>     // == 在 package a.b 中 ==
>     public let c = 2
>
>     // == 在 package test 中 ==
>     import a.*
>     import a.b.*
>
>     let v = a.b.c // will be 1
>     ```
>
> 4. 若作用域级别高的名字`f`为一个成员函数, 则按重载的规则判断`f`是否发生重载, 如无重载, 则可能发生覆盖或重定义;
>
>     如无重载且不能覆盖/重定义, 则报错(具体规则, 见[覆盖]、[重定义])
>
>     ```cangjie
>     open class A {
>         public open func foo() {
>             println(1)
>         }
>     }
>
>     class B <: A {
>         public override func foo() {  // override
>             println(2)
>         }
>
>         public func foo() {  // error, conflicting definitions
>             println(3)
>         }
>     }
>     ```
>
> 5. 若作用域级别高的名字`f`为非成员函数, 则按重载的规则判断`f`是否发生重载
>
>     如无重载, 则视为遮盖
>
>     ```cangjie
>     func foo() { 1 }
>
>     func test() {
>         func foo() { 2 } // shadows
>         func foo(x: Int64) { 3 } // overloads
>
>         println(foo()) // will print 2
>     }
>     ```
>
> 下面的例子展示了函数内变量的作用域级别和之间的遮盖关系, 以及项与类型在同一命名空间
>
> ```cangjie
> func g(): Unit {
>       f(1, 2)  // OK. f 是顶层定义, 在整个文件中可见
> }
>
> func f(x: Int32, y: Int32): Unit {
>     // Error. 变量名 Maze 遮盖了类型 Maze, 因此 Maze 不能用作类型
>     // let Maze : Maze
>     // var x = 1    // Error, x 已经在参数中引入了
>     var i = 1       // OK
>
>     for (i in 1..3) { // OK, i 是引入的一个新变量, 遮盖第9行定义的i
>         let v = 1
>     }              // i 和 v 消失
>
>     print("${i}")     // OK. 这个 i 是第9行定义的
>     // print("${v}")  // Error, v 已经消失了, 找不到 v
> }
>
> enum Maze {
>     C1 | C2
>     // | C1 // Error. C1 这个名字, 在同级作用域中已经用过了
> }
> ```
>
> 下面的例子展示了不同包之间定义的变量和类的作用域级别关系和之间遮盖关系
>
> ```cangjie
> // File a.cj
> package p1
>
> class A {}
> var v: Int32 = 10
>
> // File b.cj
> package p2
>
> import p1.A        // A 属于包 p1
> import p1.v        // v 属于包 p1
>
> // p2 定义了自己的 v, 其作用域级别高于包 p1 中的 v
> var v: Int32 = 20
>
> func displayV1(): Unit {
>     // 根据作用域级别, p2.v 遮盖了 p1.v, 因此在此处访问 p2.v
>     print("${v}")          // output: 20
> }
>
> var a = A()         // 调用 p1 中的A
> ```
>
> 下面的例子展示了继承中的遮盖关系
>
> ```cangjie
> var x = 1
>
> open class A {
>       var x = 3   // 这个 x 遮盖了顶层作用域的 x
> }
>
> class B <: A {
>     var x = 5   // error: 子类的成员不能遮盖父类的成员
>
>     func f(x!: Int32 = 7): Unit { // 这个 x 会遮盖上面的所有 x
>
>     }
> }
> ```

**成员函数不存在遮盖**, 只有覆盖、重载、重定义, 重定义就会报错

也只有存在继承关系的类型中的成员函数会出现这些情况

### 变量

> 仓颉编程语言作为一种静态类型(`statically typed`)语言, 要求每个变量的类型必须在编译时确定
>
> 根据是否可进行修改, 可将变量分为 3 类: 不可变变量(一旦初始化, 值不可改变)、可变变量(值可以改变)、`const`变量(必须编译期初始化, 不可修改)

仓颉将变量类型分为了3种, 是根据可变状态和初始化状态区分的

`const`修饰的变量, 定义时必须初始化

#### 变量的定义

> 变量定义的语法定义为:
>
> ```
> variableDeclaration
>     : variableModifier* ('let' | 'var' | 'const') patternsMaybeIrrefutable (((':' type)? ('=' expression)) | (':' type))
>     ;
>
> patternsMaybeIrrefutable
>     : wildcardPattern
>     | varBindingPattern
>     | tuplePattern
>     | enumPattern
>     ;
> ```
>
> 变量的定义均包括四个部分: 修饰符、`let/var/const`关键字、`patternsMaybeIrrefutable`和变量类型
>
> 其中:
>
> 1. 修饰符
>
>     - `top-level`变量的修饰符包括:`public`, `protected`, `private`, `internal`
>     - **局部变量不能用修饰符修饰**
>     - `class`类型的成员变量的修饰符包括:`public`, `protected`, `private`, `internal`, `static`
>     - `struct`类型的成员变量的修饰符包括:`public`, `private`, `internal`, `static`
>
> 2. 关键字`let/var/const`
>
>     - `let`用于定义不可变变量, `let`变量一旦初始化就不能再改变
>     - `var`用于定义可变变量
>     - `const`用于定义`const`变量
>
> 3. `patternsMaybeIrrefutable`
>
>     - `let`(或`var/const`)之后只能是那些一定或可能为 *`irrefutable`* 的`pattern`(见 [模式的分类])
>
>         在语义检查阶段, 会检查`pattern`是否真的是 *`irrefutable`*, 如果不是 *`irrefutable`*`pattern`, 则编译报错
>
>     - `let`(或`var/const`)之后的`pattern`中引入的新的变量, 全部都是`let`修饰(或`var`修饰)的变量
>
>     - 在`class`和`struct`中定义成员变量时, 只能使用`binding pattern`(见 [绑定模式])
>
> 4. 变量类型是可选的, 不声明变量类型时需要给变量初始值, 编译器将尝试根据初始值推断变量类型
>
> 5. 变量可以定义在`top-level`, 表达式内部, `class/struct`类型内部

顶层作用域种定义变量, 仓颉比C/C++多了修饰符, 应该是包管理的原因, C/C++包管理不完善(悲)

根据已经了解的一些内容

`patternsMaybeIrrefutable`应该是变量名的模式, `irrefutable`是不可辩驳的意思, 意思应该是没有歧义

`wildcardPattern`通配符模式, 意思是, 定义时变量名可以是通配符? 应该是解构某些数据时, 忽略数据用的:`let _ = "ignore data"`

`varBindingPattern`变量绑定模式, 应该就是普通的变量:`let val: Int64 = 30`

`tuplePattern`元组模式, 解构元组时用的:`let (_, val) = ("ignore data", 30)`

`enumPattern`是枚举模式, 暂时想不出什么作用?

`class`和`interface`的成员变量, 只能是绑定模式, 只有`varBindingPattern`吗?

<Question>

记录这两个问题, 具体了解到之后回答

</Question>

> 需要注意的是:
>
> 1. `pattern`和变量类型之间需要使用冒号(:)分隔, `pattern`的类型需要和冒号后的类型相匹配
>
> 2. 关键字`let/var/const`和`pattern`是必选的
>
> 3. 局部变量除了使用上述语法定义之外, 还有如下几种情形会引入局部变量:
>
>     - `for-in`循环表达式中, `for`和`in`中间的`pattern`, 详见 [for-in 表达式]
>     - 函数、`lambda`定义中的形参, 详见 [参数]
>     - `try-with-resource`表达式中`ResourceSpecifications`, 详见 [异常]
>     - `match`表达式中, `case`后的`pattern`, 详见 [模式匹配表达式]
>
> 4. 可以使用一对反引号(<code>\` `</code>)将关键字变为合法的标识符
>
>     (例如, <code>\`open`</code>等)
>
> 下面给出变量定义的一些实例:
>
> ```cangjie
> let b: Int32                    // 定义 Int32 类型 只读变量b
> let c: Int64                    // 定义 Int64 类型 只读变量b
> var bb: String                // 定义 String 类型 可写变量bb
> var (x, y): (Int8, Int16)        // 定义 Int8和Int16 类型 两个可写变量x, y
> var`open`= 1                // 定义 变量名为`open`的值为1的变量
> var`throw`= "throw"            // 定义 变量名为`throw`的值为"throw"的变量
> const d: Int64 = 0            // 定义 Int64 类型 const变量bb, 初始化值为0
> ```

#### 变量的初始化

##### 不可变变量和可变变量

> 不可变变量和可变变量的初始化均有两种方式: 定义时初始化和先定义后初始化
>
> 需要注意的是, **每个变量在使用前必须进行初始化**, 否则会报编译错误
>
> 关于变量的初始化, 举例如下:
>
> ```cangjie
> func f() {
>     let a = 1               // 定义并初始化不可变变量 a
>     let b: Int32            // 定义不可变变量 b, 但不初始化
>     b = 10                  // 初始化变量 b
>     var aa: Float32 = 3.14  // 定义并初始化可变变量 aa
> }
> ```
>
> 使用`let`定义的不可变变量, 只能被赋值一次(即初始化), 如果被多次赋值, 会报编译错误
>
> 使用`var`定义的可变变量, 支持多次赋值
>
> ```cangjie
> func f() {
>     let a = 1             // 定义并初始化不可变变量 a
>     a = 2                 // error: 不可变变量 a 不能被赋值
>     var b: Float32 = 3.14 // 定义并初始化可变变量 b
>     b = 3.1415            // ok: 可变变量 b 可以被赋值
> }
>
> class C {
>     let m1: Int64
>     init(a: Int64, b: Int64) {
>         m1 = a
>         if (b > 0) {
>             m1 = a * b  // OK: 不可变变量可以在 构造函数中 赋值
>         }
>     }
> }
> ```

仓颉中的变量必须要经过初始化之后, 才能正常使用

不可变变量, 可以在构造函数内多次赋值

##### 全局变量及静态变量的初始化

> 全局变量指定义在`top-level`的变量, 静态变量包含定义在`class`或`struct`中的静态变量
>
> 全局变量和静态变量的初始化必须满足以下规则:
>
> - **全局变量在声明时必须立即对其进行初始化**, 否则报错
>
>     即, 声明必须提供一个初始化表达式
>
> - **静态变量在声明时必须立即对其进行初始化**, 可以采用与全局变量的初始化相同的形式, 也**可以在静态初始化器中进行初始化**(更多细节见[静态初始化器]部分)
>
>     - 注意, 静态变量不能在其他静态变量中初始化:
>
>         ```cangjie
>         class Foo {
>             static let x: Int64
>             static let y = (x = 1) // it's forbidden
>         }
>         ```
>
> - 初始化表达式`e`不能依赖未初始化的全局变量或静态变量
>
>     编译器会进行保守的分析, 如果`e`可能会访问到未初始化的全局变量或静态变量, 则报错
>
>     详细的分析取决于编译器的实现, 在规范中没有指定

C/C++中可没有关于静态初始化器的内容

> 全局/静态变量的初始化时机和初始化顺序规则如下:
>
> - **所有的全局/静态变量都在`main`(程序入口) 之前完成初始化**
>
> - 对于同一个文件中声明的全局/静态变量, 初始化顺序根据变量的声明顺序从上到下进行
>
>     如果使用了静态初始化器, 则根据静态初始化器的初始化顺序规则执行(详见[静态初始化器]部分)
>
> - 同一个包里不同文件或不同包中声明的全局/静态变量的初始化顺序取决于文件或包之间的依赖关系
>
>     如果文件`B.cj`依赖文件`A.cj`且`A.cj`不依赖`B.cj`, 则`A.cj`中的全局/静态变量的初始化在`B.cj`中的全局/静态变量的初始化之前
>
> - 如果文件或包之间存在循环依赖或者不存在任何依赖, 那么它们之间的初始化顺序不确定, 由编译器实现决定
>
>     ```cangjie
>     /* 全局变量的初始化不能依赖于 全局在同一包的其他文件中定义的变量 */
>     // a.cj
>     let x = 2
>     let y = z       // OK, b.cj 不直接或间接依赖于此文件
>     let a = x       // OK.
>
>     let c = A()
>     let d = c.f()
>     /* c.f 是一个开放函数, 编译器无法静态确定该函数是否符合全局变量的初始化规则, 可能会报告错误 */
>
>     open class A {
>         // static var x = A.z     // Error, A.z 在初始化之前被使用
>         // static var y = B.f     // Error, B.f 在初始化之前被使用
>         static var z = 1
>         public open func f(): Int64 {
>             return 77
>         }
>     }
>
>     class B {
>         static var e = A.z    // OK.
>         static var f = x      // OK.
>     }
>
>     // b.cj
>     let z = 10
>     // let y = 10   // Error, y 已经在 a.cj 中定义了
>
>     // main.cj
>     main(): Int64 {
>         print("${x}")
>         print("${y}")
>         print("${z}")
>
>         return 1
>     }
>     ```

##### `const`变量

> 详见`const`章节

### 修饰符

> 仓颉提供了很多修饰符, 主要分以下两类:
>
> - 访问修饰符
> - 非访问修饰符
>
> 修饰符通常放在定义处的最前端, 用来表示该定义具备某些特性

#### 访问修饰符

> 详细内容请参考包和模块管理章节[访问修饰符]

#### 非访问修饰符

> 仓颉提供了许多非访问修饰符以支持其它丰富的功能
>
> - **`open`** 表示该实例成员可被子类覆盖, 或者该类能被子类继承, 详见[类]
> - **`sealed`** 表示该`class`或`interface`只能在当前包内被继承或实现, 详见[类]
> - **`override`** 表示覆盖父类的成员, 详见[类]
> - **`redef`** 表示重新定义父类的静态成员, 详见[类]
> - **`static`** 表示该成员是静态成员, 静态成员不能通过实例对象访问, 详见[类]
> - **`abstract`** 表示该`class`是抽象类, 详见[类]
> - **`foreign`** 表示该成员是外部成员, 详见[语言互操作]
> - **`unsafe`** 表示与 C 语言进行互操作的上下文, 详见[语言互操作]
> - **`sealed`** 表示该`class`或`interface`只能在当前包内被继承或实现, 详见[类]
> - **`mut`** 表示该成员是具有可变语义的, 详见[函数]
>
> 这些修饰符的具体功能详见对应的章节

看来修饰符的使用与具体的语法结合得比较深

那就具体阅读到再具体了解
