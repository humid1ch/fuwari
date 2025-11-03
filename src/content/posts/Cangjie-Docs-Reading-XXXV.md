---
title: "仓颉文档阅读-开发指南III: 基础数据类型(V) - 区间类型、Unit类型和Nothing类型"
published: 2025-11-03 14:07:12
description: "仓颉文档阅读的开发指南部分, 本篇文章介绍一些仓颉语言的基础类型: 区间类型、Unit类型和Nothing类型"
image: 'https://humid1ch.oss-cn-shanghai.aliyuncs.com/20250929154944807.webp'
category: Blogs
tags:
    - 开发语言
    - 仓颉
---

> [!NOTE]
> 
> 阅读文档版本:
> 
> 语言规约 [Cangjie-0.53.18-Spec](https://cangjie-lang.cn/docs?url=/0.53.18/Spec/source_zh_cn/Chapter_01_Lexical_Structure(zh).html)
> 
> 具体开发指南 [Cangjie-LTS-1.0.3](https://cangjie-lang.cn/docs?url=/1.0.3/index.html)
> 
> 在阅读 了解仓颉的语言规约时, 难免会涉及到一些仓颉的示例代码, 但 我们对仓颉并不熟悉, 所以可以用[仓颉在线体验](https://cangjie-lang.cn/playground)快速验证
> 
> 有条件当然可以直接[配置Canjie-SDK](https://cangjie-lang.cn/download/1.0.3)

> [!WARNING]
> 
> 博主在此之前, 基本只接触过C/C++语言, 对大多现代语言都没有了解, 所以在阅读过程中遇到相似的概念, 难免会与C/C++中的相似概念作类比, 见谅
> 
> 且, 本系列是文档阅读, 而不是仓颉的零基础教学, 所以如果要跟着阅读的话最好有一门编程语言的开发经验

> [!WARNING]
> 
> 在阅读仓颉编程语言的开发指南之前, 已经大概阅读了一遍 仓颉编程语言的语言规约，已经对仓颉编程语言有了一个大概的了解
> 
> 所以在阅读开发指南时，不会对类似：类、函数、结构体、接口等解释起来较为复杂名称 做出解释

> 此样式内容, 表示文档原文内容

## 基础数据类型

### 区间类型

> 区间类型用于表示拥有固定步长的序列，区间类型是一个泛型，使用 `Range<T>` 表示
> 
> 当 `T` 被实例化不同的类型时（要求此类型必须支持关系操作符，并且可以和 `Int64` 类型的值做加法），会得到不同的区间类型，如最常用的 `Range<Int64>` 用于表示整数区间
> 
> 每个区间类型的实例都会包含 `start`、`end` 和 `step` 三个值
> 
> 其中，`start` 和 `end` 分别表示序列的起始值和终止值，`step` 表示序列中前后两个元素之间的差值（即步长）；`start` 和 `end` 的类型相同（即 `T` 被实例化的类型），`step` 类型是 `Int64`，并且它的值不能等于 `0`
> 
> 下面的例子给出了区间类型的实例化方式（关于区间类型定义和其中的属性，详见《仓颉编程语言库 API》）：
> 
> ```cangjie
> // Range<T>(start: T, end: T, step: Int64, hasStart: Bool, hasEnd: Bool, isClosed: Bool)
> let r1 = Range<Int64>(0, 10, 1, true, true, true)         // r1 包含 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10
> let r2 = Range<Int64>(0, 10, 1, true, true, false)        // r2 包含 0, 1, 2, 3, 4, 5, 6, 7, 8, 9
> let r3 = Range<Int64>(10, 0, -2, true, true, false)       // r3 包含 10, 8, 6, 4, 2
> ```

区间`Range`, 是一个有限的每个元素之间固定步长的序列, 比如: `1, 2, 3, 4, 5, 6, 7, 8`, 元素之家的步长是`1`

实例化时, 构造函数传入`start(表示起始值)`、`end(表示终止值)`和`step(表示从起始值到终止值每个元素的步长)`

步长可以是正数和负数, 且`start`和`end`的大小要符合步长会发生的变化趋势, 但**禁止是`0`**

如果`Range<Int64>(0, 10, 2, true, true, true)`, 则 `start为0`, `end为10`, 步长为`2`

最终得到的区间序列, 值为: `0, 2, 4, 6, 8, 10`

`Range`的构造函数还存在另外三个参数: `hasStart`、`hasEnd`和`isClosed`

`isClosed`表示是否右闭区间, 即**是否包含终止值本身**

其实文档中的例子就已经很好的说明了

而`hasStart`和`hasEnd`大概率是用在数组索引上的`Range`才有用的

关于`Range`用在数组索引中, 请参考[访问`Array`成员](https://blog.humid1ch.cn/posts/cangjie-docs-reading-xxxv/#访问Array成员)

#### 区间类型字面量

> 区间字面量有两种形式：**“左闭右开”**区间和**“左闭右闭”**区间
> 
> - “左闭右开”区间的格式是 `start..end : step`，它表示一个从 `start` 开始，以 `step` 为步长，到 `end`（不包含 `end`）为止的区间；
> 
> - “左闭右闭”区间的格式是 `start..=end : step`，它表示一个从 `start` 开始，以 `step` 为步长，到 `end`（包含 `end`）为止的区间
> 
> 下面的例子定义了若干区间类型的变量：
> 
> ```cangjie
> let n = 10
> let r1 = 0..10 : 1            // r1 包含 0, 1, 2, 3, 4, 5, 6, 7, 8, 9
> let r2 = 0..=n : 1            // r2 包含 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10
> let r3 = n..0 : -2            // r3 包含 10, 8, 6, 4, 2
> let r4 = 10..=0 : -2          // r4 包含 10, 8, 6, 4, 2, 0
> ```
> 
> 区间字面量中，可以不写 `step`，此时 **`step`默认等于`1`**，但是**`step`的值不能等于`0`**
> 
> 另外，区间也有可能是空的（即不包含任何元素的空序列），举例如下：
> 
> ```cangjie
> let r5 = 0..10                // r5 的步长是 1, r5 包含 0, 1, 2, 3, 4, 5, 6, 7, 8, 9
> let r6 = 0..10 : 0            // Error, 步长不能是 0
> 
> let r7 = 10..0 : 1            // r7 to r10 是空区间
> let r8 = 0..10 : -1
> let r9 = 10..=0 : 1
> let r10 = 0..=10 : -1
> ```
> 
> > 注意：
> > 
> > - 表达式 `start..end : step` 中，当 `step > 0` 且 `start >= end`，或者 `step < 0` 且 `start <= end` 时，`start..end : step` 是一个空区间
> > 
> > - 表达式 `start..=end : step` 中，当 `step > 0` 且 `start > end`，或者 `step < 0` 且 `start < end` 时，`start..=end : step` 是一个空区间

区间字面量的格式是: `start..(=)end( : step)`

`=`表示是否有右闭区间, 可以省略

`step`指定步长, 默认为`1`, 禁止为`0` 可以省略

当`step`的值 无法表示 `start`到`end`的变化趋势时, 此字面量表示一个空区间

即:

1. 是**右开区间**时
    
    如果 `start >= end`, `step > 0`时 为空区间

    如果 `start <= end`, `step < 0`时 为空区间

2. 如果**右闭区间**时

    如果 `start > end`, `step > 0`时 为空区间

    如果 `start < end`, `step < 0`时 为空区间

### `Unit`类型

> 对于那些只关心副作用而不关心值的表达式，它们的类型是 `Unit`
> 
> 例如，`print` 函数、赋值表达式、复合赋值表达式、自增和自减表达式、循环表达式，它们的类型都是 `Unit`
> 
> `Unit` 类型只有一个值，也是它的字面量：`()`
> 
> 除了赋值、判等和判不等外，`Unit` 类型不支持其他操作

仓颉中的`Unit`可以类比C/C++中的`void`类型, 表示空, 字面量也只有一个`()`

### `Nothing`类型

> `Nothing` 是一种特殊的类型，它不包含任何值，并且 `Nothing` 类型是所有类型的子类型（这当中也包括`Unit`类型）
> 
> `break`、`continue`、`return` 和 `throw` 表达式的类型是 `Nothing`，程序执行到这些表达式时，它们**之后的代码将不会被执行**
> 
> `return` 只能在函数体中使用，`break`、`continue` 只能在循环体中使用，参考如下示例：
> 
> ```cangjie
> while (true) {
>     func f() {
>         break                         // Error, break 语句必须直接位于循环内部
>     }
>     let g = { =>
>         continue                      // Error, continue 语句必须直接位于循环内部
>     }
> }
> ```
> 
> 由于函数的形参和其默认值不属于该函数的函数体，所以下面例子中的 `return` 表达式缺少包围它的函数体——它既不属于外层函数 `f`（因为内层函数定义 `g` 已经开始），也不在内层函数 `g` 的函数体中（该用例相关内容，请参考嵌套函数）：
> 
> ```cangjie
> func f() {
>     func g(x!: Int64 = return) {      // Error, return 必须在函数体中使用
>         0
>     }
>     1
> }
> ```
> 
> > 注意：
> > 
> > 目前编译器还不允许在使用类型的地方显式地使用 `Nothing` 类型

`Nothing`表示什么都不是, 它是所有类型的子类型

`return`、`break`、`continue`和`throw`表达式, 就是`Nothing`类型的, 它们之后的代码不会被执行

`Nothing`类型目前不允许被显式使用
