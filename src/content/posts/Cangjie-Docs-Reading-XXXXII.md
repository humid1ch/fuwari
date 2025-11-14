---
title: "仓颉文档阅读-开发指南V: 结构类型 - struct 以及 mut 函数"
published: 2025-11-14 16:21:22
description: '仓颉文档阅读的开发指南部分, 本篇文章介绍仓颉语言的结构体相关内容'
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

## 结构类型

### 定义`struct`类型

> `struct`类型的定义以关键字`struct`开头, 后跟`struct`的名字, 接着是定义在一对花括号中的`struct`定义体
> 
> `struct`定义体中可以定义一系列的成员变量、成员属性(参见属性)、静态初始化器、构造函数和成员函数
> 
> ```cangjie
> struct Rectangle {
>     let width: Int64
>     let height: Int64
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
> ```
> 
> 上例中定义了名为`Rectangle`的`struct`类型, 它有两个`Int64`类型的成员变量`width`和`height`, 一个有两个`Int64`类型参数的构造函数(使用关键字`init`定义, 函数体中通常是对成员变量的初始化), 以及一个成员函数`area`(返回`width`和`height`的乘积)
> 
> > 注意：
> > 
> > `struct`只能定义在源文件的顶层作用域

仓颉也可以`struct`类型, 是值类型的

只能定义在顶层作用域

```cangjie
struct Ident {}
```

可以拥有成员变量、成员属性、构造函数、成员函数等

#### `struct`成员变量

> `struct`成员变量分为实例成员变量和静态成员变量(使用`static`修饰符修饰)
> 
> 二者访问上的区别在于实例成员变量只能通过`struct`实例(说`a`是`T`类型的实例, 指的是`a`是一个`T`类型的值)访问, 静态成员变量只能通过`struct`类型名访问
> 
> 实例成员变量定义时可以不设置初值(但必须标注类型, 如上例中的`width`和`height`), 也可以设置初值, 例如：
> 
> ```cangjie
> struct Rectangle {
>     let width = 10
>     let height = 20
> }
> ```

仓颉中, `strcut`也可以定义静态成员变量, 但与C++不同的是, 仓颉中的**静态成员变量只能通过`struct`名访问**

#### `struct`静态初始化器

> `struct`支持定义静态初始化器, 并在静态初始化器中 通过赋值表达式来**对静态成员变量进行初始化**
> 
> 静态初始化器以关键字组合**`static init`开头, 后跟无参参数列表和函数体, 且不能被访问修饰符修饰**
> 
> 函数体中**必须完成对所有未初始化的静态成员变量的初始化**, 否则编译报错
> 
> ```cangjie
> struct Rectangle {
>     static let degree: Int64
>     static init() {
>         degree = 180
>     }
> }
> ```
> 
> 一个`struct`中**最多允许定义一个静态初始化器**, 否则报重定义错误
> 
> ```cangjie
> struct Rectangle {
>     static let degree: Int64
>     static init() {
>         degree = 180
>     }
>     static init() {                   // Error, 静态初始化函数重定义
>         degree = 180
>     }
> }
> ```

仓颉中, 使用静态初始化器: `static init() {}`对静态成员变量进行初始化

且, 需要在静态初始化器中 **对所有静态成员变量完成初始化**, 否则编译报错

且`struct`中只能定义一个静态初始化器

#### `struct`构造函数

> `struct`支持两类构造函数：普通构造函数和主构造函数
> 
> 普通构造函数以关键字`init`开头, 后跟参数列表和函数体
> 
> 函数体中**必须完成对所有未初始化的实例成员变量的初始化**(如果参数名和成员变量名无法区分, 可以在成员变量前使用`this`加以区分, `this`表示`struct`的当前实例), 否则编译报错
> 
> ```cangjie
> struct Rectangle {
>     let width: Int64
>     let height: Int64
> 
>     public init(width: Int64, height: Int64) {            // Error, 'height' 在构造函数中未初始化
>         this.width = width
>     }
> }
> ```

仓颉`struct`, 使用`init`作为构造函数的函数名

函数体内必须 **完成所有未初始化成员变量的初始化**

构造函数的形参可以与已存在的成员变量同名, 此时在函数体内可以**使用`this.变量名`访问成员变量, 否则就是形参**

> 一个`struct`中可以定义多个普通**构造函数**, 但它们**必须构成重载**(参见[函数重载](https://www.humid1ch.cn/posts/cangjie-docs-reading-xxxx/#函数重载)), 否则报重定义错误
> 
> ```cangjie
> struct Rectangle {
>     let width: Int64
>     let height: Int64
> 
>     public init(width: Int64) {
>         this.width = width
>         this.height = width
>     }
> 
>     public init(width: Int64, height: Int64) {            // Ok, 与第一个 init 函数构成重载
>         this.width = width
>         this.height = height
>     }
> 
>     public init(height: Int64) {                          // Error, 与第一个 init 函数重定义
>         this.width = height
>         this.height = height
>     }
> }
> ```

仓颉`struct`的`init`函数之间可以构成重载

> 除了可以定义若干普通的以`init`为名字的构造函数外, `struct`内还可以定义**最多一个主构造函数**
> 
> **主构造函数的名字和`struct`类型名相同**, 它的参数列表中可以有两种形式的形参：普通形参和**成员变量形参**(需要在参数名前加上`let`或`var`), 成员变量形参同时扮演定义成员变量和构造函数参数的功能
> 
> 使用主构造函数通常可以简化`struct`的定义, 例如, 上述包含一个`init`构造函数的`Rectangle`可以简化为如下定义：
> 
> ```cangjie
> struct Rectangle {
>     public Rectangle(let width: Int64, let height: Int64) {}
> }
> ```
> 
> 主构造函数的参数列表中也可以定义普通形参, 例如：
> 
> ```cangjie
> struct Rectangle {
>     public Rectangle(name: String, let width: Int64, let height: Int64) {}
> }
> ```

仓颉`struct`可以存在一个 **主构造函数**

主构造函数与`struct`同名, 且 主构造函数可以参数列表中可以定义**成员变量形参**

什么是成员变量形参呢? 

仓颉`struct`的主构造函数中, 可以通过**使用`let`或`var`修饰形参**, 实现在参数列表中**定义成员变量**

成员变量形参与普通形参的区别, 就只在是否使用`let`和`var`修饰

> [!NOTE]
> 
> 仓颉中, 主构造函数会在编译时 自动转换为对应的`init`构造函数
> 
> 同时要保证 主构造函数对应的`init`构造函数, 需要**与其他显式定义的`init`函数构成重载**

> 如果`struct`定义中不存在自定义构造函数(包括主构造函数), 并且所有实例成员变量都有初始值, 则会自动为其生成一个无参构造函数(调用此无参构造函数会创建一个所有实例成员变量的值均等于其初值的对象); 否则, 不会自动生成此无参构造函数
> 
> 例如, 对于如下`struct`定义, 注释中给出了自动生成的无参构造函数：
> 
> ```cangjie
> struct Rectangle {
>     let width: Int64 = 10
>     let height: Int64 = 10
>     /* : 自动生成的无参构造函数
>     public init() {
>     }
>     */
> }
> ```

仓颉`struct`也可以不显式定义构造函数

仓颉规定, 只要`struct`的**所有成员变量在定义时均有初始值**, 那么编译器就会生成一个无参构造函数

#### `struct`成员函数

> `struct`成员函数分为**实例成员函数**和**静态成员函数**(使用`static`修饰符修饰), 二者的区别在于：
> 
> **实例成员函数只能通过`struct`实例访问, 静态成员函数只能通过`struct`类型名访问**
> 
> 静态成员函数中不能访问实例成员变量, 也不能调用实例成员函数, 但在实例成员函数中可以访问静态成员变量以及静态成员函数
> 
> 下例中, `area`是实例成员函数, `typeName`是静态成员函数
> 
> ```cangjie
> struct Rectangle {
>     let width: Int64 = 10
>     let height: Int64 = 20
> 
>     public func area() {
>         this.width * this.height
>     }
> 
>     public static func typeName(): String {
>         "Rectangle"
>     }
> }
> ```
> 
> 实例成员函数中可以通过`this`访问实例成员变量, 例如：
> 
> ```cangjie
> struct Rectangle {
>     let width: Int64 = 1
>     let height: Int64 = 1
> 
>     public func area() {
>         this.width * this.height
>     }
> }
> ```

仓颉的成员函数, 拥有实例成员函数和静态成员函数

静态成员函数内, 不能调用实例成员函数和实例成员变量, 且在`struct`外 只能通过`struct`名访问

#### `struct`成员的访问修饰符

> `struct`的成员(包括成员变量、成员属性、构造函数、成员函数、操作符函数(详见[操作符重载](https://www.humid1ch.cn/posts/cangjie-docs-reading-xxxx/#操作符重载)章节))用`4`种访问修饰符修饰：`private`、`internal`、`protected`和`public`, 缺省的修饰符是`internal`
> 
> - `private`表示在`struct`定义内可见
> 
> - `internal`表示仅当前包及子包(包括子包的子包, 详见包章节)内可见
> 
> - `protected`表示当前模块(详见[包]()章节)可见
> 
> - `public`表示模块内外均可见
> 
> 下面的例子中, `width`是`public`修饰的成员, 在类外可以访问, `height`是缺省访问修饰符的成员, 仅在当前包及子包可见, 其他包无法访问
> 
> ```cangjie
> package a
> public struct Rectangle {
>     public var width: Int64
>     var height: Int64
>     private var area: Int64
>     ...
> }
> 
> func samePkgFunc() {
>     var r = Rectangle(10, 20)
>     r.width = 8                       // Ok, public 'width' 可以在这里访问
>     r.height = 24                     // Ok, 'height' 没有显式修饰符, 可以在此处访问
>     r.area = 30                       // Error, private 'area' 在此处不可访问
> }
> ```
> 
> ```cangjie
> package b
> import a.*
> main() {
>     var r = Rectangle(10, 20)
>     r.width = 8                       // Ok, public 'width' 可以在这里访问
>     r.height = 24                     // Error, 'height' 没有显式修饰符 在此处不可访问
>     r.area = 30                       // Error, private 'area' 在此处不可访问
> }
> ```

仓颉中, 成员默认是被`internal`修饰的

除此之外, 成员还可以被`private`、`protected`和`public`修饰, 分别表示不同的可访问性:

- `private`表示在`struct`定义内可见

    即, 仅能在`struct`定义内访问

- `internal`表示仅当前包及子包(包括子包的子包, 详见[包]()章节)内可见

    即, 仅能在`struct`定义所在包, 以及其子包内访问

- `protected`表示当前模块(详见[包]()章节)可见

    即, 与`internal`不同, 被`protected`修饰的成员, 可以在父包、当前包以及子包访问

- `public`表示模块内外均可见

    被`public`修饰的成员的访问, 只要可以见到, 不受任何限制

#### 禁止递归`struct`

> 递归和互递归定义的`struct`均是非法的
> 
> 例如：
> 
> ```cangjie
> struct R1 {                   // Error, 'R1' 递归引用自身
>     let other: R1
> }
> struct R2 {                   // Error, 'R2' 和 'R3' 相互递归
>     let other: R3
> }
> struct R3 {                   // Error, 'R2' 和 'R3' 相互递归
>     let other: R2
> }
> ```

**`struct`是值类型的, 在使用时需要确定类型具体的大小, 所以不能自递归以及互递归**

### 创建`struct`实例

> 定义了`struct`类型后, 即可通过调用`struct`的构造函数来创建`struct`实例
> 
> 在`struct`定义之外, 通过`struct`类型名调用构造函数创建该类型实例, 并可以通过实例访问满足可见性修饰符(如`public`)的实例成员变量和实例成员函数
> 
> 例如, 下例中定义了一个`Rectangle`类型的变量`r`, 通过`r.width`和`r.height`可分别访问`r`中`width`和`height`的值, 通过`r.area()`可以调用`r`的成员函数`area`
> 
> ```cangjie
> struct Rectangle {
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
> let r = Rectangle(10, 20)
> let width = r.width                             // width = 10
> let height = r.height                           // height = 20
> let a = r.area()                                // a = 200
> ```

仓颉`struct`实例, 都是通过对应的构造函数创建的

通过实例, 可以访问对应`struct`的成员, 通过实例访问成员, 就不是`struct`内部访问了, 就要满足可访问性修饰符的限制

> 如果希望通过`struct`实例去修改成员变量的值, 需要将`struct`类型的变量定义为**可变变量**, 并且被修改的成员变量也必须是可变成员变量(使用`var`定义)
> 
> 举例如下：
> 
> ```cangjie
> struct Rectangle {
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
>     var r = Rectangle(10, 20)                   // r.width = 10, r.height = 20
>     r.width = 8                                 // r.width = 8
>     r.height = 24                               // r.height = 24
>     let a = r.area()                            // a = 192
> }
> ```

仓颉`struct`是值类型数据

对于`struct`实例, 如果要修改成员变量的值, 需要将`struct`变量定义为`var`的, 即 可变变量

> 在赋值或传参时, 会对`struct`实例进行复制(**成员变量为引用类型时, 仅复制引用而不会复制引用的对象**), 生成新的实例, 对其中一个实例的修改并不会影响另外一个实例
> 
> 以赋值为例, 下面的例子中, 将`r1`赋值给`r2`之后, 修改`r1`的`width`和`height`的值, 并不会影响`r2`的`width`和`height`值
> 
> ```cangjie
> struct Rectangle {
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
>     var r1 = Rectangle(10, 20)                  // r1.width = 10, r1.height = 20
>     var r2 = r1                                 // r2.width = 10, r2.height = 20
>     r1.width = 8                                // r1.width = 8
>     r1.height = 24                              // r1.height = 24
>     let a1 = r1.area()                          // a1 = 192
>     let a2 = r2.area()                          // a2 = 200
> }
> ```

仓颉的`struct`是值类型的, 在传参或赋值时 会对`struct`实例进行拷贝

但, 如果成员变量是引用类型, 就**只拷贝引用, 而不会拷贝完整的成员变量实例**

下面这段代码可以演示:

```cangjie
class RefTest {
    var num = 0

    init(num!: Int64 = 0) {
        this.num = num
    }
    
}

struct ValueTest {
    let ref: RefTest
    var num: Int64

    init(num: Int64) {
        this.num = num
        this.ref = RefTest(num: num)
    }
}

main() {
    var test1 = ValueTest(20)

    let test2 = test1

    println("test1: ref.num: ${test1.ref.num}, num: ${test1.num}")
    println("test2: ref.num: ${test2.ref.num}, num: ${test2.num}")

    test1.ref.num = 40
    test1.num = 30

    println("test1: ref.num: ${test1.ref.num}, num: ${test1.num}")
    println("test2: ref.num: ${test2.ref.num}, num: ${test2.num}")
}
```

这段代码最终会输出:

```text
test1: ref.num: 20, num: 20
test2: ref.num: 20, num: 20
test1: ref.num: 40, num: 30
test2: ref.num: 40, num: 20
```

从结果可以看出, 用`struct`实例进行赋值, 会发生拷贝, 即`let test2 = test1`发生拷贝

但, 此时, 如果修改成员变量`ref`的成员变量, 你会看到两个`struct`实例同步发生了改变

即 如果`struct`的成员变量是引用类型的, 那么 此成员变量**只会拷贝引用**而不是拷贝完整的变量实例

可以认为 **`struct`默认是浅拷贝**

### `mut`函数

> `struct`类型是值类型, 其**实例成员函数无法修改实例本身**
> 
> 例如, 下例中, 成员函数`g`中不能修改成员变量`i`的值
> 
> ```cangjie
> struct Foo {
>     var i = 0
> 
>     public func g() {
>         i += 1                // Error, 实例成员函数中不能修改实例成员变量的值
>     }
> }
> ```
> 
> `mut`函数是一种可以修改`struct`实例本身的特殊的实例成员函数
> 
> 在`mut`函数内部, `this`的语义是特殊的, 这种`this`拥有原地修改字段的能力
> 
> > 注意：
> > 
> > 只允许在`interface`、`struct`和`struct`的扩展内定义`mut`函数(`class`是引用类型, 实例成员函数不需要加`mut`也可以修改实例成员变量, 所以禁止在`class`中定义`mut`函数)

仓颉`struct`是值类型的, 实例成员函数无法修改实例本身

即, 如果在实例成员函数内 尝试修改成员变量, 此行为是被禁止的, 即使目标成员变量是`var`的

但`sturct`中可以定义`mut`函数, 声明函数的可变性

仓颉`struct`, **被`mut`修饰的函数, 才可以修改实例成员变量**

#### `mut`函数定义

> `mut`函数与普通的实例成员函数相比, 多一个`mut`关键字来修饰
> 
> 例如, 下例中在函数`g`之前增加`mut`修饰符之后, 即可在函数体内修改成员变量`i`的值
> 
> ```cangjie
> struct Foo {
>     var i = 0
> 
>     public mut func g() {
>         i += 1                                        // Ok
>     }
> }
> ```
> 
> **`mut`只能修饰实例成员函数, 不能修饰静态成员函数**
> 
> ```cangjie
> struct A {
>     public mut func f(): Unit {}                      // Ok
>     public mut operator func +(rhs: A): A {           // Ok
>         A()
>     }
>     public mut static func g(): Unit {}               // Error, 静态成员函数不能用 'mut' 修饰
> }
> ```
> 
> `mut`函数中的`this`不能被捕获, 也不能作为表达式
> 
> `mut`函数中的`lambda`或嵌套函数, 不能对`struct`的实例成员变量进行捕获
> 
> 示例：
> 
> ```cangjie
> struct Foo {
>     var i = 0
> 
>     public mut func f(): Foo {
>         let f1 = { => this }                          // Error, mut 函数中的 'this' 无法被捕获
>         let f2 = { => this.i = 2 }                    // Error, mut 函数中的 实例成员变量 无法被捕获
>         let f3 = { => this.i }                        // Error, mut 函数中的 实例成员变量 无法被捕获
>         let f4 = { => i }                             // Error, mut 函数中的 实例成员变量 无法被捕获
>         this                                          // Error, mut 函数中的 'this' 不能用作表达式
>     }
> }
> ```

`mut`函数定义时要使用`mut`关键字修饰

但, **静态成员变量禁止被`mut`修饰**

仓颉规定`mut`函数中的`this`以及实例成员变量, 不能被捕获, `this`也不能被捕获

`mut`函数内可以修改`var`成员变量

#### 接口中的`mut`函数

> 接口中的实例成员函数, 也可以使用`mut`修饰
> 
> `struct`类型在实现`interface`的函数时必须保持一样的`mut`修饰
> 
> `struct`以外的类型实现`interface`的函数时不能使用`mut`修饰
> 
> 示例：
> 
> ```cangjie
> interface I {
>     mut func f1(): Unit
>     func f2(): Unit
> }
> 
> struct A <: I {
>     public mut func f1(): Unit {}               // Ok: 在interface中, 使用了 'mut' 修饰符
>     public func f2(): Unit {}                   // Ok: 在interface中, 没有使用 'mut' 修饰符
> }
> 
> struct B <: I {
>     public func f1(): Unit {}                   // Error, 'f1' 在interface中用 'mut' 修饰, 但在struct中没有
>     public mut func f2(): Unit {}               // Error, 'f2' 在interface中没有被 'mut' 修饰, 但在struct中被修饰
> }
> 
> class C <: I {
>     public func f1(): Unit {}                   // Ok
>     public func f2(): Unit {}                   // Ok
> }
> ```

当`interface`中定义有`mut`函数时:

1. 如果`struct`实现此`interface`, 那么 实现接口函数也需要使用`mut`修饰

2. 如果`class`实现此`interface`, 那么 实现接口函数不能使用`mut`修饰

> 当`struct`的实例赋值给`interface`类型时是拷贝语义, 因此`interface`的`mut`函数并不能修改`struct`实例的值
> 
> 示例：
> 
> ```cangjie
> interface I {
>     mut func f(): Unit
> }
> struct Foo <: I {
>     public var v = 0
>     public mut func f(): Unit {
>         v += 1
>     }
> }
> main() {
>     var a = Foo()
>     var b: I = a  
>     b.f()                                       // 通过'b'调用'f()'不能修改'a'的值
>     println(a.v)                                // 0
> }
> ```
> 
> 程序输出结果为：
> 
> ```text
> 0
> ```

拿`struct`实例 赋值给`interface`变量, 是拷贝语义

此时, 如果使用`interface`变量调用`mut`函数, 也无法修改原`struct`实例成员

#### `mut`函数的使用限制

> 因为`struct`是值类型, 所以如果一个变量是`struct`类型且使用`let`声明, 那么不能通过这个变量访问该类型的`mut`函数
> 
> 示例：
> 
> ```cangjie
> interface I {
>     mut func f(): Unit
> }
> struct Foo <: I {
>     public var i = 0
>     public mut func f(): Unit {
>         i += 1
>     }
> }
> main() {
>     let a = Foo()
>     a.f()                                       // Error, 'a' 是struct, 并且使用 'let' 声明, 因此不能通过 'a' 访问 'mut' 函数
>     var b = Foo()
>     b.f()                                       // Ok
>     let c: I = Foo()
>     c.f()                                       // Ok, 变量 c 为接口 I 类型, 非 struct 类型, 此处允许访问
> }
> ```

`let`修饰的`struct`变量, 无法访问`mut`成员函数

> 为避免逃逸, 如果一个变量的类型是`struct`类型, 那么这个变量不能将该类型使用`mut`修饰的函数作为一等公民来使用, 只能调用这些`mut`函数
> 
> 示例：
> 
> ```cangjie
> interface I {
>     mut func f(): Unit
> }
> 
> struct Foo <: I {
>     var i = 0
> 
>     public mut func f(): Unit {
>         i += 1
>     }
> }
> 
> main() {
>     var a = Foo()
>     var fn = a.f                                // Error, mut 函数 'f' 不能作为 'a' 的一等公民使用
>     var b: I = Foo()
>     fn = b.f                                    // Ok
> }
> ```

`struct`变量只能调用`mut`成员函数, 不能尝试将`mut`成员函数作为一等公民使用

> 为避免逃逸, 非`mut`的实例成员函数(包括`lambda`表达式)不能直接访问所在类型的`mut`函数, 反之可以
> 
> 示例：
> 
> ```cangjie
> struct Foo {
>     var i = 0
> 
>     public mut func f(): Unit {
>         i += 1
>         g()                                     // Ok
>     }
> 
>     public func g(): Unit {
>         f()                                     // Error, mut 函数不能在非 mut 函数中调用
>     }
> }
> 
> interface I {
>     mut func f(): Unit {
>         g()                                     // Ok
>     }
> 
>     func g(): Unit {
>         f()                                     // Error, mut 函数不能在非 mut 函数中调用
>     }
> }
> ```

非`mut`实例成员函数 禁止直接访问同类型的`mut`实例成员函数

这是为了防止`mut`函数逃逸
