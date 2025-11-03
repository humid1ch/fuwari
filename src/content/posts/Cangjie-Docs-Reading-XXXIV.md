---
title: "仓颉文档阅读-开发指南III: 基础数据类型(IV) - 元组类型和数组类型"
published: 2025-10-29 15:29:12
description: "仓颉文档阅读的开发指南部分, 本篇文章介绍一些仓颉语言的基础类型: 元组类型 数组类型"
image: 'https://humid1ch.oss-cn-shanghai.aliyuncs.com/20250929154944807.webp'
category: Blogs
tags:
    - 开发语言
    - 仓颉
---

:::note

阅读文档版本:

语言规约 [Cangjie-0.53.18-Spec](https://cangjie-lang.cn/docs?url=/0.53.18/Spec/source_zh_cn/Chapter_01_Lexical_Structure(zh).html)

具体开发指南 [Cangjie-LTS-1.0.3](https://cangjie-lang.cn/docs?url=/1.0.3/index.html)

在阅读 了解仓颉的语言规约时, 难免会涉及到一些仓颉的示例代码, 但 我们对仓颉并不熟悉, 所以可以用[仓颉在线体验](https://cangjie-lang.cn/playground)快速验证

有条件当然可以直接[配置Canjie-SDK](https://cangjie-lang.cn/download/1.0.3)

:::

:::warning

博主在此之前, 基本只接触过C/C++语言, 对大多现代语言都没有了解, 所以在阅读过程中遇到相似的概念, 难免会与C/C++中的相似概念作类比, 见谅

且, 本系列是文档阅读, 而不是仓颉的零基础教学, 所以如果要跟着阅读的话最好有一门编程语言的开发经验

:::

:::warning

在阅读仓颉编程语言的开发指南之前, 已经大概阅读了一遍 仓颉编程语言的语言规约，已经对仓颉编程语言有了一个大概的了解

所以在阅读开发指南时，不会对类似：类、函数、结构体、接口等解释起来较为复杂名称 做出解释

:::

> [!CAUTION]
> tyest

> 此样式内容, 表示文档原文内容

## 基础数据类型

### 元组类型

> 元组（`Tuple`）可以将多个不同的类型组合在一起，成为一个新的类型
>
> 元组类型使用 (`T1, T2, ..., TN`) 表示，其中 `T1` 到 `TN` 可以是任意类型，不同类型间使用逗号（`,`）连接
>
> 元组至少是`二元`，例如，`(Int64, Float64)` 表示一个二元组类型，`(Int64, Float64, String)` 表示一个三元组类型
>
> 元组的长度是固定的，即一旦定义了一个元组类型的实例，它的长度不能再被更改
>
> 元组类型是**不可变类型**，即一旦定义了一个元组类型的实例，它的内容（即单个元素）不能再被更新
>
> 但整个元组可被覆盖替换，例如：
>
> ```cangjie
> let tuple1 = (8, false)
> var tuple2 = (true, 9, 20)
> tuple2 = tuple1                   // Error, mismatched types
> tuple2[0] = false                 // Error, 'tuple element' can not be assigned
>
> var tuple3 = (9, true)
> tuple3 = tuple1
> println(tuple3[0])                // 8
> println(tuple3[1])                // false
> ```

元组是一种类型, 元组类型的语法格式为: `(T1, T2, T3, ..., TN)`

`T`可以是任意类型, 不同的`T`或不同顺序, 都是一种具体的元组类型, **类型与类型之间也不相同**

即, `(Int64, Int32)`与`(Int32, Int64)`就不是相同的类型

且, 元组是不可变类型, 定义了实例之后, 元素就无法再改变了

元组实例的元素, 可以**通过下标索引进行访问**

元组**至少是二元**的, 即 元组元素至少有两个, 但没有上限

#### 元组类型的字面量

> 元组类型的字面量使用 `(e1, e2, ..., eN)` 表示，其中 `e1` 到 `eN` 是表达式，多个表达式之间使用逗号分隔
>
> 下面的例子中，分别定义了一个 `(Int64, Float64)` 类型的变量 `x`，以及一个 `(Int64, Float64, String)` 类型的变量 `y`，并且使用元组类型的字面量为它们定义了初值：
>
> ```cangjie
> let x: (Int64, Float64) = (3, 3.141592)
> let y: (Int64, Float64, String) = (3, 3.141592, "PI")
> ```
>
> 元组支持通过 `t[index]` 的方式访问某个具体位置的元素，其中 `t` 是一个元组，`index` 是下标，并且 `index` 只能是从 `0` 开始且小于元组元素个数的整数类型字面量，否则编译报错
>
> 下面的例子中，使用 `pi[0]` 和 `pi[1]` 可以分别访问二元组 `pi` 的第一个元素和第二个元素
>
> ```cangjie
> main() {
>     var pi = (3.14, "PI")
>     println(pi[0])
>     println(pi[1])
> }
> ```
>
> 编译并执行上述代码，输出结果为：
>
> ```text
> 3.140000
> PI
> ```
>
> 在赋值表达式中，可使用元组进行多赋值，参见[赋值操作符](https://www.humid1ch.cn/blog/cangjie-docs-reading-xxxi#heading-2)章节

元组的类型是`(T1, T2, T3, ..., TN)`, 元组字面量也很简单: `(expr1, expr2, expr3, ..., exprN)`

元组, 实际就是多个元素组成的一个元素组

可以通过下标访问具体的元素, 但不能修改元素值

使用元组字面量创建元组变量实例, 可以不用声明元组类型, 可以由编译器推导

```cangjie
let person = ("humid1ch", 10, 50, 100)
print("${person[0]} ")
print("${person[1]} ")
print("${person[2]} ")
print("${person[3]} ")
println()
```

#### 元组类型的类型参数

> 可以为元组类型标记显式的类型参数名，下面例子中的 `name` 和 `price` 就是 类型参数名
>
> ```cangjie
> func getFruitPrice (): (name: String, price: Int64) {
>     return ("banana", 10)
> }
> ```
>
> 对于一个元组类型，只允许**统一写**类型参数名，或者**统一不写**类型参数名，**不允许交替存在**，并且**参数名本身不能作为变量使用或用于访问元组中元素**
>
> ```cangjie
> let a: (name: String, Int64) = ("banana", 5)              // Error
> let b: (name: String, price: Int64) = ("banana", 5)       // OK
> b.name                                                    // Error
> ```

元组类型在声明时, 可以像函数参数一样, 添加类型参数名, 比如: `(name: String, age: Int64)`

但类型参数名实际上**只用作说明**, 不能像变量名一样访问目标元素

且, 如果要声明类型参数名, 那么所有类型都要声明, 否则所有类型都不能声明类型参数名

### 数组类型

仓颉中的数组有两种: **`Array`引用类型数组** 和 **`VArray`值类型数组**

#### `Array`

> 可以使用 `Array` 类型来构造单一元素类型，有序序列的数据
>
> 仓颉使用 `Array<T>` 来表示 `Array` 类型
>
> `T` 表示 `Array` 的元素类型，`T` 可以是任意类型
>
> ```cangjie
> var a: Array<Int64> = [0, 0, 0, 0]            // 元素类型为 Int64 的数组
> var b: Array<String> = ["a1", "a2", "a3"]     // 元素类型为 String 的数组
> ```
>
> **元素类型不相同的 `Array` 是不相同的类型**，所以它们之间不可以互相赋值
>
> 因此以下例子是不合法的
>
> ```cangjie
> b = a                                         // 类型不匹配
> ```

数组`Array`, 可以构造**单一元素类型**的有序序列

数组的字面量语法为: `[elem1, elem2, elem3, ..., elemN]`, 字面量中所有`elem`类型需要为同一类型

`Array`的元素类型不同, 则数组类型也不同, **`Array`数组类型只与元素类型有关**

**不同的数组类型变量之间禁止相互赋值**

数组, 也是一个各编程语言中几乎都存在的概念

数组元素可以通过下标索引访问

> 可以轻松使用字面量来初始化一个 `Array`，只需要使用方括号将逗号分隔的值列表括起来即可
>
> 编译器会根据上下文自动推断 `Array` 字面量的类型
>
> ```cangjie
> let a: Array<String> = []                 // 创建一个元素类型为 String 的 空Array数组
> let b = [1, 2, 3, 3, 2, 1]                // 创建一个元素类型为 Int64 的 Array数组，包含元素 1、2、3、3、2、1
> ```
>
> 也可以使用构造函数的方式构造一个指定元素类型的 `Array`
>
> 其中，`repeat` 属于 `Array` 构造函数中的一个命名参数
>
> 需要注意的是，当通过 `repeat` 指定的初始值初始化 `Array` 时，该构造函数不会拷贝 `repeat`，如果 `repeat` 是一个引用类型，构造后数组的每一个元素都将指向相同的引用
>
> ```cangjie
> let a = Array<Int64>()                    // 创建一个元素类型为 Int64 的 空Array数组
> let b = Array<Int64>(3, repeat: 0)        // 创建一个元素类型为 Int64、长度为 3 的 Array数组，所有元素均初始化为 0
> let c = Array<Int64>(3, {i => i + 1})     // 创建一个元素类型为 Int64、长度为 3 的 Array数组，所有元素都由初始化函数初始化
> ```
>
> 示例中 `let c = Array<Int64>(3, {i => i + 1})` 使用了 `lambda` 表达式作为初始化函数来初始化数组中的每一个元素，即 `{i => i + 1}`

除了使用`Array`字面量直接创建`Array`实例

`Array`提供有构造函数, 最常用的就是这三个构造函数:

1. `init()`

    无参, 构造一个空的`Array`数组

    ```cangjie
    let a = Array<Int64>()
    println(a)
    ```

    执行结果为:

    ```text
    []
    ```

2. `init(size: Int64, repeat!: T)`

    第一个参数: `size`, 表示构造的`Array`数组的长度

    第二个参数: `repeat!`, 表示元素初始值, 如果是此参数传入引用类型, 则数组中所有元素**指向同一引用**

    ```cangjie
    let b = Array<Int64>(3, repeat: 1)
    println(b)
    ```

    执行结果为:

    ```text
    [1, 1, 1]
    ```

3. `init(size: Int64, initElement: (Int64) -> T)`

    第一个参数: `size`, 表示构造的`Array`数组的长度

    第二个参数: `initElement`, 各索引元素的初始化函数, 即 各位置会将索引作为参数, 传入此函数 返回值作为元素值

    ```cangjie
    let c = Array<Int64>(3, {i => i + 1})
    println(c)
    ```

    执行结果为:

    ```text
    [1, 2, 3]
    ```

`Array`还有其他构造函数, 具体之后再分析

##### 访问`Array`成员

> 当需要对 `Array` 的所有元素进行访问时，可以使用 `for-in` 循环遍历 `Array` 的所有元素
>
> `Array` 是按元素插入顺序排列的，因此对 `Array` 遍历的顺序总是恒定的
>
> ```cangjie
> main() {
>     let arr = [0, 1, 2]
>     for (i in arr) {
>         println("The element is ${i}")
>     }
> }
> ```
>
> 编译并执行上面的代码，会输出：
>
> ```text
> The element is 0
> The element is 1
> The element is 2
> ```
>
> 当需要知道某个 `Array` 包含的元素个数时，可以使用 `size` 属性获得对应信息
>
> ```cangjie
> main() {
>     let arr = [0, 1, 2]
>     if (arr.size == 0) {
>         println("This is an empty array")
>     } else {
>         println("The size of array is ${arr.size}")
>     }
> }
> ```
>
> 编译并执行上面的代码，会输出：
>
> ```text
> The size of array is 3
> ```
>
> 当想访问单个指定位置的元素时，可以使用下标语法访问（下标的类型必须是 `Int64`）
>
> 非空 `Array` 的第一个元素总是从位置 `0` 开始的
>
> 可以从 `0` 开始访问 `Array` 的任意一个元素，直到最后一个位置（`Array` 的 `size - 1`）
>
> 索引值不能使用负数或者大于等于 `size`，当编译器能检查出索引值非法时，会在编译时报错，否则会在运行时抛异常
>
> ```cangjie
> main() {
>     let arr = [0, 1, 2]
>     let a = arr[0]                // a == 0
>     let b = arr[1]                // b == 1
>     let c = arr[-1]               // 数组大小为 3，但访问索引为 -1，这将导致溢出
> }
> ```

仓颉中要访问`Array`的元素, 有很多种方法:

1. `for-in`按顺序, 迭代遍历访问`Array`的元素

    `for-in`访问`Array`的元素, 是不可修改的

2. `[index]`, 通过下标索引访问`Array`的元素

    通过`[index]`访问`Array`的元素, 是可以修改索引位置的元素值的

    访问单个元素时, `index`索引只能是 **`Int64`类型**, 甚至**不能是`Int32`等其他整数类型**

    且, 仓颉`Array`的索引下标的范围也是: `[0, size - 1]` 或 `[0, size)`

仓颉的`Array`提供有`size`属性, 可以可以直接访问获取`Array`的元素个数

> 如果想获取某一段 `Array` 的元素，可以在下标中传入 `Range` 类型的值，就可以一次性取得 `Range` 对应范围的一段 `Array`
>
> ```cangjie
> let arr1 = [0, 1, 2, 3, 4, 5, 6]
> let arr2 = arr1[0..5]                 // arr2 包含元素 0, 1, 2, 3, 4
> ```
>
> 当 `Range` 字面量在下标语法中使用时，可以省略 `start` 或 `end`
>
> 当省略 `start` 时，`Range` 会从 `0` 开始；当省略 `end` 时，`Range` 的 `end` 会延续到最后一位
>
> ```cangjie
> let arr1 = [0, 1, 2, 3, 4, 5, 6]
> let arr2 = arr1[..3]                  // arr2 包含元素 0, 1, 2
> let arr3 = arr1[2..]                  // arr3 包含元素 2, 3, 4, 5, 6
> ```

仓颉的`Array`可以 以`[Range]`的语法, 获取原`Array`的一段元素的**引用**

具体的使用方式为: `array[start..(=)end]`

其中, `Range`字面量的`start`和`end`是可以省略的:

1. **`end`省略, 但`start`不省略: `array[start..]`**

    此时, 表示从索引`start`开始截取(包括索引`start`的元素), 到`Array`的结尾

    当`Range`字面量的`=`省略时, `end`不能省略

2. **`start`省略, 但`end`不省略: `array[..(=)end]`**

    此时, 表示从`Array`首位开始截取, 到索引`end`结束

    如果`Range`字面量的`=`不省略, 则包括索引`end`的元素

    如果`=`省略, 则不包括索引`end`的元素

    其实只要理解了`Range`语法, 就很简单

3. **`start`和`end`都省略: `array[..]`**

    此时, 表示截取整个`Array`, 其实就等价于`array`

通过`[Range]`截取的`Array`, 是原`Array`数组片段的引用, 通过截取的片段修改元素, 是 **会影响原`Array`**的

#### 修改`Array`

> `Array` 是一种长度不变的 `Collection` 类型，因此 `Array` 没有提供添加和删除元素的成员函数
>
> 但是 `Array` 允许对其中的元素进行修改，同样使用下标语法
>
> ```cangjie
> main() {
>     let arr = [0, 1, 2, 3, 4, 5]
>     arr[0] = 3
>     println("The first element is ${arr[0]}")
> }
> ```
>
> 编译并执行上面的代码，会输出：
>
> ```text
> The first element is 3
> ```
>
> **`Array` 虽然是 `struct` 类型，但其内部持有的只是元素的引用**，因此在作为表达式使用时不会拷贝副本，同一个 `Array` 实例的所有引用都会共享同样的元素数据
>
> 因此对 `Array` 元素的修改会影响到该实例的所有引用
>
> ```cangjie
> let arr1 = [0, 1, 2]
> let arr2 = arr1
> arr2[0] = 3
> // arr1 contains elements 3, 1, 2
> // arr2 contains elements 3, 1, 2
> ```

仓颉中`Array`实际是`struct`类型的, 但内部持有元素的引用, 所以`Array`是引用类型

通过下标索引访问`Array`的元素时, 可以直接修改元素值

### `VArray<T, $N>`类型

> 除了引用类型的数组 `Array`，仓颉还引入了值类型数组 `VArray<T, $N>`，其中 `T` 表示该值类型数组的元素类型，`$N` 是一个固定的语法
>
> 通过 `$` 加上一个 `Int64` 类型的数值字面量表示这个值类型数组的长度
>
> 需要注意的是，`VArray<T, $N>` 不能省略 `<T, $N>`，且使用类型别名时，不允许拆分 `VArray` 关键字与其泛型参数
>
> 与频繁使用引用类型 `Array` 相比，使用值类型 `VArray` 可以减少堆上内存分配和垃圾回收的压力
>
> 但是需要注意的是，由于值类型本身在传递和赋值时的拷贝，会产生额外的性能开销，因此建议不要在性能敏感场景使用较大长度的 `VArray`
>
> 值类型和引用类型的特点请参见[值类型和引用类型变量](https://www.humid1ch.cn/blog/cangjie-docs-reading-xxix#heading-5)
>
> ```cangjie
> type varr1 = VArray<Int64, $3>        // Ok
> type varr2 = VArray                   // Error
> ```
>
> > 注意：
> >
> > 由于运行时后端限制，当前 `VArray<T, $N>` 的元素类型 `T` 或 `T` 的成员不能包含引用类型、枚举类型、`Lambda` 表达式（`CFunc` 除外）以及未实例化的泛型类型

仓颉中除了 **引用类型的数组`Array`** 之外, 还有一个 **值类型数组`VArray<T, $N>`**

如果要声明一个`VArray`类型变量, 必须明确`VArray<T, $N>`中的`T`和`N`

`T`表示类型, `N`表示数组长度

`T`和`$N`一起影响`VArray`的实际类型, `T`相同`N`不同, 实际的`VArray`类型也不同, `T`不同就更不用说了

`VArray<T, $N>`中, `T`不能包含引用类型、枚举类型、`Lambda`表达式等为实例化泛型

`VArray`是值类型的, 在对其进行传值或复制时, 会**发生拷贝**

> `VArray` 可以由一个数组的字面量来进行初始化，左值 `a` 必须标识出 `VArray` 的实例化类型：
>
> ```cangjie
> var a: VArray<Int64, $3> = [1, 2, 3]
> ```
>
> 同时，它拥有两个构造函数：
>
> ```cangjie
> // VArray<T, $N>(initElement: (Int64) -> T)
> let b = VArray<Int64, $5>({ i => i })             // [0, 1, 2, 3, 4]
> // VArray<T, $N>(repeat!: T)
> let c = VArray<Int64, $5>(repeat: 0)              // [0, 0, 0, 0, 0]
> ```
>
> 除此之外，`VArray<T, $N>` 类型提供了两个成员方法：
>
> - 用于下标访问和修改的 `[]` 操作符方法：
>
>     ```cangjie
>     var a: VArray<Int64, $3> = [1, 2, 3]
>     let i = a[1]                                  // i is 2
>     a[2] = 4                                      // a is [1, 2, 4]
>     ```
>
>     下标访问的下标类型必须为 `Int64`
>
> - 用于获取 `VArray` 长度的 `size` 成员：
>
>     ```cangjie
>     var a: VArray<Int64, $3> = [1, 2, 3]
>     let s = a.size                                // s is 3
>     ```
>
>     `size` 属性的类型为 `Int64`
>
> 此外，`VArray` 还支持仓颉与 `C` 语言互操作场景使用，相关内容请参见[数组]()

`VArray`实例可以直接使用 `Array`数组字面量 创建, 但是**必须声明实例类型为`VArray<T, $N>`**, 否则会创建`Array`实例

并且, **声明的类型的`$N`必须与字面量中的元素个数一致, 否则类型不匹配**

除了使用`Array`数组字面量, 还可以使用`VArray<T, $N>`构造函数

1. `init(repeat!: T)`

    命名参数`repeat`, 类型为数组元素类型, 用于设置`VArray`数组每个元素的初始值

2. `init(initElement: (Int64) -> T)`

    参数`initElement`, 各索引元素的初始化函数, 即 各位置会将索引作为参数, 传入此函数 返回值作为元素值

`VArray`未实现`ToString`接口, 所以不能直接使用`print`或`println`

但可以使用`[]`通过下标索引访问数组元素
