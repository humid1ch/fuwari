---
title: "仓颉文档阅读-语言规约VI: 类和接口(IV)"
published: 2025-10-11 09:33:26
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

## 类和接口

任何面向对象的编程语言, 基本都存在类这个概念, C++不例外, 仓颉当然也不例外

但, C++ 中不存在接口这个东西, 接口这个词, 在业务方面听起来不陌生, 但是语言中的接口具体情况并不是非常了解

### 覆盖、重载、遮盖、重定义

#### 覆盖

##### 覆盖的定义


> 子类型中定义父类型中已经存在的同名非抽象、具有`open`语义的实例函数时, 允许使用可选的`override`进行修饰以表示 是对父类型中同名函数的覆盖
>
> 函数的覆盖需要遵循以下规则:
>
> - 覆盖函数与被覆盖函数的函数名必须相同
>
> - 覆盖函数与被覆盖函数的参数列表必须相同
>
>     > 参数列表相同指函数的参数个数、参数类型相同
>
> - 覆盖函数的返回类型与被覆盖函数的返回类型相同或为其子类型
>
> - 同一个函数覆盖多个父作用域中出现的函数, 每个函数的覆盖规则由上面的其他规则确定
>
> 示例:
>
> - 类`C1`、`C2`, 接口`I`中的函数`f`, 参数参数列表相同, 返回类型相同, 满足覆盖的要求
>
> - 类`C1`与`C2`中的函数`f1`, 参数列表相同, 在`C1`中的返回类型是`Father`, 在`C2`中的返回类型是`Child`, 由于`Child`是`Father`的子类型, 因此满足覆盖的规则
>
> ```cangjie
> open class Father {}
> class Child <: Father {}
>
> open class C1 {
>     public open func f() {}
>     public open func f1(): Father { Father() }
> }
>
> interface I {
>     func f() {}
> }
>
> class C2 <: C1 & I {
>     public override func f() {}                       // OK
>     public override func f1(): Child { Child() }      // OK
> }
> ```
>
> 需要注意的一些细节包括:
>
> 1. 类中用`private`修饰的函数不会被继承, 无法在子类中访问
>
> 2. 静态函数不能覆盖实例函数
>

覆盖一定是出现在存在继承关系的两个类型中的

只能覆盖父类型中被`open`修饰的函数

覆盖要保证函数名和参数列表完全相同, 返回值可以是父类中函数返回值类型的子类型

仓颉中的覆盖, 其实类似于C++类中的虚函数重写, 可以用以实现动态多态

##### 覆盖的调用

> 如果子类型覆盖了父类型中的函数, 并且在程序中调用了此函数, 则编译器会根据 运行时对象指向的类型 来选择执行此函数的哪个版本
>
> 以下示例中, 在类`C1`定义了函数`f`, 其子类`C2`中覆盖了`f`, 同时定义了类型为`C1`的变量`a`与`b`, 并将`C1`的对象`myC1`赋给`a`, 将`C2`的对象`myC2`赋给`b`
>
> 通过`a`和`b`来调用函数`f`时, 会在运行时根据它们实际的类型来选择对应的函数
>
> ```cangjie
> open class C1 {
>     public open func f(): Unit {
>         return
>     }
> }
> class C2 <: C1 {
>     public override func f(): Unit {
>         return
>     }
> }
>
> var myC1 = C1()
> var myC2 = C2()
>
> // 将父类 C1 的对象赋值给 C1 类型的变量
> var a: C1 = myC1
> // 将子类 C2 的对象赋值给 C1 类型的变量
> var b: C1 = myC2
>
> // 根据运行时的对象类型调用 C1 的 f
> var c = a.f()
> // 根据运行时的对象类型调用 C2 的 f
> var d = b.f()
> ```

覆盖的调用, 其实可以实现动态多态调用

#### 重载

> 关于函数重载定义及调用的详细描述, 请参见[函数重载]章节
>
> 类和接口的 静态成员函数和实例成员函数之间 不允许重载
>
> 如果类或接口的 静态成员函数和实例成员函数(包括本类/接口定义的和从父类或父接口继承过来的成员函数)同名, 则报编译错误
>
> ```cangjie
> open class Base {}
> class Sub <: Base {}
> open class C{
>     static func foo(a: Base) {
>     }
> }
> open class B <: C {
>     func foo(a: Sub) {            // Error
>         C.foo(Sub())
>     }
> }
>
> class A <: B {
>     // 静态函数和实例函数之间 不能被重载
>     static func foo(a: Sub) {}    // Error
> }
> ```
>
> 重载决议时, 父类型和子类型中定义的函数当成同一作用域优先级处理

重载是发生在同一作用域的

静态成员函数和普通成员函数禁止同名, 因为禁止两者之间重载

#### 遮盖

> 子类型的成员不可遮盖父类型的成员
>
> 如果发生遮盖, 将编译报错
>
> 如下示例中, 类`C1`和`C2`都有同名实例变量`x`, 编译报错
>
> ```cangjie
> open class C1 {
>     let x = 1
> }
> class C2 <: C1 {
>     let x = 2     // error
> }
> ```

遮盖应该发生在不同作用域级别, 父子类中不存在遮盖

#### 重定义

##### 重定义函数的定义

> 子类型中定义父类型中已经存在的 同名非抽象静态函数 时, 允许使用可选的`redef`进行修饰以表示是对父类型中同名函数的重定义
>
> 函数的重定义需要遵循以下规则:
>
> - 函数与被重定义函数的函数名必须相同
>
> - 函数与被重定义函数的参数列表必须相同
>
>     > 参数列表相同指函数的参数个数、参数类型相同
>
> - 函数的返回类型与被重定义函数的返回类型相同或为其子类型
>
> - 如果被重定义函数为泛型函数, 则要求 重定义函数的 类型变元约束 比被实现函数更宽松或相同
>
> - 同一个函数 重定义 多个父作用域中出现的函数, 每个函数的覆盖规则由上面的其他规则确定

这里的重定义不是指重复定义, 而是指重新定义

重定义与覆盖的语法类似, 但 重定义主要是针对静态成员的概念

> 示例:
>
> - 类`C1`、`C2`, 接口`I`中的函数`f`, 参数参数列表相同, 返回类型相同, 满足重定义的要求
>
> - 类`C1`与`C2`中的函数`f1`, 参数列表相同, 在`C1`中的返回类型是`Father`, 在`C2`中的返回类型是`Child`, 由于`Child`是`Father`的子类型, 因此满足重定义的规则
>
> ```cangjie
> open class Father {}
> class Child <: Father {}
>
> open class C1 {
>     public static func f() {}
>     public static func f1(): Father { Father() }
> }
>
> interface I {
>     static func f() {}
> }
>
> class C2 <: C1 & I {
>     public redef static func f() {}                       // OK
>     public redef static func f1(): Child { Child() }      // OK
> ```

> 示例: 基类`Base`中的函数`f`和`g`为泛型函数, 子类`E`和`F`满足 重定义函数 比 被重定义函数的类型变元约束 更宽松或相同的要求, 编译成功; 子类`D`不满足要求, 则编译报错
>
> ```cangjie
> // C <: B <: A
> open class Base {
>     static func f<T>(a: T): T where T <: B {...}
>     static func g<T>(): T where T <: B {...}
> }
>
> class D <: Base  {
>     redef static func f<T>(a: T): T where T <: C {...}        // Error, 更严格的约束
>     redef static func g<T>(): T where T <: C {...}            // Error, 更严格的约束
> }
>
> class E <: Base  {
>     redef static func f<T>(a: T): T where T <: A {...}        // OK, 更宽松的约束
>     redef static func g<T>(): T where T <: A {...}            // OK, 更宽松的约束
> }
>
> class F <: Base  {
>     redef static func f<T>(a: T): T where T <: B {...}        // OK, 相同的约束
>     redef static func g<T>(): T where T <: B {...}            // OK, 相同的约束
> }
> ```
>
> `redef`修饰符不能用于修饰静态初始化器(因为静态初始化器不能被显式调用), 否则编译器会报告错误

##### 重定义函数的调用

> 如果子类型重定义了父类型中的函数, 并且在程序中调用了此函数, 则编译器会根据类型来选择执行此函数的哪个版本
>
> 以下示例中, 在类`C1`定义了函数`f`, 其子类`C2`中重定义了`f`
>
> 通过`C1`和`C2`来调用函数`f`时, 会在编译时根据它们的类型来选择对应的函数
>
> ```cangjie
> open class C1 {
>     static func f(): Unit {
>         return
>     }
> }
> class C2 <: C1 {
>     redef static func f(): Unit {
>         return
>     }
> }
>
> // 调用 C1 的 f
> var c = C1.f()
> // 调用 C2 的 f
> var d = C2.f()
> ```

重定义其实不存在根据类型动态派发执行函数的情况, 因为只能通过类型调用, 此时类型是确定好的

重定义函数的调用, 是在编译期就确定好的

### 访问控制等级限制

> 根据访问修饰符所允许的可访问范围大小, 规定访问修饰符等级如下:
>
> - 类型内部访问修饰符等级为`public`>`protected`>`default`>`private`
>
> 在此等级下, 对跨访问等级的行为规定如下:
>
> - 在子类继承父类时, 子类的实例成员函数 覆盖 父类的实例成员函数, 或者子类的静态成员函数 重定义 父类的静态成员函数, 子类成员函数的可访问等级不得修改为小于父类成员函数的访问等级;
>
> - 在类型实现接口时, 类型的成员函数 实现了 接口中的抽象函数, 类型的成员函数的可访问等级不得修改为小于抽象函数的访问等级
>
> 以下是访问等级限制的代码示例:
>
> ```cangjie
> open class A {
>     protected open func f() {}
> }
> interface I {
>     func m() {}                       // 默认 public
> }
> class C <: A & I {
>     private override func f() {}      // Error: 覆盖函数的访问控制 低于 被覆盖函数
>     protected func m() {}             // Error: 实现抽象函数的函数的访问控制 低于 抽象函数
> }
> ```

我想, 如果允许子类降低访问控制等级, 那么通过父类调用子类成员函数这一多态行为, 就会成为非法行为

### 非顶层作用域的访问修饰符

> 在修饰非顶层成员时不同访问修饰符的语义如下:
>
> - `private`表示仅当前类型或扩展定义内可见
>
> - `internal`表示仅当前包及子包(包括子包的子包)内可见
>
> - `protected`表示当前`module`及当前类的子类可见
>
> - `public`表示`module`内外均可见
>
> | Type/Extend | Package & Sub-Packages | Module & Sub-Classes | All Packages |
> | ----------- | ---------------------- | -------------------- | ------------ |
> | `private` | Y | N | N | N |
> | `internal` | Y | Y | N | N |
> | `protected` | Y | Y | Y | N |
> | `public` | Y | Y | Y | Y |
>
> 类型成员的访问修饰符可以不同于类型本身
>
> 除接口外类型成员的默认修饰符(默认修饰符是指在省略情况下的修饰符语义, 这些默认修饰符也允许显式写出)是`internal`
>
> 接口中的成员函数和属性不可以写访问修饰符, 它们的访问级别等同于`public`

因为还没有了解过 仓颉中的`module`和`package`, 所以暂时先知道这些内容

### 泛型在类和接口中的使用限制

#### 实例化类型导致函数签名重复

> 在定义泛型类时, 由于函数是可以重载的, 所以下面的`C1`类与成员函数的定义是合法的
>
> ```cangjie
> open class C1<T> {
>     public func c1(a: Int32) {}       // ok
>     public func c1(a: T) {}           // ok
> }
>
> interface I1<T> {
>     func i1(a: Int32): Unit           // ok
>     func i1(a: T): Unit               // ok
> }
>
> var a = C1<Int32>()                   // error
> class C2 <: C1<Int32> {...}           // error
> var b: I1<Int32>                      // error
> class C3 <: I1<Int32> {...}           // error
> ```
>
> 但是当类`C1<T>`需要被实例化为`C1<Int32>`时, 会出现两个函数签名完全相同的情形, 此时, 在使用到`C1<Int32>`类型位置报错
>
> 同理, 当接口`I1<Int32>`需要被实例化时, 也会造成其部声明的两个函数签名重复, 此时也会报错

总之, 需要避免在实例化类型参数之后, 成员出现签名重复的情况

#### 类与接口的泛型成员函数

> 在仓颉语言中, 非静态抽象函数与类中被`open`关键字修饰的函数 **不能声明泛型参数**
>
> ```cangjie
> interface Bar {
>     func bar<T>(a: T): Unit               // error
> }
>
> abstract class Foo {
>     func foo<T>(a: T): Unit               // error
>     public open func goo<T>(a: T) { }     // error
> }
>
> class Boo {
>     public open func Boo<T>(a: T) { }     // error
> }
> ```
>
> 更多可以见第 9 章泛型 7.2 小节
