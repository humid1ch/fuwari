---
title: "仓颉文档阅读-开发指南IV: 函数(IV) - 函数调用语法糖"
published: 2025-11-12 18:06:22
description: '仓颉文档阅读的开发指南部分, 本篇文章介绍仓颉语言的函数相关内容'
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

## 函数

### 函数调用语法糖

语法糖的意思是 更友好的相关语法使用方式

#### 尾随`lambda`

> 尾随`lambda`可以使函数的调用看起来像是语言内置的语法一样, 增加语言的可扩展性
> 
> 当函数最后一个形参是函数类型, 并且函数调用对应的实参是`lambda`时, 可以使用尾随`lambda`语法, **将`lambda`放在函数调用的尾部, 圆括号外面**
> 
> 例如, 下例中定义了一个`myIf`函数, 它的第一个参数是`Bool`类型, 第二个参数是函数类型
> 
> 当第一个参数的值为`true`时, 返回第二个参数调用后的值, 否则返回`0`
> 
> 调用`myIf`时可以像普通函数一样调用, 也可以使用尾随`lambda`的方式调用
> 
> ```cangjie
> func myIf(a: Bool, fn: () -> Int64) {
>     if(a) {
>         fn()
>     } else {
>         0
>     }
> }
> 
> func test() {
>     myIf(true, { => 100 })    // 普通函数调用
> 
>     myIf(true) {              // 尾随闭包调用
>         100
>     }
> }
> ```
> 
> 当函数调用有且只有一个`lambda`实参时, 还可以省略`()`, 只写`lambda`
> 
> 示例: 
> 
> ```cangjie
> func f(fn: (Int64) -> Int64) { fn(1) }
> 
> func test() {
>     f { i => i * i }
> }
> ```

只有函数的最后一个参数是函数类型, 且实参将要传入`lambda`表达式时, 才能使用尾随`lambda`语法

尾随`lambda`, 就是在函数调用时, 不在参数列表中传入`lambda`实参, 而是在**前面的实参都传入之后, 在函数调用的`()`之后, 定义`lambda`表达式**

如果函数的参数只有一个函数类型, 那么使用尾随`lambda`时, 甚至可以省略函数调用的`()`

#### `Flow`表达式

> 流操作符包括两种: 表示数据流向的中缀操作符`|>`(称为`pipeline`)和表示函数组合的中缀操作符`~>`(称为`composition`)

这两个操作符在之前的文章中已经有过初步的认识: [仓颉文档阅读-语言规约IV: 表达式(IV)](https://www.humid1ch.cn/posts/cangjie-docs-reading-ix/#流表达式)

##### `Pipeline`表达式

> 当需要对输入数据做一系列的处理时, 可以使用`pipeline`表达式来简化描述
> 
> `pipeline`表达式的语法形式如下: `e1 |> e2`, 等价于如下形式的语法糖: `let v = e1; e2(v)`
> 
> 其中`e2`是函数类型的表达式, `e1`的类型是`e2`的参数类型的子类型
> 
> 示例: 
> 
> ```cangjie
> func inc(x: Array<Int64>): Array<Int64> {   // 将数组中每个元素的值加 '1'
>     let s = x.size
>     var i = 0
>     for (e in x where i < s) {
>         x[i] = e + 1
>         i++
>     }
>     x
> }
> 
> func sum(y: Array<Int64>): Int64 {          // 获取数组中元素的总和
>     var s = 0
>     for (j in y) {
>         s += j
>     }
>     s
> }
> 
> let arr: Array<Int64> = [1, 3, 5]
> let res = arr |> inc |> sum                 // res = 12
> ```

**`|>(pipeline)`是函数调用的语法糖**

`e1 |> e2`, 就表示 将`e1`作为`e2`的实参调用`e2`, 表达式的结果就是`e2`调用的返回值

向文档中的例子: `arr |> inc |> sum`, 其实等价于`sum(inc(arr))`, 不过`|>(pipeline)`可以更直观的看出数据流向

如果是初次见到的话, 可能会懵一下, 但毕竟是语法糖, 所以其实还是很简单的

##### `Composition`表达式

> **`composition`表达式表示两个单参函数的组合**
> 
> `composition`表达式语法为`f ~> g`, 等价于`{ x => g(f(x)) }`
> 
> 其中`f`, `g`均为**只有一个参数的函数类型的表达式**
> 
> `f`和`g`组合, 则要求`f(x)`的返回类型是`g(...)`的参数类型的子类型
> 
> 示例 1: 
> 
> ```cangjie
> func f(x: Int64): Float64 {
>     Float64(x)
> }
> func g(x: Float64): Float64 {
>     x
> }
> 
> var fg = f ~> g                           // 等价于 { x: Int64 => g(f(x)) }
> ```
> 
> 示例 2: 
> 
> ```cangjie
> func f(x: Int64): Float64 {
>     Float64(x)
> }
> 
> let lambdaComp = ({x: Int64 => x}) ~> f   // 等价于 { x: Int64 => f({x: Int64 => x}(x)) }
> ```
> 
> 示例 3: 
> 
> ```cangjie
> func h1<T>(x: T): T { x }
> func h2<T>(x: T): T { x }
> var hh = h1<Int64> ~> h2<Int64>           // 等价于 { x: Int64 => h2<Int64>(h1<Int64>(x)) }
> ```
> 
> > 注意: 
> > 
> > 表达式`f ~> g`中, 会**先对`f`求值, 然后对`g`求值**, 最后才会进行函数的组合

`|>(pipeline)`是函数调用的语法糖, 表达式的值 是**最终函数调用的返回值**

与`|>(pipeline)`不同, `~>(composition)`表达式是**单参函数按顺序调用组合**的语法糖, 最终可以等价于一个`lambda`, 即 表达式的值 其实是**一个符合语义的`lambda`**

`左函数 ~> 右函数`, 会先对左函数求值, 并作为右函数的参数

##### `|>(pipeline)`注意事项

> 另外, 流操作符不能与 **无默认值的命名形参函数**, 直接一同使用, 这是因为无默认值的命名形参函数必须给出命名实参才可以调用
> 
> 例如: 
> 
> ```cangjie
> func f(a!: Int64): Unit {}
> 
> var a = 1 |> f                            // Error
> ```
> 
> 如果需要使用, 开发者可以通过`lambda`表达式传入`f`函数的命名实参: 
> 
> ```cangjie
> func f(a!: Int64): Unit {}
> 
> var x = 1 |>  { x: Int64 => f(a: x) }     // Ok
> ```
> 
> 由于相同的原因, 当`f`的参数有默认值时, 直接与流运算符一起使用也是错误的, 例如: 
> 
> ```cangjie
> func f(a!: Int64 = 2): Unit {}
> 
> var a = 1 |> f                            // Error
> ```
> 
> 但是当命名形参都存在默认值时, 不需要给出命名实参也可以调用该函数, 函数仅需要传入非命名形参, 那么这种函数是可以同流运算符一起使用的, 例如: 
> 
> ```cangjie
> func f(a: Int64, b!: Int64 = 2): Unit {}
> 
> var a = 1 |> f                            // Ok
> ```
> 
> 当然, 如果想要在调用`f`时, 为参数`b`传入其他参数, 那么也需要借助`lambda`表达式: 
> 
> ```cangjie
> func f(a: Int64, b!: Int64 = 2): Unit {}
> 
> var a = 1 |> {x: Int64 => f(x,  b: 3)}    // Ok
> ```

`|>(pipeline)`可以与非单参函数一起使用, 但是 如果直接使用必须要保证:

1. 允许只显式传入一个参数进行调用

    因为, `|>(pipeline)`进行函数调用只能传入一个参数

2. 需要显示传入的参数, 不能是命名参数

    因为, 命名参数无论是否存在默认值, 如果需要显式传入实参, 那么**必须显式命名**

当然, 可以通过单参`lambda`, 然后在`lambda`体内 显式传参调用, 实现间接使用`|>`

### 变长参数

> 变长参数是一种特殊的函数调用语法糖
> 
> 当**形参最后一个非命名参数是`Array`类型**时, 实参中对应位置可以直接传入参数序列代替`Array`字面量(参数个数可以是`0`个或多个)
> 
> 示例如下: 
> 
> ```cangjie
> func sum(arr: Array<Int64>) {
>     var total = 0
>     for (x in arr) {
>         total += x
>     }
>     return total
> }
> 
> main() {
>     println(sum())
>     println(sum(1, 2, 3))
> }
> ```
> 
> 程序输出: 
> 
> ```text
> 0
> 6
> ```
> 
> 需要注意, **只有最后一个非命名参数可以作为变长参数**, 命名参数不能使用这个语法糖
> 
> ```cangjie
> func length(arr!: Array<Int64>) {
>     return arr.size
> }
> 
> main() {
>     println(length())                 // Error, 预期 1 个参数, 找到 0 个参数
>     println(length(1, 2, 3))          // Error, 预期 1 个参数, 找到 3 个参数
> }
> ```

仓颉函数也可以拥有变长参数, 即 **当且仅当函数的最后一个形参是 非命名的`Array`类型 时**

仓颉函数的变长参数可以是`0`个参数, 当然也可以更多

> 变长参数可以出现在全局函数、静态成员函数、实例成员函数、局部函数、构造函数、函数变量、`lambda`、函数调用操作符重载、索引操作符重载的调用处
> 
> 不支持其他操作符重载、`composition`、`pipeline`这几种调用方式
> 
> 示例如下: 
> 
> ```cangjie
> class Counter {
>     var total = 0
>     init(data: Array<Int64>) { total = data.size }
>     operator func ()(data: Array<Int64>) { total += data.size }
> }
> 
> main() {
>     let counter = Counter(1, 2)
>     println(counter.total)
>     counter(3, 4, 5)
>     println(counter.total)
> }
> ```
> 
> 程序输出: 
> 
> ```text
> 2
> 5
> ```
> 
> **函数重载决议总是会优先考虑不使用变长参数就能匹配的函数**, 只有在所有函数都不能匹配, 才尝试使用变长参数解析
> 
> 示例如下: 
> 
> ```cangjie
> func f<T>(x: T) where T <: ToString {
>     println("item: ${x}")
> }
> 
> func f(arr: Array<Int64>) {
>     println("array: ${arr}")
> }
> 
> main() {
>     f()
>     f(1)
>     f(1, 2)
> }
> ```
> 
> 程序输出: 
> 
> ```text
> array: []
> item: 1
> array: [1, 2]
> ```
> 
> 当编译器无法决议时会报错: 
> 
> ```cangjie
> func f(arr: Array<Int64>) { arr.size }
> func f(first: Int64, arr: Array<Int64>) { first + arr.size }
> 
> main() {
>     println(f(1, 2, 3))                                       // Error
> }
> ```

仓颉函数的变长参数是一种语法糖, 正常调用的话, 最后一个参数应该传入`Array`实例

函数变长参数调用, 只是`Array`实例 以 传入多个相同类型实参的形式 调用

所以, 变长参数可能与重载函数存在一定的冲突

仓颉规定 **函数重载决议总是会优先考虑不使用变长参数就能匹配的函数**, 只有在所有函数都不能匹配, 才尝试使用变长参数解析
