---
title: "仓颉文档阅读-语言规约II: 类型(II)"
published: 2025-09-22
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

## 类型

> 仓颉编程语言是一种 **静态类型(statically typed)** 语言: 大部分保证程序安全的类型检查发生在编译期
>
> 同时, 仓颉编程语言是一种 **强类型(strongly typed)** 语言: 每个表达式都有类型, 并且表达式的类型决定了它的取值空间和它支持的操作
>
> 静态类型和强类型机制可以帮助程序员在编译阶段发现大量由类型引发的程序错误

### 可变类型 和 不可变类型

> 仓颉中的类型可分为两类: 不可变类型(immutable type)和可变类型(mutable type)
>
> 其中, 不可变类型包括数值类型(分为整数类型和浮点数类型)、`Rune`类型、`Bool`类型、`Unit`类型、`Nothing`类型、`String`类型、元组(`Tuple`)类型、`Range`类型、函数(`Function`)类型、`enum`类型
>
> 可变类型包括`Array`类型、`VArray`类型、`struct`类型、`class`类型和`interface`类型
>
> 不可变类型和可变类型的区别在于: 不可变类型的值, 其数据值一经初始化后就不会发生变化; 可变类型的值, 其数据值初始化后仍然有可以修改的方法


### 可变类型

#### `Array`类型

> 仓颉编程语言使用泛型类型`Array<T>`表示`Array`类型, `Array`类型用于存储一系列相同类型(或者拥有公共父类)的元素
>
> 关于`Array<T>`, 说明如下:
>
> 1. `Array<T>`中的元素是有序的, 并且**支持通过索引(从`0`开始)访问其中的元素**
>
> 2. **`Array`类型的长度是固定的, 即一旦建立一个`Array`实例, 其长度是不允许改变的**
>
> 3. **`Array`是引用类型, 定义为引用类型的变量, 变量名中存储的是指向数据值的引用, 因此在进行赋值或函数传参等操作时, `Array`拷贝的是引用**
>
> 4. 类型变元`T`被实例化成不同的类型, 会得到不同的`Array`类型, 例如, `Array<Int32>`和`Array<Float64>`分别表示`Int32`类型的`Array`和`Float64`类型的`Array`
>
> 5. 当`Array<T>`中的`Type`类型支持使用`==`进行值判等(使用`!=`进行值判不等)时, `Array<T>`类型支持使用`==`进行值判等(使用`!=`进行值判不等);
>
>     否则, `Array<T>`类型不支持`==`和`!=`(如果使用`==`和`!=`, 编译报错)
>
>     两个同类型的`Array<T>`实例值相等, 当且仅当相同位置(`index`)的元素全部相等(意味着它们的长度相等)
>
> 6. 多维`Array`定义为`Array`的`Array`, 使用`Array<Array<...>>`表示
>
>     例如, `Array<Array<Int32>>`表示`Int32`类型的二维`Array`
>
> 7. 当`ElementType`拥有子类型时, `Array<ElementType>`中可以存放任何`ElementType`子类型的实例, 例如`Array<Object>`中可以存放任何`Object子类`的实例

**仓颉中的`Array`, 有序(不是升序降序, 而是顺序), 可以通过索引随机访问, 创建之后长度固定且无法改变, 可创建多维**

`Array`即为仓颉中的数组类型, **且`Array`类型是引用类型**, 支持整个数组的判等和判不等操作, 前提是数组内元素类型要支持

##### 创建`Array`实例

> 存在 2 种创建`Array`实例的方式:
>
> 构造函数
>
> ```cangjie
> // Array<T>()
> let emptyArr1 = Array<Int64>()  // create an empty array whose type is Array<Int64>
> let emptyArr2 = Array<String>() // create an empty array whose type is Array<String>
>
> // Array<T>(size: Int64, initElement: (Int64)->T)
> let array3 = Array<Int64>(3) { i => i * 2 } // 'array3' has 3 elememts: 0, 2, 4
> let array4 = Array<String>(2) { i => "$i" } // 'array4' has 2 elememts: "0", "1"
> ```
>
> Array 字面量
>
> `Array`字面量使用格式`[element1, element2, ... , elementN]`表示, 其中多个`element`之间使用逗号分隔, 每个`element`可以是一个`expressionElement`(普通表达式)
>
> `Array`字面量每个元素都是一个表达式:
>
> ```cangjie
> let emptyArray: Array<Int64> = []    // empty Array<Int64>
> let array0 = [1, 2, 3, 3, 2, 1]      // array0 = [1, 2, 3, 3, 2, 1]
> let array1 = [1 + 3, 2 + 3, 3 + 3]   // array1 = [4, 5, 6]
> ```
>
> - 在上下文没有明确的类型要求时, 若所有`element`的最小公共父类型是`T`, `Array`字面量的类型是`Array<T>`
> - 在上下文有明确的类型要求时, 此时要求所有`element`的类型都是上下文所要求的`element`类型的子类型

仓颉`Array`类型的构造函数, 除了创建空数组之外, 比较重要的是**指定`size`以及初始化元素的函数**

仓颉的构造函数`Array<T>(size: Int64, initElement: (Int64)->T)`

第二个参数, 是用来初始化元素的, 参数是`Int64`类型, 返回值是`T`类型

实际的初始化操作, 就是调用函数传入索引, 返回目标索引获取的`T`类型数据作为`Array`中目标索引的初始值

仓颉中的数组字面量格式为:`[elem1, elem2, ...]`, 也可以用来创建`Array`实例

##### 访问`Array`中的元素

> 对于`Array`类型的实例`arr`, 支持以下方式访问`arr`中的元素:
>
> **访问某个位置处的元素**: 通过`arr[index1]...[indexN]`的方式访问具体位置处的元素(其中`index1,...,indexN`是索引值的表达式, 它们的类型均为`Int64`), 例如:
>
> ```cangjie
> let array5 = [0, 1]
> let element0 = array5[0]      // element0 = 0
> array5[1] = 10                // change the value of the second element of 'array5' through index
>
> let array6 = [[0.1, 0.2], [0.3, 0.4]]
> let element01 = array6[0][1]  // element01 = 0.2
> array6[1][1] = 4.0            // change the value of the last element of 'array6' through index
> ```
>
> **迭代访问**: 通过`for-in`表达式迭代访问`arr`中的元素, 例如:
>
> ```cangjie
> func access() {
>     let array8 = [1, 8, 0, 1, 0]
>     for (num in array8) {
>         print("${num}")   // output: 18010
>     }
> }
> ```

仓颉中`Array`元素的访问也比较简单:`[]`下标索引 和`for-in`迭代

##### 访问`Array`的大小

> 支持通过`arr.size`的方式返回`arr`中元素的个数

```cangjie
let array9 = [0, 1, 2, 3, 4, 5]
let array10 = [[0, 1, 2], [3, 4, 5]]
let size1 = array9.size  // size1 = 6
let size2 = array10.size // size2 = 2
```

##### `Array`的切片

数组的切片, 就是截取数组中的某一段连续的序列

> **当`Range<Int64>`类型的值用作`Array`下标时, 用于截取`Array`的一个片段**(称之为`slicing`)
>
> 需要注意的是:
>
> - **`step`必须是 1, 否则运行时会抛出异常**
>
> - `slicing`返回的类型仍然是相同的`Array`类型, 并且**是原`Array`的引用**
>
>     **对切片中元素的修改会影响到原数组**
>
> - 当使用`start..end`作为`Array`下标时, 如果省略`start`, 则将`start`的值设置为`0`, 如果省略`end`, 则将`end`的值设置为`Array`的长度值
>
> - 当使用`start..=end`作为`Array`下标时, 如果`start`被省略, 则将`start`的值设置为`0`
>
>     **`start..=end`形式的`end`不能省略**
>
> - 如果下标是一个空`range`值, 那么返回的是一个空的`Array`

```cangjie
let array7 = [0, 1, 2, 3, 4, 5]

func slicingTest() {
    array7[0..5]      // [0, 1, 2, 3, 4]
    array7[0..5:1]    // [0, 1, 2, 3, 4]
    array7[0..5:2]    // step 不为 1, 运行时异常
    array7[5..0:-1]   // step 不为 1, 运行时异常
    array7[5..0:-2]   // step 不为 1, 运行时异常
    array7[0..=5]     // [0, 1, 2, 3, 4, 5]
    array7[0..=5:1]   // [0, 1, 2, 3, 4, 5]
    array7[0..=5:2]   // step 不为 1, 运行时异常
    array7[5..=0:-1]  // step 不为 1, 运行时异常
    array7[5..=0:-2]  // step 不为 1, 运行时异常
    array7[..4]       // [0, 1, 2, 3]
    array7[2..]       // [2, 3, 4, 5]
    array7[..]        // [0, 1, 2, 3, 4, 5]
    array7[..4:2]     // 语法错误: start 或 end 不存在, 禁止使用:step
    array7[2..:-2]    // 语法错误: start 或 end 不存在, 禁止使用:step
    array7[..:-1]     // 语法错误: start 或 end 不存在, 禁止使用:step

    array7[..=4]      // [0, 1, 2, 3, 4]
    array7[..=4:2]    // 语法错误: start 或 end 不存在, 禁止使用:step

    array7[0..5:-1]   // step 不为 1, 运行时异常
    array7[5..=0]     // []
    array7[..0]       // []
    array7[..=-1]     // []
    let temp: Array<Int64> = array7[..]
    temp[0] = 6 // temp == array7 == [6, 1, 2, 3, 4, 5]
}
```

> 当使用`slicing`进行赋值时, 支持两种不同的用法:
>
> - 如果`=`右侧表达式的类型是数组的元素类型, 会将这个表达式的值作为元素覆盖切片范围的元素
> - 如果`=`右侧表达式的类型与数组的类型相同, 会将这个数组拷贝覆盖当前切片范围, 这时要求右侧表达式的`size`必须与切片范围相同, 否则运行时会抛出异常

```cangjie
let arr = [1, 2, 3, 4, 5]
arr[..] = 0
// arr = [0, 0, 0, 0, 0]

arr[0..2] = 1
// arr = [1, 1, 0, 0, 0]

arr[0..2] = [2, 2]
// arr = [2, 2, 0, 0, 0]

arr[0..2] = [3, 3, 3]     // size不匹配, 运行时抛异常
arr[0..2] = [4]         // size不匹配, 运行时抛异常

let arr2 = [1, 2, 3, 4, 5]
arr[0..2] = arr2[0..2] // ok
// arr = [1, 2, 0, 0, 0]

arr[0..2] = arr2 // runtime exception
arr[..] = arr2
```

如果使用`slicing`对原数组进行赋值, 总结就两条:

1. 如果`=`右边是 数组内元素类型的值, 就将`slicing`切片中 所有元素赋值为目标值
2. 如果`=`右边是 同数组类型的数组字面量, 那么**这个数组字面量必须要与`slicing`的`size`匹配才能拷贝覆盖式赋值**, 否则会抛异常

#### `VArray`类型

仓颉中已经有了`Array`数组类型, 但`Array`是引用类型的数组

> **仓颉编程语言使用泛型类型`VArray<T, $N>`表示`VArray`类型**, `VArray`类型用于存储一系列相同类型(或者拥有公共父类型)的元素
>
> 关于`VArray<T, $N>`, 说明如下:
>
> 1. `VArray<T, $N>`中的元素是有序的, 并且支持通过索引(从`0`开始)访问其中的元素
>
>     如果提供的索引大于或等于其长度, 在编译期间能够推导出下标的则编译报错, 否则在运行时抛出异常
>
> 2. `VArray<T, $N>`类型的长度是类型的一部分
>
>     其中`N`表示`VArray`的长度, 它必须是一个整数字面量, 通过固定语法`$`加数字进行标注
>
>     当提供的整数字面量为负数时, 编译期间进行报错
>
> 3. **`VArray`是值类型, 定义为值类型的变量, 变量名中存储的是数据值本身, 因此在进行赋值或函数传参等操作时, `VArray`类型拷贝的是值**
>
> 4. **当类型变元`T`被实例化成不同的类型, 或`$N`标注长度不相等时, 会得到不同的`VArray`类型**
>
>     例`VArray<Int32, $5>`和`VArray<Float64, $5>`是不同类型的`VArray`
>
>     `VArray<Int32, $2>`和`VArray<Int32, $3>`也是不同类型的`VArray`
>
> 5. `VArray`的泛型变元`T`是`VArray`类型时, 表示一个多维的`VArray`类型

仓颉中存在两种数组类型:

1. `Array`引用类型数组, 具体类型为`Array<Type>`
2. `VArray`值类型数组, 具体类型为`VArray<Type, $N>`

`VArray<Type, $N>`不同的`Type`和不同的`N`都是不同的类型

##### 创建`VArray`实例

> `VArray`同样可以使用`Array`字面量来创建新的实例
>
> 这种方式只能在上下文中能明确该字面量是`VArray`类型时才可以使用, 否则仍然会优先推断为`Array`类型
> `VArray`的长度必须为`Int64`的字面量类型, 且必须与提供的字面量`Array`长度一致, 否则会编译报错
>
> ```cangjie
> let arr1: VArray<Int64, $5> = [1,2,3,4,5]
> let arr2: VArray<Int16, $0> = []
> let arr3: VArray<Int16, $0> = [1] // error
> let arr4: VArray<Int16, $-1> = [] // error
> ```
>
> 除此以外`VArray`也可以使用构造函数的形式创建实例
>
> ```cangjie
> // VArray<T, $N>(initElement: (Int64)->T)
> let arr5: VArray<Int64, $5> = VArray<Int64, $5> { i => i } // [0, 1, 2, 3, 4]
> // VArray<T, $N>(item!: T)
> let arr6: VArray<Int64, $5> = VArray<Int64, $5>(item: 0) // [0, 0, 0, 0, 0]
> ```
>
> `VArray`总是不允许缺省类型参数`<T, $N>`

##### 访问`VArray`中的元素

> 对于`VArray`类型的实例`arr`, 支持以下方式访问`arr`中的元素:
>
> 通过`arr[index1]...[indexN]`的方式访问具体位置处的元素(其中`index1,...,indexN`是索引值的表达式, 它们的类型均为`Int64`)
>
> 需要注意的是, `VArray`的下标取值操作会返回指定元素的拷贝
>
> 这意味着如果元素是值类型, 那么下标取值会得到一个不可修改的新实例
>
> 对于多维`VArray`, 我们也不能通过`arr[n][m] = e`的方式来修改内层的`VArray`, **因为通过`arr[n]`所获取的内层`VArray`是一个经过拷贝的新的`VArray`实例**
>
> ```cangjie
> var arr7: VArray<Int64, $2> = [0, 1]
> let element0 = arr7[0]                     // element0 = 0
> arr7[1] = 10                             // change the value of the second element of 'arr7' through index
>
> // Get and Set of multi-dimensional VArrays.
> var arr8: VArray<VArray<Int64, $2>, $2> = [[1, 2], [3, 4]]
> let arr9: VArray<Int64, $2> = [0, 1]
> let element1 = arr8[1][0]                 // element1 = 3
> arr8[1][1] = 5                             // error: function call returns immutable value
> arr8[1] = arr9                             // arr8 = [[1, 2], [0, 1]]
> ```

因为`VArray`是值类型的数组

所以, **通过`[index]`操作符对`VArray`进行取值操作, 获取到的只会是数组中`index`位置同值的值类型拷贝新实例, 即一个独立的临时副本**

如果通过`[index]`操作符直接对`VArray`数组的进行赋值, 则可以改变`VArray[index]`内的元素值

但对于**多维的`VArray`, 尝试通过`[][]`对多维的索引位置进行赋值, 是不可行的**

因为, **`[][]`的第一个`[]`操作被看作取值操作**, 是对外层维度的取值, 获取到的是外层这一维度的值类型的临时副本

不能直接拿着临时副本, 尝试修改临时副本的值

如果要直接通过`[]`修改`VArray`最外层维度的值, 可以考虑直接覆盖整个维度:

```cangjie
main() {
    var arr8: VArray<VArray<Int64, $2>, $2> = [[1, 2], [3, 4]]

    arr8[0] = [5, 6]    // 直接覆盖整个维度

    println("${arr8[0][0]} ${arr8[0][1]}")

    return 0
}
```

##### 获取`VArray`的长度

> 支持通过`arr.size`的方式返回`arr`中元素的个数
>
> ```cangjie
> let arr9: VArray<Int64, $6> = [0, 1, 2, 3, 4, 5]
> let size = arr9.size  // size = 6
> ```

这一部分与`Array`是一致的

##### `VArray`在函数签名中时

> 当`VArray`作为函数的参数或返回值时, 需要标注`VArray`的长度:
>
> ```
> func mergeArray<T>(a: VArray<T,$2>, b: VArray<T, $3>): VArray<T, $5>
> ```

因为`VArray<Type, $N>`中, `Type`和`N`都会影响实际类型

所以, 大概在实际需要声明`VArray`类型的所有地方都需要声明完整的`Type`和`N`

#### `struct`类型

> `struct`类型是一种`mutable`类型, 在其内部可定义一系列的成员变量和成员函数
>
> 定义`struct`类型的语法为:
>
> ```
> structDefinition
>     : structModifier? 'struct' identifier typeParameters? ('<:' superInterfaces)? genericConstraints? structBody
>     ;
>
> structBody
>     : '{'
>         structMemberDeclaration*
>         structPrimaryInit?
>         structMemberDeclaration*
>       '}'
>     ;
>
> structMemberDeclaration
>     : structInit
>     | staticInit
>     | variableDeclaration
>     | functionDefinition
>     | operatorFunctionDefinition
>     | macroExpression
>     | propertyDefinition
>     ;
> ```
>
> 其中`structModifier`是`struct`的修饰符, `struct`是关键字, `identifier`是`struct`类型的名字, `typeParameters`和`genericConstraints`分别是类型变元列表和类型变元的约束
>
> `structBody`中可以定义成员变量(`variableDeclaration`), 成员属性(`propertyDefinition`), 主构造函数(`structPrimaryInit`), 构造函数(`structInit`)和成员函数(包括普通成员函数和操作符函数)

从定义语法来看, 一个`struct`可以长这样:

```cangjie
(修饰符) struct 名字(<T>约束) {
    // 主构造函数(可选)
    // 成员变量
    // 成员属性
    // 构造函数
    // 成员函数
}
```

> 关于`struct`类型, 需要注意的是:
>
> 1. **`struct`是值类型, 定义为值类型的变量**, 变量名中存储的是数据值本身, 因此在进行赋值或函数传参等操作时, `struct`类型拷贝的是值
>
> 2. **`struct`类型只能定义在`top-level`**
>
> 3. 作为一种自定义类型, `struct`类型默认不支持使用`==`(`!=`)进行判等(判不等)
>
>     当然, 可以通过重载`==`(`!=`)操作符使得自定义的`struct`类型支持`==`(`!=`)
>
> 4. **`struct`不可以被继承**
>
> 5. `struct`可以实现接口
>
> 6. 如果一个`struct`类型中的某个(或多个)非静态成员变量的类型中引用了`struct`自身, 则称此`struct`为递归 `struct`类型
>
>     对于多个`struct`类型, 如果它们的非静态成员变量的类型之间构成了循环引用, 则称这些`struct` 类型间构成了互递归
>
>     递归(或互递归)定义的`struct`类型是非法的, 除非每条递归链`T_1, T_2, ..., T_N`上都存在至少一个`T_i`被封装在`Class`、`Interface`、`Enum`或函数类型中, 也就是说, 可以使用`Class`、`Interface`、`Enum`或函数类型使递归(或互递归)的`struct`定义合法化
>
> `struct`类型定义举例:
>
> ```cangjie
> struct Rectangle1 {
>     let width1: Int32
>     let length1: Int32
>     let perimeter1: () -> Int32
>
>     init (width1: Int32, length1: Int32) {
>         this.width1 = width1
>         this.length1 = length1
>         this.perimeter1 = { => 2 * (width1 + length1) }
>     }
>
>     init (side: Int32) {
>         this(side, side)
>     }
>
>     func area1(): Int32 {
>         width1 * length1
>     }
> }
>
> // Define a generic struct type.
> struct Rectangle2<T> {
>     let width2: T
>     let length2: T
>
>     init (side: T) {
>       this.width2 = side
>       this.length2 = side
>     }
>
>     init (width2!: T, length2!: T) {
>         this.width2 = width2
>         this.length2 = length2
>     }
> }
> ```

从上面文档中给的定义示例, 好像与C++中的`struct`定义没有什么明显区别

不过仓颉中可以使用`init()`定义构造函数

但从描述来看, 还有很多区别: 值类型、递归和互递归等

> 递归和互递归`struct`类型定义举例:
>
> ```cangjie
> struct R1 { // 错误: 'R1'不能有递归包含它的成员
>     let other: R1
> }
> struct R2 { // ok
>     let other: Array<R2>
> }
>
> struct R3 { // 错误: 'R3'不能有递归包含它的成员
>     let other: R4
> }
> struct R4 { // 错误: 'R4'不能有递归包含它的成员
>     let other: R3
> }
>
> struct R5 { // ok
>     let other: E1
> }
> enum E1 { // ok
>     A(R5)
> }
> ```

<Info>

示例中都忽略了`init()`构造函数

</Info>

从示例观察, 可以看到:

1. 如果直接自递归, 是禁止的

    ```cangjie
    struct R1 { // 错误: 'R1'不能有递归包含它的成员
        let other: R1
    }
    ```

    而`Array<R1>`作为成员变量类型, 是允许的

    ```cangjie
    struct R1 { // ok
        let other: Array<R1>
    }
    ```

    且, 更无法使用`VArray<R1, $N>`作为成员变量

2. 互递归, 直接互递归, 也是被禁止的

    ```cangjie
    struct R3 { // 错误: 'R3'不能有递归包含它的成员
        let other: R4
    }
    struct R4 { // 错误: 'R4'不能有递归包含它的成员
        let other: R3
    }
    ```

    但是如果另一个类型不是`struct`好像就可以:

    ```cangjie
    struct R5 { // ok
        let other: E1
    }
    enum E1 { // ok
        A(R5)
    }
    ```

    且, 经测试 这样是可以的:

    ```cangjie
    struct R5 { // ok
        let other: R6
    }
    struct R6 { // ok
        let other: Array<R5>
    }
    ```

从示例给出的错误和正确案例, 猜测 与值类型和引用类型有关

如果将引用类型变量, 类比成C/C++中的指针, 就能很简单的解释了

如果`struct`内部 值类型自递归, 是错误的, 因为`struct`是值类型, 如果成员还是值类型自递归, 那么`struct`是无法确定大小的

互递归也是同样的原因, 如果直接值类型自递归或互递归, 无法确定`struct`的大小

而引用类型不一样, 如果将引用类型看作C/C++中的指针, 那么引用类型变量的大小就与具体类型无关了, 而与平台有关, 是固定的



当然这只是猜测, 毕竟才刚开始阅读语言规约, 暂时就这样理解吧

##### `struct`成员变量

> 定义成员变量的语法为:
>
> ```
> variableDeclaration
>     : variableModifier* NL* (LET | VAR) NL* patternsMaybeIrrefutable ((NL* COLON NL* type)? (NL* ASSIGN NL* expression) | (NL* COLON NL* type))
>     ;
> ```
>
> 在定义成员变量的过程中, 需要注意的是:
>
> 1. 在主构造函数之外定义的成员变量可以有初始值, 也可以没有初始值
>
>     如果有初始值, 初始值表达式中仅可以引用定义在它之前的已经初始化的成员变量, 以及`struct`中的静态成员函数

从定义语法来看:`variableModifier`变量修饰词, `NL`换行, `(LET | VAR)`修饰词, `patternsMaybeIrrefutable`可选模式

应该这样定义:

```cangjie
(public) let(var) 变量名: Type = Type字面量
(public) let(var) 变量名: Type
```

##### 构造函数

> 在仓颉编程语言中, 有两种构造函数:主构造函数和`init`构造函数 (简称构造函数)

构造函数看起来和C++有一定的区别

###### 主构造函数

> 主构造函数的语法定义如下:
>
> ```
>
> structPrimaryInit
>     : (structNonStaticMemberModifier)? structName '('
>       structPrimaryInitParamLists? ')'
>        '{'
>             expressionOrDeclarations?
>        '}'
>     ;
>
> structName
>     : identifier
>     ;
>
> structPrimaryInitParamLists
>     : unnamedParameterList  (','  namedParameterList)?
>        (',' structNamedInitParamList)?
>     | unnamedParameterList (',' structUnnamedInitParamList)?
>        (',' structNamedInitParamList)?
>     | structUnnamedInitParamList (',' structNamedInitParamList)?
>     | namedParameterList (',' structNamedInitParamList)?
>     | structNamedInitParamList
>     ;
>
> structUnnamedInitParamList
>     : structUnnamedInitParam (',' structUnnamedInitParam)*
>     ;
>
> structNamedInitParamList
>     : structNamedInitParam (',' structNamedInitParam)*
>     ;
>
> structUnnamedInitParam
>     : structNonStaticMemberModifier?  ('let' | 'var')  identifier ':' type
>     ;
>
> structNamedInitParam
>     : structNonStaticMemberModifier?   ('let' | 'var')  identifier '!' ':' type
>       ('=' expression)?
>     ;
> ```
>
> 主构造函数的定义包括以下几个部分:
>
> 1. 修饰符:可选
>
>     可以被所有访问修饰符修饰, 默认的可访问性为`internal`
>
> 2. **主构造函数名:与类型名一致**
>
>     **主构造函数名前不允许使用`func`关键字**
>
> 3. 形参列表:主构造函数与`init`构造函数不同的是, 前者有两种形参:普通形参和成员变量形参
>
>     普通形参的语法和语义与函数定义中的形参一致
>
>     引入成员变量形参是为了减少代码冗余
>
>     **成员变量形参的定义, 同时包含形参和成员变量的定义**, 除此之外还表示了通过形参给成员变量赋值的语义
>
>     省略的定义和表达式会由编译器自动生成
>
>     - 成员变量形参的 **语法和成员变量定义语法一致**, 此外, 和普通形参一样支持使用`!`来标注是否为命名形参
>
>     - 成员变量形参的 修饰符有:`public`, `private`, `protected`, `internal`
>
>     - 成员变量形参 **只允许非静态成员变量**, 即不允许使用`static`修饰
>
>     - 成员变量形参 **不能与主构造函数外的成员变量同名**
>
>     - 成员变量形参 可以没有初始值
>
>         这是因为主构造函数会由编译器生成一个对应的构造函数, 将在构造函数体内完成将形参给成员变量的赋值;
>
>     - 成员变量形参 也可以有初始值, 初值表达式中可以引用该成员变量定义之前已经定义的其他形参或成员变量**(不包括定义在主构造函数外的实例成员变量)**, 但不能修改这些形参和成员变量的值
>
>         需要注意的是, **成员变量形参的初始值只在主构造函数中有效, 不会在成员变量定义中包含初始值**
>
>     - **成员变量形参后不允许出现普通形参**, 并且要遵循函数定义时的参数顺序, **命名形参后不允许出现非命名形参**
>
> 4. 主构造函数体:主构造函数不允许调用本`struct`中其它构造函数
>
>     主构造函数体内允许写声明和表达式, 其中声明和表达式需要满足`init`构造函数的要求
>
> 主构造函数定义的例子如下:
>
> ```cangjie
> struct Test {
>     static let counter: Int64 = 3
>     let name: String = "afdoaidfad"
>
>     private Test(
>         name: String,                     // regular parameter
>         annotation!: String = "nnn",       // regular parameter
>         var width!: Int64 = 1,             // member variable parameter with initial value
>         private var length!: Int64,        // member variable parameter
>         private var height!: Int64 = 3    // member variable parameter
>     ) {}
> }
> ```

从文档描述和示例来看:

1. **主构造函数与`struct`同名**

2. 主构造函数除了普通的形参之外, 还可以像定义变量一样 **声明(定义)成员变量形参**

    主构造函数中的成员变量形参, **会自动被定义为此`struct`的成员变量**

    与普通形参的区别是, **成员变量形参使用`let | var`被修饰**

3. 主构造函数中的成员变量形参, **直接被看作`struct`的成员变量, 且不被当作存在初始值**

    这也就意味着, **主构造函数中的成员变量形参, 也必须要显式在其他构造函数中赋初始值**

    > **成员变量形参的初始值只在主构造函数中有效, 不会在成员变量定义中包含初始值**

> 主构造函数是`init`构造函数的语法糖, 编译器会自动生成与主构造函数对应的构造函数和成员变量的定义
>
> 自动生成的构造函数形式如下:
>
> - 其修饰符与主构造函数修饰符一致
> - 其形参从左到右的顺序与主构造函数形参列表中声明的形参一致
> - 构造函数体内形式如下:
>     - 首先是对成员变量的赋值, 语法形式为`this.x = x`, 其中`x`为成员变量名
>     - 然后是主构造函数体的其它代码
>
> ```cangjie
> struct B<X,Y> {
>     B(    // 主构造函数, 与sturct同名
>         x: Int64,
>         y: X,
>         v!: Int64 = 1,     // 普通参数
>         private var z!: Y  // 成员变量形参
>     ) {}
>
>     /* 编译器自动生成与主构造函数对应的初始化构造函数
>
>     private var z: Y    // 自动生成 成员变量定义
>     init( x: Int64, y: X, v!: Int64 = 1, z!: Y) { // 自动生成的命名参数定义
>         this.z = z // 自动生成的成员变量赋值表达式
>     }
>     */
> }
> ```
>
> **一个`struct`最多可以定义一个主构造函数**, 除了主构造函数之外, 可以照常定义其他构造函数, 但要求**其他构造函数必须和主构造函数所对应的构造函数构成重载**

这段的意思是说, 主构造函数其实是仓颉提供的语法糖

实际并不存在额外的构造函数, **主构造函数编译时还是会被转换为对应的`init()`构造函数**

而且, **在显式定义其他`init()`构造函数时, 要与编译器根据主构造函数生成的`init()`构造函数 构成重载, 不能一致**

即, 下面这样是不行的:

```cangjie
struct B<X,Y> {
    B(    // 主构造函数, 与sturct同名
        x: Int64,
        y: X,
        v!: Int64 = 1,     // 普通参数
        private var z!: Y  // 成员变量形参
    ) {}

    // 显式定义init()构造函数
    init(x: Int64, y: X, v!: Int64 = 1, z!: Y) {
        this.z = z
    }
}
```

因为显式定义的`init()`会与主构造函数生成的`init()`重名重参数列表, 构不成重载, 只会重定义

###### `init`构造函数

> 除了主构造函数, 还可以自定义构造函数, 构造函数使用关键字`init`加上参数列表和函数体的方式定义
>
> 一个`struct`中可以定义多个构造函数, 但要求**它们和主构造函数所对应的构造函数必须构成重载**
>
> 另外:
>
> 1. 构造函数的参数可以有默认值
>
>     **禁止使用实例成员变量`this.variableName`及其语法糖`variableName`作为构造函数参数的默认值**
>
> 2. 构造函数在所有实例成员变量完成初始化之前, 禁止使用 隐式传参或捕获了`this`的函数或`lambda`, 但允许使用`this.variableName`或其语法糖`variableName`来访问已经完成初始化的成员变量`variableName`
>
> 3. **构造函数中的`lamdba`和嵌套函数不能捕获`this`, `this`也不能作为表达式单独使用**
>
> 4. **构造函数中允许通过`this`调用其他构造函数**
>
>     如果调用, **必须在构造函数体内的第一个表达式处**, 在此之前不能有任何表达式或声明
>
>     **在构造函数体外, 不允许通过`this`调用表达式来调用构造函数**
>
> 5. 若构造函数没有显式调用其他构造函数, 则**需要确保`return`之前本`struct`声明的所有实例成员变量均完成初始化, 否则编译报错**
>
> 6. 编译器会对所有构造函数之间的调用关系进行依赖分析, 循环的调用将编译报错
>
> 7. **构造函数的返回类型为`Unit`**
>
> 如果一个`struct`既没有定义主构造函数, 也没有定义`init`构造函数, 则会尝试生成一个(`public`修饰的)无参构造函数
>
> 如果存在本`struct`的实例成员变量没有初始值, 则编译报错

总的来看, 仓颉`struct`的构造函数:

1. 不能将成员变量, 作为参数的默认值
2. 所有成员变量完成初始化之前, 不能以传参、捕获等方式将`this`跳出构造函数使用
3. 构造函数内, 可以调用其他构造函数, 但只能是构造函数内的第一个表达式
4. 构造函数必须完成所有成员变量的初始化
5. 构造函数返回类型为`Unit`

##### `struct`的其他成员

> struct 中也可以定义成员函数、操作符函数、成员属性和静态初始化器
>
> - 定义普通成员函数参见[函数定义]
> - 定义操作符函数的语法参见[操作符重载]
> - 定义成员属性的语法参见[属性的定义]
> - 定义静态初始化器的语法参见[静态初始化器]

没招了, 具体阅读到再说

##### `struct`中的修饰符

> `struct`及其成员可以使用访问修饰符进行修饰, 详细内容请参考包和模块管理章节[访问修饰符]
>
> 成员函数和变量可以使用`static`修饰, 这些成员是静态成员, 静态成员属于`struct`类型的成员, 而不是`struct`实例的成员
>
> 在`struct`定义外部, 实例成员变量和实例成员函数只能通过`struct`实例访问, **静态成员变量和静态成员函数只能通过`struct`类型的名字访问**
>
> 另外函数还可以被`mut`修饰, `mut`函数是一种特殊的实例成员函数, `mut`函数详细介绍参考函数章节[`mut`函数]
>
> **`struct`构造函数以及主构造函数内部定义的成员变量只能使用访问修饰符修饰, 不能使用`static`修饰**

本片文档只主要说明了`static`修饰符, 其他的在其他文档中

而`static`与C++中的差不太多, 区别在于:

**C++的静态成员可以通过对象的实例访问, 但仓颉中的静态成员只能通过类型访问**

##### `struct`的实例化

> 定义完`struct`类型之后, 就可以创建对应的`struct`实例
>
> `struct`实例的定义方式按照是否包含类型变元可分为两种:
>
> 1. 定义非泛型`struct`的实例:`StructName(arguments)`
>
>     其中`StructName`为`struct`类型的名字, `arguments`为实参列表
>
>     `StructName(arguments)`会根据重载函数的调用规则调用对应的构造函数, 然后生成`StructName`的一个实例
>
>     举例如下:
>
>     ```cangjie
>     let newRectangle1_1 = Rectangle1(100, 200)    // Invoke the first custom constructor
>     let newRectangle1_2 = Rectangle1(300)         // Invoke the second custom constructor
>     ```
>
> 2. 定义泛型`struct`的实例:`StructName<Type1, Type2, ... , TypeK>(labelValue1, labelValue2, ... , labelValueN)`
>
>     与定义非泛型`struct`的实例的差别仅在于需要对泛型参数进行实例化
>
>     举例如下:
>
>     ```cangjie
>     let newRectangle2_1 = Rectangle2<Int32>(100)                         // Invoke the custom constructor
>     let newRectangle2_1 = Rectangle2<Int32>(width2: 10, length2: 20)    // Invoke another custom constructor
>     ```
>
> 最后, 需要注意的是:
>
> 对于`struct`类型的变量`structInstance`, 如果它使用`let`定义, 不支持通过`structInstance.varName = expr`的方式修改成员变量`varName`的值(即使`varName`使用`var`定义)
>
> **如果`structInstance`使用`var`定义, 并且`varName`同样使用`var`定义**, **支持通过`structInstance.varName = expr`的方式修改成员变量`varName`的值**

实例化`struct`, 就是通过类型名调用构造函数, 然后创建`struct`实例

只有非泛型和泛型的区别, 泛型需要指明类型

有一点需要主要:

**只有`struct`变量和其成员变量, 均用`var`修饰是, 才能通过`structInstance.varName = expr`的方式修改成员变量的值**

<Question>

`struct`是值类型的

那么, 给`struct`成员赋值, 是与值类型变量本身赋值类似, 还是与引用类型变量赋值类似呢?

这里猜测, 与引用类型变量赋值类似, 是直接替换了结构体成员内存中的数据, 不会重新创建实例, 更不会创建一个新的结构体

当然只是猜测

> 仓颉好像提供有`CPointer`类型, 到时可以验证一下

</Question>

#### `class`类型和`interface`类型

> `class`和`interface`是引用类型, 定义为引用类型的变量, 变量名中存储的是指向数据值的引用, 因此在进行赋值或函数传参等操作时, `class`和`interface`拷贝的是引用
>
> 请参见[类和接口]

`class`和`interface`应该是比较占篇幅的, 需要单独领出来的类型

具体到时再看
