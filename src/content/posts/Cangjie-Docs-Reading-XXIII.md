---
title: "仓颉文档阅读-语言规约XIV: 语言互操作"
published: 2025-10-17 15:56:21
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

## 语言互操作

### C语言互操作

#### `unsafe`上下文

> 因为 C语言太容易造成不安全, 所以仓颉规定所有和 C语言互操作的功能都只能发生在`unsafe`上下文中
>
> `unsafe`上下文是用`unsafe`关键字引入的
>
> `unsafe`关键字可以有以下几种用法:
>
> - 修饰一个代码块
>
>     `unsafe`表达式的类型就是这个代码块的类型
>
> - 修饰一个函数
>
> 所有的`unsafe`函数和`CFunc`类型函数必须在`unsafe`上下文中调用
>
> 所有的`unsafe`函数, 均不能作为一等公民使用, 包括不能赋值给变量, 不能作为实参或返回值使用, 不能作为表达式使用, 只能在`unsafe`上下文中被调用
>
> 语法定义为:
>
> ```
> unsafeExpression
>     : 'unsafe' '{' expressionOrDeclarations '}'
>     ;
>
> unsafeFunction
>     : 'unsafe' functionDefinition
>     ;
> ```

因为, C语言不安全, 所以相关操作只能在`unsafe`上下文中使用

`unsafe`可以修饰函数 或 代码块

#### 调用 C函数

##### `foreign`关键字 和`@C`

> 仓颉编程语言要调用 C函数, 需要先在仓颉代码中声明这个函数且用`foreign`关键字修饰
>
> `foreign`函数只能存在于`top-level`作用域中, 且仅包内可见, 故不能使用其他任何函数修饰符
>
> `@C`只支持修饰`foreign`函数、`top-level`作用域中的非泛型函数和`struct`类型
>
> 在修饰`foreign`函数时, `@C`可省略
>
> 如未特别说明, C语言互操作章节中的`foreign`函数均视为`@C`修饰的`foreign`函数
>
> 在`@C`修饰下, `foreign`关键字只允许修饰`top-level`作用域中的非泛型函数
>
> `foreign`函数只是声明, 没有函数体, 其参数和返回类型均**不可省略**
>
> 用`foreign`修饰的函数使用**原生 C ABI**, 且不会做名字修饰
>
> ```cangjie
> foreign func foo(): Unit
> foreign var a: Int32 = 0          // 编译错误, 只能声明非泛型函数
> foreign func bar(): Unit {        // 编译错误, 不能由函数体
>     return
> }
> ```
>
> 多个`foreign`函数声明, 可以使用`foreign`块来声明
>
> `foreign`块是指在`foreign`关键字后使用一对花括号括起来的声明序列
>
> `foreign`块内仅可包含函数
>
> 在`foreign`块上添加注解等同于为`foreign`块中的每一个成员加上注解
>
> ```cangjie
> foreign {
>     func foo(): Unit
>     func bar(): Unit
> }
> ```
>
> `foreign`函数需要能链接到**同名的 C函数**, 且参数和返回类型需要保持一致
>
> **只有满足于`CType`约束的类型, 可以被用于`foreign`函数的参数和返回类型**
>
> 关于`CType`的定义, 请参考下文的`CType`接口部分
>
> `foreign`函数不支持命名参数和参数默认值
>
> `foreign`函数允许变长参数, 使用`...`表达, 只能用于参数列表的最后
>
> 变长参数均需要满足`CType`约束, 但不必是同一类型

`foreign`只能修饰顶层作用域的非泛型函数, 且只是声明, 不能拥有函数体

仓颉要调用C函数, 就必须要使用`foreign`声明同名函数

`@C`只支持修饰`foreign`函数、`top-level`作用域中的非泛型函数和`struct`类型

不过`@C`修饰`foreign`函数时, `@C`可省略

`@C`应该表示这是C语言可调用的函数, 而`foreign`应该用来修饰会被仓颉调用的C函数

##### `CFunc`

> 仓颉中的`CFunc`类型函数是指可以被 C语言代码调用的函数, 共有以下三种形式:
>
> - `@C`修饰的`foreign`函数
>
> - `@C`修饰的仓颉函数
>
> - 类型为`CFunc`的`lambda`表达式
>
>     与普通的`lambda`表达式不同, `CFunc lambda`**不能捕获变量**
>
> ```cangjie
> // 情况 1: @C 修饰的 foreign 函数
> foreign func free(ptr: CPointer<Int8>): Unit
>
> // 情况 2: @C 修饰的仓颉函数
> @C
> func callableInC(ptr: CPointer<Int8>) {
>     print("This function is defined in Cangjie.")
> }
>
> // 情况 3: 类型为 CFunc 的 lambda 表达式
> let f1: CFunc<(CPointer<Int8>) -> Unit> = { ptr =>
>     print("This function is defined with CFunc lambda.")
> }
> ```
>
> 以上三种形式声明/定义的函数的类型均为`CFunc<(CPointer<Int8>) -> Unit>`

`foreign`函数是仓颉可以调用的C函数, 也是C代码可调用的C函数

`@C`修饰仓颉函数, 即 表示此为 C代码可调用的C函数

`CFunc`是一个泛型类型, 类型参数可以是仓颉函数类型, 可以赋值一个`lambda`表达式, 但不是闭包

> `CFunc`对应 C语言的**函数指针类型**
>
> 这个类型为泛型类型, 其泛型参数表示该`CFunc`入参和返回值类型, 使用方式如下:
>
> ```cangjie
> foreign func atexit(cb: CFunc<()->Unit>)
> ```
>
> 与`foreign`函数一样, 其他形式的`CFunc`的参数和返回类型必须满足`CType`约束, 且不支持命名参数和参数默认值

`CFunc`的参数和返回类型都需要满足`CType`约束

或者说, 能够被C语言调用的C函数, 类型都需要满足`CType`约束

> `CFunc`在仓颉代码中被调用时, 需要处在`unsafe`上下文中
>
> 仓颉支持将一个`CPointer<T>`类型的变量类型转换为一个具体的`CFunc`, 其中`CPointer`的泛型参数`T`可以是满足`CType`约束的任意类型, 使用方式如下:
>
> ```cangjie
> main() {
>     var ptr = CPointer<Int8>()
>     var f = CFunc<() -> Unit>(ptr)
>     unsafe { f() }                                                // 运行时 core dump, 因为指针是 nullptr
> }
> ```
>
> `CFunc`的参数和返回类型不允许依赖外部的泛型参数
>
> ```cangjie
> func call<T>(f: CFunc<(T) -> Unit>, x: T) where T <: CType {      // 错误
>     unsafe { f(x) }
> }
>
> func f<T>(x: T) where T <: CType {
>     let g: CFunc<(T) -> T> = { x: T => x }                        // 错误
> }
>
> class A<T> where T <: CType {
>     let x: CFunc<(T) -> Unit>                                     // 错误
> }
> ```

任意的`CPointer<T>`类型变量都可以转换成一个`CFunc`, 即使`T`并不表示一个函数类型

但, 如果此变量不指向一个有效的函数, 那么转换之后调用时会出现问题

且, `T`要满足`CType`约束

##### `inout`参数

> 在仓颉中调用`CFunc`时, 其 实参可以使用`inout`关键字修饰 组成**引用传值表达式**, 此时, 该参数按**引用传递**
>
> 引用传值表达式的类型为`CPointer<T>`, 其中`T`为`inout`表达式修饰的表达式的类型
>
> 引用传值表达式具有以下约束:
>
> - 仅可用于对`CFunc`的调用处
>
> - 其修饰对象的类型必须满足`CType`约束, 但不可以是`CString`
>
> - 其修饰对象必须是用`var`定义的变量
>
> - 通过仓颉侧引用传值表达式传递到 C 侧的指针, 仅保证在函数调用期间有效, 即此种场景下 C 侧不应该保存指针以留作后用
>
> `inout`修饰的变量, 可以是定义在`top-level`作用域中的变量、局部变量、`struct`中的成员变量, 但**不能直接或间接来源于`class`的实例**
>
> ```cangjie
> foreign func foo1(ptr: CPointer<Int32>): Unit
>
> @C
> func foo2(ptr: CPointer<Int32>): Unit {
>     let n = unsafe { ptr.read() }
>     println("*ptr = ${n}")
> }
>
> let foo3: CFunc<(CPointer<Int32>) -> Unit> = { ptr =>
>     let n = unsafe { ptr.read() }
>     println("*ptr = ${n}")
> }
>
> struct Data {
>     var n: Int32 = 0
> }
>
> class A {
>     var data = Data()
> }
>
> main() {
>     var n: Int32 = 0
>     unsafe {
>         foo1(inout n)                   // OK
>         foo2(inout n)                   // OK
>         foo3(inout n)                   // OK
>     }
>     var data = Data()
>     var a = A()
>     unsafe {
>         foo1(inout data.n)              // OK
>         foo1(inout a.data.n)            // Error, n 是通过类 A 的实例成员变量间接派生的
>     }
> }
> ```

`inout`关键字, 可以在调用C函数传参时 修饰实参, 表示传引用传参

但, 实参不能来自`class`类型实例, 因为需要在调用过程中保证参数的有效, 但类是由运行时自动管理的

其次, 因为类实例与成员之间已经存在一定的引用关系, 可能不能简单地映射到C指针语义

##### 调用约定

> 函数调用约定, 描述调用者和被调用者双方 如何进行函数调用(如参数如何传递、栈由谁清理等), 函数调用和被调用双方 必须使用相同的调用约定才能正常运行
>
> 仓颉编程语言通过`@CallingConv`来表示各种调用约定, 支持的调用约定如下:
>
> - `CDECL`, `CDECL`表示`clang`的 C 编译器在不同平台上默认使用的调用约定
>
> - `STDCALL`, `STDCALL`表示`Win32 API`使用的调用约定
>
> 通过C语言互操作机制调用的C函数, 未指定调用约定时将采用默认的`CDECL`调用约定
>
> 如下调用 C 标准库函数`clock`示例:
>
> ```cangjie
> @CallingConv[CDECL]               // 默认情况下可省略
> foreign func clock(): Int32
>
> main() {
>     println(clock())
> }
> ```
>
> `@CallingConv`只能用于修饰`foreign`块、单个`foreign`函数和`top-level`作用域中的`CFunc`函数
>
> 当`@CallingConv`修饰`foreign`块时, 会为`foreign`块中的每个函数分别加上相同的`@CallingConv`修饰

文档说, 调用和被调用双方 都要使用相同的调用约定

但调用时, 省略了修饰说明

#### 类型映射

> 在仓颉编程语言中声明`foreign`函数时, 参数以及返回值的类型 必须和要调用的 C 函数的参数和返回类型相一致
>
> 仓颉编程语言类型与C语言类型不同, 有些简单值类型, 如 C 语言`int32_t`类型可以直接使用仓颉编程语言`Int32`与之对应, 但是一些相对复杂类型如结构体则需要在仓颉侧声明对应的同样内存布局的类型
>
> 仓颉编程语言可以用于和 C 交互的类型都满足`CType`约束, 它们可以分成基础类型和复杂类型
>
> 基础类型包括整型、浮点型、`Bool`类型、`CPointer`类型、`Unit`类型
>
> 复杂类型包括 用`@C`修饰的`struct`、`CFunc`类型等

仓颉的类型 与 C语言的类型不同, 但仓颉在声明`foreign`函数时, 要求 声明函数的参数和返回值必须要与调用的C函数保持一致

所以, 仓颉的类型可以与C语言类型进行映射

##### 基础类型

> 仓颉编程语言和C之间传递参数时, 基础的值类型会进行**复制**, 如`int`、`short`等
>
> 仓颉编程语言与 C 语言相匹配的基础类型映射关系表, 如下:
>
> | Cangjie Type | C type | Size |
> | ------------ | ------ | ---- |
> | `Unit` | `void` | 0 |
> | `Bool` | `bool` | 1 |
> | `Int8` | `int8_t` | 1 |
> | `UInt8` | `uint8_t` | 1 |
> | `Int16` | `int16_t` | 2 |
> | `UInt16` | `uint16_t` | 2 |
> | `Int32` | `int32_t` | 4 |
> | `UInt32` | `uint32_t` | 4 |
> | `Int64` | `int64_t` | 8 |
> | `UInt64` | `uint64_t` | 8 |
> | `IntNative` | `ssize_t` | platform dependent |
> | `UIntNative` | `size_t` | kplatform dependent |
> | `Float32` | `float` | 4 |
> | `Float64` | `double` | 8 |
>
> 注意:
>
> - `int`类型、`long`类型等由于其在不同平台上的不确定性, 需要程序员自行指定对应仓颉编程语言类型

> - `IntNative/UIntNative`在 C 互操作场景中, 其与 C 语言中的`ssize_t/size_t`是一致的

> - 在 C 互操作场景中, 与 C 语言类似, `Unit`类型仅可作为`CFunc`中的返回类型和`CPointer`的泛型参数
>
> 仓颉编程语言中`foreign`函数的参数、返回值的类型需要与 C 函数参数、返回值的类型相对应
>
> 对于类型**映射关系明确且平台无关**的类型(参考基础类型映射关系表), 可以直接使用标准对应的仓颉编程语言基础类型
>
> 比如在 C 语言中有一个`add`函数声明如下:
>
> ```c
> int64_t add(int64_t X,  int64_t Y) { return X+Y; }
> ```
>
> 在仓颉编程语言中调用add函数, 代码示例如下:
>
> ```cangjie
> foreign func add(x: Int64, y: Int64): Int64
>
> main() {
>     let x1: Int64 = 42
>     let y1: Int64 = 42
>     var ret1 = unsafe { add(x1, y1) }
>     ...
> }
> ```

对于基础类型, 如果仓颉中的类型 与 C语言中的类型 映射关系明确且与平台无关

那么在仓颉中调用C函数时, 可以是指使用对应的仓颉类型

上部分中描述的基础类型, 仓颉和C之间都是可以直接相互映射的

##### 指针

> 仓颉编程语言提供`CPointer<T>`类型对应C语言的指针`T*`类型, 其中`T`必须满足`CType`约束
>
> `CPointer`类型必须满足:
>
> - 大小和对齐与平台相关
>
> - 对它做加减法算术运算、读写内存, 是需要在`unsafe`上下文操作的
>
> - `CPointer<T1>`可以在`unsafe`上下文中使用类型强制转换, 变成`CPointer<T2>`类型
>
> `CPointer`有一些成员方法, 如下所示:
>
> ```cangjie
> func isNull() : bool
>
> // 操作符重载
> unsafe operator func + (offset: int64) : CPointer<T>
> unsafe operator func - (offset: int64) : CPointer<T>
>
> // 读和写访问
> unsafe func read() : T
> unsafe func write(value: T) : Unit
>
> // 带偏移的读和写
> unsafe func read(idx: int64) : T
> unsafe func write(idx: int64, value: T) : Unit
> ```
>
> `CPointer`可以使用类型名构造一个实例, 它的值对应 C 语言的`NULL`
>
> ```cangjie
> func test() {
>     let p = CPointer<Int64>()
>     let r1 = p.isNull()             // r1 == true
> }
> ```
>
> 仓颉支持将一个`CFunc`变量的类型转换为一个`CPointer`类型, 其中`CPointer`的泛型参数`T`可以是满足`CType`约束的任意类型, 使用方式如下:
>
> ```cangjie
> foreign func rand(): Int32
> main() {
>     var ptr = CPointer<Int8>(rand)
>     0
> }
> ```

仓颉提供了`CPointer<T>`类型, 映射C语言的指针, `T`可以是满足`CType`约束的任意类型

并且, `CPointer`默认拥有一些成员方法

##### 字符串

> C语言字符串实际是以`'\0'`终止的一维字符数组
>
> 仓颉编程语言提供`CString`与 C语言字符串相匹配
>
> 通过`CString`的构造函数或`LibC`的`mallocCString`创建 C 语言字符串, 如需在仓颉端释放, 则调用`LibC`的`free`方法
>
> 声明`foreign`函数时, 需要根据要调用的 C语言函数的声明来确定函数名称、参数类型、返回值类型
>
> C语言标准库`printf`函数的声明如下:
>
> ```c
> int printf(const char *format, ...)
> ```
>
> 参数`const char *`类型对应仓颉的类型`CString`
>
> 返回类型`int`对应的仓颉的`Int32`类型
>
> 创建字符串并调用`printf`函数示例如下:
>
> ```cangjie
> package demo
>
> foreign func printf(fmt: CString, ...): Int32
>
> main() {
>     unsafe {
>         let str: CString = LibC.mallocCString("hello world!\n")
>         printf(str)
>         LibC.free(str)
>     }
> }
> ```

`CString`类型不支持直接使用仓颉的字符串字面量进行初始化, 只支持使用`CPointer<UInt>`的指针进行初始化

最方便的创建`CString`的方式, 还是`LibC.mallocCString("")`的方式, 不过要手动释放

##### `Array`类型

> 仓颉的`Array`类型是不满足`CType`约束的, 因此它不能用于`foreign`函数的参数和返回值
>
> 但是当`Array`内部的元素类型满足`CType`约束的时候, 仓颉允许使用下面这两个函数 获取和释放 指向数组内部元素的指针
>
> ```cangjie
> unsafe func acquireArrayRawData<T>(arr: Array<T>): CPointerHandle<T> where T <: CType
> unsafe func releaseArrayRawData<T>(h: CPointerHandle<T>):  Unit where T <: CType
>
> struct CPointerHandle<T> {
>     let pointer: CPointer<T>
>     let array: Array<T>
> }
> ```
>
> 参考如下示例, 假设我们要把一个`Array<UInt8>`写入到文件中, 可以这样做:
>
> ```cangjie
> foreign func fwrite(buf: CPointer<UInt8>, size: UIntNative, count: UIntNative, stream: CPointer<Unit>): UIntNative
>
> func writeFile(buffer: Array<UInt8>, file: CPointer<Unit>) {
>     unsafe {
>         let h = acquireArrayRawData(buffer)
>         fwrite(h.pointer, 1, buffer.size, file)
>         releaseArrayRawData(h)
>     }
> }
> ```

仓颉的`Array`类型, 当内部元素类型满足`CType`约束时, 可以使用仓颉提供的函数获取指向数组内部元素的指针, 只能整体获取

获取完成之后, 需要对指针进行释放操作, 释放只是解除了`CPointer`与`Array`数据的关联, 而非释放原`Array`数据内存

`CPointerHandle.pointer`表示当前指针指向的元素地址, `CPointerHandle.array`表示当前指针指向的数组(可以使用`[]`读写), 可以以此修改原`Array`中的数据

##### `VArray`类型

> 仓颉使用`VArray`类型与 C数组类型映射
>
> 当`VArray<T, $N>`中的元素类型`T`满足`CType`约束时, `VArray<T, $N>`类型也满足`CType`约束
>
> ```cangjie
> struct A {}                         // A 不是一个 CType.
> let arr1: VArray<A, $2>             // arr1 不是一个 CType.
> let arr2: VArray<Int64, $2>         // arr2 是一个 CType.
> ```
>
> `VArray`允许作为`CFunc`签名的参数类型, 但**不允许为返回类型**
>
> 如果`CFunc`签名中参数类型被声明为`VArray<T, $N>`, 对应的实参也只能是被`inout`修饰的`VArray<T, $N>`类型的表达式, 但参数传递时, 仍然以`CPointer<T>`传递
>
> 如果`CFunc`签名中参数类型被声明为`CPointer<T>`, 对应的实参可以是被`inout`修饰的`VArray<T, $N>`类型的表达式, 参数传递时, 仍然为`CPointer<T>`类型
>
> `CPointer<VArray<T, $N>>`等同于`CPointer<T>`
>
> ```cangjie
> foreign func foo1(a: VArray<Int32, $3>): Unit
> foreign func foo2(a: CPointer<Int32>): Unit
>
> var a: VArray<Int32, $3> = [1, 2, 3]
> unsafe {
>     foo1(inout a)               // Ok.
>     foo2(inout a)               // Ok.
> }
> ```

`Array`不满足`CType`约束, 但是`VArray`可以满足`CType`约束

但, `VArray`只允许作为`CFunc`的参数类型, 而不允许作为返回类型

且, 参数传参只能使用`inout`修饰的`VArray`类型对象

但是, 有一点不理解, 为什么 **`CPointer<VArray<T, $N>>`等同于`CPointer<T>`**

##### 结构体

> `foreign`函数的签名中包含结构体类型时, 需要在仓颉侧定义同样内存布局的`struct`, 使用`@C`修饰

仓颉中, `struct`应该可以映射C结构体, 结构体需要用`@C`修饰, 且其内的成员类型应该需要满足`CType`约束

> 参考如下示例, 一个 C 图形库(`libskialike.so`)中有一个计算两点之间距离的函数`distance`, 其在 C 语言头文件中相关结构体和函数声明如下:
>
> ```c
> struct Point2D {
>     float x;
>     float y;
> };
> float distance(struct Point2D start, struct Point2D end);
> ```
>
> 声明`foreign`函数时, 需要根据要调用的 C 语言函数的声明来确定函数名称、参数类型、返回值类型
>
> 当创建 C侧结构体时, 需要确定结构体各个成员名称和类型
>
> 代码示例如下:
>
> ```cangjie
> package demo
>
> @C
> struct Point2D {
>     var x: Float32
>     var y: Float32
> }
>
> foreign func distance(start: Point2D, end: Point2D): Float32
> ```
>
> 用`@C`修饰的`struct`必须满足以下限制:
>
> - 成员变量的类型必须满足`CType`约束
>
> - 不能实现或者扩展`interfaces`
>
> - 不能作为`enum`的关联值类型
>
> - 不允许被闭包捕获
>
> - 不能具有泛型参数
>
> 用`@C`修饰的`struct`自动满足`CType`约束
>
> `@C struct`中的`VArray`类型成员变量保证与 C 中数组的内存布局一致
>
> 例如, 对于以下 C 的结构体类型:
>
> ```c
> struct S {
>     int a[2];
>     int b[0];
> }
> ```
>
> 在仓颉中, 可以声明为如下结构体与 C 代码对应:
>
> ```cangjie
> @C
> struct S {
>     var a: VArray<Int32, $2> = VArray<Int32, $2>(item: 0)
>     var b: VArray<Int32, $0> = VArray<Int32, $0>(item: 0)
> }
> ```
>
> 注意: C 语言中允许结构体的最后一个字段为未指明长度的数组类型, 该数组被称为柔性数组(flexible array), 仓颉**不支持包含柔性数组的结构体的映射**

和预想的没错, `@C struct`成员需要保证类型满足`CType`约束

且, C语言结构体成员可以使用柔性数组, 但仓颉中不支持

如果需要调用的是C库中已存在的函数, 且C代码定义有自己的结构体, 需要保证`@C struct`的内存布局与C代码的结构体内存布局完全一致

<Warning>

仓颉中只是与C语言的互操作, 所以`struct`需要满足C语言标准, 而不是C++中的`struct`

</Warning>

##### 函数

仓颉中的函数类型是不满足`CType`约束的, 故提供了`CFunc`作为 C 语言中函数指针的映射

如下 C 语言代码中定义的函数指针类型, 在仓颉中可以映射为`CFunc<() -> Unit>`

```c
// Function pointer in C.
typedef void (*FuncPtr) ();
```

`CFunc`的具体细节参见前面 [`CFunc`一节]()

#### `CType`接口

> `CType`接口是一个语言内置的空接口, 它是`CType`约束的具体实现, 所有 C 互操作支持的类型都隐式地实现了该接口, 因此所有 C 互操作支持的类型都可以作为`CType`类型的子类型使用
>
> ```cangjie
> @C
> struct Data {}
>
> @C
> func foo() {}
>
> main() {
>     var c: CType = Data()               // ok
>     c = 0                               // ok
>     c = true                            // ok
>     c = CString(CPointer<UInt8>())      // ok
>     c = CPointer<Int8>()                // ok
>     c = foo                             // ok
> }
> ```
>
> `CType`接口是仓颉中的一个`interface`类型, 它本身不满足`CType`约束
>
> 同时, `CType`接口不允许被继承、显式实现、扩展
>
> `CType`接口不会突破其子类型的使用限制
>
> ```cangjie
> @C
> struct A {}                         // 隐式实现 CType
>
> class B <: CType {}                 // error, CType 无法被显式实现
>
> class C {}
>
> extend C <: CType {}                // error, CType 无法被显式实现
>
> class D<T> where T <: CType {}
>
> main() {
>     var d0 = D<Int8>()              // ok
>     var d1 = D<A>()                 // ok
> }
> ```

`CType`在仓颉中是一个空接口, 主要用于约束
