---
title: "仓颉文档阅读-开发指南VII: 类和接口(II) - This、创建对象以及class的继承"
published: 2025-11-19 16:33:21
description: '仓颉文档阅读的开发指南部分, 本篇文章介绍仓颉语言的类的相关内容'
image: 'https://humid1ch.oss-cn-shanghai.aliyuncs.com/20250929154944807.webp'
tags: ["开发语言", "仓颉"]
category: Blogs
draft: true
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

### 类

> `class` 类型是面向对象编程中的经典概念，仓颉中同样支持使用 `class` 来实现面向对象编程
> 
> `class` 与 `struct` 的主要区别在于：`class` 是引用类型，`struct` 是值类型，它们在赋值或传参时行为是不同的
> 
> **`class` 之间可以继承，但 `struct` 之间不能继承**
> 
> 本节依次介绍如何定义 `class` 类型，如何创建对象，以及 `class` 的继承

### `This` 类型

> 在类内部，支持 `This` 类型占位符，代指当前类的类型
> 
> 它**只能被作为实例成员函数的返回类型**来使用
> 
> 当使用子类对象调用在父类中定义的返回 `This` 类型的函数时，该函数调用的类型会被识别为子类类型，而非定义所在的父类类型
> 
> 如果实例成员函数没有声明返回类型，并且只存在返回`This`类型表达式时，当前函数的返回类型会推断为 `This`
> 
> 示例如下：
> 
> ```cangjie
> open class C1 {
>     func f(): This {                      // 函数类型是 `() -> C1`
>         return this
>     }
> 
>     func f2() {                           // 函数类型是 `() -> C1`
>         return this
>     }
> 
>     public open func f3(): C1 {
>         return this
>     }
> }
> class C2 <: C1 {
>     // 成员函数 f 是从 C1 继承的，现在 此函数的类型是`() -> C2`
>     public override func f3(): This {     // Ok
>         return this
>     }
> }
> 
> var obj1: C2 = C2()
> var obj2: C1 = C2()
> 
> var x = obj1.f()                          // 在编译过程中, x 的类型为 C2
> var y = obj2.f()                          // 在编译过程中, y 的类型为 C1
> ```

`This`是仓颉中的一个类型占位符, 它表示所在`class`这个类型, 但只能用与`class`成员函数的返回值类型

但要注意, 如果使用子类实例 调用父类的`This`作为返回值类型的函数, `This`表示子类类型

文档中:

```cangjie
var obj1: C2 = C2()
var obj2: C1 = C2()

var x = obj1.f()                          // 在编译过程中, x 的类型为 C2
var y = obj2.f()                          // 在编译过程中, y 的类型为 C1
```

编译过程中, `x`和`y`的识别类型是不同的, 即 是`obj1`和`obj2`的声明类型

但, 运行过程中, 是一种动态多态行为, 实际调用类型均为`C2`

为什么存在这个差异呢?

因为要保证语法合法, 你不能尝试使用`obj2`直接调用`C2`自己的成员函数, 这是非法行为

```cangjie
import std.reflect.*

main() {
    println(TypeInfo.of(x))
    println(TypeInfo.of(y))

    return 0
}
```

这个打印:

```text
Main.C2
Main.C2
```

### 创建对象

> 定义了 `class` 类型后，即可通过调用其构造函数来创建对象（通过 `class` 类型名调用构造函数）
> 
> 例如，下例中通过 `Rectangle(10, 20)` 创建 `Rectangle` 类型的对象并赋值给变量 `r`
> 
> 创建对象之后，可以通过对象访问(`public` 修饰的）实例成员变量和实例成员函数
> 
> 例如，下例中通过 `r.width` 和 `r.height` 可分别访问 `r` 中 `width` 和 `height` 的值，通过 `r.area()` 可以调用成员函数 `area`
> 
> ```cangjie
> class Rectangle {
>     let width: Int64
>     let height: Int64
> 
>     public init(width: Int64, height: Int64) {
>         this.width = width
>         this.height = height
>     }
> 
>     public func area() {
>         this.width * this.height
>     }
> }
> let r = Rectangle(10, 20) // r.width = 10, r.height = 20
> let width = r.width       // width = 10
> let height = r.height     // height = 20
> let a = r.area()          // a = 200
> ```
> 
> 如果希望通过对象去修改成员变量的值（不鼓励这种方式，最好还是通过成员函数去修改），需要将 `class` 类型中的成员变量定义为可变成员变量（即使用 `var` 定义）
> 
> 举例如下：
> 
> ```cangjie
> class Rectangle {
>     public var width: Int64
>     public var height: Int64
> 
>     public init(width: Int64, height: Int64) {
>         this.width = width
>         this.height = height
>     }
> 
>     public func area() {
>         width * height
>     }
> }
> 
> main() {
>     let r = Rectangle(10, 20) // r.width = 10, r.height = 20
>     r.width = 8               // r.width = 8
>     r.height = 24             // r.height = 24
>     let a = r.area()          // a = 192
> }
> ```

`class`对象(实例)的创建, 很简单, 即 使用类名调用构造函数: `类名(参数列表)`

创建完成之后, 就可以通过实例访问可访问的成员变量、成员函数

使用`var`定义的成员变量可以被修改, 但 仓颉不建议直接通过实例修改成员变量的值, 最好还是通过成员函数来修改

> 不同于 `struct`，对象在赋值或传参时，不会将对象进行复制，多个变量指向的是同一个对象，通过一个变量去修改对象中成员的值，其他变量中对应的成员变量也会被修改
> 
> 以赋值为例，下面的例子中，将 `r1` 赋值给 `r2` 之后，修改 `r1` 的 `width` 和 `height` 的值，`r2` 的 `width` 和 `height` 值也同样会被修改
> 
> ```cangjie
> class Rectangle {
>     var width: Int64
>     var height: Int64
> 
>     public init(width: Int64, height: Int64) {
>         this.width = width
>         this.height = height
>     }
> 
>     public func area() {
>         this.width * this.height
>     }
> }
> main() {
>     var r1 = Rectangle(10, 20) // r1.width = 10, r1.height = 20
>     var r2 = r1                // r2.width = 10, r2.height = 20
>     r1.width = 8               // r1.width = 8
>     r1.height = 24             // r1.height = 24
>     let a1 = r1.area()         // a1 = 192
>     let a2 = r2.area()         // a2 = 192
> }
> ```

`class`是引用类型的, `class`对象在赋值或传参是, 不会发生拷贝, 而是引用

即, 新变量或形参 会引用(指向)原对象

多次赋值, 所有变量都是引用同一个原对象

此时, 通过其中一个变量修改对象成员的值, 所有变量再访问对象成员的值都会发生改变

因为, 都引用同一个对象

这样理解仓颉中的引用类型:

创建引用类型实例(对象) 赋值给变量, 就是变量引用(指向)了实例, 并不表示此变量就是实例, **变量与引用类型实例之间的关系是引用(指向)关系**

如此, 你通过变量传参或赋值, 都只是**建立了新的引用关系**

如果, 给此变量重新赋值, 就是改变了此变量的引用关系, 此变量就不再引用之前的实例

仓颉的引用类型, 变量与实例之间的关系, **不同于C++的引用变量**, C++引用变量定义之后无法改变引用对象

**变量和实例是两个有关联的东西**, 变量也可以重新修改指向, 可以类比一下C/C++的指针, 但又有很大不同

### `class` 的继承

> 像大多数支持 `class` 的编程语言一样，仓颉中的`class`同样**支持继承**
> 
> 如果类 `B` 继承自类 `A`，则称 `A` 为父类，`B` 为子类
> 
> 子类将继承父类中 除`private`成员和构造函数以外 的所有成员
> 
> **抽象类总是可被继承的**，故抽象类定义时的 `open` 修饰符是可选的，也可以使用 **`sealed`修饰符修饰抽象类**，表示该抽象类**只能在本包被继承**
> 
> 但非抽象的类可被继承是有条件的：定义时必须使用修饰符 `open` 修饰
> 
> 当带 `open` 修饰的实例成员被 `class` 继承时，该 `open` 的修饰符也会被继承
> 
> 当非 `open` 修饰的类中存在 `open` 修饰的成员时，编译器会给出告警
> 
> 可以在子类定义处通过 `<:` 指定其继承的父类，但要求父类必须是可继承的
> 
> 例如，下面的例子中，`class A` 使用 `open` 修饰，是可以被类 `B` 继承的，但是因为类 `B` 是不可继承的，所以 `C` 在继承 `B` 的时候会报错
> 
> ```cangjie
> open class A {
>     let a: Int64 = 10
> }
> 
> class B <: A {                    // Ok, 'B' 继承自 'A'
>     let b: Int64 = 20
> }
> 
> class C <: B {                    // Error, 'B' 不能被继承
>     let c: Int64 = 30
> }
> ```
> 
> **`class`仅支持单继承**，因此下面这样一个类继承两个类的代码是不合法的（**`&`是类实现多个接口时的语法**，详见 [接口]() ）
> 
> ```cangjie
> open class A {
>     let a: Int64 = 10
> }
> 
> open class B {
>     let b: Int64 = 20
> }
> 
> class C <: A & B {                // Error, 'C' 只能继承与一个类(单继承)
>     let c: Int64 = 30
> }
> ```

仓颉`class`存在继承机制, 但 只允许单继承

继承的语法为:

```cangjie
open class ClassBase {}

class ClassSuper <: ClassBase {}
```

只有`open`修饰的类能够被继承, 即 作为父类

如果要继承于一个父类, **类名后紧跟`<:`并指明父类**, 如果尝试使用`&`符号连接多个父类进行继承, 是不被允许的, 因为仓颉只允许单继承

> [!TIP]
> 
> 仓颉`class`继承的语法, 与实现接口的语法是一致的
> 
> 不同的是, 实现接口可以使用`&`连接多个接口, 以实现多个接口, 但多继承不被允许
> 
> 如果同时 进行继承和实现接口, 那么 父类必须为`<:`之后的首个标识符
> 
> 即, `class ClassSuper <: ClassBase & Interface1 & Interface2 & Interface3 {}`

抽象类总是可被继承的, 默认具有`open`语义, 所以显式的`open`声明是可忽略的

抽象类还可以使用`sealed`修饰, 以此限制此抽象类只能在本包中被继承

> 因为类是单继承的，所以任何类都最多只能有一个直接父类
> 
> 对于定义时指定了父类的`class`，它的直接父类就是定义时指定的类，对于定义时未指定父类的 `class`，它的直接父类是 `Object` 类型
> 
> **`Object`是所有类的父类**（注意，`Object` 没有直接父类，并且 `Object` 中不包含任何成员）
> 
> 因为子类是继承自父类的，所以**子类的对象天然可以当做父类的对象使用**，但是反之不然
> 
> 例如，下例中 `B` 是 `A` 的子类，那么 `B` 类型的对象可以赋值给 `A` 类型的变量，但是 `A` 类型的对象不能赋值给 `B` 类型的变量
> 
> ```cangjie
> open class A {
>     let a: Int64 = 10
> }
> 
> class B <: A {
>     let b: Int64 = 20
> }
> 
> let a: A = B()                // Ok, 子类对象可以赋值给父类变量
> 
> open class A {
>     let a: Int64 = 10
> }
> 
> class B <: A {
>     let b: Int64 = 20
> }
> 
> let b: B = A()                // Error, 父类对象不能赋值给子类变量
> ```
> 
> `class`定义的类型不允许继承类型本身
> 
> ```cangjie
> class A <: A {}               // Error, 'A' 不能继承于自身
> ```

仓颉所有类都是`Object`类的子类, 即 如果一个类没有显式指定继承的父类, 也默认继承于`Object`类(此类没有任何成员, 且没有直接父类)

子类是父类的子类型, 所以子类对象天然可以做为父类对象使用

所以, 子类对象能够赋值给父类变量, 但反之的不行

> 抽象类可以使用 **`sealed`修饰符**，表示被修饰的类定义 **只能在本定义所在的包内被其他类继承**
> 
> `sealed`已经蕴含了`public/open`的语义，因此定义`sealed abstract class`时若提供`public/open`修饰符，编译器将会告警
> 
> `sealed`的子类可以不是`sealed`类，仍可被 `open/sealed` 修饰，或不使用任何继承性修饰符
> 
> 若 `sealed` 类的子类被 `open` 修饰，则其子类可在包外被继承
> 
> `sealed` 的子类可以 不被 `public` 修饰(注意, 不是 不可以, 而是 可以不)
> 
> ```cangjie
> package A
> public sealed abstract class C1 {}                // Warning, 冗余修饰符，'sealed' 已存在 'public' 语义
> sealed open abstract class C2 {}                  // Warning, 冗余修饰符，'sealed' 已存在 'open' 语义
> sealed abstract class C3 {}                       // OK, 使用 'sealed' 时, 'public' 是可选的
> 
> class S1 <: C1 {}                                 // OK
> public open class S2 <: C1 {}                     // OK
> public sealed abstract class S3 <: C1 {}          // OK
> open class S4 <: C1 {}                            // OK
> ```
> 
> ```cangjie
> package B
> import A.*
> 
> class SS1 <: S2 {}                                // OK
> class SS2 <: S3 {}                                // Error, S3 在 package A 中被 sealed 修饰, 不能在这里被继承
> sealed class SS3 {}                               // Error, 'sealed' 不能修饰非抽象类
> ```

`sealed`修饰符, 用来修饰抽象类, 不能修饰非抽象类, 隐式包含`public`和`open`语义

被`sealed`修饰的类, 被限制 只能在定义此类的包中被继承

但, `sealed`类的子类, 其可访问性、可继承性与其他普通的类保持一致, 即 根据 **是否`open`和其可访问性修饰符决定**

#### 父类构造函数调用

子类的 `init` 构造函数可以使用 `super(args)` 的形式调用父类构造函数，或使用 `this(args)` 的形式调用本类其他构造函数，但两者之间只能调用一个

如果调用，**必须在构造函数体内的第一个表达式处**，在此之前不能有任何表达式或声明

```cangjie
open class A {
    A(let a: Int64) {}
}

class B <: A {
    let b: Int64
    init(b: Int64) {
        super(30)
        this.b = b
    }

    init() {
        this(20)
    }
}
```

子类的主构造函数中，可以使用 `super(args)` 的形式调用父类构造函数，但不能使用 `this(args)` 的形式调用本类其他构造函数

如果子类的构造函数没有显式调用父类构造函数，也没有显式调用其他构造函数，编译器会在该构造函数体的开始处插入直接父类的无参构造函数的调用

如果此时父类没有无参构造函数，则会编译报错

```cangjie
open class A {
    let a: Int64
    init() {
        a = 100
    }
}

open class B <: A {
    let b: Int64
    init(b: Int64) {
        // OK, `super()` added by compiler
        this.b = b
    }
}

open class C <: B {
    let c: Int64
    init(c: Int64) {  // Error, there is no non-parameter constructor in super class
        this.c = c
    }
}
```

#### 覆盖和重定义

子类中可以覆盖（`override`）父类中的同名非抽象实例成员函数，即在子类中为父类中的某个实例成员函数定义新的实现

覆盖时，要求父类中的成员函数使用 `open` 修饰，子类中的同名函数使用 `override` 修饰，其中 `override` 是可选的

例如，下面的例子中，子类 `B` 中的函数 `f`覆盖了父类 `A` 中的函数 `f`

```cangjie
open class A {
    public open func f(): Unit {
        println("I am superclass")
    }
}

class B <: A {
    public override func f(): Unit {
        println("I am subclass")
    }
}

main() {
    let a: A = A()
    let b: A = B()
    a.f()
    b.f()
}
```

对于被覆盖的函数，调用时将根据变量的运行时类型（由实际赋给该变量的对象决定）确定调用的版本（即所谓的动态派发）

例如，上例中 `a` 的运行时类型是 `A`，因此 `a.f()` 调用的是父类 `A` 中的函数 `f`

`b` 的运行时类型是 `B`（编译时类型是 `A`），因此 `b.f()` 调用的是子类 `B` 中的函数 `f`

所以程序会输出：

```text
I am superclass
I am subclass
```

对于静态函数，子类中可以重定义父类中的同名非抽象静态函数，即在子类中为父类中的某个静态函数定义新的实现

重定义时，要求子类中的同名静态函数使用 `redef` 修饰，其中 `redef` 是可选的

例如，下面的例子中，子类 `D` 中的函数 `foo` 重定义了父类 `C` 中的函数 `foo`

```cangjie
open class C {
    public static func foo(): Unit {
        println("I am class C")
    }
}

class D <: C {
    public redef static func foo(): Unit {
        println("I am class D")
    }
}

main() {
    C.foo()
    D.foo()
}
```

对于被重定义的函数，调用时将根据 `class` 的类型决定调用的版本

例如，上例中 `C.foo()` 调用的是父类 `C` 中的函数 `foo`，`D.foo()` 调用的是子类 `D` 中的函数 `foo`

```text
I am class C
I am class D
```

如果抽象函数或 `open` 修饰的函数有命名形参，那么实现函数或 `override` 修饰的函数也需要保持同样的命名形参

```cangjie
open class A {
    public open func f(a!: Int32): Int32 {
        a + 1
    }
}

class B <: A {
    public override func f(a!: Int32): Int32 { // Ok
        a + 2
    }
}

class C <: A {
    public override func f(b!: Int32): Int32 { // Error
        b + 3
    }
}

main() {
    B().f(a: 0)
    C().f(b: 0)
}
```

还需要注意的是，当实现或重定义的函数为泛型函数时，子类型函数的类型变元约束需要比父类型中对应函数更宽松或相同

```cangjie
open class A {}
open class B <: A {}
open class C <: B {}

open class Base {
    public open func foo<T>(a: T): Unit where T <: B {}
    public open func bar<T>(a: T): Unit where T <: B {}
    public static func f<T>(a: T): Unit where T <: B {}
    public static func g<T>(): Unit where T <: B {}
}

class D <: Base {
    public override func foo<T>(a: T): Unit where T <: C {} // Error, stricter constraint
    public override func bar<T>(a: T): Unit where T <: C {} // Error, stricter constraint
    public redef static func f<T>(a: T): Unit where T <: C {} // Error, stricter constraint
    public redef static func g<T>(): Unit where T <: C {} // Error, stricter constraint
}

class E <: Base {
    public override func foo<T>(a: T): Unit where T <: A {} // OK: looser constraint
    public override func bar<V>(a: V): Unit where V <: A {} // OK: looser constraint, names of generic parameters do not matter
    public redef static func f<T>(a: T): Unit where T <: A {} // OK: looser constraint
    public redef static func g<T>(): Unit where T <: A {} // OK: looser constraint
}

class F <: Base {
    public override func foo<T>(a: T): Unit where T <: B {} // OK: same constraint
    public override func bar<V>(a: V): Unit where V <: B {} // OK: same constraint
    public redef static func f<T>(a: T): Unit where T <: B {} // OK: same constraint
    public redef static func g<T>(): Unit where T <: B {} // OK: same constraint
}
```
