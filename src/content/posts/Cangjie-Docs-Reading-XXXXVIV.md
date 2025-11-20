---
title: "仓颉文档阅读-开发指南VII: 类和接口(IV) - 属性的定义与使用"
published: 2025-11-20 17:06:22
description: '仓颉文档阅读的开发指南部分, 本篇文章介绍仓颉语言的 属性 的相关内容'
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

### 属性

> 属性（`Properties`）提供了一个 `getter` 和一个可选的 `setter` 来间接获取和设置值
> 
> **使用属性的时候与普通变量无异**，只需要对数据操作，对内部的实现无感知，可以更便利地实现访问控制、数据监控、跟踪调试、数据绑定等机制
> 
> 属性在使用时可以**作为表达式或被赋值**
> 
> 此处以类和接口为例进行说明，但属性不仅限于类和接口
> 
> 以下是一个简单的例子，`b` 是一个典型的属性，封装了外部对 `a` 的访问：
> 
> ```cangjie
> class Foo {
>     private var a = 0
> 
>     public mut prop b: Int64 {
>         get() {
>             println("get")
>             a
>         }
>         set(value) {
>             println("set")
>             a = value
>         }
>     }
> }
> 
> main() {
>     var x = Foo()
>     let y = x.b + 1       // get
>     x.b = y               // set
> }
> ```
> 
> 此处 `Foo` 提供了一个名为 `b` 的属性，针对 `getter/setter` 这两个功能，仓颉提供了 `get` 和 `set` 两种语法来定义
> 
> 当一个类型为 `Foo` 的变量 `x` 在访问 `b` 时，会调用 `b` 的 `get` 操作返回类型为 `Int64` 的值，因此可以用来与 `1` 相加
> 
> 而当 `x` 在对 `b` 进行赋值时，会调用 `b` 的 `set` 操作，将 `y` 的值传给 `set` 的 `value`，最终将 `value` 的值赋值给 `a`
> 
> 通过属性 `b`，外部对 `Foo` 的成员变量 `a` 完全不感知，但却可以通过 `b` 做到同样地访问和修改操作，实现了有效的封装性
> 
> 所以程序的输出如下：
> 
> ```text
> get
> set
> ```

属性, 使用关键字`prop`声明, 语法为:

```cangjie
class Sample {
    prop p: Int64
}
```

仓颉属性, 提供了两个功能:

1. `getter`, 可以通过属性获取目标数据

    通过属性的`get()`语法实现

2. `setter`, 可以通过属性设置目标数据

    通过属性的`set()`语法实现

虽然属性可以声明类型, 但是**属性本身并不存储数据**, 属性的类型是是给`getter`和`setter`使用的

`getter`的功能是, 通过属性获取目标数据

属性自身并不存储数据, 但是通过 访问属性, 可以**调用对应属性实现的`get()`**, `get()`返回值就是要获取的目标数据

同样的, 如果 给属性赋值, 可以调用对应属性的`set(value)`, `set()`就会接收数据并执行内部代码

而`get()`返回值类型 与 `set(value)`接收的`value`的类型, 就是属性声明的类型

**简单理解, 属性的类型, 其实是给`get()`声明返回值类型, 是给`set()`声明参数类型**

就像文档中提供的例子, 属性`b`, `getter`实际返回的是私有成员变量`a: Int64`, `setter`实际是给私有成员变量`a: Int64`赋值

所以, 而私有成员变量`a`是`Int64`类型的, 很显然 获取它要使用`Int64`类型, 给它赋值也要接收`Int64`类型数据, 所以属性`b`声明为`Int64`类型

这就是属性需要声明类型的原因, 这也是 接口能够定义成员属性, 但也可以多继承的原因(属性只是提供了一个间接的访问数据、修改数据的方法而不存储数据)

#### 属性定义

> 属性可以在 `interface`、`class`、`struct`、`enum`、`extend` 中定义
> 
> 一个典型的属性语法结构如下：
> 
> ```cangjie
> class Foo {
>     public prop a: Int64 {
>         get() { 0 }
>     }
>     public mut prop b: Int64 {
>         get() { 0 }
>         set(v) {}
>     }
> }
> ```
> 
> 其中使用 `prop` 声明的 `a` 和 `b` 都是属性，`a` 和 `b` 的类型都是 `Int64`
> 
> `a` 是 **无`mut`修饰符** 的属性，这类属性**有且仅有定义`getter`（对应取值）**实现
> 
> `b` 是 **使用`mut`修饰** 的属性，这类属性**必须分别定义`getter`（对应取值）和`setter`（对应赋值）**的实现
> 
> > 注意：
> > 
> > 对于数值、元组、函数、`Bool`、`Unit`、`Nothing`、`String`、`Range` 和 `enum` 类型，在它们的扩展和声明中不能声明`mut`修饰的属性，也不能实现有`mut`属性的接口
> 
> 属性的 `getter` 和 `setter` 分别对应两个不同的函数
> 
> 1. `getter` 函数类型是 `() -> T`，`T` 是该属性的类型，当使用该属性作为表达式时会执行 `getter` 函数
> 
> 2. `setter` 函数类型是 `(T) -> Unit`，`T` 是该属性的类型，形参名需要显式指定，当对该属性赋值时会执行 `setter` 函数
> 
> `getter` 和 `setter` 的实现中可以和函数体一样包含声明和表达式，与函数体的规则一样，详见 [函数体](https://www.humid1ch.cn/posts/cangjie-docs-reading-xxxvi/#函数体) 章节
> 
> `setter` 中的参数对应的是赋值时传入的值
> 
> ```cangjie
> class Foo {
>     private var j = 0
>     public mut prop i: Int64 {
>         get() {
>             j
>         }
>         set(v) {
>             j = v
>         }
>     }
> }
> ```
> 
> 需要注意的是，**在属性的`getter`和`setter`中访问属性自身属于递归调用**，与函数调用一样可能会出现死循环的情况

仓颉属性, 没有`mut`的属性只能实现且必须实现`getter`, 有`mut`修饰的属性才能实现且必须额外实现`setter`

与上面分析的一致, 属性的类型声明, 是为`getter`和`setter`使用的, 是`get()`的返回值类型, 是`set(value)`的参数类型

##### 修饰符

> 可以在 `prop` 前面声明需要的修饰符
> 
> ```cangjie
> class Foo {
>     public prop a: Int64 {
>         get() {
>             0
>         }
>     }
>     private prop b: Int64 {
>         get() {
>             0
>         }
>     }
> }
> ```
> 
> 和成员函数一样，成员属性也支持 `open`、`override`、`redef` 修饰，所以也可以在子类型中覆盖/重定义父类型属性的实现
> 
> 子类型覆盖父类型的属性时，如果父类型属性带有 `mut` 修饰符，则子类型属性也需要带有 `mut` 修饰符，同时也必须保持一样的类型
> 
> 如下代码所示，`A` 中定义了 `x` 和 `y` 两个属性，`B` 中可以分别对 `x` 和 `y` 进行 `override/redef`：
> 
> ```cangjie
> open class A {
>     private var valueX = 0
>     private static var valueY = 0
> 
>     public open prop x: Int64 {
>         get() { valueX }
>     }
> 
>     public static mut prop y: Int64 {
>         get() { valueY }
>         set(v) {
>             valueY = v
>         }
>     }
> }
> class B <: A {
>     private var valueX2 = 0
>     private static var valueY2 = 0
> 
>     public override prop x: Int64 {
>         get() { valueX2 }
>     }
> 
>     public redef static mut prop y: Int64 {
>         get() { valueY2 }
>         set(v) {
>             valueY2 = v
>         }
>     }
> }
> ```

属性也可以通过可访问性修饰符修饰, 具体可访问性限制不再赘述, 与其他成员的可访问性限制保持一致

属性除了实例成员属性之外, 还可以定义**静态成员属性**

既然有静态成员属性, 那么就有实例成员和静态成员之间的访问问题

1. 静态成员属性

    只能访问其他静态成员变量, 不能访问实例成员变量

    只能访问其他静态成员属性, 不能访问实例成员属性

    只能访问其他静态成员函数, 不能访问实例成员函数

2. 实例成员属性

    可以通过类型名访问其他静态成员变量, 也可以访问其他实例成员变量

    可以通过类型名访问其他静态成员属性, 也可以访问其他实例成员属性

    可以通过类型名访问其他静态成员函数, 也可以访问其他实例成员函数

其实总结起来, 仓颉各类型:

**静态成员函数或成员属性, 只能访问其他静态成员, 不能访问实例成员**

**而实例成员函数和成员属性, 可以访问其他实例成员, 也可以通过类型名访问静态成员**

另外, 属性也可以用`open`修饰, 表示可以覆盖或重定义, `override`和`redef`同样也是可选的

覆盖或重定义时, 属性是否被`mut`修饰, 属性类型 都要与父类型保持一致

##### 抽象属性

> **类似抽象函数**，在 `interface`和抽象类中也可以声明抽象属性，这些抽象属性**没有实现**
> 
> ```cangjie
> interface I {
>     prop a: Int64
> }
> 
> abstract class C {
>     public prop a: Int64
> }
> ```
> 
> 当实现类型实现 `interface`或者非抽象子类继承抽象类时，**必须要实现这些抽象属性**
> 
> 与覆盖的规则一样，实现类型或子类在实现这些属性时，如果父类型属性带有`mut`修饰符，则子类型属性也需要带有`mut`修饰符，同时也必须保持一样的类型
> 
> ```cangjie
> interface I {
>     prop a: Int64
>     mut prop b: Int64
> }
> class C <: I {
>     private var value = 0
> 
>     public prop a: Int64 {
>         get() { value }
>     }
> 
>     public mut prop b: Int64 {
>         get() { value }
>         set(v) {
>             value = v
>         }
>     }
> }
> ```
> 
> 通过抽象属性，可以让接口和抽象类对一些数据操作能以更加易用的方式进行约定，相比函数的方式要更加直观
> 
> 如下代码所示，如果要对一个 `size` 值的获取和设置进行约定，使用属性的方式(`I1`)相比使用函数的方式(`I2`)代码更少，也更加符合对数据操作的意图
> 
> ```cangjie
> interface I1 {
>     mut prop size: Int64
> }
> 
> interface I2 {
>     func getSize(): Int64
>     func setSize(value: Int64): Unit
> }
> 
> class C <: I1 & I2 {
>     private var mySize = 0
> 
>     public mut prop size: Int64 {
>         get() {
>             mySize
>         }
>         set(value) {
>             mySize = value
>         }
>     }
> 
>     public func getSize() {
>         mySize
>     }
> 
>     public func setSize(value: Int64) {
>         mySize = value
>     }
> }
> 
> main() {
>     let a: I1 = C()
>     a.size = 5
>     println(a.size)
> 
>     let b: I2 = C()
>     b.setSize(5)
>     println(b.getSize())
> }
> ```
> 
> ```text
> 5
> 5
> ```

抽象属性, 就是没有具体实现, 只有声明的属性

可以在`interface`和抽象类中定义, 实现或继承的类型必须要实现
