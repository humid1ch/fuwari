---
title: "仓颉文档阅读-语言规约XVI: 常量求值"
published: 2025-10-21 00:06:11
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

## 常量求值

### `const`变量

> `const`变量是一种特殊的变量, 它**可以定义在编译时完成求值**, 并且在运行时不可改变的变量
>
> `const`变量与`let/var`声明的变量的区别是**必须在定义时就初始化**, 且必须用`const`表达式初始化
>
> 因此`const`变量的类型只能是`const`表达式支持的类型
>
> 与`let/var`一样, `const`变量也支持省略类型
>
> `const`表达式见下文定义
>
> ```cangjie
> const a: Int64 = 0                    // ok
> const b: Int64                        // error, b 为初始化
> const c = f()                         // error, f() 不是 const 表达式
> const d = 0                           // ok, d 的类型是 Int64
>
> func f(): Unit {}
> ```
>
> `const`变量定义后可以像`let/var`声明的变量一样使用
>
> 与`let/var`声明的变量不同, `const`由于在编译时就可以得到结果, 可以大幅减少程序运行时需要的计算
>
> ```cangjie
> const a = 0
>
> main(): Int64 {
>     let b: Int64 = a                  // ok
>     print(a)                          // ok
>     let c: VArray<Int64, $0> = []     // ok
>     return 0
> }
> ```
>
> `const`变量可以是全局变量, 局部变量, 静态成员变量
>
> `const`变量**不能在扩展中定义**
>
> ```cangjie
> const a = 0                           // ok
>
> class C {
>     const b = 0                       // error, const 成员字段必须由 static 修改
>     static const c = 0                // ok
>
>     var v: Int64
>
>     init() {
>         const d = 0                   // ok
>         v = b + c + d
>     }
> }
>
> extend C {
>     const e = 0                       // error, const 不能在 extend 中定义
> }
> ```
>
> `const`变量可以访问对应类型的所有实例成员, 也可以调用对应类型的所有**非`mut`**实例成员函数
>
> ```cangjie
> struct Foo {
>     let a = 0
>     var b = 0
>
>     const init() {}
>
>     func f1() {}
>     const func f2() {}
>     mut func f3() {
>         b = 123
>     }
> }
>
> main(): Int64 {
>     const v = Foo()
>     print(v.a)                        // ok
>     print(v.b)                        // ok
>     v.f1()                            // ok
>     v.f2()                            // ok
>     v.f3()                            // error, f3 是 mut 函数
>     return 0
> }
> ```
>
> `const`变量初始化后该类型实例的所有成员都是`const`的(深度`const`, 包含成员的成员), 因此不能被用于左值
>
> ```cangjie
> struct Foo {
>     let a = 0
>     var b = 0
>
>     const init() {}
> }
>
> func f() {
>     const v = Foo()
>     v.a = 1                           // error
>     v.b = 1                           // error
> }
> ```

`const`修饰变量, 定义常量, 整个程序不允许修改次变量的值

`const`变量, 必须在定义时进行初始化

`const`可以修饰成员变量, 但**必须是静态成员变量**

`const`修饰一个实例, 此实例可以访问所有成员变量, 但只能访问非`mut`成员函数

`const`修饰一个实例, 此实例的所有成员变量默认会被`const`修饰, 禁止赋值(当左值)

### `const`表达式


> 某些特定形式的表达式, 被称为`const`表达式, 这些表达式**具备了可以在编译时求值的能力**
>
> 在`const`上下文中, 这些是唯一允许的表达式, 并且始终会在编译时进行求值
>
> 而在其它非`const`上下文, `const`表达式**不保证在编译时求值**
>
> 以下表达式都是`const`表达式
>
> 1. 数值类型、`Bool`、`Unit`、`Rune`、`String`类型的**字面量(不包含插值字符串)**
>
> 2. 所有元素都是`const`表达式的`array`**字面量(不能是 Array 类型)**, `tuple`**字面量**
>
> 3. `const`变量, `const`函数形参, `const`函数中的局部变量
>
> 4. `const`函数, 包含使用`const`声明的函数名、符合`const`函数要求的`lambda`、以及这些函数返回的函数表达式
>
> 5. `const`函数调用(包含`const`构造函数), 该函数的表达式必须是`const`表达式, 所有实参必须都是`const`表达式
>
> 6. 所有参数都是`const`表达式的`enum`构造器调用, 和无参数的`enum`构造器
>
> 7. 数值类型、`Bool`、`Unit`、`Rune`、`String`类型的算数表达式、关系表达式、位运算表达式, **所有操作数都必须是`const`表达式**
>
> 8. `if`、`match`、`try`、控制转移表达式(包含`return`、`break`、`continue`、`throw`)、`is`、`as`
>
>     这些表达式内的表达式必须都是`const`表达式
>
> 9. `const`表达式的成员访问(不包含属性的访问), `tuple`的索引访问
>
> 10. `const init`和`const`函数中的`this`和`super`表达式
>
> 11. `const`表达式的`const`实例成员函数调用, 且所有实参必须都是`const`表达式

首先, 各类型的字面量都是`const`表达式

`const`修饰的各种函数、参数、变量等都是`const`表达式

各种元素是`const`表达式的运算表达式 等

`const`表达式应该不是很难区分, 除了`let`之外, 不能被修改的, 不能被当作左值的等

### `const`上下文

> `const`上下文是一类特定上下文, 在这些上下文内的表达式都必须是`const`表达式, 并且这些表达式**始终在编译时求值**
>
> `const`上下文是指 **`const`变量初始化表达式**

常量上下文, 始终再编译时求值, 其实是特指`const`变量初始化表达式

### `const`函数

> `const`函数是一类特殊的函数, 这些函数具备了**可以在编译时求值的能力**
>
> 在`const`上下文中调用这种函数时, 这些函数会在编译时执行计算
>
> 而在其它非`const`上下文, `const`函数会和普通函数一样在运行时执行
>
> `const`函数与普通函数的区别是 限制了部分影响编译时求值的功能, `const`函数中只能出现声明、`const`表达式、受限的部分赋值表达式
>
> 1. `const`函数声明必须使用`const`修饰
>
> 2. 全局`const`函数和`static const`函数中只能访问`const`声明的外部变量, 包含`const`全局变量、`const`静态成员变量, 其它外部变量都不可访问
>
>     `const init`函数和`const`实例成员函数, 除了能访问`const`声明的外部变量, 还可以访问当前类型的实例成员变量
>
> 3. `const`函数中的表达式都必须是`const`表达式, `const init`函数除外
>
> 4. `const`函数中可以使用`let`、`const`声明新的局部变量, 但不支持`var`
>
> 5. `const`函数中的参数类型和返回类型没有特殊规定
>
>     如果该函数调用的实参不符合`const`表达式要求, 那这个函数调用不能作为`const`表达式使用, 但仍然可以作为普通表达式使用
>
> 6. `const`函数不一定都会在编译时执行, 例如可以在非`const`函数中运行时调用
>
> 7. `const`函数与非`const`函数重载规则一致
>
> 8. 数值类型、`Bool`、`Unit`、`Rune`、`String`类型 和`enum`支持定义`const`实例成员函数
>
> 9. 对于`struct`和`class`, 只有定义了`const init`才能定义`const`实例成员函数
>
>     `class`中的`const`实例成员函数不能是`open`的
>
>     `struct`中的`const`实例成员函数不能是`mut`的
>
> ```cangjie
> const func f(a: Int64): Int64 {   // ok
>     let b = 6
>     if (a > 5 && (b + a) < 15 ) {
>         return a * a
>     }
>     return a
> }
>
> class A {
>     var a = 0
> }
>
> const func f1(a: A): Unit {       // ok
>     return
> }
>
> const func f2(): A {
>     return A()                    // error, A 没有包含 const init
> }
> ```

`const`函数使用`const`修饰, 其内的表达式全部为`const`表达式(`const init`除外)

`const`函数在`const`上下文中调用时, 会在编译时求值

#### 接口中的`const`函数

> 接口中也可以定义`const`函数, 但会受到以下规则限制
>
> - 接口中的`const`函数, 实现类型必须也用`const`函数才算实现接口
>
> - 接口中的非`const`函数, 实现类型使用`const`或非`const`函数都算实现接口
>
> - 接口中的`const`函数与接口的`static`函数一样, 只有在该接口作为泛型约束的时候, 受约束的泛型变元或变量才能使用这些`const`函数
>
> ```cangjie
> interface I {
>     const func f(): Int64
>     const static func f2(): Int64
> }
>
> const func g<T>(i: T) where T <: I {
>     return i.f() + T.f2()
> }
> ```

接口中的`const`函数, 只有实际实现`const`函数才算实现接口函数

接口中的普通函数, 实际实现`const`或非`const`函数, 都算实现接口函数

### `const init`

> 有无`const init`决定了哪些自定义`struct/class`可以用在`const`表达式上
>
> 一个类型能否定义`const init`取决于以下条件
>
> - 如果当前类型是`class`, 则不能具有`var`声明的实例成员变量, 否则不允许定义`const init`
>
>     如果当前类型具有父类, 当前的`const init`必须调用父类的`const init`(可以显式调用或者隐式调用无参`const init`), 如果父类没有`const init`则报错
>
> - `Object`类型的无参`init`也是`const init`, 因此`Object`的子类可以使用该`const init`
>
> - 当前类型的实例成员变量如果有初始值, 初始值必须要是`const`表达式, 否则不允许定义`const init`
>
> - `const init`内可以使用赋值表达式对实例成员变量赋值, 除此以外不能有其它赋值表达式
>
> `const init`与`const`函数的区别是`const init`内允许对实例成员变量进行赋值(需要使用赋值表达式)
>
> ```cangjie
> struct R1 {
>     var a: Int64
>     let b: Int64
>
>     const init() {        // ok
>         a = 0
>         b = 0
>     }
> }
>
> struct R2 {
>     var a = 0
>     let b = 0
>
>     const init() {}       // ok
> }
>
> func zero(): Int64 {
>     return 0
> }
>
> struct R3 {
>     let a = zero()
>
>     const init() {}       // error, a 的初始化不是 const 表达式
> }
>
> class C1 {
>     var a = 0
>
>     const init() {}       // error, a 不能是 var 绑定
> }
>
> struct R4 {
>     var a = C1()
>
>     const init() {}       // error, a 的初始化不是 const 表达式
> }
>
> open class C2 {
>     let a = 0
>     let b = R2()
>
>     const init() {}       // ok
> }
>
> class C3 <: C2 {
>     let c = 0
>
>     const init() {}       // ok
> }
> ```

如果, `class`想要实现`const init`, 那么成员变量**不允许存在`var`变量**

且`struct/class`成员如果存在定义直接初始化, 需要是**`const`表达式**, 否则不能实现`const init`

`const init`内出现的赋值表达式, 只能是对成员变量进行赋值的表达式, 不允许出现其他赋值表达式

`const init`的作用, 感觉更大是优化性能, 因为在编译时会实例化对象

**要实例化一个`const`变量, 那么此类型就必须实现`const init`**
