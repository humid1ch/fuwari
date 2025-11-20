---
title: "仓颉文档阅读-开发指南VII: 类和接口(III) - 接口的定义、继承与实现"
published: 2025-11-20 14:23:31
description: '仓颉文档阅读的开发指南部分, 本篇文章介绍仓颉语言的接口的相关内容'
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

## 类和接口

### 接口

> 接口用来定义一个抽象类型, 它**不包含数据**, 但可以定义类型的行为
> 
> 一个类型如果声明实现某接口, 并且**实现了该接口中所有的成员**, 就被称为**实现了该接口**
> 
> 接口的成员可以包含: 
> 
> - 成员函数
> 
> - 操作符重载函数
> 
> - 成员属性
> 
> 这些成员都是抽象的, 要求实现类型必须拥有对应的成员实现

如果你理解了类, 那么 简单了解接口, 就是禁止包含成员变量的、本身禁止创建实例的可继承的类

即 接口只能有成员函数、成员属性, 无法单独创建接口实例, 默认可继承

其他类可以实现接口的成员函数, 接口也可以继承接口

#### 接口定义

> 一个简单的接口定义如下: 
> 
> ```cangjie
> interface I { // 'open' 修饰符是可选的
>     func f(): Unit
> }
> ```
> 
> 接口使用关键字`interface`声明, 其后是接口的标识符`I`和接口的成员
> 
> 接口成员可被`open`修饰符修饰, 并且 **`open`修饰符是可选的**
> 
> 当接口`I`声明了一个成员函数`f`之后, 要为一个类型实现 I 时, 就必须在该类型中实现一个对应的`f`函数
> 
> 因为`interface`默认具有`open`语义, 所以`interface`定义时的 **`open`修饰符是可选的**
> 
> 如下面的代码所示, 定义了一个`class Foo`, 使用`Foo <: I`的形式声明了`Foo`实现`I`接口
> 
> 在`Foo`中**必须包含`I`声明的所有成员的实现**, 即需要定义一个相同类型的`f`, 否则会由于没有实现接口而编译报错
> 
> ```cangjie
> class Foo <: I {
>     public func f(): Unit {
>         println("Foo")
>     }
> }
> 
> main() {
>     let a = Foo()
>     let b: I = a
>     b.f() // "Foo"
> }
> ```
> 
> 当某个类型实现了某个接口之后, 该类型就会**成为该接口的子类型**
> 
> 对于上面的例子, `Foo`是`I`的子类型, 因此任何一个`Foo`类型的实例, 都可以当作`I`类型的实例使用
> 
> 在`main`中将一个`Foo`类型的变量`a`, 赋值给一个`I`类型的变量`b`
> 
> 然后再调用`b`中的函数`f`, 就会打印出`Foo`实现的`f`版本
> 
> 程序的输出结果为: 
> 
> ```text
> Foo
> ```

接口使用`interface`作为定义关键字 

语法为: 

```cangjie
interface IIdent {
    public func functiong(): Unit {}
}
```

接口默认拥有`open`语义, 成员函数和成员属性也默认拥有`open`语义

其他类型要实现接口, 语法为, `class Sample <: IIdent {}`, 同时 **需要包含目标接口的所有成员的实现**

实现了接口的类型, 就是此接口的子类型, 所以可以赋值给 接口类型变量使用

> `interface`也可以使用`sealed`修饰符表示 **只能在`interface`定义所在的包内** 继承、实现或扩展该`interface`
> 
> `sealed`已经蕴含了`public/open`的语义, 因此定义`sealed interface`时若提供`public/open`修饰符, 编译器将会告警
> 
> 继承`sealed`接口的子接口或实现`sealed`接口的抽象类, 仍可 被`sealed`修饰 或 不使用`sealed`修饰
> 
> 若`sealed`接口的子接口被`public`修饰, 且不被`sealed`修饰, 则其子接口可在包外被继承、实现或扩展
> 
> 继承、实现`sealed`接口的类型 可以不被`public`修饰
> 
> ```cangjie
> package A
> public interface I1 {}
> sealed interface I2 {}                // OK
> public sealed interface I3 {}         // Warning, 冗余修饰符, 'sealed' 含有 'public' 语义
> sealed open interface I4 {}           // Warning, 冗余修饰符, 'sealed' 含有 'open' 语义
> 
> class C1 <: I1 {}
> public open class C2 <: I1 {}
> sealed abstract class C3 <: I2 {}
> extend Int64 <: I2 {}
> ```
> 
> ```cangjie
> package B
> import A.*
> 
> class S1 <: I1 {}                     // OK
> class S2 <: I2 {}                     // Error, I2 在 package A 中被 'sealed' 修饰 , 不能在这里实现
> ```

接口也可以被`sealed`修饰, 同样表示只能在本包中被继承、实现、扩展

> 通过接口的这种约束能力, 可以**对一系列的类型约定共同的功能**, 达到对功能进行抽象的目的
> 
> 例如下面的代码, 可以定义一个`Flyable`接口, 并且让其他具有`Flyable`属性的类实现它
> 
> ```cangjie
> interface Flyable {
>     func fly(): Unit
> }
> 
> class Bird <: Flyable {
>     public func fly(): Unit {
>         println("Bird flying")
>     }
> }
> 
> class Bat <: Flyable {
>     public func fly(): Unit {
>         println("Bat flying")
>     }
> }
> 
> class Airplane <: Flyable {
>     public func fly(): Unit {
>         println("Airplane flying")
>     }
> }
> 
> func fly(item: Flyable): Unit {
>     item.fly()
> }
> 
> main() {
>     let bird = Bird()
>     let bat = Bat()
>     let airplane = Airplane()
>     fly(bird)
>     fly(bat)
>     fly(airplane)
> }
> ```
> 
> 编译并执行上面的代码, 会看到如下输出: 
> 
> ```text
> Bird flying
> Bat flying
> Airplane flying
> ```

接口, 一般就是将一系列实体共同的功能 约定、抽象出来, 定义为一个接口, 然后供其他类型定义时使用

文档中, 举了个例子 将可飞定义为一个接口, 可以给`bird``bat``airplane`定义使用

> 接口的成员可以是实例的或者静态的, 以上的例子已经展示过实例成员函数的作用, 接下来来看看静态成员函数的作用
> 
> 静态成员函数和实例成员函数类似, 都要求实现类型提供实现
> 
> 例如下面的例子, 定义了一个`NamedType`接口, 这个接口含有一个静态成员函数`typename`用来获得每个类型的字符串名称
> 
> 这样其他类型在实现`NamedType`接口时就必须实现`typename`函数, 之后就可以安全地在`NamedType`的子类型上获得类型的名称
> 
> ```cangjie
> interface NamedType {
>     static func typename(): String
> }
> 
> class A <: NamedType {
>     public static func typename(): String {
>         "A"
>     }
> }
> 
> class B <: NamedType {
>     public static func typename(): String {
>         "B"
>     }
> }
> 
> main() {
>     println("the type is ${ A.typename() }")
>     println("the type is ${ B.typename() }")
> }
> ```
> 
> 程序输出结果为: 
> 
> ```text
> the type is A
> the type is B
> ```
> 
> 接口中的静态成员函数(或属性)**可以没有默认实现**
> 
> 当其没有默认实现时, 将无法通过接口类型名对其进行访问
> 
> 例如下面的代码, 直接访问`NamedType`的`typename`函数会发生编译报错, 因为`NamedType`不具有`typename`函数的实现
> 
> ```text
> interface NamedType {
>     static func typename(): String
> }
> main() {
>     NamedType.typename() // Error
> }
> ```
> 
> 接口中的静态成员函数(或属性)也**可以拥有默认实现**, 当另一个类型继承拥有默认静态函数(或属性)实现的接口时, 该类型可以不再实现这个静态成员函数(或属性), 该函数(或属性)可以通过接口名和该类型名直接访问
> 
> 如下用例, `NamedType`的成员函数`typename`拥有默认实现, 且在`A`中都可以不用再重新实现它, 同时, 也可以通过接口名和该类型名对其进行直接访问
> 
> ```cangjie
> interface NamedType {
>     static func typename(): String {
>         "interface NamedType"
>     }
> }
> 
> class A <: NamedType {}
> 
> main() {
>     println(NamedType.typename())
>     println(A.typename())
>     0
> }
> ```
> 
> 程序输出结果为: 
> 
> ```text
> interface NamedType
> interface NamedType
> ```

仓颉接口, 除了实例成员函数, 还可以拥有静态成员函数

静态成员函数 **可以拥有默认实现**, 也可以没有默认实现

如果没有默认实现, 那么 要实现此接口的其他类型, 就必须要实现此静态成员函数

如果接口的静态成员变量已经有了默认实现, 那么 要实现此接口的其他类型, **不用必须实现此静态成员函数**

其实不止静态成员函数, **实例成员函数也可以拥有默认实现**J 并不一定必须是抽象的、没有函数体的

接口中有默认实现的静态成员函数, 与类的静态成员函数一样, 可以直接通过接口名(类名)调用此静态成员函数

> 通常会通过泛型约束, 在泛型函数中使用这类静态成员
> 
> 例如下面的`printTypeName`函数, 当约束泛型变元`T`是`NamedType`的子类型时, 需要保证`T`的实例化类型中所有的静态成员函数(或属性)都必须拥有实现, 以保证可以使用`T.typename`的方式访问泛型变元的实现, 达到了对静态成员抽象的目的(详见 [泛型]() )
> 
> ```cangjie
> interface NamedType {
>     static func typename(): String
> }
> 
> interface I <: NamedType {
>     static func typename(): String {
>         f()
>     }
>     static func f(): String
> }
> 
> class A <: NamedType {
>     public static func typename(): String {
>         "A"
>     }
> }
> 
> class B <: NamedType {
>     public static func typename(): String {
>         "B"
>     }
> }
> 
> func printTypeName<T>() where T <: NamedType {
>     println("the type is ${ T.typename() }")
> }
> 
> main() {
>     printTypeName<A>()        // Ok
>     printTypeName<B>()        // Ok
>     printTypeName<I>()        // Error, 'I' 必须实现所有静态函数  否则, 会调用未实现的 'f', 出现问题
> }
> ```

文档中提道, 接口的静态成员函数, 通常通过泛型约束在泛型函数中使用

> 接口中可以定义 **泛型实例成员函数** 或 **泛型静态成员函数**, 与非泛型函数一样具有`open`语义
> 
> ```cangjie
> import std.collection.*
> 
> interface M {
>     func foo<T>(a: T): T
>     static func toString<T>(b: ArrayList<T>): String where T <: ToString
> }
> class C <: M {
>     public func foo<S>(a: S): S { // implements M::foo, names of generic parameters do not matter
>         a
>     }
>     public static func toString<T>(b: ArrayList<T>) where T <: ToString {
>         var res = ""
>         for (s in b) {
>             res += s.toString()
>         }
>         res
>     }
> }
> ```
> 
> 需要注意的是, **接口的成员默认就被`public`修饰**, 不可以声明额外的访问控制修饰符, 同时也要求 **实现类型必须使用`public`实现**
> 
> ```cangjie
> interface I {
>     func f(): Unit
> }
> 
> open class C <: I {
>     protected func f() {} // Compiler Error, f needs to be public semantics
> }
> ```

仓颉接口中, 还可以定义泛型成员, 特性当然与普通成员一样

但要注意: 

**接口的成员默认是`public`的, 且 只能是`public`, 不能显式使用其他可访问性修饰符修饰, 且 要求实现时 也必须`public`修饰**

#### 接口继承与接口实现

> 当想为一个类型实现多个接口, 可以在声明处使用`&`分隔多个接口, 实现的接口之间没有顺序要求
> 
> 例如下面的例子, 可以让`MyInt`同时实现`Addable`和`Subtractable`两个接口
> 
> ```cangjie
> interface Addable {
>     func add(other: Int64): Int64
> }
> 
> interface Subtractable {
>     func sub(other: Int64): Int64
> }
> 
> class MyInt <: Addable & Subtractable {
>     var value = 0
>     public func add(other: Int64): Int64 {
>         value + other
>     }
>     public func sub(other: Int64): Int64 {
>         value - other
>     }
> }
> ```

其他类型可以一次性实现多个接口, 需要实现的多个接口 在声明接口实现时 **使用`&`连接起来**

而且, 如果是`class`, 类的继承和接口的实现是可以同时存在的, 但类的继承必须要声明为`<:`之后的首个类型, 之后可以使用`&`连接要实现的接口, 且只能是接口(仓颉`class`只允许单继承):

```cangjie
open class ClassBase {}

interface I1 {}

interface I2 {}

class ClassSuper <: ClassBase & I1 & I2 {}
// 类的继承 与 接口实现 共存
```

继承必须要声明为`<:`之后的首个类型, 之后使用`&`连接实现的接口(顺序无关)

> **接口可以继承一个或多个接口, 但不能继承类**
> 
> 与此同时, 接口继承的时候可以添加新的接口成员
> 
> 例如下面的例子, `Calculable`接口继承了`Addable`和`Subtractable`两个接口, 并且增加了乘除两种运算符重载
> 
> ```cangjie
> interface Addable {
>     func add(other: Int64): Int64
> }
> 
> interface Subtractable {
>     func sub(other: Int64): Int64
> }
> 
> interface Calculable <: Addable & Subtractable {
>     func mul(other: Int64): Int64
>     func div(other: Int64): Int64
> }
> ```
> 
> 这样实现类型实现`Calculable`接口时就必须同时实现加减乘除四种运算符重载, 不能缺少任何一个成员
> 
> ```cangjie
> class MyInt <: Calculable {
>     var value = 0
>     public func add(other: Int64): Int64 {
>         value + other
>     }
>     public func sub(other: Int64): Int64 {
>         value - other
>     }
>     public func mul(other: Int64): Int64 {
>         value * other
>     }
>     public func div(other: Int64): Int64 {
>         value / other
>     }
> }
> ```
> 
> `MyInt`实现`Calculable`的同时, 也同时实现了`Calculable`继承的所有接口, 因此`MyInt`也实现了`Addable`和`Subtractable`, 即同时是它们的子类型
> 
> ```cangjie
> main() {
>     let myInt = MyInt()
>     let add: Addable = myInt
>     let sub: Subtractable = myInt
>     let calc: Calculable = myInt
> }
> ```
> 
> 对于`interface`的继承, 子接口 如果继承了 父接口中有默认实现的 函数或属性, 则在子接口中 **不允许仅写此函数或属性的声明(即没有默认实现)**, 而是**必须要给出新的默认实现**, 并且函数定义前的`override`修饰符(或`redef`修饰符)是可选的
> 
> 子接口 如果继承了 父接口中没有默认实现的函数或属性, 则在子接口中允许仅写此函数或属性的声明(当然也允许定义默认实现), 并且函数声明或定义前的`override`修饰符(或`redef`修饰符)是可选的
> 
> ```cangjie
> interface I1 {
>    func f(a: Int64) {
>         a
>    }
>    static func g(a: Int64) {
>         a
>    }
>    func f1(a: Int64): Unit
>    static func g1(a: Int64): Unit
> }
> 
> interface I2 <: I1 {
>     /* 'override' 是可选的 */ func f(a: Int64) {
>        a + 1
>     }
>     override func f(a: Int32) {}                          // Error, 重写函数 'f', 但 在其父类型中没有 可以重写的函数
>     static /* 'redef' 是可选的 */ func g(a: Int64) {
>        a + 1
>     }
>     /* 'override' 是可选的 */ func f1(a: Int64): Unit {}
>     static /* 'redef' 是可选的 */ func g1(a: Int64): Unit {}
> }
> ```

如果其他类型要实现接口, 那么在`<:`之后声明接口就行了

而, 如果 **定义接口时, 在`<:`之后声明其他接口, 此时就叫做接口的继承**

接口的继承:

1. 如果父接口的成员函数、属性没有默认实现
   
    子接口对继承的成员函数、属性, 也可以不默认实现, 可以只声明

2. 如果父接口的成员函数、属性**有默认实现**

    子接口对继承的成员函数、属性:

    1. 要不就干脆不做任何显式声明, 此时 会直接继承父接口的默认实现

    2. 但不能只做声明而不提供默认实现, 即 如果子接口要显式声明, 就必须提供默认实现

3. 子接口是可以定义自己的成员函数、属性的

4. 接口的继承是可以多继承的, 而仓颉类 只允许单继承

5. 子接口在覆盖、重定义父类的成员函数时, 也可以省略`override`和`redef`修饰符

接口允许多继承, 因为接口成员实际都是抽象语义, 且接口不允许成员变量存在, 不会造成语义冗余和冲突

##### 接口实现的要求

> 仓颉除`Tuple`、`VArray`和函数外的其他类型都可以实现接口
> 
> 一个类型实现接口有三种途径: 
> 
> 1. 在定义类型时就声明实现接口, 在以上的内容中已经见过相关例子
> 
> 2. 通过扩展实现接口, 这种方式详见 [扩展]()
> 
> 3. 由语言内置实现, 具体详见《仓颉编程语言库 API》相关文档
> 
> 实现类型声明实现接口时, 需要实现接口中要求的所有成员, 为此需要满足下面一些规则:
> 
> 1. 对于成员函数 和 操作符重载函数, 要求实现类型提供的函数实现与接口对应的函数名称相同、参数列表相同、返回类型相同
> 
> 2. 对于成员属性, 要求是否被`mut`修饰保持一致, 并且属性的类型相同
> 
> 所以大部分情况都如同上面的例子, 需要让实现类型中包含与接口要求的一样的成员的实现
> 
> 但有个地方是个例外, 如果 接口中的 成员函数 或 操作符重载函数 的**返回值类型是`class`类型**, 那么**允许实现函数的返回类型是其子类型**
> 
> 例如下面这个例子, `I`中的`f`返回类型是一个`class`类型`Base`, 因此`C`中实现的`f`返回类型可以是`Base`的子类型`Sub`
> 
> ```cangjie
> open class Base {}
> class Sub <: Base {}
> 
> interface I {
>     func f(): Base
> }
> 
> class C <: I {
>     public func f(): Sub {
>         Sub()
>     }
> }
> ```

仓颉对类型实现扩展时 也可以实现接口

实现接口的成员函数时, 绝大多数情况下需要保证 函数实现与接口对应的成员函数 **同名、同形参列表、同返回值类型**, 但 如果返回值类型是`class`类型, 则实现时 可以改为其子类型

> 除此以外, 接口的成员还可以提供默认实现
> 
> 例如下面的代码中, `SayHi`中的`say`拥有默认实现, 因此`A`实现`SayHi`时可以继承`say`的实现, 而`B`也可以选择提供自己的`say`实现
> 
> ```cangjie
> interface SayHi {
>     func say() {
>         "hi"
>     }
> }
> 
> class A <: SayHi {}
> 
> class B <: SayHi {
>     public func say() {
>         "hi, B"
>     }
> }
> ```
> 
> 特别地, 如果一个类型在实现多个接口时, 多个接口中包含同一个成员的默认实现, 这时会发生多重继承的冲突, 语言无法选择最适合的实现, 因此这时接口中的默认实现也会失效, 需要实现类型提供自己的实现
> 
> 例如下面的例子, `SayHi`和`SayHello`中都包含了`say`的实现, `Foo`在实现这两个接口时就必须提供自己的实现, 否则会出现编译错误
> 
> ```cangjie
> interface SayHi {
>     func say() {
>         "hi"
>     }
> }
> 
> interface SayHello {
>     func say() {
>         "hello"
>     }
> }
> 
> class Foo <: SayHi & SayHello {
>     public func say() {
>         "Foo"
>     }
> }
> ```

仓颉接口的成员是可以拥有默认实现的

但因为接口允许多继承, 避免不了 不同父接口中可能存在 同名、同参数列表、同返回值类型的成员

此时如果不同父接口的此成员 **都有默认实现**, 那么 子接口就**必须对此函数提供自己的默认实现**, 否则会因为无法决议出现编译报错

如果不同父接口的此成员 只有一个默认实现, 那么还是遵循正常的继承规则, 要不就不显式声明 直接继承唯一的默认实现, 要不就必须提供自己的默认实现

> `struct`、`enum`和`class`在实现接口时, 函数或属性定义前的`override`修饰符(或`redef`修饰符)是可选的, 无论接口中的函数或属性是否存在默认实现
> 
> ```cangjie
> interface I {
>     func foo(): Int64 {
>         return 0
>     }
> }
> enum E <: I{
>     elem
>     public override func foo(): Int64 {
>         return 1
>     }
> }
> struct S <: I {
>     public override func foo(): Int64 {
>         return 1
>     }
> }
> ```

好像无论是类`class`还是接口`interface`, 继承或者实现, **`override`和`redef`都是可选的**

#### `Any`类型

> `Any`类型是一个内置的接口, 它的定义如下: 
> 
> ```cangjie
> interface Any {}
> ```
> 
> 仓颉中所有接口都默认继承它, 所有非接口类型都默认实现它, 因此所有类型都可以作为`Any`类型的子类型使用
> 
> 如下面的代码, 可以将一系列不同类型的变量赋值给`Any`类型的变量
> 
> ```cangjie
> main() {
>     var any: Any = 1
>     any = 2.0
>     any = "hello, world!"
> }
> ```

在阅读 类 的文档时, 提到: 仓颉拥有一个`Object`类 不拥有任何成员, 它是所有`class`的父类, 没有显式指定继承的类 默认继承`Object`

此时, 接口也存在一个`Any`接口, 同样不拥有任何成员, 但他比`class Object`使用更广

`interface Any`, 所有接口都继承它, 所有类型都实现它, 包括任意的`class`

所以, `calss`不仅一定拥有一个父类叫`Object`, 而且一定是实现了接口`Any`

即, 也可以说仓颉中所有类型都是`interface Any`的子类型

所以 所有值都可以赋值给`Any`类型变量, 包括`Unit`类型字面量`()`

