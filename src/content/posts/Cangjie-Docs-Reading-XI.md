---
title: "仓颉文档阅读-语言规约V: 函数(II)"
published: 2025-09-30 15:13:16
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

## 函数

> 函数是一段完成特定任务的独立代码片段, 可以通过函数名字来标识, 这个名字可以被用来调用函数
>
> 仓颉编程语言中函数是**一等公民**, 函数可以赋值给变量, 或作为参数传递, 或作为返回值返回

### `Lambda`表达式

> 一个 `Lambda` 表达式是一个匿名的函数
>
> `Lambda` 表达式的语法如下:
> ```
> lambdaExpression
>         : '{' NL* lambdaParameters? '=>' NL* expressionOrDeclarations '}'
>         ;
>
> lambdaParameters
>     : lambdaParameter (',' lambdaParameter)* ','?
>     ;
>
> lambdaParameter
>     : (identifier | '_') (':' type)?
>     ;
> ```
>
> `lambda` 表达式有两种形式, 一种是有形参的 `{a: Int64 => e1; e2 }`
>
> 另一种是无形参的 `{ => e1; e2 }` (e1 和 e2 是表达式或声明序列)
>
> ```cangjie
> let f1: (Int64, Int64)->Int64 = {a: Int64, b: Int64 => a + b}
>
> var f2: () -> Int32 = { => 123 }
> ```

现代语言中基本都存在`lambda`表达式

仓颉中的`lambda`表达式的书写并不复杂:

```cangjie
{ 形参列表 => 表达式序列 }
```

仓颉的`lambda`表达式, 语法规定 形参列表不用`()`包裹, 执行的语句序列也不用`{}`包裹

> 对于有形参的 `lambda` 表达式, 可以使用一个 `_` 代替一个形参, `_` 代表在 `lambda` 的函数体中不会使用 到的参数
>
> ```cangjie
> let f2 = { n: Int64, _: Int64 => return n * n }
> let f3: (Int32, Int32) -> Int32 = { n, _ => return n * n }
> let f4: (String) -> String = { _ => return "Hello" }
> ```


> `Lambda` 表达式中不支持声明返回类型
>
> 若上下文明确指定了 `lambda` 表达式的返回类型, 则其返回类型为上下文指定的类型
>
> 如果指定的返回类型是 `Unit`, 则仓颉编译器还会在 `lambda` 表达式的函数体中所有可能返回的地方插入 `return ()`, 使其总是返回 `Unit` 类型
>
> 指定了 `Unit` 返回类型的示例如下:
>
> ```cangjie
> func column(c: (Data) -> Unit) {...}
> func row(r: (Data) -> Unit) {...}
>
> func build(): Unit {
>     column { _ =>
>         row { _ =>
>             buildDetail()
>             buildCalendar()
>         } // OK. 由于插入了'return', 所以类型正确
>         width(750)
>         height(700)
>         backgroundColor("#ff41444b")
>     }  // OK. 由于插入了'return', 所以类型正确
> }
> ```
>
> 若不存在这样的上下文, 则 `=>` 右侧的表达式的类型会被视为 `lambda` 表达式的返回类型
>
> 同函数一样, 若无法推导出返回类型, 则会编译报错

仓颉中的`lambda`表达式不支持声明返回值类型

如果`lambda`表达式的上下文中指定了返回值类型, 那么编译器也会在`lambda`表达式的可能返回的位置插入`return`表达式, 这一点和函数是一样的

如果不存在相关的上下文, 那么就将`=>`右侧的表达式类型当作`lambda`表达式的返回类型, 这里的右侧 是指`lambda`表达式会执行的完整的表达式序列的类型

> `Lambda` 表达式中参数的类型标注可缺省, 编译器会尝试从上下文进行类型推断, 当编译器无法推断出类型时会编译报错
>
> ```cangjie
> var sum1: (Int32, Int32) -> Int32 = {a, b => a + b}
>
> var sum2: (Int32, Int32) -> Int32 = {a: Int32, b => a + b}
>
> var display = { => print("Hello") }
>
> var a = { => return 1 }
> ```
>
> `=>` 右侧的内容与普通函数体的规则一样, 同样可以省略 `return`
>
> 若 `=>` 的右侧为空, 返回值为 `()`
>
> ```cangjie
> sum1 = {a, b => a + b}
>
> sum2 = {a, b => return a + b}		// 与前面的一行一致
> ```
>
> `Lambda` 表达式支持原地调用, 例如:
> ```cangjie
> let r1 = {a: Int64, b: Int64 => a + b}(1, 2)  // r1 = 3
> let r2 = { => 123 }()                         // r2 = 123
> ```

仓颉中`lambda`表达式的原地调用, 类似将整个`lambda`表达式当作函数名, 直接加上`(实参列表)`就可以调用

### 闭包

> 闭包指的是自包含的函数或 `lambda`
>
> 闭包可以从定义它的静态作用域中**捕获**变量, 即使对闭包调用不在定义的作用域, 仍可访问其捕获的变量
>
> 变量捕获发生在闭包定义时

自包含, 表示自己内部包含有 部分或全部 执行所需的数据, 不需要完全依赖调用时传入的参数执行

当一个函数或`lambda`被定义时, 如果你在函数体或`lambda`执行体中引用了 在定义位置所在静态作用域中 能够访问到的变量, 这些变量会被自动捕获到闭包中

被捕获的变量, 会被包含到函数或`lambda`中, 此时形成闭包

其实可以类比C++中的`lambda`表达式的手动捕获可见变量功能, 不过C++需要手动捕获

> 不是所有闭包内的变量访问都称为捕获, 以下情形的变量访问不是捕获:
>
> - 对定义在本函数或本 `lambda` 内的局部变量的访问;
>
> - 对本函数或本 `lambda` 的形参的访问;
>
> - 全局变量和静态成员变量在函数或 `lambda` 中的访问;
>
> - 实例成员变量在实例成员函数中的访问
>
>     由于实例成员函数将`this`作为参数传入, 在实例成员函数内通过`this`访问所有实例成员变量
>
> 关于变量的捕获, 可以分为以下两种情形:
>
> - 捕获`let`声明的变量:
>
>     在闭包中不能修改这些变量的值
>
>     如果捕获的变量是引用类型, 可修改其可变实例成员变量的值
>
>     仅捕获了`let`声明的**局部变量**的函数或 `lambda` 可以作为一等公民使用, 即可以赋值给变量, 可以作为实参或返回值使用, 可以作为表达式使用
>
> - 捕获`var`声明的变量:
>
>     捕获了可变变量的函数或 `lambda` 只能被调用, 不能作为一等公民使用, 包括不能赋值给变量, 不能作为实参或返回值使用, 不能作为表达式使用
>
> > 需要注意的是, 捕获具有传递性, 如果一个函数`f`调用了捕获`var`变量的函数`g`, 且存在`g`捕获的`var`变量不在函数`f`内定义, 那么函数`f`同样捕获了`var`变量, 此时, `f`也不能作为一等公民使用
>
> 以下示例中, `h`仅捕获了`let`声明的变量`y`, `h`可以作为一等公民使用
>
> ```cangjie
> func f(){
>     let y = 2
>     func h() {
>         print(y)  // OK, 捕获了不可变变量
>     }
>     let d = h     // OK, h 可以被赋值到变量
>
>     return h      // OK, h 可以被当做返回值
> }
> ```
>
> 以下示例中, `g`捕获了`var`声明的变量`x`, `g`不可以作为一等公民使用, **仅能被调用**
>
> ```cangjie
> func f() {
>     var x = 1
>
>     func g() {
>         print(x)  // OK, 捕获了一个可变变量
>     }
>     let b = g     // Error, g 不能被赋值到变量
>
>     g     // Error, g 不能用作表达式
>     g()   // OK, g 可以被调用
>
>     // Lambda 捕获了一个可变变量, 不能赋值给变量
>     let e = { => print("${x}") }  // Error
>
>     let i = { => x * x }()        // OK, Lambda 捕获了一个可变变量, 可以被调用
>
>     return g                      // Error, g 不能用作返回值
> }
> ```
>
> 以下示例中, `g`捕获了`var`声明的变量`x`, `f`调用了`g`, 且`g`捕获的`x`不在`f`内定义, `f`同样不能作为一等公民使用:
>
> ```cangjie
> func h(){
>     var x = 1
>
>     func g() { x }    // 捕获了一个可变变量
>
>     func f() {
>         g()           // 调用 g
>     }
>
>     return f          // error
> }
> ```
>
> 以下示例中, `g`捕获了`var`声明的变量`x`, `f`调用了`g`
>
> 但`g`捕获的`x`在`f`内定义, `f`没有捕获其它`var`声明的变量
>
> 因此, `f`仍作为一等公民使用:
>
> ```cangjie
> func h(){
>     func f() {
>         var x = 1
>         func g() { x }    // 捕获一个可变变量
>
>         g()
>     }
>
>     return f              // ok
> }
> ```
>
> 访问了`var`修饰的全局变量、静态成员变量、实例成员变量的函数或 `lambda` 仍可作为一等公民使用
>
> ```cangjie
> class C {
>     static var a: Int32 = 0
>     static func foo() {
>         a++                   // OK
>         return a
>     }
> }
>
> var globalV1 = 0
>
> func countGlobalV1() {
>     globalV1++
>     C.a = 99
>     let g = C.foo             // OK
> }
>
> func g(){
>     let f = countGlobalV1     // OK
>     f()
> }
> ```

对全局变量、静态变量的访问不被看作捕获

从文档介绍来看, 如果暂时先忽略捕获的传递性, 那么 只要函数或`lambda`捕获了`var`变量, 那么此闭包就只能调用, 而不能被当作一等公民使用

> 捕获的变量必须满足以下规则:
>
> - 被捕获的变量必须在闭包定义时可见, 否则编译报错
>
> - 变量被捕获时必须已经完成初始化, 否则编译报错
>
> - 如果函数外有变量, 同时函数内有同名的局部变量, 函数内的闭包因局部变量的作用域未开始而捕获了函数外的变量时, 为避免用户误用, 报 warning
>
> ```cangjie
> // 1. 捕获的变量必须在闭包之前定义
> let x = 4
> func f() {
>     print("${x}")     // Print 4.
>     let x = 99
>     func f1() {
>         print("${x}")
>     }
>     let f2 = { =>
>         print("${x}")
>     }
>     f1()              // Print 99
>     f2()              // Print 99
> }
>
> // 2. 变量在被捕获之前必须被初始化
> let x = 4
> func f() {
>     print("${x}")         // Print 4
>     let x: Int64
>     func f1() {
>         print("${x}")     // Error: x 还没有初始化
>     }
>     x = 99
>     f1()
> }
>
> // 3. 如果在代码块中存在局部变量, 则 闭包捕获外部作用域中同名变量 时将警告
> let x = 4
> func f() {
>     print("${x}")         // Print 4
>     func f1() {
>         print("${x}")     // warning
>     }
>     let f2 = { =>
>         print("${x}")  	// warning
>     }
>     let x = 99
>     f1()                  // print 4
>     f2()                  // print 4
> }
> ```

### 函数重载

#### 函数重载定义

> 在仓颉编程语言中, 一个作用域中可见的同一个函数名对应的函数定义 不构成重定义时 便构成重载
>
> 函数存在重载时, 在进行函数调用时 需要根据函数调用表达式中的实参的类型和上下文信息 明确是哪一个函数定义被使用
>
> 被重载的多个函数必须遵循以下规则:
>
> - 函数名必须相同, 且满足以下情况之一:
>
>     - 通过`func`关键字引入的函数名
>
>     - 在一个 `class` 或 `struct` 内定义的构造函数, 包括主构造函数和`init`构造函数
>
> - 参数类型必须不同, 即满足以下情况之一:
>
>     - 函数参数数量不同
>
>     - 函数参数数量相同, 但是对应位置的参数类型不同
>
> - 必须在同一个作用域中可见
>
> 注意, 参数类型不受"是否是命名形参"、"是否有默认值"影响; 且讨论参数类型时, 一个类型与它的 `alias` 被视为相同的类型
>
> 例如下面例子中的四个函数的参数类型是完全相同的:
>
> ```cangjie
> type Boolean = Bool
> func f(a : Bool) {}
> func f(a! : Bool) {}
> func f(a! : Bool = false) {}
> func f(a! : Boolean) {}
> ```

仓颉的函数重载基本与C++中的函数重载定义一致

1. 函数名相同

2. 参数类型或参数数量存在一定差异

> 示例一: 定义源文件顶层的 2 个函数的参数类型不同, f构成了重载
>
> ```cangjie
> // f 构成重载
> func f() {...}
> func f(a: Int32) {...}
> ```
>
> 示例二: 接口I、类C1、C2中的函数f1构成了重载
>
> ```cangjie
> interface I {
>     func f1() {...}
> }
>
> open class C1 {
>     func f1(a: Int32) {...}
> }
>
> class C2 <: C1 & I {
>     // f1 构成重载
>     func f1(a: Int32, b: String) {...}
> }
> ```
>
> 示例三: 类型内的构造函数之间构成重载
>
> ```cangjie
> class C {
>     var name: String = "abc"
>
>     // 构造函数构成重载
>     init(){
>         print(name)
>     }
>
>     init(name: String){
>         this.name = name
>     }
> }
> ```
>
> 示例四: 如下示例, 函数参数数量相同, 相同位置的类型相同, 仅参数类型中包含的类型变元的约束不同, 不构成重载
>
> ```cangjie
> interface I1{}
> interface I2{}
>
> func f<T>(a: T) where T <: I1 {}
> func f<T>(a: T) where T <: I2 {} // Error, 不构成重载
> ```

### `mut`函数

> `mut` 函数是一种特殊的实例成员函数
>
> 在 `struct` 类型的 `mut` 成员函数中, 可以通过 `this` 修改成员变量, 而在 `struct` 类型的非 `mut` 成员函数中, 不可以通过 `this` 修改成员变量

仓颉中, `strcut`的普通成员函数是不能通过`this`修改成员变量的

仓颉中, 可以用来`mut`修饰成员函数, 使成员函数可以通过`this`修改成员变量

#### 定义

> `mut` 函数使用 `mut` 关键字修饰, 只允许在 `interface`、`struct` 和 `struct` 扩展中定义, 并且只能作用于实例成员函数(不支持静态成员函数)
>
> ```cangjie
> struct A {
>     mut func f(): Unit {}             // ok
>     mut static func g(): Unit {}      // error
>     mut operator func +(rhs: A): A {  // ok
>         return A()
>     }
> }
>
> extend A {
>     mut func h(): Unit {}             // ok
> }
>
> class B {
>     mut func f(): Unit {}             // error
> }
>
> interface I {
>     mut func f(): Unit                // ok
> }
> ```

`mut`只能用到`interface`、`struct`和`struct`扩展的非静态成员函数中, 无法用在`class`中

> 在 `mut` 函数中可以对 `struct` 实例的字段进行赋值, 这些赋值会修改实例并立刻生效
>
> 与实例成员函数相同, `this` 不是必须的, 可以由编译器推断
>
> ```cangjie
> struct Foo {
>     var i = 0
>     mut func f() {
>         this.i += 1   // ok
>         i += 1        // ok
>     }
> }
>
> main() {
>     var a = Foo()
>     print(a.i)        // 0
>     a.f()
>     print(a.i)        // 2
>     a.f()
>     print(a.i)        // 4
>
>     return 0
> }
> ```
>
> `mut` 函数中的 `this` 具有额外的限制:
>
> - 不能被捕获(意味着当前实例的字段也不能被捕获)
>
> - 不能作为表达式
>
> ```cangjie
> struct Foo {
>     var i = 0
>     mut func f(): Foo {
>         let f1 = { => this }          // error
>         let f2 = { => this.i = 2 }    // error
>         let f3 = { => this.i }        // error
>         let f4 = { => i }             // error
>         return this                   // error
>     }
> }
> ```

`mut`函数的作用是 `mut`函数可以通过`this`修改成员变量

因为, 仓颉中, `struct`的普通成员函数是无法修改成员变量的

不过, 仓颉中的`mut`函数内的`this`不能当作表达式, 即不能被返回、赋值给其他变量等

至于原因, 应该是为了防止出现`this`暴露, 导致其他地方能够通过`this`访问成员变量?

毕竟`struct`是值类型的

#### 接口中的`mut`函数


> `struct` 类型在实现 `interface` 的函数时 **必须保持一样的 `mut` 修饰**
>
> `struct` 以外的类型实现 `interface` 的函数时禁止使用 `mut` 修饰
>
> ```cangjie
> interface I {
>     mut func f1(): Unit
>     func f2(): Unit
> }
>
> struct A <: I {
>     public mut func f1(): Unit {}     // ok
>     public func f2(): Unit {}         // ok
> }
> struct B <: I {
>     public func f1(): Unit {}         // error
>     public mut func f2(): Unit {}     // error
> }
> class C <: I {
>     public func f1(): Unit {}         // ok
>     public func f2(): Unit {}         // ok
> }
> ```
>
> 需要注意的是, 当 `struct` 的实例赋值给 `interface` 类型时 是拷贝语义, 因此 `interface` 的 `mut` 函数并不能修改原本 `struct` 实例的值
>
> ```cangjie
> interface I {
>     mut func f(): Unit
> }
>
> struct Foo <: I {
>     var v = 0
>     public mut func f(): Unit {
>         v += 1
>     }
> }
>
> main() {
>     var a = Foo()
>     var b: I = a
>     b.f()
>     print(a.v)    // 0
>
>     return 0
> }
> ```

继承自`interface`的`struct`类型, `struct`实例赋值给`interface`类型, 是拷贝语义

#### 访问规则

> 如果一个变量使用 `let` 声明, 并且类型可能是 `struct`(包含静态类型是 `struct` 类型, 或者类型变元可能是 `struct` 类型), 那么这个变量不能访问该类型使用 `mut` 修饰的函数
>
> 其它情况均允许访问
>
> ```cangjie
> interface I {
>     mut func f(): Unit
> }
> struct Foo <: I {
>     var i = 0
>     public mut func f(): Unit {
>         i += 1
>     }
> }
> class Bar <: I {
>     var i = 0
>     public func f(): Unit {
>         i += 1
>     }
> }
>
> main() {
>     let a = Foo()
>     a.f()             // error
>     var b = Foo()
>     b.f()             // ok
>     let c: I = Foo()
>     c.f()             // ok
>
>     return 0
> }
>
> func g1<T>(v: T): Unit where T <: I {
>     v.f()             // error
> }
>
> func g2<T>(v: T): Unit where T <: Bar & I {
>     v.f()             // ok
> }
> ```

仓颉中, `let`修饰的变量, 只要静态类型可能是`strcut`, 就禁止调用`mut`函数

> 如果一个变量的类型可能是 `struct`(包含静态类型是 `struct` 类型, 或者类型变元可能是 `struct` 类型), 那么这个变量不能 将该类型使用 `mut` 修饰的函数被作为 高阶函数使用, 只能调用这些 `mut` 函数
>
> ```cangjie
> interface I {
>     mut func f(): Unit
> }
> struct Foo <: I {
>     var i = 0
>     public mut func f(): Unit {
>         i += 1
>     }
> }
> class Bar <: I {
>     var i = 0
>     public func f(): Unit {
>         i += 1
>     }
> }
>
> main() {
>     var a = Foo()
>     var fn = a.f          // error
>     var b: I = Foo()
>     fn = b.f              // ok
>
>     return 0
> }
>
> func g1<T>(v: T): Unit where T <: I {
>     let fn = v.f          // error
> }
> func g2<T>(v: T): Unit where T <: Bar & I {
>     let fn = v.f          // ok
> }
> ```

仓颉中, `struct`类型实例的`mut`函数, 不能被当作高阶函数使用

但文档中给出的例子, 并不是`mut`函数被当作高阶函数使用, 而是尝试当作第一公民使用

> 非 `mut` 的实例成员函数(包括 `lambda` 表达式)不能访问 `this` 的 `mut` 函数, 反之可以
>
> ```cangjie
> struct Foo {
>     var i = 0
>     mut func f(): Unit {
>         i += 1
>         g()           // ok
>     }
>     func g(): Unit {
>         f()           // error
>     }
> }
>
> interface I {
>     mut func f(): Unit {
>         g()           // ok
>     }
>     func g(): Unit {
>         f()           // error
>     }
> }
> ```

仓颉中, 非`mut`函数不能访问`mut`函数

---

仓颉中, `struct`的普通成员函数是不能修改成员变量的, 但是是可以访问成员变量的

这表示, `struct`的`this`默认是不可变的

根本的原因是, 仓颉中的`struct`被设计为值类型, 如果`struct`实例被`let`修饰, 因为是值类型实例, 那么这个实例的成员应该是默认禁止被修改的

如果`this`是可变的, 那么即使是被`let`修饰的实例, 通过普通的成员函数就可能会出现, 普通成员函数能够修改成员变量的情况

这不符合`struct`的设计哲学
