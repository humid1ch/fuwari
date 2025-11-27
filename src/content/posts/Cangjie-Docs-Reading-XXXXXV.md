---
title: "仓颉文档阅读-开发指南VIV: 扩展(II) - 扩展访问规则"
published: 2025-11-26 16:27:23
description: '仓颉文档阅读的开发指南部分, 本篇文章介绍仓颉语言的 扩展访问规则 的相关内容'
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

## 扩展

### 访问规则

扩展成员的可访问性, 并不完全与原类型成员的可访问性保持一致

#### 扩展的修饰符

> **扩展本身不能使用修饰符修饰**
> 
> 例如, 下面的例子中对`A`的直接扩展前使用了`public`修饰, 将编译报错
> 
> ```cangjie
> public class A {}
> 
> public extend A {}            // Error, 在 extend 之前不该存在修饰符
> ```
> 
> 扩展成员可使用的修饰符有: `static`、`public`、`protected`、`internal`、`private`、`mut`
> 
> - 使用 **`private`** 修饰的成员 **只能在本扩展内使用**, 外部不可见
> 
> - 使用 **`internal`** 修饰的成员 **可以在当前包及子包(包括子包的子包)内使用**, 这是默认行为
> 
> - 使用 **`protected`** 修饰的成员 **在本模块内可以被访问(受导出规则限制)**
> 
>     当被扩展类型是`class`时, 该 **`class`的子类定义体也能访问**
> 
> - 使用 **`static`** 修饰的成员, **只能通过类型名访问**, 不能通过实例对象访问
> 
> - 对 **`struct`** 类型的扩展 **可以定义`mut`函数**
> 
> ```cangjie
> package p1
> 
> public open class A {}
> 
> extend A {
>     public func f1() {}
>     protected func f2() {}
>     private func f3() {}
>     static func f4() {}
> }
> 
> main() {
>     A.f4()
>     var a = A()
>     a.f1()
>     a.f2()
> }
> ```
> 
> 扩展内的成员定义不支持使用`open`、`override`、`redef`修饰
> 
> ```cangjie
> class Foo {
>     public open func f() {}
>     static func h() {}
> }
> 
> extend Foo {
>     public override func f() {}       // Error
>     public open func g() {}           // Error
>     redef static func h() {}          // Error
> }
> ```

扩展定义本身不允许用任何修饰符修饰

但扩展的成员可以使用 可访问性修饰符: `private`、`internal`、`protected`和`public`, 以及`static`和`mut`(仅`struct`)修饰

同时, 扩展的成员不允许使用`open`、`override`和`redef`修饰, 即扩展不能被继承

> [!TIP]
> 
> - `private`表示仅在当前扩展内可见
> 
>     即, 仅能在扩展定义时可访问
> 
> - `internal`表示仅当前包及子包(包括子包的子包)内可见
> 
>     即, 仅能在当前扩展定义所在包, 以及其子包内访问
> 
> - `protected`表示当前模块可见
> 
>     即, 与`internal`不同, 被`protected`修饰的成员, 可以在父包、当前包以及子包访问
> 
> - `public`表示模块内外均可见
> 
>     被`public`修饰的成员的访问, 只要可以见到, 不受任何限制

#### 扩展的孤儿规则

> 为一个其他`package`的类型实现另一个`package`的接口, 可能造成理解上的困扰
> 
> 为了防止一个类型被意外实现不合适的接口, 仓颉不允许定义孤儿扩展, 即 既不与接口(包含接口继承链上的所有接口)定义在同一个包中, 也不与被扩展类型定义在同一个包中的接口扩展
> 
> 如下代码所示, 不能在`package c`中, 为`package a`里的`Foo`实现`package b`里的`Bar`
> 
> 只能在`package a`或者在`package b`中为`Foo`实现`Bar`
> 
> ```cangjie
> // package a
> public class Foo {}
> 
> // package b
> public interface Bar {}
> 
> // package c
> import a.Foo
> import b.Bar
> 
> extend Foo <: Bar {}      // Error
> ```

关于孤儿扩展, 是只针对 接口扩展的

孤儿扩展是指, 在定义接口扩展时, 在 **既不是类型定义的包 也不是接口定义的包中** 定义接口扩展

这里的接口, 甚至包含被扩展实现接口继承链上的所有接口

#### 扩展的访问和遮盖

> 扩展的实例成员与类型定义处一样可以使用`this`, `this`的功能保持一致
> 
> 同样也可以省略`this`访问成员
> 
> **扩展的实例成员不能使用`super`**
> 
> ```cangjie
> class A {
>     var v = 0
> }
> 
> extend A {
>     func f() {
>         print(this.v)     // Ok
>         print(v)          // Ok
>     }
> }
> ```
> 
> **扩展不能访问被扩展类型中`private`修饰的成员**
> 
> ```cangjie
> class A {
>     private var v1 = 0
>     protected var v2 = 0
> }
> 
> extend A {
>     func f() {
>         print(v1)         // Error
>         print(v2)         // Ok
>     }
> }
> ```
> 
> **扩展不能遮盖被扩展类型的任何成员**
> 
> ```cangjie
> class A {
>     func f() {}
> }
> 
> extend A {
>     func f() {}           // Error
> }
> ```
> 
> **扩展也不允许遮盖其他扩展增加的任何成员**
> 
> ```cangjie
> class A {}
> 
> extend A {
>     func f() {}
> }
> 
> extend A {
>     func f() {}           // Error
> }
> ```
> 
> 在同一个包内, 对**同一类型可以扩展多次**, 并且在扩展中**可以直接调用**被扩展类型的**其他扩展中非`private`修饰的函数**
> 
> ```cangjie
> class Foo {}
> 
> extend Foo {              // OK
>     private func f() {}
>     func g() {}
> }
> 
> extend Foo {              // OK
>     func h() {
>         g()               // OK
>         f()               // Error
>     }
> }
> ```

扩展定义时, 可以访问原类型的非`private`的成员以及`this`, 也可以访问其他扩展的非`private`成员, 但无法调用`super`

扩展定义时, 不允许遮盖 原类型的成员以及其他扩展定义的成员

> 扩展泛型类型时, 可以使用额外的泛型约束
> 
> 泛型类型的任意两个扩展之间的可见性规则如下: 
> 
> - 如果两个扩展的**约束相同**, 则两个扩展相互可见, 即两个扩展内可以直接使用对方内的函数或属性
> 
> - 如果两个扩展的**约束不同**, 且两个扩展的**约束有包含关系**, 约束更宽松的扩展对约束更严格的扩展可见, 反之, 不可见
> 
> - 当两个扩展的**约束不同**时, 且两个**约束不存在包含关系**, 则两个扩展均互相不可见
> 
> 示例: 假设对同一个类型`E<X>`的两个扩展分别为扩展1和扩展2, `X`的约束在扩展1 中比扩展2 中更严格, 那么扩展1 中的函数和属性对扩展2 均不可见, 反之, 扩展2 中的函数和属性对扩展1 可见
> 
> ```cangjie
> open class A {}
> class B <: A {}
> class E<X> {}
> 
> interface I1 {
>     func f1(): Unit
> }
> interface I2 {
>     func f2(): Unit
> }
> 
> extend<X> E<X> <: I1 where X <: B {   // extension 1
>     public func f1(): Unit {
>         f2()                          // OK
>     }
> }
> 
> extend<X> E<X> <: I2 where X <: A {   // extension 2
>     public func f2(): Unit {
>         f1()                          // Error
>     }
> }
> ```

#### 扩展的导入导出

> 扩展也是可以被导入和导出的, 但是扩展本身不能使用可见性修饰符修饰, **扩展的导出有一套特殊的规则**
> 
> 对于直接扩展, **当扩展与被扩展的类型在同一个包中**, 扩展是否导出, **由被扩展类型与泛型约束(如果有)的访问修饰符同时决定**, **当所有的泛型约束都是导出类型(修饰符与导出规则, 详见顶层声明的可见性章节)时, 该扩展将被导出**
> 
> **当扩展与被扩展类型不在同一个包中时, 该扩展不会导出**
> 
> 如以下代码所示, `Foo`是导出的, `f1`函数所在的扩展由于不导出泛型约束, 故该扩展不会被导出;`f2`和`f3`函数所在的扩展的泛型约束均被导出, 故该扩展被导出;`f4`函数所在的扩展包含多个泛型约束, 且泛型约束中`I1`未被导出, 故该扩展不会被导出;`f5`函数所在的扩展包含多个泛型约束, 所有的泛型约束均是导出的, 故该扩展会被导出
> 
> ```cangjie
> // package a.b
> package a.b
> 
> private interface I1 {}
> internal interface I2 {}
> protected interface I3 {}
> 
> extend Int64 <: I1 & I2 & I3 {}
> 
> public class Foo<T> {}
> 
> // 该扩展不会被导出
> extend<T> Foo<T> where T <: I1 {
>     public func f1() {}
> }
> 
> // 该扩展会被导出, 只有同时导入 Foo 和 I2 的包才能访问它
> extend<T> Foo<T> where T <: I2 {
>     public func f2() {}
> }
> 
> // 该扩展会被导出, 只有同时导入 Foo 和 I3 的包才能访问它
> extend<T> Foo<T> where T <: I3 {
>     public func f3() {}
> }
> 
> // 扩展不会被导出, 具有最低访问级别的 I1 决定了导出
> extend<T> Foo<T> where T <: I1 & I2 & I3 {
>     public func f4() {}
> }
> 
> // 扩展会被导出, 只有导入 Foo、I2 和 I3 的包才能访问扩展
> extend<T> Foo<T> where T <: I2 & I3 {
>     public func f5() {}
> }
> ```
> 
> ```cangjie
> // package a.c
> package a.c
> import a.b.*
> 
> main() {
>     Foo<Int64>().f1()         // 无法访问
>     Foo<Int64>().f2()         // 无法访问, 只在子包中可见
>     Foo<Int64>().f3()         // Ok
>     Foo<Int64>().f4()         // 无法访问
>     Foo<Int64>().f5()         // 无法访问, 只在子包中可见
> }
> ```
> 
> ```cangjie
> // package a.b.d
> package a.b.d
> import a.b.*
> 
> main() {
>     Foo<Int64>().f1()         // 无法访问
>     Foo<Int64>().f2()         // Ok
>     Foo<Int64>().f3()         // Ok
>     Foo<Int64>().f4()         // 无法访问
>     Foo<Int64>().f5()         // Ok
> }
> ```

仓颉中, 扩展可以被导入和导出, 但**不是由用户手动进行的**, 而是根据 与扩展相关的可访问性修饰符 和 扩展定义位置 一起决定, 且**由编译器自己决定**

当相关类型满足条件时, **扩展自动被导出或导入**

针对直接扩展:

1. 如果 扩展定义与被扩展类型, 不在同一个包中, 则 该扩展一定不被导出

    此时, 扩展就只能在扩展定义时所在的包使用

2. 如果 扩展定义拥有泛型约束, 那么扩展的导出 与 被扩展类型和泛型约束的目标类型的可访问性修饰符保持一致
   
    即, 如果泛型约束的目标类型, 被`private`修饰, 那么此扩展不被导出

    如果被`internal`修饰, 那么被导出, 且在扩展定义的子包中可访问(被导入)

    如果被`protected`修饰, 那么被导出, 且在扩展定义的所在模块中可访问(被导入)

    如果被`public`修饰, 那么被导出, 且在所有位置可访问(被导入)

    但, 扩展可访问(扩展导入)的要求是: **泛型约束的所有目标类型 以及 被扩展类型 在目标包中可访问**

> 对于接口扩展则分为两种情况: 
> 
> - 当接口扩展与被扩展类型在相同的`package`时, 扩展会与被扩展类型以及泛型约束(如果有)一起被导出, 不受接口类型的访问级别影响, 包外不需要导入接口类型也能访问该扩展的成员
> 
> - 当接口扩展与被扩展类型在不同的`package`时, 接口扩展是否导出由接口类型以及泛型约束(如果有)里用到的类型中最小的访问级别决定
> 
>     其他`package`必须导入被扩展类型、相应的接口以及约束用到的类型(如果有), 才能访问对应接口包含的扩展成员
> 
> 如下代码所示, 在包`a`中, 虽然接口访问修饰符为`private`, 但`Foo`的扩展仍然会被导出
> 
> ```cangjie
> // package a
> package a
> 
> private interface I0 {}
> 
> public class Foo<T> {}
> 
> // 扩展会被导出
> extend<T> Foo<T> <: I0 {}
> ```
> 
> 当在其他包中为`Foo`类型扩展时, 扩展是否导出由实现接口和泛型约束的访问修饰符决定
> 
> 实现接口至少存在一个导出的接口, 且所有的泛型约束均可导出时, 该扩展将被导出
> 
> ```cangjie
> // package b
> package b
> 
> import a.Foo
> 
> private interface I1 {}
> internal interface I2 {}
> protected interface I3 {}
> public interface I4 {}
> 
> // 该扩展不会导出, 因为 I1 在文件外部不可见
> extend<T> Foo<T> <: I1 {}
> 
> // 该扩展会被导出
> extend<T> Foo<T> <: I2 {}
> 
> // 该扩展会被导出
> extend<T> Foo<T> <: I3 {}
> 
> // 该扩展会被导出
> extend<T> Foo<T> <: I1 & I2 & I3 {}
> 
> // 扩展不会导出, 具有最低访问级别的 I1 将决定导出
> extend<T> Foo<T> <: I4 where T <: I1 & I2 & I3 {}
> 
> // 该扩展会被导出
> extend<T> Foo<T> <: I4 where T <: I2 & I3 {}
> 
> // 该扩展会被导出
> extend<T> Foo<T> <: I4 & I3 where T <: I2 {}
> ```
> 
> 特别的, **接口扩展导出的成员仅限于接口中包含的成员**
> 
> ```cangjie
> // package a
> package a
> 
> public class Foo {}
> ```
> 
> ```cangjie
> // package b
> package b
> 
> import a.Foo
> 
> public interface I1 {
>     func f1(): Unit
> }
> 
> public interface I2 {
>     func f2(): Unit
> }
> 
> extend Foo <: I1 & I2 {
>     public func f1(): Unit {}
>     public func f2(): Unit {}
>     public func f3(): Unit {}         // f3 不会被导出
> }
> ```
> 
> ```cangjie
> // package c
> package c
> 
> import a.Foo
> import b.I1
> 
> main() {
>     let x: Foo = Foo()
>     x.f1()                            // OK, 因为 f1 是 I1 的成员
>     x.f2()                            // error, f2 没有被导入
>     x.f3()                            // error, f3 未找到
> }
> ```

针对接口扩展:

1. 如果 被扩展类型和接口扩展定义都 **在同一个`package`中**, 那么**此扩展与被扩展类型一起被导出**, 且与扩展接口的访问限制无关

2. 如果 被扩展类型和接口扩展定义 **不在同一个`package`中**

    - 扩展是否被导出, 与 被扩展类型、扩展的接口 以及泛型约束 中, 所用到类型的最小的可访问性限制决定

    - 其他`package`如果要访问扩展, 此`package`需要同时导入 被扩展类型、所有扩展接口类型 以及 泛型约束所用的所有类型

3. 接口扩展, 只有 被实现接口的成员会被导出, **如果扩展中定义了非实现接口中的成员, 此新成员不会被导出**

> 与扩展的导出类似, 扩展的导入也**不需要显式地用`import`导入**, **扩展的导入只需要导入被扩展的类型、接口和泛型约束, 就可以导入可访问的所有扩展**
> 
> 如下面的代码所示, 在`package b`中, 只需要导入`Foo`就可以使用`Foo`对应的扩展中的函数`f`
> 
> 而对于接口扩展, 需要同时导入被扩展的类型、扩展的接口和泛型约束(如果有)才能使用
> 
> 因此在`package c`中, 需要同时导入`Foo`和`I`才能使用对应扩展中的函数`g`
> 
> ```cangjie
> // package a
> package a
> public class Foo {}
> extend Foo {
>     public func f() {}
> }
> ```
> 
> ```cangjie
> // package b
> package b
> import a.Foo
> 
> public interface I {
>     func g(): Unit
> }
> extend Foo <: I {
>     public func g() {
>         this.f()          // OK
>     }
> }
> ```
> 
> ```cangjie
> // package c
> package c
> import a.Foo
> import b.I
> 
> func test() {
>     let a = Foo()
>     a.f()                 // OK
>     a.g()                 // OK
> }
> ```
