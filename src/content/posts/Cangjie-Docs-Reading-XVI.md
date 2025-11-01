---
title: "仓颉文档阅读-语言规约VII: 属性"
published: 2025-10-11 11:08:21
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

## 属性

> 属性是一种特殊的语法, 它不像字段一样会存储值, 相反它们提供了一个`getter`和一个可选的`setter`来间接检索和设置值
>
> 通过使用属性可以将数据操作封装成访问函数, 使用的时候与普通字段无异, 我们只需要对数据操作, 对内部的实现无感知, 可以更便利地实现访问控制、数据监控、跟踪调试、数据绑定等机制
>
> 属性在使用时语法与字段一致, 可以作为表达式或被赋值
>
> 以下是一个简单的例子, `b`是一个典型的属性, 封装了外部对`a`的访问:
>
> ```cangjie
> class Foo {
>     private var a = 0
>     mut prop b: Int64 {
>         get() {
>             print("get")
>             a
>         }
>         set(value) {
>             print("set")
>             a = value
>         }
>     }
> }
>
> main() {
>     var x = Foo()
>     let y = x.b + 1   // get
>     x.b = y           // set
> }
> ```

属性, 是C++中没有的概念

从简单介绍来看, 属性可以用来操作成员变量, 且提供有`getter`和`setter`, 用于对属性取值以及赋值

### 属性的语法

> 属性的语法规则为:
>
> ```
> propertyDefinition
>     : propertyModifier* 'prop' identifier ':' type propertyBody?
>     ;
>
> propertyBody
>     : '{' propertyMemberDeclaration+ '}'
>     ;
>
> propertyMemberDeclaration
>     : 'get' '(' ')' block end*
>     | 'set' '(' identifier ')' block end*
>     ;
>
> propertyModifier
>     : 'public'
>     | 'private'
>     | 'protected'
>     | 'internal'
>     | 'static'
>     | 'open'
>     | 'override'
>     | 'redef'
>     | 'mut'
>     ;
> ```

一个属性基本的定义长这样:

```cangjie
class PropTest {
    mut prop prop1: Int64 {
        get() {}
        set() {}
    }
}
```

还可以使用`public`、`open`、`static`等其他修饰符修饰

>
> 没有`mut`修饰符声明的属性需要定义`getter`实现
>
> 用`mut`修饰符声明的属性需要单独的`getter`和`setter`实现
>
> 特别是, 对于数值类型、`Bool`、`Unit`、`Nothing`、`Rune`、`String`、`Range`、`Function`、`Enum`和`Tuple`类型, 在它们的扩展或定义体中不能定义`mut`修饰的属性, 也不能实现有`mut`属性的接口
>
> 如下面的示例所示, `a`是没有`mut`声明的属性, `b`是使用`mut`声明的属性
>
> ```cangjie
> class Foo {
>     prop a: Int64 {
>         get() {
>             0
>         }
>     }
>     mut prop b: Int64 {
>         get() {
>             0
>         }
>         set(v) {}
>     }
> }
> ```
>
> 没有`mut`声明的属性没有`setter`, 而且与用`let`声明的字段一样, 它们不能被赋值
>
> ```cangjie
> class A {
>     prop i: Int64 {
>         get() {
>             0
>         }
>     }
> }
> main() {
>     var x = A()
>     x.i = 1           // error
> }
> ```
>
> 特别是, 当使用`let`声明`struct`的实例时, 不能为`struct`中的属性赋值, 就像用`let`声明的字段一样
>
> ```cangjie
> struct A {
>     var i1 = 0
>     mut prop i2: Int64 {
>         get() {
>             i1
>         }
>         set(value) {
>             i1 = value
>         }
>     }
> }
> main() {
>     let x = A()
>     x.i1 = 2          // error
>     x.i2 = 2          // error
> }
> ```
>
> 属性与字段不同, 属性不可以赋初始值, 必须要声明类型

属性不是字段, 即 不是变量, 它不能存储值, 不能给初始值, 所以必须要声明类型

虽然不能存储值, 但是要**声明类型, 是为了`getter`返回目标类型数据, 以及`setter`的传参类型**, 即 决定了访问属性时获取的数据类型, 以及赋值时参与计算的数据类型

属性是否用`mut`修饰, 决定此属性是否需要实现`setter`以及此属性是否允许被赋值

`let`实例化的`strcut`, 即使属性为`mut`, 也禁止修改属性

**基础类型在扩展时, 不能定义`mut`属性, 也不能实现拥有`mut`属性的接口, 因为基础类型都是值类型**

### 属性的定义

> 属性可以在`interface`, `class`, `struct`, `enum`, `extend`中定义
>
> ```cangjie
> class A {
>     prop i: Int64 {
>         get() {
>             0
>         }
>     }
> }
>
> struct B {
>     prop i: Int64 {
>         get() {
>             0
>         }
>     }
> }
>
> enum C {
>     prop i: Int64 {
>         get() {
>             0
>         }
>     }
> }
>
> extend A {
>     prop s: String {
>         get() {
>             ""
>         }
>     }
> }
> ```

> 可以在`interface`和`abstract class`中声明抽象属性, 它的定义体可以省略
>
> 在实现类型中实现抽象属性时, 它必须保持相同的名称、相同的类型和相同的`mut`修饰符
>
> ```cangjie
> interface I {
>     prop a: Int64
> }
>
> class A <: I {
>     public prop a: Int64 {
>         get() {
>             0
>         }
>     }
> }
> ```
>
> 如同`interface`中的抽象函数可以拥有默认实现, `interface`中的抽象属性也同样可以拥有默认实现
>
> 拥有默认实现的抽象属性, 实现类型可以不提供自己的实现(必须符合默认实现的使用规则)
>
> ```cangjie
> interface I {
>     prop a: Int64 {   // ok
>         get() {
>             0
>         }
>     }
> }
> class A <: I {}       // ok
> ```

接口和抽象类中可以声明抽象属性, 但在子类或子接口中实现时, 必须要与原抽象属性声明保持完全一致

> 属性分为实例成员属性和静态成员属性
>
> 其中, 实例成员属性只能由实例访问, 在`getter`或`setter`的实现中可以访问`this`、实例成员和其它静态成员
>
> 而静态成员属性只能访问静态成员
>
> ```cangjie
> class A {
>     var x = 0
>     mut prop X: Int64 {
>         get() {
>             x + y
>         }
>         set(v) {
>             x = v + y
>         }
>     }
>     static var y = 0
>     static mut prop Y: Int64 {
>         get() {
>             y
>         }
>         set(v) {
>             y = v
>         }
>     }
> }
> ```
>
> 属性不支持重载, 也不支持遮盖, 不能和其它同级别成员重名
>
> ```cangjie
> open class A {
>     var i = 0
>     prop i: Int64 {       // error
>         get() {
>             0
>         }
>     }
> }
> class B <: A {
>     prop i: Int64 {       // error
>         get() {
>             0
>         }
>     }
> }
> ```

静态属性的`getter`和`setter`中 只能访问静态成员

普通属性的`getter`和`setter`中 也可以访问静态成员, 同时可以访问`this`

**属性的名字不能与其他成员存在任何冲突, 也不支持重载, 也不支持遮盖, 但是支持覆盖**

### 属性的实现

> 属性的`getter`和`setter`分别对应两个不同的函数
>
> - `getter`函数类型是`()->T`, `T`是该属性的类型, 当使用该属性作为表达式时会执行`getter`函数
>
> - `setter`函数类型是`(T)->Unit`, `T`是该属性的类型, 形参名需要显式指定, 当对该属性赋值时会执行`setter`函数
>
> 属性的实现同函数的实现规则一样, 其中可以包含声明和表达式, 可以省略`return`, 其返回值必须符合返回类型
>
> ```cangjie
> class Foo {
>     mut prop a: Int64 {
>         get() {           // () -> Int64
>             "123"         // error
>         }
>         set(v) {          // (Int64) -> Unit
>             123
>         }
>     }
> }
> ```
>
> 无论在属性内部还是外部, 访问属性的行为都是一致的, 因此属性递归访问时与函数一样可能会造成死循环
>
> ```cangjie
> class Foo {
>     prop i: Int64 {
>         get() {
>             i             // dead loop
>         }
>     }
> }
> ```
>
> 需要注意的是, `struct`的`setter`是`mut`函数, 因此也可以在`setter`内部修改其它字段的值, 并且`this`会受到`mut`函数的限制

属性声明的类型, 决定了`setter`和`getter`函数的类型

**禁止在`getter`函数内访问属性本身, 禁止在`setter`函数内给属性本身赋值, 会出现无限递归**

### 属性的修饰符

> 属性跟函数一样可以使用修饰符修饰, 但只允许**对整个属性修饰**, 不能对`getter`或`setter`独立修饰
>
> ```cangjie
> class Foo {
>     public mut prop a: Int64 {        // ok
>         get() {
>             0
>         }
>         set(v) {}
>     }
>     mut prop b: Int64 {
>         public get() {                // error
>             0
>         }
>         public set(v) {}              // error
>     }
> }
> ```

> 属性可以使用访问控制修饰符有`private`, `protected`, `public`
>
> ```cangjie
> class Foo {
>     private prop a: Int64 {           // ok
>         get() { 0 }
>     }
>     protected prop b: Int64 {         // ok
>         get() { 0 }
>     }
>     public static prop c: Int64 {     // ok
>         get() { 0 }
>     }
> }
> ```

> 实例属性像实例函数一样, 可以使用`open`和`override`修饰
>
> 使用`open`修饰的属性, 子类型可以使用`override`覆盖父类型的实现(`override`是可选的)
>
> ```cangjie
> open class A {
>     public open mut prop i: Int64 {
>         get() { 0 }
>         set(v) {}
>     }
> }
> class B <: A {
>     override mut prop i: Int64 {
>         get() { 1 }
>         set(v) {}
>     }
> }
> ```

> 静态属性像静态函数一样, 可以使用`redef`修饰(`redef`是可选的), 子类型可以重新实现父类型的静态属性
>
> ```cangjie
> open class A {
>     static mut prop i: Int64 {
>         get() { 0 }
>         set(v) {}
>     }
> }
> class B <: A {
>     redef static mut prop i: Int64 {
>         get() { 1 }
>         set(v) {}
>     }
> }
> ```

> 子类型`override/redef`使用`let`声明的实例属性必须要重新实现`getter`
>
> 子类型`override/redef`父类型中使用`mut`修饰符声明的的属性时, 允许只重新实现`getter`或`setter`, 但不能均不重新实现
>
> ```cangjie
> open class A {
>     public open mut prop i1: Int64 {
>         get() { 0 }
>         set(v) {}
>     }
>     static mut prop i2: Int64 {
>         get() { 0 }
>         set(v) {}
>     }
> }
>
> // case 1
> class B <: A {
>     public override mut prop i1: Int64 {
>         get() { 1 }                       // ok
>     }
>     redef static mut prop i2: Int64 {
>         get() { 1 }                       // ok
>     }
> }
>
> // case 2
> class B <: A {
>     public override mut prop i1: Int64 {
>         set(v) {}                         // ok
>     }
>     redef static mut prop i2: Int64 {
>         set(v) {}                         // ok
>     }
> }
>
> // case 3
> class B <: A {
>     override mut prop i1: Int64 {}        // error
>     redef static mut prop i2: Int64 {}    // error
> }
> ```

> 子类型的属性`override/redef`必须与父类保持相同的`mut`修饰符, 并且还必须保持相同的类型
>
> ```cangjie
> class P {}
> class S {}
>
> open class A {
>     open prop i1: P {
>         get() { P() }
>     }
>     static prop i2: P {
>         get() { P() }
>     }
> }
>
> // case 1
> class B <: A {
>     override mut prop i1: P {         // error
>         set(v) {}
>     }
>     redef static mut prop i2: P {     // error
>         set(v) {}
>     }
> }
> // case 2
> class B <: A {
>     override prop i1: S {             // error
>         get() { S() }
>     }
>     redef static prop i2: S {         // error
>         get() { S() }
>     }
> }
> ```

我不理解这一句话: **子类型`override/redef`使用`let`声明的实例属性必须要重新实现`getter`**

1. 属性并不能被`let`和`var`修饰

2. 经测试, 就算父类型属性基于`let`成员变量, 如果不显式覆盖, 也可以直接不重新实现属性

3. 假设要显式覆盖属性, 就必须要重新实现`getter`, 那跟`let`或`var`也没有任何关系

所以我不是很理解, 这一句话是什么意思?

> 子类`override`父类的属性时, 可以使用`super`调用父类的实例属性
>
> ```cangjie
> open class A {
>     open prop v: Int64 {
>         get() { 1 }
>     }
> }
> class B <: A {
>     override prop v: Int64 {
>         get() { super.v + 1 }
>     }
> }
> ```

其实不仅是通过`super`访问父类的实例属性, 还能访问其他成员
