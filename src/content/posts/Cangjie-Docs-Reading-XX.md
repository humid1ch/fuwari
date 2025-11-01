---
title: "仓颉文档阅读-语言规约XI: 包和模块管理"
published: 2025-10-14 18:26:44
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

---

终于到包和模块管理了

## 包和模块管理

> 在仓颉编程语言中, 程序以包的形式进行组织, **包是最小的编译单元**
>
> 包可以定义子包, 从而构成树形结构
>
> 没有父包的包称为`root`包, `root`包及其子包(包括子包的子包)构成的整棵树称为`module`
>
> `module`的名称与`root`包相同
>
> `module`是仓颉的最小发布单元
>
> 包由一个或多个源码文件组成, **同一个包的源码文件必须在同一个目录**, 并且**同一个目录里的源码文件只能属于同一个包**
>
> 子包的目录是其父包目录的子目录

仓颉中, 包可以拥有子包, 没有父包就是`root`包, `root`包及其子包构成整个`module`

`module`是仓颉的最小发布单元, 包是最小编译单元

### 包

#### 包的声明


> 包声明以关键字`package`开头, 后接`root`包至当前包路径上所有包的包名, 以`.`分隔(路径本身不是包名)
>
> 包声明的语法如下:
>
> ```cangjie
> packageHeader
>     : packageModifier? (MACRO NL*)? PACKAGE NL* fullPackageName end+
>     ;
>
> fullPackageName
>     : (packageNameIdentifier NL* DOT NL*)* packageNameIdentifier
>     ;
>
> packageNameIdentifier
>     : Ident
>     | contextIdent
>     ;
>
> packageModifier
>     : PUBLIC
>     | PROTECTED
>     | INTERNAL
>     ;
> ```
>
> 每个包都有包名, 包名是这个包可唯一识别的标识符
>
> 除`root`包外, 包名必须和其所在的目录名一致
>
> **包声明必须在文件的首行**(注释和空白字符不计), 每个文件**只能有一个`package`声明**
>
> 特别地, `root`包的文件可以不声明`package`, 对于不包含`package`声明的文件, 它会用`default`作为包名
>
> ```cangjie
> // ./src/main.cj
> // 省略包声明相当于'package default'
>
> main() {
>     println("Hello World")
> }
> ```
>
> `package`可以被`internal`、`protected`或者`public`修饰
>
> `package`的默认修饰符(默认修饰符是指在省略情况下的修饰符语义, 这些默认修饰符也允许显式写出)为`public`
>
> 同一个包在不同文件中的`package`声明必须使用相同的访问修饰符修饰
>
> 特别地, `root`包不能被`internal`或者`protected`修饰
>
> - **`internal`表示仅当前包及子包(包括子包的子包)内可见**
>
>     当前包的子包(包括子包的子包)内可以导入这个包或者这个包的成员
>
> - **`protected`表示仅当前`module`内可见**
>
>     同一个`module`内的其它包可以导入这个包或者这个包的成员, 不同`module`的包无法访问
>
> - **`public`表示`module`内外均可见**
>
>    其它包可以导入这个包或者这个包的成员

仓颉中, 声明包的语法是`包修饰词 package 包名`

在`.cj`文件中声明包, 就表示此文件属于所声明的包, 当然 前提是目录结构合法

如果不显式声明包, 则`package default`

包名是一个路径, 如果是子包, 需要声明为`package 父包.子包`, 用`.`分割路径

### 包的成员

> 包的成员是在顶层声明的类、接口、`struct`、`enum`、类型别名、全局变量、扩展、函数
>
> 当前包的父包和子包 并不是当前包的成员, **访问父包或者子包需要 包的导入机制**, 未被导入的包名不在`top-level`作用域中
>
> 如下所示的例子中, `package a.b`是`package a`的子包
>
> ```cangjie
> // src/a.cj
> package a
> let a = 0                 // ok
>
> // src/b/b.cj
> package a.b
> let a = 0                 // ok
> let b = 0                 // ok
> ```
>
> 如下所示的例子中, `package a.b`是`package a`的子包
>
> ```cangjie
> // src/a.cj
> package a
>
> let u = 0                 // ok
> let _ = b.x               // Error: 未声明的标识符 'b'
> let _ = a.u               // Error: 未声明的标识符 'a'
> let _ = a.b.x             // Error: 未声明的标识符 'a'
>
> // src/b/b.cj
> package a.b
>
> let x = 1                 // ok
> let _ = a.u               // Error: 未声明的标识符 'a'
> let _ = a.b.x             // Error: 未声明的标识符 'a'
> let _ = b.x               // Error: 未声明的标识符 'b'
> ```
>
> 特别地, **子包不能和当前包的成员同名**, 这是为了保证访问路径中的名称是唯一的

不能尝试通过当前包的包名访问当前包的成员

### 访问修饰符


> 仓颉中, 可以使用访问修饰符来保护对类型、变量、函数等元素的访问
>
> 仓颉有 4 种不同的访问修饰符
>
> - `private`
>
> - `internal`
>
> - `protected`
>
> - `public`

这四种访问修饰符, 并不只是修饰包用的

#### 修饰顶层元素


> 在修饰顶层元素时不同访问修饰符的语义如下:
>
> - **`private`表示仅当前文件内可见**
>
>     不同的文件无法访问这类成员
>
> - **`internal`表示仅当前包及子包(包括子包的子包)内可见**
>
>     同一个包内可以不导入就访问这类成员, 当前包的子包(包括子包的子包)内可以通过导入来访问这类成员
>
> - **`protected`表示仅当前`module`内可见**
>
>     同一个包的文件可以不导入就访问这类成员, 不同包但是在同一个`module`内的其它包可以通过导入访问这些成员, 不同`module`的包无法访问
>
> - **`public`表示`module`内外均可见**
>
>     同一个包的文件可以不导入就访问这类成员, 其它包可以通过导入访问这些成员
>
> |   | File | Package & Sub-Packages | Module | All Packages |
> | - | ---- | ---------------------- | ------ | ------------ |
> | `private` | Y | N | N | N |
> | `internal` | Y | Y | N | N |
> | `protected` | Y | Y | Y | N |
> | `public` | Y | Y | Y | Y |
>
> 不同顶层声明支持的访问修饰符和默认修饰符规定如下:
>
> - `pacakge`支持使用`internal`、`protected`、`public`, 默认修饰符为`public`
>
> - `import`支持使用全部访问修饰符, 默认修饰符为`private`
>
> - 其他顶层声明支持使用全部访问修饰符, 默认修饰符为`internal`

除`internal`之外, C++中也存在另外三个修饰词, 但 C++中的访问修饰符是用于类成员的, 用来声明成员可访问性

仓颉中的访问修饰符, 还可以修饰包成员的可访问性

#### 修饰非顶层成员

> 在修饰非顶层成员时不同访问修饰符的语义如下:
>
> **`private`表示仅当前类型或扩展定义内可见**
>
> **`internal`表示仅当前包及子包(包括子包的子包)内可见**
>
> **`protected`表示当前`module`及当前类的子类可见**
>
> **`public`表示`module`内外均可见**
>
> |   | Type/Extend | Package & Sub-Packages | Module & Sub-Classes | All Packages |
> | - | ----------- | ---------------------- | -------------------- | ------------ |
> | `private` | Y | N | N | N |
> | `internal` | Y | Y | N | N |
> | `protected` | Y | Y | Y | N |
> | `public` | Y | Y | Y | Y |
>
> 类型成员的访问修饰符 可以不同于 类型本身
>
> 除接口外类型成员的默认修饰符(默认修饰符是指在省略情况下的修饰符语义, 这些默认修饰符也允许显式写出)是`internal`
>
> 接口中的成员函数和属性**不可以写访问修饰符**, 它们的访问级别等同于`public`

包内可见, 即表示 在本包内定义的实例, 可以通过实例访问类成员

`protected`是`module`以及子类可以见的, 也就表示 如果不同属一个`module`, 此类成员是无法访问的

`protected`和`internal`, 这两个修饰符, 在刚接触包可能会出现混淆:

一个是`module`及子类可见, 一个是当前包及子包可见, 但`module`就是一个包含父子关系的包链, 那么这两个有什么区别?

区别在于, `protected`修饰类成员, 父包也是可见的, 而`internal`只能当前包和子包

而`private`和`public`则是两个极端, 只类内可见 或 全部可见没有限制

#### 访问修饰符的合法性检查

> 仓颉的访问级别排序为`public`>`protected`>`internal`>`private`
>
> 类型的访问级别:
>
> - 非泛型类型的访问级别 由 类型声明的访问修饰符 决定
>
> - 泛型实例化类型的访问级别等同于该泛型类型与该泛型类型实参的访问级别中最低的一个
>
> 一个声明的访问修饰符 不得高于 该声明中用到的类型的访问修饰符的级别
>
> 具体地:
>
> - 变量、属性声明的访问级别不得高于其类型的访问级别
>
> - 函数声明的访问级别不得高于参数类型、返回值类型, 以及`where`约束中的类型上界的访问级别
>
> - 类型别名的访问级别不得高于原类型的访问级别
>
> - 类型声明的访问级别不得高于`where`约束中的类型上界的访问级别
>
> - 子包的访问级别不得高于其父包的访问级别
>
> - `import`的访问修饰符不得高于其导入声明的访问级别
>
> ```cangjie
> private open class A {}
>
> protected let a = A()                     // error: 使用 private 类型 A, 声明 protected 变量 a
> let (a, b) = (A(), 1)                     // error: 使用 private 类型 A, 声明 internal 变量 a
>
> func f(_: A) {}                           // error: 使用 private 类型 A, 声明 internal 函数 f
> func f() { A() }                          // error: 使用 private 类型 A, 声明 internal 函数 f
> func f<T>() where T <: A {}               // error: 使用 private 类型 A, 声明 internal 函数 f
>
> public type X = A                         // error: 使用 private 类型 A, 声明 public 类型 X
> public type ArrayA = Array<A>             // error: 使用 private 类型 A, 声明 public 类型 ArrayA
>
> protected struct S<T> where T <: A {}     // error: 使用 private 类型 A, 约束 protected struct S<T>
>
> // src/a.cj
> public package a
>
> // src/a/b/b.cj
> protected package a.b                     // ok
>
> // src/a/b/c/c.cj
> public package a.b.c                      // error
> ```

当你在使用一个类型时, 可能使用这个类型进行 定义变量、声明别名、导入此包 等

这些操作也可以使用访问修饰符修饰, 所以你不能 在进行这些操作的时候 尝试使用更大权限的修饰符去进行提权

> 特别地, 类继承时 子类访问级别 与 父类访问级别、类型实现/继承接口时 子类型访问级别 与 父接口访问级别 **不受上述规则限制**
>
> ```cangjie
> private open class A {}
> public enum E { U | V }
> interface I {}
>
> public class C <: A {}                    // ok
>
> public interface J <: I {}                // ok
>
> extend E <: I {}                          // ok
> ```

继承和实现接口时, 不需要在意访问修饰符的限制

### 包的导入

> 导入是一种用来将其他包或其他包中的成员引入到当前仓颉程序中的机制
>
> 当源码中没有`import`声明的时候, 当前文件只能访问 当前包中的成员 和 编译器默认导入的成员
>
> 通过`import`声明, 可以让编译器在编译这个仓颉文件时 找到所需要的外部名称
>
> `import`语句在文件中的位置必须在包声明之后, 其他声明或定义之前
>
> `import`相关的语法如下:
>
> ```
> importList
>     : importModifier? NL* IMPORT NL* importContent end+
>     ;
>
> importSingle
>     : (packageNameIdentifier NL* DOT NL*)* (identifier | packageNameIdentifier)
>     ;
>
> importSpecified
>     : (identifier '.')+ identifier
>     ;
>
> importAlias
>    : importSingle NL* AS NL* identifier
>    ;
>
> importAll
>    : (packageNameIdentifier NL* DOT NL*)+ MUL
>    ;
>
> importMulti
>    : (packageNameIdentifier NL* DOT NL*)* LCURL NL*
>    (importSingle | importAlias | importAll) NL*
>    (COMMA NL* (importSingle | importAlias | importAll))* NL*
>    COMMA? NL* RCURL
>    ;
> ```
>
> `import`语法有如下几种形式:
>
> - 单导入
>
> - 别名导入
>
> - 全导入

先导入包, 才能使用包中可访问的成员

就像C/C++中引入头文件, 但并不是头文件, 原理也不同

可以导入包, 也可以导入包的子包, 可以直接导入目标包的所有成员, 也可以单独导入包的成员

导入包需要导入**目标包的完整路径**

> 通过`import`, 可以导入一个或多个其他包或者其他包中的成员, 也可以通过`as`语法为导入的名称定义别名
>
> 如果导入的名称是包, 则可以用它继续访问包中的成员(子包不是包的成员), 但**包名本身不能作为表达式**
>
> ```cangjie
> package a
> public let x = 0
> ```
>
> ```cangjie
> package demo
>
> import a
>
> main() {
>     println(a.x)      // ok, prints 0
> }
> ```

导入的包, 包名本身不能作为表达式

导入包时, 可以通过`as`为导入的内容取别名:`import a as someIdent`

> 任意包和包之间**不能产生循环依赖**, 即使是同一个`module`下的包之间也不可以
>
> 对于任意两个包`p1`和`p2`, 如果`p1`导入了`p2`或者`p2`的成员, 那么我们称`p1`和`p2`具有依赖关系, `p1`依赖`p2`
>
> 依赖关系具有传递性, 如果`p1`依赖`p2`, `p2`依赖`p3`, 那么`p1`依赖`p3`
>
> 包的循环依赖是指存在包相互依赖的情况
>
> ```cangjie
> package p.a
> import p.b              // error
>
> pacakge p.b
> import p.a              // error
> ```
>
> 禁止使用`import`导入当前包或当前包中的成员
>
> ```cangjie
> package a
>
> import a                // error
> import a.x              // error
>
> public let x = 0
> ```

包循环依赖是禁止的, 即 不允许两个包互相导入

> 导入的成员的作用域级别 低于 当前包声明的成员
>
> 导入的**非函数成员会被当前包的同名成员遮盖**; 导入的函数成员 若可以和当前包的同名函数构成重载, 调用时会根据 [泛型函数重载] 和 [函数重载] 的规则进行函数决议; 导入的函数成员若和当前包的同名函数不构成重载, 则按照遮盖处理
>
> ```cangjie
> package a
> public let x = 0
> public func f() {}
>
> import a.x                    // Warning: 导入的 x 被遮盖
> import a.f
>
> let x = 1
> func f(x: Int64) { x }
>
> let _ = f()                   // ok, 找到 导入的 a.f
> let _ = f(1)                  // ok, 找到 在本包中定义的 f
> ```

导入的包的成员, 是可以与当前包中的成员构成重载或遮盖的

#### 单导入

> 单导入语法用来**导入单个成员**, 目标成员必须是对当前包**可见的**
>
> 导入的成员名称会作为当前作用域内可以访问的名称
>
> `import`语法中的路径**最后一个名称表示指定的成员**, 这个名称可以是顶层变量、函数、类型, 也可以是包
>
> 下面是导入顶层变量、函数、类型的例子:
>
> 有两个包分别是`a`和`b`, 在`b`包中导入`a`包的成员
>
> ```cangjie
> package a
>
> public let x = 0
> public func f() { 0 }
> public class A {}
> ```
>
> ```cangjie
> import a.x
> import a.f
> import a.A
>
> private func g(_: A) { x + f() }        // ok
> ```

单导入, 是指导入某个包, 或包中的指定可见成员

需要导入完整的路径, 最后一个标识符可以在包内直接使用

> 如下所示的例子中, `c`是`a`的子包
>
> ```cangjie
> package a.c
>
> public let y = 1
>
> import a
> ```
>
> ```cangjie
> private func g(_: a.A) { a.x + a.f() }    // ok
> privage func h(_: A) {                    // error: 未声明的标识符 A
>     x + f()                               // error: 未声明的标识符 x and f
> }
> let _ = a.c.y                             // error: c 不是 a 的成员
> let _ = a                                 // error: 未声明的标识符 a
> ```
>
> ```cangjie
> import a.c
>
> let _ = c.y                               // ok
> ```
>
> 单导入的成员被当前包成员遮盖时, 编译器会给出告警提示无用导入
>
> ```cangjie
> import a.x                                // warning: 导入的 x 被遮盖
> import a.f                                // warning: 导入的 f 被遮盖
>
> func f() { 1 }
>
> let x = 1
> let _ = f()                               // ok, 调用在本包中定义的 f(), 值为 1
> ```

单导入, 如果导入包, 则可以通过包名访问目标包的可见成员

#### 别名导入

> 别名导入可以使用`as`语法为导入成员重命名
>
> 以别名导入的内容, 在当前包中只会以别名的形式引入作用域, 而不会引入原来的名称(但**不禁止分别导入原名和别名**)
>
> 导入的内容可以是包或者包的成员
>
> ```cangjie
> package a
>
> public let x = 0
> public let y = 1
> public func f() { 0 }
> ```
>
> ```cangjie
> import a as pkgA
> import a.x as x1
> import a.x as x2          // ok
>
> let _ = 5                 // error: 未声明的标识符 'x'
> let _ = a.x               // error: 未声明的标识符 'a'
> let _ = x1                // ok
> let _ = x2                // ok
> let _ = pkgA.x            // ok
> let _ = pkgA.x1           // error: 'x1' 不是 'pkgA' 的成员
> ```

#### 全导入


> 全导入通过`*`语法导入其他包中所有对当前包可见的顶层成员(不包括子包)
>
> 示例如下:
>
> ```cangjie
> package a
>
> public let x = 0
> public func f() { 0 }
> public class A {}
> ```
>
> ```cangjie
> import a.*
>
> private func g(_: A) { x + f() }    // ok
> ```
>
> 与单导入不同, 当全导入的成员被当前包成员遮盖时, 编译器不会给出告警
>
> ```cangjie
> import a.*
>
> let x = 1
> func f() { x }                      // ok, x 定义在本包中
>
> let _ = f()                         // ok, 调用在本包中定义的 f(), 值为 1
> ```

全导入, 顾名思义 即 一次性导入目标包中的所有可见成员

导入的成员同样可以构成 重载和遮盖

> 如果导入的成员 **不被当前包的成员遮盖**, 但多个导入成员重名时, 编译器不会给出告警
>
> 但如果这些重名的导入**不构成重载**, 这个名字在本包中不可用, 在使用该名称时 编译器会因**无法找到唯一的名称而报错**
>
> ```cangjie
> package b
>
> public let x = 1
> public func f(x: Int64) { x }
> ```
>
> ```cangjie
> import a.*
> import b.*
>
> let _ = x                     // error: 不可确定的 x
> ```
>
> 如果导入的重名成员**可以构成函数重载**, 调用时会根据 [泛型函数重载] 和 [函数重载] 的规则进行函数决议
>
> ```
> import a.*
> import b.*
>
> func f(b: Bool) { b }
>
> let _ = f()                   // ok, 调用 a.f
> let _ = f(1)                  // ok, 调用 b.f
> let _ = f(true)               // ok, 调用在本包中定义的 f()
> ```

导入多个不构成遮盖也不构成重载的同名成员时, 不会警告, 只是会在调用时无法找到唯一可用名称报错

> 带访问修饰符的全导入 不会导入 比其访问级别低的声明
>
> ```cangjie
> package a
> protected import a.b.*
> let _ = x                   // ok
> let _ = y                   // ok
> let _ = z                   // error: 未声明的标识符 'z'
> ```
>
> ```cangjie
> package a.b
>
> public let x = 0
> protected let y = 1
> internal let z = 2
> ```

包导入时, 也是可以使用访问修饰符的, 可以在导入成员时 指定原包中满足目标可访问等级的成员被导入

#### 批量导入

> 批量导入使用`{}`语法, 在一个`import`声明里同时导入多个成员
>
> 通常用来省略重复的包路径前缀
>
> 批量导入的`{}`中支持单导入、别名导入和全导入, 但**不允许嵌套批量导入**
>
> ```cangjie
> import std.{
>     time,
>     fs as fileSystem,
>     io.*,
>     collection.{ HashMap, HashSet }       // syntax error
> }
> ```
>
> `{}`的前缀可以为空
>
> ```cangjie
> import {
>     std.time,
>     std.fs as fileSystem,
>     std.io.*,
> }
> ```
>
> 使用批量导入语法与使用多个独立`import`的语法是等价的
>
> ```cangjie
> import std.{
>     os.process,
>     time,
>     io.*,
>     fs as fileSystem
> }
> ```
>
> 等价于:
>
> ```cangjie
> import std.os.process
> import std.time
> import std.io.*
> import std.fs as fileSystem
> ```

嵌套导入, 可以在`{}`前加上包路径, 然后在`{}`内批量添加要导入的成员, 用`, `分隔

#### 导入名称冲突检查

> 如果多个单导入的名称产生重名(包括重复导入)且不构成函数重载, 并且该名字在本包中没有被遮盖, 编译器会给出**名称冲突告警**, 这个名字在本包中不可用, 在使用该名称时编译器会因无法找到唯一的名称而报错
>
> 若该名称被当前包成员遮盖时, 编译器会给出告警提示无用导入
>
> ```cangjie
> package b
>
> public let x = 1
> public func f(x: Int64) { x }
> ```
>
> ```cangjie
> package c
> public let f = 0
> ```
>
> ```cangjie
> import a.x                      // warning: 导入的 'x' 被遮盖
> import a.x                      // warning: 导入的 'x' 被遮盖
> import b.x                      // warning: 导入的 'x' 被遮盖
>
> let x = 0
> let y = x                       // y = 0
> ```
>
> ```cangjie
> import a.x
> import a.x                      // warning: 'x' 已经被导入了
> import b.x                      // warning: 'x' 已经被导入了
>
> let _ = x                       // error: 无法确定的 'x'
> ```
>
> 如果导入的重名成员之间或者导入的成员与当前包中的同名函数之间可以构成函数重载, 调用时会根据 [泛型函数重载] 和 [函数重载] 的规则进行函数决议
>
> ```cangjie
> import a.f
> import b.f
>
> func f(b: Bool) { b }
>
> let _ = f()                   // ok, 调用 'a.f'
> let _ = f(1)                  // ok, 调用 'b.f'
> let _ = f(true)               // ok, 调用 在本包中定义的 f()
> ```

当重复导入 但发生遮盖时, 不会出现错误, 只会出现被遮盖的警告

当重复导入 发生重载时, 更可以正常的使用

只有重复导入, 但没有发生重载也没有发生遮盖, 才会出现无法确定的目标的错误

> 多个别名导入**同名**, 或者别名导入 和本包定义同名时的处理规则 与单导入相同
>
> ```cangjie
> import a.x as x1            // warning: 导入的 x1 被遮盖
>
> let x1 = 10
> let _ = x1                  // ok, 'x1' 在本包中被定义
> ```
>
> ```cangjie
> package b
>
> public let x = 1
> public func f(x: Int64) { x }
> ```
>
> ```cangjie
> import a.x as x1
> import a.x as x1            // warning: 'x1' 已经被导入
> import b.x as x1            // warning: 'x1' 已经被导入
>
> let _ = x1                  // error: 无法确定的 'x1'
> ```
>
> ```cangjie
> import a.f as g
> import b.f as g
>
> func g(b: Bool) { b }
>
> let _ = g()                 // ok, 调用 'a.f'
> let _ = g(1)                // ok, 调用 'b.f'
> let _ = g(true)             // ok, 调用 在本包中定义的 'g'
> ```
>
> 如果导入名称冲突的其中一方来自全导入, 这种情况下编译器也不会给出报警, 但冲突的声明都不可用
>
> 批量导入依据其等价的单导入、别名导入、多导入做名称冲突检查

当导入的别名冲突时, 与单导入冲突处理相同

被遮盖就不用, 构成重载就按照重载规则调用

#### `import`的访问修饰符

> `import`可以被`private`、`internal`、`protected`、`public`访问修饰符修饰
>
> 其中, 被`public`、`protected`或者`internal`修饰的`import`可以把导入的成员重导出(如果这些导入的成员没有因为名称冲突或者被遮盖导致在本包中不可用)
>
> 其他包可以根据 [访问修饰符] 的访问规则通过`import`导入这些被重导出的对象
>
> 具体地:
>
> - `private import`表示导入的内容仅当前文件内可访问, `private`是`import`的**默认修饰符**, 不写访问修饰符的`import`等价于`private import`
>
> - `internal import`表示导入的内容在当前包及其子包(包括子包的子包)均可访问, 非当前包访问需要显式`import`
>
> - `protected import`表示导入的内容在当前`module`内都可访问, 非当前包访问需要显式`import`
>
> - `public import`表示导入的内容外部都可访问, 非当前包访问需要显式`import`

包导入也可以使用访问修饰符

如果导入其他成员, 则可以使用`public`、`protected`、`internal`修饰, 这些导入的成员 可以按照访问修饰符规则被重导出

但, 按照全导入时介绍的规则, 如果使用访问修饰符 修饰`import`, 那么只能导入目标包中**满足访问修饰符限制**的成员

> 在下面的例子中, `b`是`a`的子包, 在`a`中通过`public import`重导出了`b`中定义的函数`f`
>
> ```cangjie
> package a
>
> public let x = 0
> public import a.b.f
> ```
>
> ```cangjie
> internal package a.b
>
> public func f() { 0 }
> ```
>
> ```cangjie
> import a.f          // ok
> let _ = f()         // ok
> ```
>
> ```cangjie
> import a.f                          // ok
> //// case 1
> package demo
>
> public import std.time.Duration     // warning: 导入的 'Duration' 被遮盖
>
> struct Duration {}
> ```
>
> ```cangjie
> //// case 2
> // ./a.cj
> package demo
>
> public import std.time.Duration
>
> // ./b.cj
> package demo
>
> func f() {
>     let a: Duration = Duration.second       // ok,  访问重导出的 'Duration'
> }
> ```
>
> ```cangjie
> //// case 3
> // ./a/a.cj
> package demo.a
>
> public let x = 0
>
> // ./b/b.cj
> package demo.b
>
> public import demo.a.*              // warning: 导入的 'x' 被遮盖, 将不能重导出 'demo.a.x'
>
> var x = 0
> ```
>
> ```cangjie
> //// case 4
> // ./a/a.cj
> package demo.a
>
> public let x = 0
>
> // ./b/b.cj
> package demo.b
>
> public let x = 0
>
> // ./c/c.cj
> package demo.c
>
> public import demo.a.*              // warning, 因为存在重复名称, 将不能重导出 'demo.a.x'
> public import demo.b.*              // warning, 因为存在重复名称, 将不能重导出 'demo.b.x'
> ```
>
> 特别地, **包不可以被重导出**: 如果被`import`导入的是包, 那么该`import`不允许被`public`、`protected`或者`internal`修饰
>
> ```cangjie
> public import a.b                   // error: 不能重新导出包
> ```

如果是导入的包名, 只能用`private`即 默认修饰符

重导出, 非当前包才需要 显式`import`

**被遮盖的不能重导出**
