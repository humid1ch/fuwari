---
title: "仓颉文档阅读-开发指南VII: 类和接口(I) - 类的定义"
published: 2025-11-18 16:26:06
description: '仓颉文档阅读的开发指南部分, 本篇文章介绍仓颉语言的类的相关内容'
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

### 类

> `class`类型是面向对象编程中的经典概念, 仓颉中同样支持使用`class`来实现面向对象编程
> 
> `class`与`struct`的主要区别在于: `class`是引用类型, `struct`是值类型, 它们在赋值或传参时行为是不同的
> 
> **`class`之间可以继承, 但`struct`之间不能继承**
> 
> 本节依次介绍如何定义`class`类型, 如何创建对象, 以及`class`的继承

仓颉的类也是用`class`定义的

具体本篇文章会介绍

#### `class`定义

> `class`类型的定义以关键字`class`开头, 后跟`class`的名字, 接着是定义在一对花括号中的`class`定义体
> 
> `class`定义体中可以定义一系列的成员变量、成员属性(参见 [属性]() )、静态初始化器、构造函数、成员函数和操作符函数(详见 [操作符重载]() )
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
>         width * height
>     }
> }
> ```
> 
> 上例中定义了名为`Rectangle`的`class`类型, 它有两个`Int64`类型的成员变量`width`和`height`, 一个有两个`Int64`类型参数的构造函数, 以及一个成员函数`area`(返回`width`和`height`的乘积)
> 
> > 注意: 
> > 
> > `class`只能定义在源文件的顶层作用域

仓颉`class`, 语法很简单, 也可以定义成员变量、函数等

但与C++不同的是, 仓颉`class`可以定义属性

且 仓颉`class`的成员的可访问性, 是使用可访问性修饰符**单独修饰**的, 而不是使用修饰符修饰范围

```cangjie
class ClassIdent {
    let memberVar1 = 0
    public var memberVar2 = 0               // 使用可访问性修饰符, 单独修饰成员
    
    func memberFunction1() {}

    public func memberFunction2() {}        // 使用可访问性修饰符, 单独修饰成员
}
```

仓颉`class`中, 可以使用`this`表示本实例, 可以以此访问成员, 与C++类中的`this`指针有些类似, 但又不同

> 使用`abstract`修饰的类为抽象类, 与普通类不同的是, 在抽象类中除了可以定义普通的函数, 还允许声明抽象函数(没有函数体)
> 
> 抽象类定义时的`open`修饰符是可选的, 也可以使用`sealed`修饰符修饰抽象类, 表示该抽象类只能在本包被继承
> 
> 下例中在抽象类`AbRectangle`中定义了抽象函数`foo`
> 
> ```cangjie
> abstract class AbRectangle {
>     public func foo(): Unit
> }
> ```
> 
> > 注意: 
> > 
> > - 抽象类中禁止定义`private`的抽象函数
> > 
> > - 不能为抽象类创建实例
> > 
> > - 抽象类的非抽象子类必须实现父类中的所有抽象函数

仓颉的抽象类, 使用`abstract`修饰符显式修饰

关于抽象类, 自动包含`open`属性, 可以**使用`sealed`修饰, 限定只能在本包中被继承**

仓颉中, 只有显式使用`abstract`修饰符修饰的类, 才能定义抽象函数(无函数体)

这与C++不同, C++是存在纯虚函数的类 就是抽象类

但抽象类的其他概念是一样的, 不能创建实例, 不能定义`private`抽象函数, 子类必须实现抽象函数

##### `class`成员变量

> `class`成员变量分为**实例成员变量**和**静态成员变量**
> 
> **静态成员变量**使用`static`修饰符修饰, **没有静态初始化器时必须有初值**, **只能通过类型名访问**, 参考如下示例: 
> 
> ```cangjie
> class Rectangle {
>     let width = 10
>     static let height = 20
> }
> 
> let l = Rectangle.height        // l = 20
> ```
> 
> **实例成员变量**定义时可以不设置初值(但必须标注类型), 也可以设置初值, 只能通过对象(即类的实例)访问, 参考如下示例: 
> 
> ```cangjie
> class Rectangle {
>     let width = 10
>     let height: Int64
>     init(h: Int64){
>         height = h
>     }
> }
> let rec = Rectangle(20)
> let l = rec.height              // l = 20
> ```

仓颉`class`的成员变量, 分为 **实例成员变量(属于某个具体实例)** 和 **静态成员变量(只属于`class`)**

静态成员变量, 只能 直接在定义时初始化 或 在静态初始化器中初始化, 二者必选其一

实例成员变量, 也是一样的 要不 直接在定义时初始化, 要不 需要在构造函数中初始化

成员变量, 如果不直接在定义时初始化时, 定义时**必须声明变量类型**

从实际来看, **所有成员变量必须要在 定义时、静态初始化器内或构造函数内 完成初始化**

> [!WARNING]
> 
> 当前版本的文档描述, 存在一定的误导性
> 
> 文档中只针对静态成员变量强调: **没有静态初始化器时必须有初值**
> 
> 但实际上, 还要在静态初始化器中对其完成初始化
> 
> 而且, 即使是实例成员变量, 也是一样的 如果没有在构造函数中完成初始化, 也必须有初值

##### `class`静态初始化器

> `class`支持定义静态初始化器, 并在静态初始化器中通过赋值表达式来对静态成员变量进行初始化
> 
> 静态初始化器以关键字组合`static init`开头, 后跟无参参数列表和函数体, 且不能被访问修饰符修饰
> 
> 函数体中必须完成对所有未初始化的静态成员变量的初始化, 否则编译报错
> 
> ```cangjie
> class Rectangle {
>     static let degree: Int64
>     static init() {
>         degree = 180
>     }
> }
> ```
> 
> 一个`class`中**最多允许定义一个静态初始化器**, 否则报重定义错误
> 
> ```cangjie
> class Rectangle {
>     static let degree: Int64
>     static init() {
>         degree = 180
>     }
>     static init() {               // Error, 与之前静态初始化器 重定义
>         degree = 180
>     }
> }
> ```

仓颉`class`的静态初始化器, 是专为静态成员变量初始化用的

静态初始化器, 无参数无返回值:

```cangjie
class StaticInit {
    static let staticVar: Int64
    static init() {
        staticVar = 20
    }
}
```

##### `class`构造函数

> 和`struct`一样, `class`中也支持定义**普通构造函数**和**主构造函数**
> 
> 普通构造函数以关键字`init`开头, 后跟参数列表和函数体, 函数体中**必须完成所有未初始化实例成员变量的初始化**, 否则编译报错
> 
> ```cangjie
> class Rectangle {
>     let width: Int64
>     let height: Int64
> 
>     public init(width: Int64, height: Int64) {            // Error, 'height' 在构造函数中未初始化
>         this.width = width
>     }
> }
> ```
> 
> 一个类中可以定义多个普通构造函数, 但它们必须构成重载(参见 [函数重载]() ), 否则报重定义错误
> 
> ```cangjie
> class Rectangle {
>     let width: Int64
>     let height: Int64
> 
>     public init(width: Int64) {
>         this.width = width
>         this.height = width
>     }
> 
>     public init(width: Int64, height: Int64) {            // Ok: 与第一个构造函数 构成重载
>         this.width = width
>         this.height = height
>     }
> 
>     public init(height: Int64) {                          // Error, 与 第一个构造函数 重定义
>         this.width = height
>         this.height = height
>     }
> }
> ```
> 
> 除了可以定义若干普通的以`init`为名字的构造函数外, `class`内还可以定义**最多一个主构造函数**
> 
> 主构造函数的名字和`class`类型名相同, 它的参数列表中可以有两种形式的形参: **普通形参**和**成员变量形参(需要在参数名前加上`let`或`var`)**, 成员变量形参同时具有**定义成员变量和构造函数参数**的功能
> 
> 使用主构造函数通常可以简化`class`的定义, 例如, 上述包含一个`init`构造函数的`Rectangle`可以简化为如下定义: 
> 
> ```cangjie
> class Rectangle {
>     public Rectangle(let width: Int64, let height: Int64) {}
> }
> ```
> 
> 主构造函数的参数列表中也可以定义普通形参, 例如: 
> 
> ```cangjie
> class Rectangle {
>     public Rectangle(name: String, let width: Int64, let height: Int64) {}
> }
> ```

仓颉`class`的构造函数与仓颉`struct`一样, 可定义 普通构造函数(函数名`init`) 和 主构造函数(函数名同`class`名)

仓颉`class`的构造函数:

1. 必须在构造函数内 对**所有未初始化的成员变量** 完成初始化

2. 可以定义多个构造函数, 但 构造函数之间**必须能够构成重载**

```cangjie
class ClassInit {
    public init(参数列表) {}
}
```

仓颉`class`还可以定义主构造函数, 即 **与`class`同名的函数**

主构造函数除普通形参外, 还可以定义**成员变量形参**, 这是主构造函数与普通构造函数最大的区别

成员变量形参, 就是在参数列表中使用`let`或`var`修饰普通形参, 可以同时实现 **定义成员变量和构造函数参数**

关于主构造函数的成员变量形参, 存在一个限制: 无法在主构造函数内使用形参进行赋值, 为什么呢? 

因为函数体内**直接使用成员变量形参名, 访问的是形参, 是不可变变量**

不过, 可以通过`this`访问成员变量, 以此实现对成员变量的再此赋值, 当然 能够赋值的前提是 成员变量是`var`的

> [!NOTE]
> 
> 仓颉`class`的主构造函数, 在编译时会被转为等价的`init`构造函数, 然后再进行编译
> 
> 所以, 在定义主构造函数时, 要注意不要与其他`init`构造函数重定义了, 要确保转换后的`init`构造函数 与 定义的其他`init`构造函数构成重载
> 
> 按照语法规约, 携带成员变量形参的主构造函数, 会发生这样的转换:
> 
> ```cangjie
> class Sample {
>     public Sample(
>         name: String,
>         var width: Int64,
>         private var length!: Int64,
>         private let height!: Int64 = 3
>     ) {}
> 
>     /* 将会转换为
>     var width: Int64
>     private var length: Int64
>     private let height: Int64
>     public init(name: String, width: Int64, length!: Int64, height!: Int64 = 3) {
>         this.width = width
>         this.length = length
>         this.height = height
>     }
>     */ 
> }
> ```
> 
> 根据这样的规则, 可以看到其实会将成员变量形参 先定义为成员变量, 然后声明同名形参, 然后再函数体内自动生成初始化语句
> 
> 且, 在 主构造函数参数列表中 出现的**成员变量形参**, 也**需要在其他`init`构造函数内进行初始化**
> 
> 因为, 成员变量形参 实际上就是成员变量

> 创建类的实例时调用的构造函数, 将**根据以下顺序执行**类中的表达式: 
> 
> 1. 先初始化主构造函数之外定义的有缺省值的变量
> 
> 2. 如果构造函数体内未显式调用父类构造函数或本类其他构造函数, 则调用父类的无参构造函数`super()`, 如果父类没有无参构造函数, 则报错
> 
> 3. 执行构造函数体内的代码
> 
> ```cangjie
> func foo(x: Int64): Int64 {
>     println("I'm foo, got ${x}")
>     x
> }
> 
> open class A {
>     init() {
>         println("I'm A")
>     }
> }
> 
> class B <: A {
>     var x = foo(0)
>     init() {
>         x = foo(1)
>         println("init B finished")
>     }
> }
> 
> main() {
>     B()
>     0
> }
> ```
> 
> 上述例子中, 调用`B`的构造函数时, **首先初始化有缺省值的变量`x`**, 此时`foo(0)`被调用; **之后调用父类的无参构造函数**, 此时`A`的构造函数被调用; **接下来执行构造函数体内的代码**, 此时`foo(1)`被调用, 并打印字符串
> 
> 因此上例的输出为: 
> 
> ```text
> I'm foo, got 0
> I'm A
> I'm foo, got 1
> init B finished
> ```
> 
> 如果`class`定义中不存在自定义构造函数(包括主构造函数), 并且**所有实例成员变量都有初始值**, 则会自动为其生成一个无参构造函数(调用此无参构造函数会创建一个所有实例成员变量的值均等于其初值的对象); 否则, 不会自动生成此无参构造函数
> 
> 例如, 对于如下`class`定义, 编译器会为其自动生成一个无参构造函数: 
> 
> ```cangjie
> class Rectangle {
>     let width = 10
>     let height = 20
> 
>     /* 自动生成 无参构造函数:
>     public init() {
> 
>     }
>     */
> }
> 
> // 调用自动生成的无参数构造函数
> let r = Rectangle() // r.width = 10, r.height = 20
> ```

仓颉`class`创建实例 调用构造函数时, 是按照这样的执行顺序执行的:

1. 最先初始化, 不在主构造函数内的 拥有缺省值的 成员变量

2. 如果有父类:

    > [!TIP]
    > 
    > **仓颉规定, 可以在构造函数内 显式通过`this(参数)`调用本类其他构造函数, 如果存在父类 还可以显式通过`super(参数)`调用父类构造函数**
    > 
    > 且 均只能作为构造函数内的第一个表达式, 所以 构造函数内 显式调用本类其他构造函数 和 显式调用父类构造函数 **不能共存**

    1. 如果在构造函数内 没有显式调用父类构造函数, 也没有显式调用本类其他构造函数

        那么, 就**自动通过`super()`调用父类的无参构造函数**, 如果父类没有无参构造函数 则报错

    2. 如果构造函数内 没有显式调用父类构造函数 显式调用了其他构造函数

        那么, 就执行函数体内的代码, 即 先去执行其他构造函数(因为是 构造函数内首个表达式)

        但同样会判断是否 显式调用父类构造函数 和 本类其他构造函数

        要注意 不能存在构造函数之间相互递归

    3. 然后再执行, 函数体内的代码

3. 如果没有父类, 就执行函数体内的代码

仓颉`class`, 如果所有成员变量均在定义时就完成了初始化, 那么 如果不显式实现`init()`构造函数, 编译器也会自动生成一个无参构造函数

##### `class`终结器

> `class`支持定义终结器, 这个函数在类的实例**被垃圾回收的时候**被调用
> 
> **终结器的函数名固定为`~init`**
> 
> 终结器通常用于释放**系统资源**: 
> 
> ```cangjie
> class C {
>     var p: CString
> 
>     init(s: String) {
>         p = unsafe { LibC.mallocCString(s) }
>         println(s)
>     }
>     ~init() {
>         unsafe { LibC.free(p) }
>     }
> }
> ```
> 
> 使用终结器有些限制条件, 需要开发者注意: 
> 
> 1. 终结器没有参数, 没有返回类型, 没有泛型类型参数, 没有任何修饰符, 也**不可以被显式调用**
> 
> 2. 带有终结器的类不可被`open`修饰, 只有非`open`的类可以拥有终结器
> 
> 3. 一个类最多**只能定义一个终结器**
> 
> 4. 终结器**不可以定义在扩展中**
> 
> 5. 终结器被触发的时机是不确定的
> 
> 6. 终结器可能在任意一个线程上执行
> 
> 7. 多个终结器的执行顺序是不确定的
> 
> 8. 终结器**向外抛出未捕获异常属于未定义行为**
> 
> 9. 终结器中**创建线程或者使用线程同步功能属于未定义行为**
> 
> 10. 终结器执行结束之后, 如果这个对象还可以被继续访问, 则属于未定义行为
> 
> 11. 如果对象在初始化过程中抛出异常, 这样未完整初始化的对象的终结器不会执行
> 
> 12. 依赖终结器的同步行为属于未定义行为
> 
>     例如, 下例中`main`函数通过`while (Test.t0 != 0)`等待`Test`类中的终结器修改`t0`的值, 属于未定义行为
>     
>     ```cangjie
>     import std.collection.*
>     import std.runtime.*
>         
>     class Test {
>         public static var t0 : Int32 = 0
>         public init () {
>             t0++
>         }
>         ~init () {
>             t0--
>         }
>     }
>         
>     var list: ArrayList<Test> = ArrayList<Test>()
>         
>     func foo() : Int32 {
>         let o1 = Test()
>         list.add(o1)
>         if (Test.t0 != 1) {
>             return 1
>         }
>         list.remove(at: 0)
>         return 0
>     }
>         
>     main() : Int64 {
>         var i : Int64 = 0
>         while (i < 100) {
>             if (foo() != 0) {
>                 print("fail: obj is freed before gc!")
>                 return 1
>             }
>             gc(heavy: true)               // 阻塞 GC 预期
>                                           // 等 ~init() 被执行
>             while (Test.t0 != 0) {        // error, 这是未定义行为
>                 continue
>             }
>             i++
>         }
>         return 0
>     }
>     ```

仓颉`class`除了构造函数之外, 还存在终结器, 类比C++中的析构函数

`class`终结器拥有固定的函数标识`~init()`

仓颉`class`的终结器, 不能手动显式调用, 它是由运行时的垃圾回收机制自动调用的, 所以执行的时机是不确定的

终结器是为了清理资源用的, 这里的资源 指不由仓颉运行时管理的 系统资源, 比如调用C函数创建的堆空间、其他资源句柄等

> [!CAUTION]
> 
> 仓颉`class`终结器就只应该用做清理资源, 而不应该用做其他用途, 因为可能会造成一系列未定义行为

##### `class`成员函数

> `class`成员函数同样分为**实例成员函数**和**静态成员函数(使用`static`修饰符修饰)**
> 
> 实例成员函数只能通过对象访问, **静态成员函数只能通过`class`类型名访问**
> 
> 静态成员函数中 **不能访问**实例成员变量, 也不能调用实例成员函数
> 
> 但在实例成员函数中 **可以访问**静态成员变量以及静态成员函数
> 
> 下例中, `area`是实例成员函数, `typeName`是静态成员函数
> 
> ```cangjie
> class Rectangle {
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
> 根据是否有函数体, 实例成员函数又可以分为**抽象成员函数**和**非抽象成员函数**
> 
> 抽象成员函数没有函数体, 只能定义在抽象类或接口(详见 [接口]() )中
> 
> 需要注意的是, 抽象实例成员函数**默认具有`open`的语义**, `open`修饰符是可选的, 且**必须使用`public`或`protected`进行修饰**
> 
> 非抽象函数必须有函数体, 在函数体中可以通过`this`访问实例成员变量, 例如: 
> 
> ```cangjie
> class Rectangle {
>     let width: Int64 = 10
>     let height: Int64 = 20
> 
>     public func area() {
>         this.width * this.height
>     }
> }
> ```

仓颉`class`的可以定义实例成员函数 和 静态成员函数

实例成员函数可以访问静态成员变量和静态成员函数

但静态成员函数不能访问实例成员变量和实例成员函数

实例成员函数中还存在 抽象成员函数(无函数体), 但只能定义在抽象类和接口中

##### `class`成员的访问修饰符

> 对于`class`的成员(包括成员变量、成员属性、构造函数、成员函数), 可以使用的访问修饰符有`4`种访问修饰符修饰: `private`、`internal`、`protected`和`public`, 缺省的含义是`internal`
> 
> - `private`表示在`class`定义内可见
> 
> - `internal`表示仅当前包及子包(包括子包的子包, 详见 [包]() )内可见
> 
> - `protected`表示当前模块(详见 [包]() )及当前类的子类可见
> 
> - `public`表示模块内外均可见
> 
> ```cangjie
> package a
> public open class Rectangle {
>     public var width: Int64
>     protected var height: Int64
>     private var area: Int64
>     public init(width: Int64, height: Int64) {
>         this.width = width
>         this.height = height
>         this.area = this.width * this.height
>     }
>     init(width: Int64, height: Int64, multiple: Int64) {
>         this.width = width
>         this.height = height
>         this.area = width * height * multiple
>     }
> }
> 
> func samePkgFunc() {
>     var r = Rectangle(10, 20)         // Ok, public 'Rectangle'构造函数 可以在这里访问, public 任何地方 导入类就可见
>     r.width = 8                       // Ok, public 'width' 可以在这里访问, public 任何地方 类可见 成员width就可见
>     r.height = 24                     // Ok, protected 'height' 可以在这里访问, public 当前模块内 类可见 成员height就可见
>     r.area = 30                       // Error, private 'area' 不能在这里访问, private 只能在类内可见
> }
> ```
> 
> ```cangjie
> package b
> import a.*
> public class Cuboid <: Rectangle {
>     private var length: Int64
>     public init(width: Int64, height: Int64, length: Int64) {
>         super(width, height)
>         this.length = length
>     }
>     public func volume() {
>         this.width * this.height * this.length    // Ok, protected 'height' 可以在这里访问
>     }
> }
> 
> main() {
>     var r = Rectangle(10, 20, 2)                  // Error, 'Rectangle' 不存在 public 的三参数构造函数
>     var c = Cuboid(20, 20, 20)
>     c.width = 8                                   // Ok, public 'width' 可以在这里访问
>     c.height = 24                                 // Error, protected 'height' 不能在这里访问
>     c.area = 30                                   // Error, private 'area' 不能在这里访问
> }
> ```

`private`、`internal`、`protected`和`public`四种可访问性修饰符, 可以用来修饰`class`成员

这四种可访问性修饰符, 具体可见性, 与包管理息息相关

他们在修饰`class`成员时

- `private`表示在`class`定义内可见
  
    这个不用了解包 就能理解, 可访问性最小, 只能在类内访问

- `internal`表示仅当前包及子包(包括子包的子包, 详见 [包]() )内可见

    这个, 是`class`成员的默认修饰符, 表示 当前包以及子包 可以访问, 但前提是 要能见到此类

- `protected`表示当前模块(详见 [包]() )及当前类的子类可见

    这个, 表示 当前模块可以访问, 同样前提是 要能见到此类

    当前模块, 表示 父包、当前包、子包

- `public`表示模块内外均可见

    这个也不用了解包 就能理解, 可访问性最大, 只要类可见, 就能访问
