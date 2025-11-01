---
title: "仓颉文档阅读-语言规约VIII: 扩展"
published: 2025-10-11 15:18:21
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

## 扩展

> 扩展可以为除函数、元组、接口外的在当前`package`可见的任何类型添加新功能
>
> 可以添加的功能包括:
>
> - 添加实例成员函数
>
> - 添加静态成员函数
>
> - 添加操作符重载
>
> - 添加实例成员属性
>
> - 添加静态成员属性
>
> - 实现接口
>
> 扩展是在类型定义之后追加的功能, 扩展不能破坏原有类型的封装性, 因此以下功能是禁止的
>
> - 扩展不能增加字段
>
> - 扩展不能增加抽象成员
>
> - 扩展不能增加`open`成员
>
> - 扩展不能`override/redef`原有的成员
>
> - 扩展不能访问原类型的私有成员

扩展, 是C++中没有的概念, C++中类定义完成之后, 就不能再其他地方直接扩展了

从文档描述看, 扩展更多的是为类型添加 成员函数、属性或实现新接口, 禁止对原有成员做覆盖等修改, 也禁止添加新的可继承成员

而且, 扩展不能访问原类型的私有成员

最核心是添加, 但不能修改原类型的一些继承语义

### 扩展语法

> 一个简单的扩展示例如下:
>
> ```cangjie
> extend String {
>     func printSize() {
>         print(this.size)
>     }
> }
>
> "123".printSize()         // 3
> ```
>
> 扩展定义的语法如下:
>
> ```
> extendDefinition
>     : 'extend' extendType
>       ('<:' superInterfaces)? genericConstraints?
>       extendBody
>     ;
>
> extendType
>     : (typeParameters)? (identifier NL* DOT  NL*)* identifier (NL* typeArguments)?
>     | INT8
>     | INT16
>     | INT32
>     | INT64
>     | INTNATIVE
>     | UINT8
>     | UINT16
>     | UINT32
>     | UINT64
>     | UINTNATIVE
>     | FLOAT16
>     | FLOAT32
>     | FLOAT64
>     | CHAR
>     | BOOLEAN
>     | NOTHING
>     | UNIT
>     ;
>
> extendBody
>     : '{' extendMemberDeclaration* '}'
>     ;
>
> extendMemberDeclaration
>     : (functionDefinition
>     | operatorFunctionDefinition
>     | propertyDefinition
>     | macroExpression
>     ) end*
>     ;
> ```
>
> 扩展的定义使用`extend`关键字, 扩展定义依次为`extend`关键字、可选的泛型形参、被扩展的类型、可选的实现接口、可选的泛型约束, 以及扩展体的定义
>
> 扩展体的定义不允许省略`{}`
>
> 扩展只能定义在`top level`
>
> 扩展分为直接扩展和接口扩展两种用法, 直接扩展不需要声明额外的接口

扩展的关键字是`extend`, 之后要跟被扩展的类型名, 然后就类似类的定义

但要按照扩展要求进行实现

#### 直接扩展

> 直接扩展不需要声明额外的接口, 可以用来直接为现有的类型添加新功能
>
> ```cangjie
> class Foo {}
>
> extend Foo {
>     func f() {}
> }
>
> main() {
>     let a = Foo()
>     a.f() // call extension function
> }
> ```

直接扩展比较简单, 就是按语法添加允许添加的成员就可以了

#### 接口扩展

> 接口扩展可以用来为现有的类型添加新功能并实现接口, 增强抽象灵活性
>
> 使用接口扩展时必须要声明被实现的接口
>
> ```cangjie
> interface I {
>     func f(): Unit
> }
>
> class Foo {}
>
> extend Foo <: I {
>     public func f() {}
> }
> ```

> 对一个类型使用接口扩展功能实现接口`I`, 其等价于在类型定义时实现接口`I`, 但使用范围会受到扩展导入导出的限制, 详细见[扩展的导入导出]
>
> ```cangjie
> func g(i: I) {
>     i.f()
> }
>
> main() {
>     let a = Foo()
>     g(a)
> }
> ```
>
> 我们可以在同一个扩展内同时实现多个接口, 多个接口之间使用`&`分开, 接口的顺序没有先后关系
>
> ```cangjie
> interface I1 {
>     func f1(): Unit
> }
>
> interface I2 {
>     func f2(): Unit
> }
>
> interface I3 {
>     func f3(): Unit
> }
>
> class Foo {}
>
> extend Foo <: I1 & I2 & I3 {
>     public func f1() {}
>     public func f2() {}
>     public func f3() {}
> }
> ```

接口扩展, 与类型定义时接口实现基本一致, 需要声明被实现的接口, 可以同时扩展实现多个接口

但扩展的接口, 存在使用范围限制, 在之后有提及


> 如果被扩展类型**已经实现**过某个接口, 则 不能通过扩展重复实现该接口, 包含使用扩展实现过的接口
>
> ```cangjie
> interface I {
>     func f(): Unit
> }
>
> class Foo <: I {
>     func f(): Unit {}
> }
> extend Foo <: I {}        // error, 不能重复实现接口
>
> class Bar {}
> extend Bar <: I {
>     func f(): Unit {}
> }
> extend Bar <: I {}        // error, 通过扩展已经实现的接口, 也不能重复实现
> ```

接口不允许重复实现, 扩展也同样如此, 因为扩展接口可以等价实现接口

> 如果被扩展类型已经直接实现 某个非泛型接口, 则不能使用扩展重新实现该接口
>
> 如果被扩展类型已经直接实现某个泛型接口, 则不能使用相同的类型参数扩展重新实现该接口
>
> ```cangjie
> interface I1 {}
>
> class Foo <: I1 {}
>
> extend Foo <: I1 {}               // error
>
> interface I2<T> {}
>
> class Goo<T> <: I2<Int32> {}
>
> extend Goo<T> <: I2<Int32> {}     // error
>
> extend Goo<T> <: I2<T> {}         // ok
> ```

依旧是不允许重复实现接口的原则, 泛型接口的类型变元相同也被看作同一接口类型

> 如果被扩展的类型已经包含 接口要求的函数, 则接口扩展 不能再重新实现这些函数, 也不会再使用接口中的默认实现
>
> ```cangjie
> class Foo {
>     public func f() {}
> }
>
> interface I {
>     func f(): Unit
> }
>
> extend Foo <: I {}                // ok
>
> extend Foo {
>     public func g(): Unit {
>         print("In extend!")
>     }
> }
>
> interface I2 {
>     func g(): Unit {
>         print("In interface!")
>     }
> }
>
> extend Foo <: I2 {}               // ok, I2 中 g 的默认实现不再使用
> ```

接口的本质类似一个不拥有成员变量的抽象类

所以类是有可能 本身已经实现了 接口中的函数的, 此时 扩展接口的对应函数, 将不能在扩展时被重新实现


> 禁止定义孤儿扩展, 孤儿扩展指的是 **接口扩展既不与接口(包含接口继承链上的所有接口)定义在同一个包中, 也不与被扩展类型定义在同一个包中**
>
> 即接口扩展只允许以下两种情况:
>
> - 接口扩展与类型定义处在同一个包
>
> - 接口扩展实现的接口, 和接口的继承链上 所有未被 被扩展类型实现的接口, 都必须在同一个包中
>
> 其它情况的接口扩展都是不允许定义的
>
> ```cangjie
> // package pkg1
> public class Foo {}
> public class Goo {}
>
> // package pkg2
> public interface Bar {}
> extend Goo <: Bar {}
>
> // package pkg3
> import pkg1.Foo
> import pkg2.Bar
>
> extend Foo <: Bar {}          // error
>
> interface Sub <: Bar {}
>
> extend Foo <: Sub {}          // error
>
> extend Goo <: Sub {}          // ok, 'Goo' 在 pkg2 中的继承链上实现了接口 'Bar'
> ```

接口扩展的限制, 文档的说明有一些绕

明确一些总结就是:

接口扩展这个动作(定义):

1. 要么 与被扩展类型定义在一个包中

    意思是, 接口扩展 要与 被扩展的类型 定义在同一个包中

    就像这样:

    ```cangjie
    import pkg1.Iter

    class AClass {}

    extend AClass <: Iter {}
    ```

    保证 接口扩展的定义 与 被扩展类型的定义 在同一个包中

2. 要么 与 所有 被扩展类型还未实现的接口 定义在同一个包中

    意思是, 接口扩展 要与 所有 被扩展类型还未实现 但需要被扩展实现的接口 定义在同一个包中

    就像这样:

    ```cangjie
    // case 1
    // AClass在另一个包 pkg1 中
    import pkg1.AClass

    interface Iter1 {}

    interface Iter2 {}

    extend AClass <: Iter1 & Iter2 {}       // OK, 接口扩展定义 与 所有未实现的接口 定义在同一个包中

    // case 2
    // AClass在包 pkg1 中
    import pkg1.AClass
    // Iter1在包 pkg2 中
    import pkg2.Iter1

    interface Iter2 {}

    extend AClass <: Iter1 & Iter2 {}       // Error, 接口扩展定义 未与AClass 定义在同一个包中, 也未与 Iter1 定义在一个包中(Iter1是未实现的接口)

    // case 3
    // AClass和Iter1 都在包 pkg1 中, 但AClass未实现Iter1
    import pkg1.AClass
    import pkg1.Iter1

    interface Iter2 {}

    extend AClass <: Iter1 & Iter2 {}       // Error, 接口扩展定义 未与AClass 定义在同一个包中, 也未与 Iter1 定义在一个包中(Iter1是未实现的接口)

    // case 4
    // AClass和Iter1 都在包 pkg1 中, 但AClass未实现Iter1
    import pkg1.AClass
    import pkg1.Iter1

    interface Iter2 <: Iter1 {}

    extend AClass <: Iter2 {}       // Error, 接口扩展定义 未与AClass 定义在同一个包中, 也未与 Iter1 定义在一个包中(Iter1是接口继承链上未实现的接口)

    // case 5
    // AClass和Iter1 都在包 pkg1 中, 且AClass已经实现或扩展了Iter1
    import pkg1.AClass
    import pkg1.Iter1

    interface Iter2 <: Iter1 {}

    extend AClass <: Iter2 {}       // OK, 接口扩展定义 未与AClass 定义在同一个包中, 但与 所有未实现的接口 定义在同一个包中(AClass已经实现了Iter1)
    ```

    保证 接口扩展定义 与 所有需要扩展但被扩展类型未实现的接口 定义在同一个包中

那么 孤儿扩展, 就是这两种情况都不符合

### 扩展的成员

> 扩展的成员包括: 静态成员函数、实例成员函数、静态成员属性、实例成员属性、操作符重载函数

基本都是函数和属性, 不能扩展变量字段等

#### 函数

> 扩展可以对被扩展类型添加函数, 这些函数可以是泛型函数, 支持泛型约束, 支持重载, 也支持默认参数和命名参数
>
> 这些函数都不能是抽象函数
>
> 例如:
>
> ```cangjie
> interface I {}
> extend Int64 {
>     func f<T>(a: T, b!: Int64 = 0) where T <: I {}
>     func f(a: String, b: String) {}
> }
> ```

##### 修饰符

> 扩展内的函数定义支持使用`private`, `protected`(仅限于被扩展类型是`class`类型)或`public`修饰
>
> 使用`private`修饰的函数只能在本扩展内使用, 外部不可见
>
> 使用`protected`修饰的成员函数除了能在本包内被访问, 对包外的当前`class`子类也可以访问
>
> 没有使用`private`, `protected`或`public`修饰的函数只能在本包内使用
>
> ```cangjie
> // file1 在 包p1
> package p1
>
> public open class Foo {}
>
> extend Foo {
>     private func f1() {}          // ok
>     public func f2() {}           // ok
>     protected func f3() {}        // ok
>     func f4() {}                  // 没有被 private public protected 修饰, 包内可见
> }
>
> main() {
>     let a = Foo()
>     a.f1()                        // error, 无法访问 private 函数
>     a.f2()                        // ok
>     a.f3()                        // ok
>     a.f4()                        // ok
> }
>
> // file2 在 包p2
> package p2
>
> import p1.*
>
> class Bar <: Foo {
>     func f() {
>         f1()                      // error, 无法访问 private 函数
>         f2()                      // ok
>         f3()                      // ok
>         f4()                      // error, 无法在其他包访问 default 函数
>     }
> }
> ```
>
> 扩展内的函数支持使用`static`修饰
>
> ```cangjie
> class Foo {}
>
> extend Foo {
>     static func f() {}
> }
> ```
>
> 对`struct`类型的扩展可以定义`mut`函数
>
> ```cangjie
> struct Foo {
>     var i = 0
> }
>
> extend Foo {
>     mut func f() {                // ok
>         i += 1
>     }
> }
> ```

不影响原类型继承语义的函数, 在扩展中都可以被定义、添加

> 扩展内的函数定义不支持使用`open`、`override`、`redef`修饰
>
> ```cangjie
> class Foo {
>     public open func f() {}
>     static func h() {}
> }
>
> extend Foo {
>     public override func f() {}   // error
>     public open func g() {}       // error
>     redef static func h() {}      // error
> }
> ```

扩展内的函数定义, 不允许新增可继承函数、不允许尝试覆盖原类型父类的`open`函数、不允许重定义原类型父类的静态函数

#### 属性

> 扩展可以对被扩展类型添加属性
>
> 这些属性都不能是抽象属性
>
> 例如:
>
> ```cangjie
> extend Int64 {
>     mut prop i: Int64 {
>         get() {
>             0
>         }
>         set(value) {}
>     }
> }
> ```

##### 修饰符

> 扩展内的属性定义支持使用`private`, `protected`(仅限于被扩展类型是`class`类型)或`public`修饰
>
> 使用`private`修饰的属性只能在本扩展内使用, 外部不可见
>
> 使用`protected`修饰的属性除了能在本包内被访问, 对包外的当前`class`子类也可以访问
>
> 没有使用`private`, `protected`或`public`修饰的属性只能在本包内使用
>
> ```cangjie
> // file1 在 包p1
> package p1
>
> public open class Foo {}
>
> extend Foo {
>     private prop v1: Int64 {          // ok
>         get() { 0 }
>     }
>     public prop v2: Int64 {           // ok
>         get() { 0 }
>     }
>     protected prop v3: Int64 {        // ok
>         get() { 0 }
>     }
>     prop v4: Int64 {                  // 在本包内可见
>         get() { 0 }
>     }
> }
>
> main() {
>     let a = Foo()
>     a.v1                              // error, 无法访问 private 函数
>     a.v2                              // ok
>     a.v3                              // ok
>     a.v4                              // ok
> }
>
> // file2 在 包p2
> package p2
>
> import p1.*
>
> class Bar <: Foo {
>     func f() {
>         v1                            // error, 无法访问 private 函数
>         v2                            // ok
>         v3                            // ok
>         v4                            // error, 无法在其他包访问 default 函数
>     }
> }
> ```
>
> 扩展内的属性支持使用`static`修饰
>
> ```cangjie
> class Foo {}
>
> extend Foo {
>     static prop i: Int64 {
>         get() { 0 }
>     }
> }
> ```

上面的不影响原类型继承语义的扩展属性的定义, 与属性的正常定义方式被无二致

> 扩展内的属性定义不支持使用`open`、`override`、`redef`修饰
>
> ```cangjie
> class Foo {
>     open prop v1: Int64 {
>         get() { 0 }
>     }
>     static prop v2: Int64 {
>         get() { 0 }
>     }
> }
>
> extend Foo {
>     override prop v1: Int64 {             // error
>         get() { 0 }
>     }
>     open prop v3: Int64 {                 // error
>         get() { 0 }
>     }
>     redef static prop v2: Int64 {         // error
>         get() { 0 }
>     }
> }
> ```

依旧是禁止扩展`open`、`override`、`redef`属性

### 泛型扩展

> 如果被扩展的类型是泛型类型, 有**两种**扩展语法可以对泛型类型扩展功能

> 一种是上面介绍的非泛型扩展
>
> 对于泛型类型, 非泛型扩展可以针对特定的 泛型实例化类型 进行扩展
>
> 在`extend`后允许是一个任意实例化完全的泛型类型
>
> 为这些类型增加的功能只有在类型完全匹配时才能使用
>
> ```cangjie
> extend Array<String> {}   // ok
> extend Array<Int> {}      // ok
> ```
>
> 在扩展中, 泛型类型的类型实参 必须符合 泛型类型定义处的约束要求, 否则会编译报错
>
> ```cangjie
> class Foo<T> where T <: ToString {}
>
> extend Foo<Int64> {}                  // ok
>
> class Bar {}
> extend Foo<Bar> {}                    // error
> ```

被扩展类型是泛型类型时, 如果扩展时指定所有的类型参数, 那么扩展方式就与上面介绍的方式保持一致

> 另一种是在`extend`后面引入泛型形参的泛型扩展
>
> 泛型扩展可以用来扩展 未实例化 或 未完全实例化 的泛型类型
>
> 在`extend`后允许声明泛型形参, 这些泛型形参必须被 直接或间接 使用在被扩展的泛型类型上
>
> 为这些类型增加的功能只有在 类型和约束完全匹配 时才能使用
>
> ```cangjie
> extend<T> Array<T> {}                     // ok
> ```
>
> 泛型扩展引入的泛型变元必须在被扩展类型中使用, 否则报未使用错误
>
> ```cangjie
> extend<T> Array<T> {}                 // ok
> extend<T> Array<Option<T>> {}         // ok
> extend<T> Array<Int64> {}             // error
> extend<T> Int64 <: Equatable<T> {     // error
>     ...
> }
> ```

如果被扩展类型未指定或未指定所有类型参数, 那么扩展语法就为:`extend<T> Array<T> {}`

要求`extend<T>`中的`T`, 必须要在被扩展类型中使用

> 不管是泛型扩展还是非泛型扩展, 对泛型类型扩展都不能重定义成员和重复实现接口

> 当泛型扩展的泛型变元被直接使用在被扩展类型的类型实参时, 对应的泛型变元会隐式引入被扩展类型定义时的泛型约束
>
> ```cangjie
> class Foo<T> where T <: ToString {}
>
> extend<T> Foo<T> {}                   // 隐式引入 T <: ToString
> ```

> 泛型扩展引入的泛型形参不能用于被扩展类型或被实现的接口, 否则会编译报错
>
> ```cangjie
> extend<T> T {} // error
> extend<T> Unit <: T {} // error
> ```
>
> 我们可以在泛型扩展中使用 **额外的泛型约束**
>
> 通过这种方式添加的成员或接口, 只有当该类型的实例在 满足 扩展的泛型约束 时才可以使用, 否则会报错
>
> ```cangjie
> class Foo<T> {
>     var item: T
>     init(it: T) {
>         item = it
>     }
> }
>
> interface Eq<T> {
>     func equals(other: T): Bool
> }
>
> extend<T> Foo<T> <: Eq<Foo<T>> where T <: Eq<T> {
>     public func equals(other: Foo<T>) {
>         item.equals(other.item)
>     }
> }
>
> class A {}
> class B <: Eq<B> {
>     public func equals(other: B) { true }
> }
>
> main() {
>     let a = Foo(A())
>     a.equals(a)                       // error, A 尚未实现 Eq
>     let b = Foo(B())
>     b.equals(b)                       // ok, B 实现了 Eq
> }
> ```

> **泛型类型不能重复实现同一接口**
>
> 对于同一个泛型类型, **无论它是否完全实例化, 都不能重复实现同一接口**
>
> 如果重复实现会在编译时报错
>
> ```cangjie
> interface I {}
> extend Array<String> <: I {}              // error, 不能重复实现同一个接口
> extend<T> Array<T> <: I {}                // error, 不能重复实现同一个接口
>
> interface Bar<T> {}
> extend Array<String> <: Bar<String> {}    // error, 不能重复实现同一个接口
> extend<T> Array<T> <: Bar<T> {}           // error, 不能重复实现同一个接口
> ```
>
> 对于同一个泛型类型, 无论它是否完全实例化, 都不能定义相同类型或相同签名的成员
>
> 如果重复定义会在编译时报错
>
> ```cangjie
> // case 1
> extend Array<String> {
>     func f() {}                           // error, 不能重复定义
> }
> extend<T> Array<T> {
>     func f() {}                           // error, 不能重复定义
> }
>
> // case 2
> extend Array<String> {
>     func g(a: String) {}                  // error, 不能重复定义
> }
> extend<T> Array<T> {
>     func g(a: T) {}                       // error, 不能重复定义
> }
>
> // case 3
> extend<T> Array<T> where T <: Int {
>     func g(a: T) {}                       // error, 不能重复定义
> }
> extend<V> Array<V> where V <: String {
>     func g(a: V) {}                       // error, 不能重复定义
> }
>
> // case 4
> extend Array<Int>  {
>     func g(a: Int) {}                     // ok
> }
> extend Array<String>  {
>     func g(a: String) {}                  // ok
> }
> ```

泛型扩展, 会自动引入 泛型类型的类型约束, 且可以额外添加约束

泛型扩展, 对于同一个泛型类型, 无论它是否完全实例化 都不能重复扩展同一接口, 更不能出现扩展函数重复定义的情况

这里的实例化, 指 类型参数

### 扩展的访问和遮盖

> 扩展的实例成员与类型定义处一样可以使用`this`, `this`的含义与类型定义中的保持一致
>
> 同样也可以省略`this`访问成员
>
> 扩展的实例成员不能使用`super`
>
> ```cangjie
> class A {
>     var v = 0
> }
>
> extend A {
>     func f() {
>         print(this.v)         // ok
>         print(v)              // ok
>     }
> }
> ```

扩展的实例成员可以使用`this`, 但不能使用`super`

> 扩展不能访问被扩展类型的`private`成员, 其它修饰符修饰的成员遵循可见性原则
>
> ```cangjie
> class A {
>     private var v1 = 0
>     protected var v2 = 0
> }
>
> extend A {
>     func f() {
>         print(v1)             // error
>         print(v2)             // ok
>     }
> }
> ```
>
> 扩展不允许遮盖被扩展类型的任何成员
>
> ```cangjie
> class A {
>     func f() {}
> }
>
> extend A {
>     func f() {}               // error
> }
> ```
>
> 扩展也不允许遮盖被扩展类型的其它扩展中已增加的任何成员
>
> ```cangjie
> class A {}
>
> extend A {
>     func f() {}
> }
>
> extend A {
>     func f() {}               // error
> }
> ```
>
> 在同一个`package`内对同一类型可以扩展任意多次

扩展不允许出现遮盖, 其实也是不会出现遮盖情况, 因为扩展的成员看作与原类型成员同一作用域层级

> 在扩展中可以直接使用(不加任何前缀修饰)其它对同一类型的扩展中的非`private`修饰的成员
>
> ```cangjie
> class Foo {}
>
> extend Foo {                  // OK
>     private func f() {}
>     func g() {}
> }
>
> extend Foo {                  // OK
>     func h() {
>         g()                   // OK
>         f()                   // Error
>     }
> }
> ```

> 扩展泛型类型时, 可以使用额外的泛型约束
>
> 泛型类型的扩展中能否直接使用其它对同一类型的扩展中的成员, 除了满足上述可访问性规则之外, 还需要满足以下约束规则:
>
> - 如果两个扩展的约束**相同**, 则两个扩展中可以直接使用对方的成员
>
> - 如果两个扩展的约束**不同**, 且两个扩展的约束**有包含关系**, 约束更严格的扩展 中可以直接使用 约束更宽松的扩展 中的成员, 反之, 则不可直接使用
>
> - 当两个扩展的约束不同时, 且两个约束**不存在包含关系**, 则两个扩展中 不可以直接使用对方的成员
>
> 示例: 假设对同一个类型`E<X>`的两个扩展分别为扩展 1 和扩展 2 , `X`的约束在扩展 1 中比扩展 2 中更严格, 那扩展 1 中可直接使用扩展2中的成员, 反之, 扩展 2 中不可直接使用扩展 1的成员
>
> ```cangjie
> // B <: A
> class E<X> {}
>
> interface I1 {
>     func f1(): Unit
> }
> interface I2 {
>     func f2(): Unit
> }
>
> extend<X> E<X> <: I1 where X <: B {       // 扩展 1
>     public func f1(): Unit {
>         f2()                              // OK
>     }
> }
>
> extend<X> E<X> <: I2 where X <: A   {     // 扩展 2
>     public func f2(): Unit {
>         f1()                              // Error
>     }
> }
> ```

非泛型扩展可以直接使用其他扩展中的非`private`成员

而泛型扩展, 因为可能存在类型约束, 所以各扩展成员之间的调用遵循一定的限制

### 扩展的继承

> 如果被扩展的类型是 **`class`**, 那么扩展的成员会被子类继承
>
> ```cangjie
> open class A {}
>
> extend A {
>     func f() {}
> }
>
> class B <: A {
>     func g() {
>         f()               // ok
>     }
> }
>
> main() {
>     let x = B()
>     x.f()                 // ok
> }
> ```
>
> 需要注意的是, 如果在父类中扩展了成员, 由于继承规则限制, 子类中就无法再定义同名成员, 同时也无法覆盖或重新实现(允许函数重载)
>
> ```cangjie
> open class A {}
>
> extend A {
>     func f() {}
>     func g() {}
> }
>
> class B <: A {
>     func f() {}               // error
>     override func g() {}      // error
> }
> ```
>
> 如果在同包内对同一个类型分别扩展实现父接口和子接口, 那么编译器会先检查实现父接口的扩展, 然后再检查实现子接口的扩展
>
> ```cangjie
> class Foo {}
>
> interface I1 {
>    func f() {}
> }
> interface I2 <: I1 {
>    func g() {}
> }
>
> extend Foo <: I1 {}           // first check
> extend Foo <: I2 {}           // second check
> ```

如果是对`class`类型的扩展, 那么扩展成员会被继承

但是不能被覆盖、重新实现, 因为扩展成员禁止被`open`修饰

对同一个类型进行扩展时, 如果 分别扩展实现**存在继承关系**的接口, 那么 会先检查父接口的函数, 完毕之后再检查子接口的函数

这样可以保证如果子接口覆盖了父接口的成员, 最终扩展实现 会采取子接口的成员

### 扩展的导入与导出

> `extend`前不允许有修饰符, 扩展只能与被扩展类型或接口一起导入导出

这个意思应该时, 不能单独导入或导出扩展, 而是扩展之后 导入或导出 被扩展的类型或接口

#### 直接扩展的导出

> 当直接扩展与被扩展类型的定义处在相同的`package`时, 如果被扩展类型是导出的, 扩展会与被扩展类型一起被导出, 否则扩展不会被导出
>
> ```cangjie
> package pkg1
>
> public class Foo {}
>
> extend Foo {
>     public func f() {}
> }
>
> ///////
>
> package pkg2
> import pkg1.*
>
> main() {
>     let a = Foo()
>     a.f()                 // ok
> }
> ```
>
> 当直接扩展与被扩展类型的定义处在不同的`package`时, 扩展永远不会被导出, 只能在当前`package`使用
>
> ```cangjie
> package pkg1
> public class Foo {}
>
> ///////
>
> package pkg2
> import pkg1.*
>
> extend Foo {
>     public func f() {}
> }
>
> func g() {
>     let a = Foo()
>     a.f()                 // ok
> }
>
> ///////
>
> package pkg3
> import pkg1.*
> import pkg2.*
>
> main() {
>     let a = Foo()
>     a.f()                 // error
> }
> ```

直接扩展要想被导出, 必须要保证直接扩展与被扩展类型在同一包中

#### 接口扩展的导出

> 当接口扩展与被扩展类型**在相同的包**时, 对于 外部可见类型 只可以扩展 同样外部可见的接口
>
> ```cangjie
> package pkg1
> public class Foo {}
>
> interface I1 {
>     func f(): Unit
> }
>
> extend Foo <: I1 {                    // error
>     public func f(): Unit {}
> }
>
> public interface I2 {
>     func g(): Unit
> }
>
> extend Foo <: I2 {                    // ok
>     public func g(): Unit {}
> }
>
> ///////
>
> package pkg2
> import pkg1.*
>
> main() {
>     let a = Foo()
>     a.g()                             // ok
> }
> ```
>
> 当接口扩展与被扩展类型**不在相同的包**时, 扩展的访问等级 与 相应接口相同
>
> 如果扩展多个接口, 扩展的访问等级是`public`当且仅当**所有接口**都是`public`的
>
> ```cangjie
> package pkg1
> public class Foo {}
>
> public class Bar {}
>
> ///////
>
> package pkg2
>
> import pkg1.*
>
> interface I1 {
>     func f(): Unit
> }
>
> public interface I2 {
>     func g(): Unit
> }
>
> extend Foo <: I1 & I2 {               // 不可外部访问
>     public func f(): Unit {}
>     public func g(): Unit {}
> }
>
> extend Bar <: I2 {                    // 可外部访问
>     public func g(): Unit {}
> }
> ```

对于接口扩展, 如果与被扩展类型定义在同一个包中, 限制`public`类型只能扩展实现`public`类型的接口

如果与被扩展类型不在同一个包中, 则扩展的访问等级与扩展实现的接口保持一致, 扩展多个接口时, 接口均为`public`时, 整个接口扩展才为`public`

### 导入扩展

> 扩展会与被扩展的类型和扩展实现的接口一起被导入, 不能指定导入扩展
>
> ```cangjie
> package pkg1
> public class Foo {}
>
> extend Foo {
>     public func f() {}
> }
>
> ///////
>
> package pkg2
> import pkg1.Foo               // import Foo
>
> main() {
>     let a = Foo()
>     a.f()                     // ok
> }
> ```
>
> 特别的, 由于扩展不支持遮盖任何被扩展类型的成员, 当导入的扩展发生重定义时将会报错
>
> ```cangjie
> package pkg1
> public class Foo {}
>
> ///////
>
> package pkg2
> import pkg1.Foo
>
> public interface I1 {
>     func f(): Unit {}
> }
>
> extend Foo <: I1 {}           // ok, 可外部访问
>
> ///////
>
> package pkg3
> import pkg1.Foo
>
> public interface I2 {
>     func f(): Unit {}
> }
>
> extend Foo <: I2 {}           // ok, 可外部访问
>
> ///////
>
> package pkg4
> import pkg1.Foo
> import pkg2.I1                // error
> import pkg3.I2                // error
> ```

最后一个例子, 当在`pkg4`中导入`pkg1.Foo`并同时导入`pkg2.I1`和`pkg3.I2`时, 会报错, 禁止遮蔽

从此例中可以得到一个关于接口扩展的语法规则:

当 在接口定义的包中 进行接口扩展时, 完成的扩展默认只能在接口的包中使用

即, 假设包`pkg1`中存在`class Foo`, 包`pkg2`中存在`interface I1`, 包`pkg3`中存在`interface I2`

如果在接口所在的包中, 对`Foo`进行接口扩展, 那么在其他包中导入`Foo`时, `Foo`的扩展并不能直接使用, 而是要 将需要使用的接口扩展的 接口 也导入到包中

即, **至少 被扩展类型、所扩展的接口 同时被导入, 才能访问对应的扩展成员**
