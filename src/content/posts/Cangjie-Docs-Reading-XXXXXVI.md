---
title: "仓颉文档阅读-开发指南X: Collection 类型(I) - 基础Collection概述 与 ArrayList"
published: 2025-11-27 14:15:35
description: '仓颉文档阅读的开发指南部分, 本篇文章介绍仓颉语言的 Collection类型 的相关内容'
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

## `Collection`类型

### 基础`Collection`类型概述

> 本章介绍仓颉语言中常用的几种基础`Collection`类型, 包括`Array`、`ArrayList`、`HashSet`和`HashMap`
> 
> 可以在不同的场景中选择适合对应业务的类型: 
> 
> - `Array`: 不需要增加和删除元素, 但需要修改元素
> 
> - `ArrayList`: 需要频繁对元素增删查改
> 
> - `HashSet`: 希望每个元素都是唯一的
> 
> - `HashMap`: 希望存储一系列的映射关系
> 
> 下表是这些类型的基础特性: 
> 
> | 类型名称 | 元素可变 | 增删元素 | 元素唯一性 | 有序序列 |
> | :------: | :------: | :------: | :--------: | :------: |
> | `Array<T>` | `Y` | `N` | `N` | `Y` |
> | `ArrayList<T>` | `Y` | `Y` | `N` | `Y` |
> | `HashSet<T>` | `N` | `Y` | `Y` | `N` |
> | `HashMap<K, V>` | `K: N`, `V: Y` | `Y` | `K: Y`, `V: N` | `N` |

关于`Collection`类型, 可以类比C++中STL容器的概念

就是一系列通过各种数据结构实现的可以存储其他数据的容器类型

仓颉中基础的容器类型:

1. `Array`, 定长数组, 已有元素可原地修改, 但不可增加元素(数组长度无法修改), 每个元素有序号

2. `ArrayList`, 变长数组, 已有元素可原地修改, 也可以增删元素(数组长度可以修改), 每个元素有序号

3. `HashSet`, 元素唯一, 已有元素不可原地修改, 只能增删元素, 元素无序号

4. `HashMap`, 用于存储`键:值`映射, `键:值`作为元素, 保证键唯一, 但已有键不可原地修改, 键对应的值可原地修改, 可增删元素, 元素无序号

### `ArrayList`

> 使用`ArrayList`类型需要导入`collection`包: 
> 
> ```cangjie
> import std.collection.*
> ```
> 
> 仓颉使用`ArrayList<T>`表示`ArrayList`类型, `T`表示`ArrayList`的元素类型, `T`可以是任意类型
> 
> `ArrayList`具备非常好的扩容能力, 适合于需要频繁增加和删除元素的场景
> 
> 相比`Array`, `ArrayList`既可以原地修改元素, 也可以原地增加和删除元素
> 
> `ArrayList`的可变性是一个非常有用的特征, 可以让同一个`ArrayList`实例的所有引用都共享同样的元素, 并且对它们统一进行修改
> 
> ```cangjie
> var a: ArrayList<Int64> = ...     // ArrayList 元素类型为 Int64
> var b: ArrayList<String> = ...    // ArrayList 元素类型为 String
> ```
> 
> 元素类型不相同的`ArrayList`是不相同的类型, 所以它们之间不可以互相赋值
> 
> 因此以下例子是不合法的
> 
> ```cangjie
> b = a                             // 类型不匹配
> ```
> 
> 仓颉中可以使用构造函数的方式构造一个指定的`ArrayList`
> 
> ```cangjie
> let a = ArrayList<String>()                                   // 创建一个 元素类型为 String 的空 ArrayList
> let b = ArrayList<String>(100)                                // 创建一个 元素类型为 String 的 ArrayList, 并分配了 100 个空间
> let c = ArrayList<Int64>([0, 1, 2])                           // 创建一个 元素类型为 Int64 的 ArrayList, 包含元素 0、1、2
> let d = ArrayList<Int64>(c)                                   // 使用另一个`Collection`来初始化一个 ArrayList
> let e = ArrayList<String>(2, {x: Int64 => x.toString()})      // 创建一个元素类型为 String、大小为 2 的 ArrayList, 所有元素均按指定规则函数初始化
> ```

`ArrayList`是在`std.collection`包中定义的, 使用时要将其导入

可以传入不同的类型实参, 声明存储不同类型元素的`ArrayList`, 不同元素类型的`ArrayList`属于不同类型

创建实例时, 可以调用`ArrayList`的构造函数构造: `ArrayList<T>(一些参数)`

从文档的描述看, **不同`Collection`类型之间应该是 可以通过构造函数传参 相互创建新实例的, 但应该要保证元素类型一致**

#### 访问`ArrayList`成员

> 当需要对`ArrayList`的所有元素进行访问时, 可以使用`for-in`循环遍历`ArrayList`的所有元素
> 
> ```cangjie
> import std.collection.*
> 
> main() {
>     let list = ArrayList<Int64>([0, 1, 2])
>     for (i in list) {
>         println("The element is ${i}")
>     }
> }
> ```
> 
> 编译并执行上面的代码, 会输出: 
> 
> ```text
> The element is 0
> The element is 1
> The element is 2
> ```
> 
> 当需要知道某个`ArrayList`包含的元素个数时, 可以使用`size`属性获得对应信息
> 
> ```cangjie
> import std.collection.*
> 
> main() {
>     let list = ArrayList<Int64>([0, 1, 2])
>     if (list.size == 0) {
>         println("This is an empty arraylist")
>     } else {
>         println("The size of arraylist is ${list.size}")
>     }
> }
> ```
> 
> 编译并执行上面的代码, 会输出: 
> 
> ```text
> The size of arraylist is 3
> ```
> 
> 当想访问单个指定位置的元素时, **可以使用下标语法访问**(下标的类型必须是`Int64`)
> 
> 非空`ArrayList`的第一个元素总是从位置`0`开始的
> 
> 可以从`0`开始访问`ArrayList`的任意一个元素, 直到最后一个位置(`ArrayList`的`size - 1`)
> 
> 使用负数或大于等于`size`的索引会**触发运行时异常**
> 
> ```cangjie
> let a = list[0]   // a == 0
> let b = list[1]   // b == 1
> let c = list[-1]  // 运行时异常
> ```
> 
> `ArrayList`也支持下标中使用`Range`的语法, 详见 [Array]() 章节

`ArrayList`的元素, 可以通过`for-in`遍历访问, 也提供有`size()`接口获取元素数量

`ArrayList`也可以通过下表索引 使用`[index]`语法 随机访问元素, 下标范围是: `[0, size - 1]`

如果尝试超出下标范围进行随机访问, 则会触发运行时异常

`ArrayList`的`[]`也支持使用`Range`语法截取范围

`ArrayList`的元素访问语法, 基本与`Array`保持一致

#### 修改 ArrayList

> 可以使用下标语法对某个位置的元素进行修改
> 
> ```cangjie
> let list = ArrayList<Int64>([0, 1, 2])
> list[0] = 3
> ```
> 
> `ArrayList`是引用类型, `ArrayList`在作为表达式使用时不会拷贝副本, 同一个`ArrayList`实例的所有引用都会共享同样的数据
> 
> 因此对`ArrayList`元素的修改会影响到该实例的所有引用
> 
> ```cangjie
> let list1 = ArrayList<Int64>([0, 1, 2])
> let list2 = list1
> list2[0] = 3
> // list1 包含元素 3, 1, 2
> // list2 包含元素 3, 1, 2
> ```
> 
> 如果需要将单个元素添加到`ArrayList`的末尾, 请使用`add`函数
> 
> 如果希望同时添加多个元素到末尾, 可以使用`add(all!: Collection<T>)`函数, 这个函数可以**接受其他相同元素类型的`Collection`类型**(例如`Array`)
> 
> ```cangjie
> import std.collection.*
> 
> main() {
>     let list = ArrayList<Int64>()
>     list.add(0)                     // list 包含元素 0
>     list.add(1)                     // list 包含元素 0, 1
>     let li = [2, 3]
>     list.add(all: li)               // list 包含元素 0, 1, 2, 3
> }
> ```
> 
> 可以通过`add(T, at!: Int64)`和`add(all!: Collection<T>, at!: Int64)`函数将指定的单个元素或相同元素类型的`Collection`值 **插入到指定索引的位置**
> 
> 该索引处的元素和后面的元素会 **被挪后以腾出空间**
> 
> ```cangjie
> let list = ArrayList<Int64>([0, 1, 2])  // list 包含 0, 1, 2
> list.add(4, at: 1)                      // list 包含 0, 4, 1, 2
> ```
> 
> 从`ArrayList`中删除元素, 可以使用`remove`函数, 需要 **指定删除的索引**
> 
> 该索引处**后面的元素会前移以填充空间**
> 
> ```cangjie
> let list = ArrayList<String>(["a", "b", "c", "d"])  // list 包含元素 "a", "b", "c", "d"
> list.remove(at: 1)                                  // 删除下标1的元素, 现在 list 包含元素 "a", "c", "d"
> ```

`ArrayList`是引用类型, 实例被共享引用, 出现 一处修改 处处修改的情况

**原地修改元素**, 可以通过`[]`下标索引访问特定索引元素 并修改此索引的元素

**添加元素**, 可以使用`add()`函数, 可以新增单个元素到末尾, 也可以新增一个`Collection`的所有元素到末尾

更可以指定索引位置在索引位置新增元素, 此时可以使用`at`明明形参传入目标索引位置

如果指定索引位置, 那么原索引位置及后面的元素, 都会后移为新元素挪位置

**删除元素**, 可以使用`remove(at!: T)`函数, 可以删除指定索引位置的元素, 删除之后, 后面的有效元素都会向前移动

#### 增加`ArrayList`的大小

> 每个`ArrayList`都需要特定数量的内存来保存其内容
> 
> 当向`ArrayList`添加元素并且该`ArrayList`开始超出其保留容量时, 该`ArrayList`会分配更大的内存区域并 **将其所有元素复制到新内存中**
> 
> 这种增长策略意味着触发重新分配内存的添加操作具有性能成本, 但随着`ArrayList`的保留内存变大, 它们**发生的频率会越来越低**
> 
> 如果知道大约需要添加多少个元素, 可以在添加之前预备足够的内存以避免中间重新分配, 这样可以提升性能表现
> 
> ```cangjie
> import std.collection.*
> 
> main() {
>     let list = ArrayList<Int64>(100)  // 一次性分配空间
>     for (i in 0..100) {
>         list.add(i)                   // 不会触发空间重新分配
>     }
>     
>     list.reserve(100)                 // 准备更多空间
>     for (i in 0..100) {
>         list.add(i)                   // 不会触发空间重新分配
>     }
> }
> ```

添加元素可能会导致`ArrayList`剩余空间不足, 当前剩余空间为`0`就是剩余空间不足, 如果再添加新元素, 就会自动扩容

扩容是新开辟一块更大的空间, 然后将**旧数据拷贝到新空间**, 这会出现很大的性能消耗

所以, 可以在创建实例时 就调用相关构造函数预开辟好空间, 或者 可以使用`reserve(additional)`尝试增加`additional`块新空间, 但不一定按照`additional`扩容

> [!TIP]
> 
> 查看文档发现, 调用`reserve(additional: Int64)`扩容时, 遵循以下规则:
> 
> 1. 当`additional <= 0`时, 不扩容
> 
> 2. 当`剩余空间 >= additional`时, 不扩容
> 
> 3. 当`剩余空间 < additional`时, 取`max(Int64(原始容量 * 1.5), 原始容量 + additional)`, 进行扩容
> 
>     即, 最终容量是: `max(Int64(原始容量 * 1.5), 原始容量 + additional)`
> 
> 由此规则可以推测, 自动扩容时 是按照原容量的`1.5`倍进行扩容的
> 
> 且推测, `reserve()`触发扩容时, 也是开辟新空间且拷贝原数据
> 
> **`reserve(additional)`不是将容量扩容到`additional`, 而是尝试新增`additional`块空间**
