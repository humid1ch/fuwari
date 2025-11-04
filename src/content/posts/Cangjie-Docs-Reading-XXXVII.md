---
title: '仓颉文档阅读-开发指南IV: 函数(II) - 函数的类型'
published: 2025-11-04 14:22:13
description: '仓颉文档阅读的开发指南部分, 本篇文章介绍仓颉语言的函数相关内容'
image: 'https://humid1ch.oss-cn-shanghai.aliyuncs.com/20250929154944807.webp'
tags: ['开发语言', '仓颉']
category: Blogs
---

> [!NOTE]
> 
> 阅读文档版本:
> 
> 语言规约 [Cangjie-0.53.18-Spec](https://cangjie-lang.cn/docs?url=/0.53.18/Spec/source_zh_cn/Chapter_01_Lexical_Structure(zh).html)
> 
> 具体开发指南 [Cangjie-LTS-1.0.3](https://cangjie-lang.cn/docs?url=/1.0.3/index.html) 
> 
> 在阅读 了解仓颉的语言规约时, 难免会涉及到一些仓颉的示例代码, 但 我们对仓颉并不熟悉, 所以可以用 [仓颉在线体验](https://cangjie-lang.cn/playground) 快速验证
> 
> 有条件当然可以直接 [配置Canjie-SDK](https://cangjie-lang.cn/download/1.0.3) 

> [!WARNING]
> 
> 博主在此之前, 基本只接触过C/C++语言, 对大多现代语言都没有了解, 所以在阅读过程中遇到相似的概念, 难免会与C/C++中的相似概念作类比, 见谅
> 
> 且, 本系列是文档阅读, 而不是仓颉的零基础教学, 所以如果要跟着阅读的话最好有一门编程语言的开发经验

> [!WARNING]
> 
> 在阅读仓颉编程语言的开发指南之前, 已经大概阅读了一遍 仓颉编程语言的语言规约，已经对仓颉编程语言有了一个大概的了解
> 
> 所以在阅读开发指南时，不会对类似: 类、函数、结构体、接口等解释起来较为复杂名称 做出解释

> 此样式内容, 表示文档原文内容

## 函数

### 函数类型

> 仓颉编程语言中，函数是**一等公民(`first-class citizens`)**，可以作为函数的参数或返回值，也可以赋值给变量

仓颉中的函数是一等公民, 一等公民表示 它可以:

1. 作为值, 赋值给变量

2. 作为实参传递给函数

3. 作为函数返回值返回

4. ...

就像是一个普通的字面量一样, 它可以出现在其他字面量能出现的位置

> 因此**函数本身也有类型**，称之为函数类型
> 
> 函数类型由函数的参数类型和返回类型组成，参数类型和返回类型之间使用 `->` 连接
> 
> 参数类型使用圆括号 `()` 括起来，可以有 `0` 个或多个参数，如果参数超过一个，参数类型之间使用逗号`(,)`分隔
> 
> 例如: 
> 
> ```cangjie
> func hello(): Unit {
>     println("Hello!")
> }
> ```
> 
> 上述示例定义了一个函数，函数名为 `hello`，其类型是 `() -> Unit`，表示该函数没有参数，返回类型为 `Unit`
> 
> 以下给出另一些示例: 
> 
> - 示例: 函数名为 `display`，其类型是 `(Int64) -> Unit`，表示该函数有一个参数，参数类型为 `Int64`，返回类型为 `Unit`
> 
>     ```cangjie
>     func display(a: Int64): Unit {
>         println(a)
>     }
>     ```
> 
> - 示例: 函数名为 `add`，其类型是 `(Int64, Int64) -> Int64`，表示该函数有两个参数，两个参数类型均为 `Int64`，返回类型为 `Int64`
> 
>     ```cangjie
>     func add(a: Int64, b: Int64): Int64 {
>         a + b
>     }
>     ```
> 
> - 示例: 函数名为 `returnTuple`，其类型是 `(Int64, Int64) -> (Int64, Int64)`，两个参数类型均为 `Int64`, 返回类型为元组类型: `(Int64, Int64)`
> 
>     ```cangjie
>     func returnTuple(a: Int64, b: Int64): (Int64, Int64) {
>         (a, b)
>     }
>     ```

为了函数能够作为一等公民, 所以仓颉中的函数是拥有具体类型的

C/C++中的函数并不原生具有具体类型, 不过存在函数指针/`function`包装器等, 结合函数地址可以实现类似的效果

但 仓颉中的每一个函数, 都原生具有具体的类型

仓颉函数的类型的结构是: `(参数类型列表) -> 返回值类型`

参数类型列表, 即为 函数定义时 参数列表的参数类型按顺序排列的列表

返回值类型, 即为 函数的返回值类型

举例:

```cangjie
func add(a: Int64, b: Int64): Int64 {
    return a + b
}

// add函数的类型就是 (Int64, Int64) -> Int64
let funcAdd: (Int64, Int64) -> Int64 = add
```

上面这个使用函数名, 将函数作为一等公民赋值给`funcAdd`变量, 是合法的


#### 函数类型的类型参数

> 可以为 函数类型 标记 **显式的类型参数名**，下面例子中的 `name` 和 `price` 就是 类型参数名
> 
> ```cangjie
> func showFruitPrice(name: String, price: Int64) {
>     println("fruit: ${name} price: ${price} yuan")
> }
> 
> main() {
>     let fruitPriceHandler: (name: String, price: Int64) -> Unit
>     fruitPriceHandler = showFruitPrice
>     fruitPriceHandler("banana", 10)
> }
> ```
> 
> 另外对于一个函数类型，**只允许统一写类型参数名，或者统一不写类型参数名，不能交替存在**
> 
> ```cangjie
> let handler: (name: String, Int64) -> Int64               // Error
> ```

仓颉函数, 在声明使用函数类型时, 可以为 函数类型的 参数类型, 显式声明参数名

什么意思呢?

函数类型是这样的: `(参数类型列表) -> 返回值类型`

参数类型列表中声明了函数的参数类型, 在函数类型中 其实可以为每个类型 再标明参数名

仓颉函数作为一等公民, 可以赋值给变量

举例: 

```cangjie
func add(a: Int64, b: Int64): Int64 {
    return a + b
}

// add函数的类型就是 (Int64, Int64) -> Int64
let funcAdd1: (a: Int64, b: Int64) -> Int64 = add
// 下面这样也可以
let funcAdd2: (c: Int64, d: Int64) -> Int64 = add
```

虽然可以为函数类型标明参数名, 但 此参数名 可以不与 函数实际的形参名 保持一致

不过, 函数类型中的参数名 并没有什么实际的作用, 只是一种类似注释的作用?

但 标明参数名或不标明参数名, 是可选的, 但是只能二选一, 要不 都标明, 要不 都不标明

并且, **仓颉函数类型声明中的参数名, 不能在函数体内作为变量使用, 这里的参数名更多的还是作为一个标识、注释使用**

#### 函数类型作为参数类型

> 示例: 函数名为 `printAdd`，其类型是 `((Int64, Int64) -> Int64, Int64, Int64) -> Unit`，表示该函数有三个参数，参数类型分别为函数类型`(Int64, Int64) -> Int64` 和两个 `Int64`，返回类型为 `Unit`
> 
> ```cangjie
> func printAdd(add: (Int64, Int64) -> Int64, a: Int64, b: Int64): Unit {
>     println(add(a, b))
> }
> ```

仓颉函数作为一等公民, 可以当作函数参数

所以 **函数类型中的参数类型, 也可以是函数类型**

#### 函数类型作为返回类型

> 函数类型可以作为另一个函数的返回类型
> 
> 如下示例中，函数名为 `returnAdd`，其类型是 `() -> (Int64, Int64) -> Int64`，表示该函数无参数，返回类型为函数类型 `(Int64, Int64) -> Int64`
> 
> 注意，`->` 是右结合的
> 
> ```cangjie
> func add(a: Int64, b: Int64): Int64 {
>     a + b
> }
> 
> func returnAdd(): (Int64, Int64) -> Int64 {
>     add
> }
> 
> main() {
>     var a = returnAdd()
>     println(a(1,2))
> }
> ```

仓颉函数作为一等公民, 可以当作函数的返回值

所以 **函数类型中的返回值类型, 可以是函数类型**

```cangjie
func add(a: Int64, b: Int64): Int64 {
    a + b
}

func returnAdd(): (Int64, Int64) -> Int64 {
    add
}
```

上面例子中, `returnAdd`的类型是 `() -> (Int64, Int64) -> Int64`

> [!TIP]
> 
> `->`是右结合的, `() -> (Int64, Int64) -> Int64`, 实际可以看作`() -> ((Int64, Int64) -> Int64)`

#### 函数类型作为变量类型

> 函数名本身也是表达式，它的类型为对应的函数类型
> 
> ```cangjie
> func add(p1: Int64, p2: Int64): Int64 {
>     p1 + p2
> }
> 
> let f: (Int64, Int64) -> Int64 = add
> ```
> 
> 上述示例中，函数名是 `add`，其类型为 `(Int64, Int64) -> Int64`
> 
> 变量 `f` 的类型与 `add` 类型相同，`add` 被用来初始化 `f`
> 
> 若一个函数在当前作用域中被重载(见函数重载)了，那么直接使用该函数名作为表达式可能产生歧义，如果产生歧义编译器会报错，例如: 
> 
> ```cangjie
> func add(i: Int64, j: Int64) {
>     i + j
> }
> 
> func add(i: Float64, j: Float64) {
>     i + j
> }
> 
> main() {
>     var f = add                                   // Error,  模棱两可的 function 'add'
>     var plus: (Int64, Int64) -> Int64 = add       // OK
> }
> ```

仓颉函数作为一等公民, 可以**赋值给变量**

所以, 函数类型是可以作为变量类型的, 上面已经见过了使用方式

不过, 定义函数变量时 也可以不显式声明 函数类型, 由编译器自动推导

但, **如果存在函数重载, 就必须显示声明 函数类型, 否则会编译报错, 无法推导确定**
