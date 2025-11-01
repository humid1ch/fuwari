---
title: "仓颉文档阅读-语言规约VI: 类和接口(III)"
published: 2025-10-10 15:02:22
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

### 接口

> 接口用来定义一个抽象类型, 它不包含数据, 但可以定义类型的行为
>
> 一个类型如果声明实现某接口, 并且实现了该接口中所有的成员, 就被称为实现了该接口
>
> 接口的成员可以包含实例成员函数、静态成员函数、操作符重载函数、实例成员属性和静态成员属性, 这些成员都是抽象的, 但函数和属性可以定义默认实现

仓颉中的接口, 像是一个只有成员函数、成员属性、操作符重载函数的抽象类, 不包含成员变量

C++中不严格存在接口的概念

#### 接口定义

##### 接口定义的语法

> 接口的定义使用`interface`关键字, 接口定义依次为: 可缺省的修饰符、`interface`关键字、接口名、可选的类型参数、是否指定父接口、可选的泛型约束、接口体的定义
>
> 以下示例是一些接口定义:
>
> ```cangjie
> interface I1 {}
> public interface I2<T> {}
> public interface I3<U> {}
> public interface I4<V> <: I2<V> & I3<Int32> {}
> ```
> 接口定义的语法如下:
>
> ```
> interfaceDefinition
>     : interfaceModifierList? 'interface' identifier
>     typeParameters?
>     ('<:' superInterfaces)?
>     genericConstraints?
>     interfaceBody
>     ;
> ```
>
> 接口只可以定义在`top level`
>
> 接口体的`{}`不允许省略
>
> ```cangjie
> interface I {}		// {} 不允许被省略
> ```

接口的定义语法, 除了关键词是`interface`之外, 基本与 类的定义语法保持一致

##### 接口的修饰符

> 在此小节中, 我们将介绍定义接口可以使用的修饰符

###### 访问修饰符

> 访问修饰符: 默认, 即不使用访问修饰符, 表示只能在包内部访问
>
> 也可以使用`public`修饰符表示包外部可访问
>
> ```cangjie
> public interface I {}     // 可以在包外被访问
> interface I2 {}           // 只能在包内访问
> ```

###### 继承性修饰符

> 继承修饰符: 默认, 即不使用继承修饰符即可表示接口具有`open`修饰的语义, 能在任何可以访问`interface`的位置继承、实现或扩展`interface`
>
> 当然, 也可以显式地使用`open`修饰符修饰; 也可以使用`sealed`修饰符表示只能在`interface`定义所在的包内继承、实现或扩展该`interface`
>
> `sealed`已经蕴含了`public`的语义, 所以定义`sealed interface`时的`public`修饰符是可选的
>
> 继承`sealed`接口的子接口或实现`sealed`接口的类仍可被`sealed`修饰或不使用`sealed`修饰
>
> 若`sealed`接口的子接口被`public`修饰, 且不被`sealed`修饰, 则其子接口可在包外被继承、实现或扩展;
>
> 继承、实现`sealed`接口的类型可以不被`public`修饰
>
> ```cangjie
> // package A
> package A
> public sealed interface I1 {}     // OK
> sealed interface I2  {}           // OK, 在`sealed`被使用时,`public`是可选的
> open interface I3 {}              // OK
> interface I4  {}                  // OK,`open`是可选的
>
> class C1 <: I1 {}                 // OK
> public open class C2 <: I1 {}     // OK
> public sealed class C3 <: I1 {}   // OK
> extend Int64 <: I1 {}             // OK
>
> // package B
> package B
> import A.*
>
> class S1 <: C2 {}                 // OK
> class S2 <: C3 {}                 // Error, C3 是 sealed class, 无法在这里继承
> ```

再次遇到`sealed`, 仓颉中, `sealed`用于**限制**`class`、`interface`和`struct`的包外**可继承性**

而`public`则是赋予他们包外**可访问性**, 这两个是不冲突的

<Info>

仓颉中, `interface`与`interface`之间, 为**继承关系**, 即 存在`interface2`继承自`interface1`

而`interface`与`class`或`struct`之间, 可以为**实现关系**, 即 存在`class`或`struct`实现了`interface`, 也可以叫做继承

</Info>

#### 接口成员

> 接口的成员包括:
>
> -  在接口体中声明(即在`{}`中声明)的成员: 静态成员函数、实例成员函数、操作符重载函数、静态成员属性和实例成员属性
>
> -  从其他接口继承而来的成员
>
>     如以下示例中, I2的成员将包括I1中允许被继承的成员
>
>     ```cangjie
>     interface I1 {
>        func f(): Unit
>     }
>     interface I2 <: I1 {}
>     ```
>
> 接口成员的 BNF 如下:
>
> ```
> interfaceMemberDeclaration
>        : (functionDefinition|macroExpression|propertyDefinition) end*
>        ;
> ```

接口的成员只包括 函数或属性成员, 不存在成员变量

##### 接口中的函数

###### 接口中函数的声明与定义

> 接口中可以包含实例成员函数和静态成员函数, 这些函数与普通实例成员函数和静态成员函数的编写方式一样, 但可以没有函数实现, 这些函数被称为抽象函数
>
> 对于拥有实现的抽象函数, 我们称该函数拥有默认实现
>
> 以下是包含抽象函数的示例:
>
> ```cangjie
> interface MyInterface {
>     func f1(): Unit               // 不包含默认实现
>     static func f2(): Unit        // 不包含默认实现
>
>     func f3(): Unit {             // 包含默认实现
>         return
>     }
>     static func f4(): Unit {      // 包含默认实现
>         return
>     }
> }
> ```
>
> 抽象函数可以拥有命名参数, 但不能拥有参数默认值
>
> ```cangjie
> interface MyInterface {
>     func f1(a!: Int64): Unit      // OK
>     func f2(a!: Int64 = 1): Unit  // Error, 不能有参数默认值
> }
> ```

仓颉中, 接口的静态成员函数也可以不实现函数体, 为 抽象静态成员函数

接口的成员函数可以有命名形参, 但不能存在默认值

###### 接口中函数和属性的修饰符

> 接口中定义的函数或属性已经蕴含`public`语义, 使用`public`修饰会产生编译告警
>
> 不允许使用`protected`、`internal`和`private`修饰符修饰接口中定义的函数或属性
>
> 以下是错误的示例:
>
> ```cangjie
> interface MyInterface {
>     public func f1(): Unit            // 不能使用访问修饰符
>     private static func f2(): Unit    // 不能使用访问修饰符
> }
> ```
>
> 使用`static`修饰的函数被称为静态成员函数, 可以没有函数体
>
> 没有函数体的`static`函数不能直接使用接口类型调用, 拥有函数体的`static`函数可以使用接口类型调用
>
> 当使用`interface`类型名直接调用其静态成员函数时, 如果这个函数里面直接或间接调用了接口中(自己或其它接口)没有实现的其它静态函数, 则编译报错
>
> ```cangjie
> interface I {
>     static func f1(): Unit
>     static func f2(): Unit {}
>     static func f3(): Unit {
>         f1()
>     }
> }
>
> main() {
>     I.f1()        // Error, 不能通过接口类型直接调用
>     I.f2()        // OK
>     I.f3()        // Error, f1 未实现
> }
>
> ```
> 接口中的实例成员函数默认具有`open`的语义
>
> 在接口中定义实例成员函数时, `open`修饰符是可选的
>
> ```cangjie
> interface I {
>     open func foo1() {}       // ok
>     func foo2() {}            // ok
> }
> ```
>
> 使用`mut`修饰的函数是一种特殊的实例成员函数, 可以用于抽象`struct`类型的可变行为
>
> ```cangjie
> interface I {
>     mut func f(): Unit
> }
> ```

仓颉接口中的成员函数, 默认都是`public`的, 不能用其他访问修饰符修饰

接口的成员函数, 默认都是`open`的, 即 默认都被实现

接口的成员函数也可以用`mut`修饰, 接口本身实际是不需要`mut`的, 因为接口不存在成员变量, 也就不存在成员变量是否可修改

接口的`mut`成员函数更多是为了给实现了接口的`strcut`准备的

接口的`static`成员函数是可以不实现函数体的, 而类中的`static`成员函数必须拥有函数体的实现

##### 接口中的成员属性

> `interface`中也可以定义成员属性, 定义成员属性的语法参见[属性]

##### 接口的默认实现

> `interface`中的抽象函数和抽象属性都可以拥有默认实现
>
> `interface`被其它`interface`或类型继承或实现的时候, 如果该`interface`的默认实现没有被重新实现, 则这些默认实现会被拷贝子类型中
>
> - 实例成员函数的默认实现中可以使用`this`, `this`的类型是当前`interface`
>
> - 默认实现和非抽象的成员函数一样, 可以访问当前作用域中所有可访问的元素
>
> - 默认实现是一种语法糖, 可以给实现类型提供默认行为, 子接口可以沿用父接口的默认实现
>
> - 默认实现不属于继承语义, 因此不能使用`override/redef`, 也不能使用`super`
>
> - 继承接口的子类型为类时, 默认实现保留`open`语义, 可以被类的子类重写
>
> ```cangjie
> interface I {
>     func f() {            // f: () -> I
>         let a = this      // a: I
>         return this
>     }
> }
> ```

上述内容中存在一句描述: ***默认实现不属于继承语义, 因此不能使用`override/redef`, 也不能使用`super`***

但经过测试, 在`interface`的继承链中, `override`和`redef`都是可以使用的, 但`super`无法被使用

所以, 此描述存在混淆 或 存在描述错误, 可能是在强调 不存在继承或实现关系的`interface`的默认实现不能使用`override/redef`, 但这是一定的, 好像并不需要特别强调

或者是在说, 接口的继承关系不是传统的继承语义, 因为`super`无法被使用

#### 接口继承

> 接口允许继承一个或多个接口
>
> ```cangjie
> interface I1<T> {}
> interface I2 {}
> interface I3<U> <: I1<U> {}               // 继承泛型接口
> interface I4<V> <: I1<Int32> & I2 {}      // 继承多个接口
> ```

仓颉中, `interface`可以多继承, 但`class`不允许多继承, 其中一个重要原因是`interface`不存在成员变量, 但不是唯一原因

> 子接口继承父接口时, 会继承父接口的所有成员:
>
> 子接口继承父接口时, 对于非泛型接口不能直接继承多次, 对于泛型接口不能使用相同类型参数直接继承多次
>
> 例如:
>
> ```cangjie
> interface I1 {}
> interface I2 <: I1 & I1 {}                    // error
>
> interface I3<T> {}
> interface I4 <: I3<Int32> & I3<Int32> {}      // error
> interface I5<T> <: I3<T> & I3<Int32> {}       // ok
> ```
>
> 如果泛型接口`I3`在使用时被给了类型参数为`Int32`, 那么编译器会在类型使用的位置报错:
>
> ```cangjie
> interface I1<T> {}
> interface I2<T> <: I1<T> & I1<Int32> {}       // ok
> interface I3 <: I2<Int32> {}                  // error
>
> main() {
>     var a: I2<Int32>                          // error
> }
> ```

接口的继承关系中, 不允许存在直接继承多个相同接口类型的情况

即, 如果存在`I1`, 那么不允许存在`interface I2 <: I1 & I1 {}`, 如果是泛型接口类型, 类型参数也不允许相同, 因为相同类型参数的泛型接口类型实际属于相同类型

##### 子接口中的默认实现

> 子接口如果继承了 父接口中 没有默认实现的函数或属性, 则在子接口中 允许仅写此函数或属性的声明(当然也允许定义默认实现), 并且函数声明或定义前的`override`或`redef`修饰符是可选的
>
> 示例如下:
>
> ```cangjie
> interface I1 {
>     func f1(): Unit
>     static func f2(): Unit
> }
> interface I2 <: I1 {
>     func f1(): Unit                   // ok
>     static func f2(): Unit            // ok
> }
> interface I3 <: I1 {
>     override func f1(): Unit          // ok
>     redef static func f2(): Unit      // ok
> }
> interface I4 <: I1 {
>     func f1(): Unit {}                // ok
>     static func f2(): Unit {}         // ok
> }
> interface I5 <: I1 {
>     override func f1(): Unit {}       // ok
>     redef static func f2(): Unit {}   // ok
> }
> ```

子接口中的成员函数是可以使用`override`或`redef`修饰的

> 子接口如果继承了 父接口中 有默认实现的函数或属性, 则在子接口中 不允许 仅写此函数或属性的声明而 没有实现, 如果子接口中给出新的默认实现, 那么定义前的`override`或`redef`修饰符是可选的
>
> 示例如下:
>
> ```cangjie
> interface I1 {
>     func f1(): Unit {}
>     static func f2(): Unit {}
> }
> interface I2 <: I1 {
>     func f1(): Unit                   // Error, 'f1' 必须有一个新的实现
>     static func f2(): Unit            // Error, 'f2' 必须有一个新的实现
> }
> interface I3 <: I1 {
>     override func f1(): Unit {}       // ok
>     redef static func f2(): Unit {}   // ok
> }
> interface I4 <: I1 {
>     func f1(): Unit {}                // ok
>     static func f2(): Unit {}         // ok
> }
> ```

父接口的成员函数或属性, 如果存在默认实现, 则子接口中不允许只声明不实现, 要不就不声明, 要不就要重新实现

> 如果子接口继承的多个父接口中拥有相同签名成员的默认实现, 子接口必须提供自己版本的新默认实现, 否则会编译报错
>
> ```cangjie
> interface I1 {
>     func f() {}
> }
> interface I2 {
>     func f() {}
> }
> interface I3 <: I1 & I2 {}        // error, I3 必须实现 f: () -> Unit
> ```

子接口如果是多继承, 且不同父接口中存在同签名成员函数且 都有默认实现 或 都无默认实现

子接口就会出现冲突, 子接口实现一个自己的同签名成员函数 可以解决冲突

但, 如果不同父类接口中 存在同函数名、同参数列表 但返回值类型毫无关系的成员函数, 那么**冲突无法解决**

#### 实现接口

##### 实现接口时的覆盖与重载


> 一个类型实现一个或多个接口时规则如下:
>
> - 抽象类以外的类型实现接口时, 必须实现所有的函数、属性
>
> - 抽象类实现接口时, 允许不实现接口中的函数和属性
>
> - 实现函数的函数名、参数列表必须与接口中对应函数的相同
>
> - 实现函数的返回类型 应该与 接口中对应函数的返回类型相同 或者 为其子类型
>
> - 如果接口中的函数为泛型函数, 则要求实现函数的类型变元约束 比 接口中对应函数更宽松或相同
>
> - 实现属性的`mut`修饰符必须与接口中相应的属性相同
>
> - 实现属性的类型, 必须与接口中对应的属性相同
>
> - 如果多个接口中 只有同一函数或属性的 一个默认实现, 则实现类型可以 不实现该函数或属性, 使用默认实现
>
> - 如果多个接口中 包含同一函数或属性的 多个默认实现, 则实现类型 必须要实现该函数或属性, 无法使用默认实现
>
> - 如果实现类型已经存在 (从父类继承或本类定义) 接口中同一函数或属性的实现, 则不会再使用任何接口中的默认实现
>
> - 类型在实现接口时, 函数或属性定义前的`override`修饰符(或`redef`修饰符)是可选的, 无论接口中的函数或属性是否存在默认实现

仓颉中, 抽象类可以只实现接口的部分函数或属性, 非抽象类 需要实现接口的所有的函数或属性

> 抽象类以外的类型实现接口时:
>
> 必须对接口中的抽象函数和抽象属性进行实现, 如下示例中的`f1`, `f3`
>
> 允许不使用接口中的函数默认实现, 如下示例中的`f2`
>
> 抽象类允许不实现接口中的实例成员函数, 如下示例中抽象类`C1`并没有实现接口`I`中的`f1`
>
> ```cangjie
> interface I {
>     func f1(): Unit
>     func f2(): Unit {
>         return
>     }
>     static func f3(): Int64
> }
> class C <: I {
>     public func f1(): Unit {}
>     public func f2(): Unit {
>         return
>     }
>     public static func f3(): Int64 {
>         return 0
>     }
> }
>
> abstract class C1 <: I {
>     public static func f3(): Int64 {
>         return 1
>     }
> }
> ```
>
> 示例: 接口`I`中的函数`f`和`g`为泛型函数, `class E`和`F`满足实现函数的类型变元约束 比 被实现函数的**更宽松或相同**的要求, 编译成功;`class D`不满足要求, 则编译报错
>
> ```cangjie
> // C <: B <: A
> interface I {
>     static func f<T>(a: T): Unit where  T <: B
>     static func g<T>(): Unit where T <: B
> }
>
> class D <: I  {
>     public static func f<T>(a: T) where T <: C {}         // Error, 约束过于严格
>     public static func g<T>() where T <: C {}             // Error, 约束过于严格
> }
>
> class E <: I  {
>     public static func f<T>(a: T) where T <: A {}         // OK, 约束更宽松
>     public static func g<T>() where T <: A {}             // OK, 约束更宽松
> }
>
> class F <: I  {
>     public static func f<T>(a: T) where T <: B {}         // OK, 相同的约束
>     public static func g<T>() where T <: B {}             // OK, 相同的约束
> }
> ```
>
> 更多例子:
>
> ```cangjie
> // case 1
> interface I1 {
>     func f(): Unit
> }
> interface I2 {
>     func f(): Unit
> }
> class A <: I1 & I2 {
>     public func f(): Unit {}                  // ok
> }
>
> // case 2
> interface I1 {
>     func f(): Unit
> }
> interface I2 {
>     func f(): Unit {}
> }
> open class A {
>     public open func f(): Unit {}             // ok
> }
> class B <: A & I1 & I2 {
>     public override func f(): Unit {}         // ok
> }
>
> // case 3
> interface I1 {
>     func f(): Unit
> }
> interface I2 {
>     func f(): Unit {}
> }
> class A <: I1 & I2 {}                         // ok, f from I2
>
> // case 4
> interface I1 {
>     func f(): Unit {}
> }
> interface I2 {
>     func f(): Unit {}
> }
> class A <: I1 & I2 {}                         // error
> class B <: I1 & I2 {                          // ok,
>     public func f(): Unit {}
> }
>
> // case 5
> interface I1 {
>     func f(a: Int): Unit {}
> }
> interface I2  {
>     func f(a: Int): Unit {}
> }
>
> open class A  {
>     public open func f(a: Int): Unit {}
> }
> open class B <: A & I1 & I2 {                 // ok, f from A
> }
> ```

对于泛型`interface`的成员函数, 在子类或子接口中实现时, 类型变元约束不能变得更严格

###### 实现接口时函数重载的规则

> 父作用域函数与子作用域函数的函数名必须相同, 但参数列表必须不同
>
> 以下几个例子中给出了类型实现接口时构成函数重载的一些情形, 需要注意的是 当类型实现接口时 需要为被重载的函数声明提供实现
>
> 示例一: 分别在`I1`和`I2`中**声明**了函数名相同、参数列表不同的函数`f`
>
> 需要在实现类`C`中同时实现参数类型为`Unit`和参数类型为`Int32`的f
>
> ```cangjie
> interface I1 {
>     func f(): Unit
> }
> interface I2 {
>     func f(a: Int32): Unit
> }
> class C <: I1 & I2 {
>     public func f(): Unit {}              // I1 中的 f 需要实现
>     public func f(a: Int32): Unit {}      // I2 中的 f 需要实现
> }
> ```
>
> 示例二: 分别在`I1`和`I2`中**定义**了函数相同、参数列表不同的默认函数`f`
>
> 不需要在`C`中实现`f`
>
> ```cangjie
> interface I1 {
>     func f() {}
> }
> interface I2 {
>     func f(a: Int32) {}
> }
> class C <: I1 & I2 {
> }
> ```
>
> 示例三: 在`I1`中**声明**了函数名为`f`, 参数类型为`Unit`的函数; 在`I2`中**定义**了函数名为`f`, 参数类型为`Int32`的函数
>
> 类`C`中必须实现参数类型为`Unit`的函数`f`
>
> ```cangjie
> interface I1 {
>     func f(): Unit
> }
> interface I2 {
>     func f(a: Int32):Unit {
>         return
>     }
> }
> class C <: I1 & I2 {
>     public func f(): Unit {
>         return
>     }
> }
> ```

#### `Any`接口

> `Any`接口是一个语言内置的空接口, 所有`interface`类型都默认继承它, 所有非`interface`类型都默认实现它, 因此所有类型都可以作为`Any`类型的子类型使用
>
> ```cangjie
> class A {}
>
> struct B {}
>
> enum C { D }
>
> main() {
>     var i: Any = A()      // ok
>     i = B()               // ok
>     i = C.D               // ok
>     i = (1, 2)            // ok
>     i = { => 123 }        // ok
>
>     return 0
> }
> ```
>
> 在类型定义处可以显式声明实现`Any`接口, 如果没有则会由编译器隐式实现, 但不能使用扩展重新实现`Any`接口
>
> ```cangjie
> class A <: Any {}     // ok
>
> class B {}            // 隐式实现 Any
>
> extend B <: Any {}    // error
> ```

`Any`之于接口, 类似`Object`之于类

不过`Any`也是被所有类型实现的接口
