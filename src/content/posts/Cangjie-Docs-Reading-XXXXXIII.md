---
title: "仓颉文档阅读-开发指南VIII: 泛型(III) - 泛型的子类型关系、类型别名以及泛型约束"
published: 2025-11-24 17:36:32
description: '仓颉文档阅读的开发指南部分, 本篇文章介绍仓颉语言的 泛型 的相关内容'
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

## 泛型

### 泛型类型的子类型关系 **

> 实例化后的泛型类型间也有子类型关系
> 
> 例如：
> 
> ```cangjie
> interface I<X, Y> { }
> 
> class C<Z> <: I<Z, Z> { }
> ```
> 
> 根据 `class C<Z> <: I<Z, Z> { }`，便知 `C<Bool> <: I<Bool, Bool>` 以及 `C<D> <: I<D, D>` 等
> 
> 可以解读为“于所有的(不含类型变元的)`Z`类型，都有`C<Z> <: I<Z, Z>`成立”
> 
> 但是对于下列代码：
> 
> ```cangjie
> open class C { }
> class D <: C { }
> 
> interface I<X> { }
> ```
> 
> **`I<D> <: I<C>`是不成立的**(即使 `D <: C` 成立)，这是因为在仓颉语言中，用户定义的类型构造器 在其类型参数处是**不型变**的
> 
> **型变的具体定义为**：
> 
> 如果`A`和`B`是(实例化后的)类型，`T`是类型构造器，设有一个类型参数`X`(例如`interface T<X>`)，那么
> 
> - 如果`T(A) <: T(B)`当且仅当`A = B`，则`T`是不型变的
> 
> - 如果`T(A) <: T(B)`当且仅当`A <: B` ，则`T`在`X`处是协变的
> 
> - 如果`T(A) <: T(B)`当且仅当`B <: A` ，则`T`在`X`处是逆变的
> 
> 因为现阶段的仓颉中，**所有 用户自定义的泛型类型 在其所有的类型变元处都是不变的**
> 
> 所以给定`interface I<X>`和类型`A`、`B`，只有`A = B`，才能得到 `I<A> <: I<B>`
> 
> 反过来，如果知道了 `I<A> <: I<B>`，也可推出 `A = B`
> 
> 内建类型除外：**内建的元组类型**对其每个元素类型来说，都是**协变**的；**内建的函数类型**在其入参类型处是**逆变**的，在其返回类型处是**协变**的
> 
> > 注意：
> > 
> > `class` 以外的类型实现接口，该类型和该接口之间的子类型关系不能作为协变和逆变的依据
> 
> 不型变限制了一些语言的表达能力，但也避免了一些安全问题，例如“协变数组运行时抛异常”的问题

仓颉泛型的父子类型关系, 与一个概念密切相关: **型变**

关于**型变** 仓颉规定, 针对自定义类型`Type`, 如果存在类型形参`X`, 以及另外的已实例化类型`A`和`B`:

1. 如果 有且仅有`A = B`时, 才存在`Type(A) <: Type(B)`, 那么 可称`Type`是**不型变**的

2. 如果 有且仅有`A <: B`时, 才存在`Type(A) <: Type(B)`, 那么 可称`Type`是**协变**的

3. 如果 有且仅有`B <: A`时, 才存在`Type(A) <: Type(B)`, 那么 可称`Type`是**逆变**的

同时仓颉规定, **现阶段, 所有用户自定义类型 都是不型变的**

这也就意味着, 如果类型形参不同, 那么泛型自定义类型之间不可能存在父子类型关系

> [!TIP]
> 
> 文档描述, 内建类型除外:
> 
> 1. 内建元组类型 对其每个元素类型, 都是协变的
> 
> 2. 内建函数类型 对其形参列表类型是逆变的, 对其返回类型是协变的
> 
> 这表示, 元组类型, 只要对应位置元素类型具有同向的父子类型关系, 那么元组类型之间就具有相同的父子类型关系
> 
> 而 函数类型, 只要 参数列表每个形参对应位置类型 具有同向的父子类型关系, 且 返回值 具有与参数列表相反的父子类型关系, 那么 函数类型就具有与返回值类型相同的父子类型关系

### 类型别名

> 当某个类型的名字比较复杂或者在特定场景中不够直观时，可以选择使用类型别名的方式为此类型设置一个别名
> 
> ```cangjie
> type I64 = Int64
> ```
> 
> 类型别名的定义以 **关键字`type`** 开头，接着是类型的别名（如上例中的 `I64`），然后是等号 `=`，最后是原类型（即被取别名的类型，如上例中的 `Int64`）
> 
> **只能在源文件顶层定义类型别名，并且原类型必须在别名定义处可见**
> 
> 例如，下例中 `Int64` 的别名定义在 `main` 中将报错，`LongNameClassB` 类型在为其定义别名时不可见，同样报错
> 
> ```cangjie
> main() {
>     type I64 = Int64          // Error, 类型别名只能在 源文件的顶层 定义
> }
> 
> class LongNameClassA { }
> type B = LongNameClassB       // Error, 'LongNameClassB' 类型没有被定义
> ```
> 
> 一个（或多个）类型别名定义中**禁止出现（直接或间接的）循环引用**
> 
> ```cangjie
> type A = (Int64, A)           // Error, 'A' 循环引用自己
> type B = (Int64, C)           // Error, 'B' 和 'C' 相互循环引用
> type C = (B, Int64)
> ```
> 
> 类型别名并**不会定义一个新的类型**，它仅仅是为原类型定义了另外一个名字，它有如下几种使用场景：
> 
> 1. **作为类型使用**，例如：
> 
>     ```cangjie
>     type A = B
>     class B {}
>     var a: A = B()            // 使用 类型别名A 作为类型 B
>     ```
> 
> 2. 当类型别名实际指向的类型为`class`、`struct`时，可以 **作为构造器名称使用**：
> 
>     ```cangjie
>     type A = B
>     class B {}
>     func foo() { A() }        // 使用 类型别名A 作为 B 的构造函数
>     ```
> 
> 3. 当类型别名实际指向的类型为`class`、`interface`、`struct`时，可以 **作为访问内部静态成员变量或函数的类型名**：
> 
>     ```cangjie
>     type A = B
>     class B {
>         static var b : Int32 = 0;
>         static func foo() {}
>     }
>     func foo() {
>         A.foo()               // 使用 类型别名A 访问 类B 的静态方法
>         A.b
>     }
>     ```
> 
> 4. 当类型别名实际指向的类型为 `enum` 时，可以 **作为`enum`声明的构造器的类型名**：
> 
>     ```cangjie
>     enum TimeUnit {
>         Day | Month | Year
>     }
>     type Time = TimeUnit
>     var a = Time.Day  
>     var b = Time.Month        // 使用 类型别名Time 来访问 TimeUnit 中的构造函数
>     ```
> 
> 需要注意的是，当前用户自定义的类型别名 **暂不支持在类型转换表达式中使用**，参考如下示例：
> 
> ```cangjie
> type MyInt = Int32
> MyInt(0)                      // Error, 没有匹配的函数用于 操作符'()' 函数调用
> ```

仓颉允许针对类型起别名， 可以类比C语言的`typedef`

语法为:

```cangjie
type 别名 = 实际类姓名
```

类型别名并不是定义一个新的类型, 而是为已有类型重新起一个名字

但, 类型别名只能在顶层作用域定义, 且 不能在类型转换表达式中使用

#### 泛型别名

> 类型别名也是**可以声明类型形参**的，但是不能对其形参使用 `where` 声明约束，对于泛型变元的约束会在后面给出解释
> 
> 当一个泛型类型的名称过长时，可以使用类型别名来为其声明一个更短的别名
> 
> 例如，有一个类型为 `RecordData` ，可以把他用类型别名简写为`RD`：
> 
> ```cangjie
> struct RecordData<T> {
>     var a: T
>     public init(x: T){
>         a = x
>     }
> }
> 
> type RD<T> = RecordData<T>
> 
> main(): Int64 {
>     var struct1: RD<Int32> = RecordData<Int32>(2)
>     return 1
> }
> ```
> 
> 在使用时就可以用 `RD<Int32>` 来代指 `RecordData<Int32>` 类型

仓颉类型别名可以声明类型形参, 也就意味着 也可以给泛型类型起别名, 被称为泛型别名

但 **泛型别名不能使用泛型约束**

泛型别名的语法是:

```cangjie
type 别名<类型形参列表> = 泛型类型<类型形参列表>
```

泛型别名在使用上与泛型类型保持一致

### 泛型约束

> 泛型约束的作用是在`function`、`class`、`interface`、`struct`、`enum`声明时, **明确泛型形参所具备的操作与能力**
> 
> 只有声明了这些约束才能调用相应的成员函数
> 
> 在很多场景下泛型形参是需要加以约束的
> 
> 以 `id` 函数为例：
> 
> ```cangjie
> func id<T>(a: T) {
>     return a
> }
> ```
> 
> 开发者唯一能做的事情就是将函数形参`a`这个值返回，而不能进行 `a + 1`，`println("${a}")` 等操作，因为它可能是一个任意的类型，比如 `(Bool) -> Bool`，这样就无法与整数相加，同样因为是函数类型，也不能通过 `println` 函数来输出在命令行上
> 
> 而如果这一泛型形参上有了约束，那么就可以做更多操作了
> 
> 约束大致分为接口约束与 `class` 类型约束
> 
> 语法为在函数、类型的声明体之前使用 `where` 关键字来声明，对于声明的泛型形参 `T1`, `T2`，可以使用 `where T1 <: Interface`, `T2 <: Class` 这样的方式来声明泛型约束，同一个类型变元的多个约束可以使用 `&` 连接
> 
> 例如：`where T1 <: Interface1 & Interface2`
> 
> 仓颉中的 `println` 函数能接受类型为字符串的参数
> 
> 如果需要把一个泛型类型的变量转为字符串后打印在命令行上，可以对这个泛型类型变元加以约束，这个约束是 `core` 中定义的 `ToString` 接口，显然它是一个接口约束：
> 
> ```cangjie
> package std.core              // `ToString` 被定义在`core`里
> 
> public interface ToString {
>     func toString(): String
> }
> ```
> 
> 这样就可以利用这个约束，定义一个名为 `genericPrint` 的函数：
> 
> ```cangjie
> func genericPrint<T>(a: T) where T <: ToString {
>     println(a)
> }
> 
> main() {
>     genericPrint<Int64>(10)
>     return 0
> }
> ```
> 
> 结果为：
> 
> ```text
> 10
> ```
> 
> 如果 `genericPrint` 函数的类型实参没有实现 `ToString` 接口，那么编译器会报错
> 
> 例如传入一个函数做为参数时：
> 
> ```cangjie
> func genericPrint<T>(a: T) where T <: ToString {
>     println(a)
> }
> 
> main() {
>     genericPrint<(Int64) -> Int64>({ i => 0 })
>     return 0
> }
> ```
> 
> 如果对上面的文件进行编译，那么编译器会抛出 **泛型类型参数不满足约束的错误**
> 
> 因为 `genericPrint` 函数的泛型的类型实参不满足约束 `(Int64) -> Int64 <: ToString`
> 
> 除了上述通过接口来表示约束，还可以使用 `class` 类型来约束一个泛型类型变元
> 
> 例如：当要声明一个动物园类型 `Zoo<T>`，但是需要这里声明的类型形参 `T` 受到约束，这个约束就是 `T` 需要是动物类型 `Animal` 的子类型， `Animal` 类型中声明了 `run` 成员函数
> 
> 这里声明两个子类型 `Dog` 与 `Fox` 都实现了 `run` 成员函数，这样在 `Zoo<T>` 的类型中，就可以对于 `animals` 数组列表中存放的动物实例调用 `run` 成员函数：
> 
> ```cangjie
> import std.collection.*
> 
> abstract class Animal {
>     public func run(): String
> }
> 
> class Dog <: Animal {
>     public func run(): String {
>         return "dog run"
>     }
> }
> 
> class Fox <: Animal {
>     public func run(): String {
>         return "fox run"
>     }
> }
> 
> class Zoo<T> where T <: Animal {
>     var animals: ArrayList<Animal> = ArrayList<Animal>()
>     public func addAnimal(a: T) {
>         animals.add(a)
>     }
> 
>     public func allAnimalRuns() {
>         for(a in animals) {
>             println(a.run())
>         }
>     }
> }
> 
> main() {
>     var zoo: Zoo<Animal> = Zoo<Animal>()
>     zoo.addAnimal(Dog())
>     zoo.addAnimal(Fox())
>     zoo.allAnimalRuns()
>     return 0
> }
> ```
> 
> 程序的输出为：
> 
> ```text
> dog run
> fox run
> ```
> 
> > 注意：
> > 
> > 泛型变元的约束只能是具体的 `class` 类型或 `interface`，且变元如果存在多个 `class` 类型的上界时，它们必须在同一继承链路上

仓颉泛型约束其实并不复杂:

```cangjie
class 类姓名<T> where T <: 目标约束类型 {}

func 函数名<T>() where T <: 目标约束类型 {}

// ...
```

泛型约束的语法, 就是在定义泛型时, 后接`where 目标类型变元 <: 目标约束类型`

泛型约束是为了:

1. 保证类型变元, 属于目标类型的子类型

    如此, 在 泛型类型、泛型函数体 内使用类型变元时, 就能访问目标类型的成员

    文档中举例子, `genericPrint<T>(a: T) where T <: ToString`

    此时, 在函数体内, 就能够通过`print`或`println`等函数, 直接打印形参`a`

    因为泛型约束, 保证了`a`的类型实现了`ToString`接口, 所以可以直接调用`print`或`println`

    否则, 就不能直接调用`print`或`println`打印

    约束其他类型时, 也是相同的作用

    只要使用了泛型约束, **在泛型内 就能访问目标约束类型中的成员**

2. 限制类型实参, 属于目标类型的子类

    同样以`genericPrint`为例, 在调用`genericPrint`时, 必须保证调用时 类型实参实现了`ToString`接口

    否则编译直接报错
