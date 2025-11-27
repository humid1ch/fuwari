---
title: "仓颉文档阅读-开发指南X: Collection 类型(II) - HashSet 与 HashMap"
published: 2025-11-27 16:17:44
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

仓颉的`Collection`类型, 新增元素和删除元素函数的用法基本相似

### `HashSet`

> 使用`HashSet`类型需要导入`collection`包: 
> 
> ```cangjie
> import std.collection.*
> ```
> 
> 可以使用`HashSet`类型来构造**只拥有不重复元素**的`Collection`
> 
> 仓颉使用`HashSet<T>`表示`HashSet`类型, `T`表示`HashSet`的元素类型, `T`必须是实现了`Hashable`和`Equatable<T>`接口的类型, 例如数值或`String`
> 
> ```cangjie
> var a: HashSet<Int64> = ...   // 一个元素类型为 Int64 的 HashSet
> var b: HashSet<String> = ...  // 一个元素类型为 String 的 HashSet
> ```
> 
> 元素类型不相同的`HashSet`是不相同的类型, 所以它们之间不可以互相赋值
> 
> 因此以下例子是不合法的
> 
> ```cangjie
> b = a         // 类型不匹配
> ```
> 
> 仓颉中可以使用构造函数的方式构造一个指定的`HashSet`
> 
> ```cangjie
> let a = HashSet<String>()                             // 创建一个 空 HashSet, 其元素类型为 String
> let b = HashSet<String>(100)                          // 创建一个容量为 100 的 HashSet
> let c = HashSet<Int64>([0, 1, 2])                     // 创建一个元素类型为 Int64 的 HashSet, 包含元素 0、1、2
> let d = HashSet<Int64>(c)                             // 使用另一个`Collection`来初始化一个 HashSet
> let e = HashSet<Int64>(10, {x: Int64 => (x * x)})     // 创建一个元素类型为 Int64、大小为 10 的 HashSet, 所有元素均通过指定的规则函数初始化
> ```

`HashSet`是在`std.collection`包中定义的, 使用时要将其导入

`HashSet`与`ArrayList`一样, 可以传入不同的类型实参, 声明存储不同类型元素的`HashSet`, 不同元素类型的`HashSet`属于不同类型

传入的类型实参, 必须是**实现了`Hashable`和`Equatable<T>`接口的类型**

`HashSet`添加元素一定是根据元素的哈希值来确定存储位置的

创建实例, 可以调用`HashSet`的构造函数进行构造

同样也可以通过 其他同元素类型的`Collection`进行构造



#### 访问`HashSet`成员

> 当需要对`HashSet`的所有元素进行访问时, 可以使用`for-in`循环遍历`HashSet`的所有元素
> 
> 需要注意的是, `HashSet`并不保证按插入元素的顺序排列, 因此遍历的顺序和插入的顺序可能不同
> 
> ```cangjie
> import std.collection.*
> 
> main() {
>     let mySet = HashSet<Int64>([0, 1, 2])
>     for (i in mySet) {
>         println("The element is ${i}")
>     }
> }
> ```
> 
> 编译并执行上面的代码, 有可能会输出: 
> 
> ```text
> The element is 0
> The element is 1
> The element is 2
> ```
> 
> 当需要知道某个`HashSet`包含的元素个数时, 可以使用`size`属性获得对应信息
> 
> ```cangjie
> import std.collection.*
> 
> main() {
>     let mySet = HashSet<Int64>([0, 1, 2])
>     if (mySet.size == 0) {
>         println("This is an empty hashset")
>     } else {
>         println("The size of hashset is ${mySet.size}")
>     }
> }
> ```
> 
> 编译并执行上面的代码, 会输出: 
> 
> ```text
> The size of hashset is 3
> ```
> 
> 当想判断某个元素是否被包含在某个`HashSet`中时, 可以使用`contains`函数
> 
> 如果该元素存在会返回`true`, 否则返回`false`
> 
> ```cangjie
> let mySet = HashSet<Int64>([0, 1, 2])
> let a = mySet.contains(0)                 // a == true
> let b = mySet.contains(-1)                // b == false
> ```

`HashSet`不能使用下标随机访问元素

只能在遍历所有元素时 访问元素, 比如使用`for-in`循环遍历`HashSet`的元素

但, `HashSet`可以通过`contains()`函数, 判断目标元素是否存在, 存在返回`true`, 不存在返回`false`

#### 修改`HashSet`

> `HashSet`是一种可变的引用类型, `HashSet`类型提供了添加元素、删除元素的功能
> 
> `HashSet`的可变性是一个非常有用的特征, 可以让同一个`HashSet`实例的所有引用都共享同样的元素, 并且对它们统一进行修改
> 
> 如果需要将单个元素添加到`HashSet`里, 请使用`add`函数
> 
> 如果希望同时添加多个元素, 可以使用`add(all!: Collection<T>)`函数, 这个函数可以接受另一个相同元素类型的`Collection`类型(例如`Array`)
> 
> 当元素不存在时, `add`函数会执行添加的操作, 当`HashSet`中存在相同元素时, `add`函数将不会有效果
> 
> ```cangjie
> let mySet = HashSet<Int64>()
> mySet.add(0)                  // mySet 包含元素 0
> mySet.add(0)                  // mySet 包含元素 0
> mySet.add(1)                  // mySet 包含元素 0, 1
> let li = [2, 3]
> mySet.add(all: li)            // mySet 包含元素 0, 1, 2, 3
> ```
> 
> `HashSet`是引用类型, `HashSet`在作为表达式使用时不会拷贝副本, 同一个`HashSet`实例的所有引用都会共享同样的数据
> 
> 因此对`HashSet`元素的修改会影响到该实例的所有引用
> 
> ```cangjie
> let set1 = HashSet<Int64>([0, 1, 2])
> let set2 = set1
> set2.add(3)
> // set1 包含元素 0, 1, 2, 3
> // set2 包含元素 0, 1, 2, 3
> ```
> 
> 从`HashSet`中删除元素, 可以使用`remove`函数, 需要指定删除的元素
> 
> ```cangjie
> let mySet = HashSet<Int64>([0, 1, 2, 3])
> mySet.remove(1)                           // mySet 包含元素 0, 2, 3
> ```

`HashSet`中已存在的元素, 是**无法原地修改**的, 但可以先删除旧元素 再添加新元素

`HashSet`提供有 添加元素(`add()`函数)和删除元素(`remove()`函数) 的功能

`HashSet`内不允许存在重复的元素, 所以当使用`add()`函数 **不会向`HashSet`添加已存在的元素**

`HashSet`的`add()`也可以传入其他同元素类型的`Collection`实例

`HashSet`的`remove()`函数, 可以删除目标元素

### `HashMap`

> 使用`HashMap`类型需要导入`collection`包: 
> 
> ```cangjie
> import std.collection.*
> ```
> 
> 可以使用`HashMap`类型来构造元素为键值对的`Collection`
> 
> `HashMap`是一种哈希表, 提供对其包含的元素的快速访问
> 
> 表中的每个元素都使用其键作为标识, 可以使用键来访问相应的值
> 
> 仓颉使用`HashMap<K, V>`表示`HashMap`类型, `K`表示`HashMap`的键类型, `K`必须是实现了`Hashable`和`Equatable<K>`接口的类型, 例如数值或`String`
> 
> `V`表示`HashMap`的值的类型, `V`可以是任意类型
> 
> ```cangjie
> var a: HashMap<Int64, Int64> = ...    // 键类型为 Int64, 值类型为 Int64 的 HashMap
> var b: HashMap<String, Int64> = ...   // 键类型为 String, 值类型为 Int64 的 HashMap
> ```
> 
> 元素类型不相同的`HashMap`是不相同的类型, 所以它们之间不可以互相赋值
> 
> 因此以下例子是不合法的
> 
> ```cangjie
> b = a         // 类型不匹配
> ```
> 
> 仓颉中可以使用构造函数的方式构造一个指定的`HashMap`
> 
> ```cangjie
> let a = HashMap<String, Int64>()                                  // 创建一个空的 HashMap, 其键类型为 String, 值类型为 Int64
> let b = HashMap<String, Int64>([("a", 0), ("b", 1), ("c", 2)])    // 其键类型为 String, 值类型为 Int64, 包含元素("a", 0), ("b", 1), ("c", 2)
> let c = HashMap<String, Int64>(b)                                 // 使用另一个`Collection`来初始化一个 HashMap
> let d = HashMap<String, Int64>(10)                                // 创建一个键类型为 String、值类型为 Int64、容量为 10 的 HashMap
> let e = HashMap<Int64, Int64>(10, {x: Int64 => (x, x * x)})       // 创建一个键值类型为 Int64、大小为 10 的 HashMap, 所有元素都通过指定的规则函数进行初始化
> ```

`HashMap`是在`std.collection`包中定义的, 使用时要将其导入

`HashMap`也可以传入不同的类型实参, 声明存储不同类型元素的`HashMap`, 不同元素类型的`HashMap`属于不同类型

不过`HashMap`需要传入两个类型实参, 第一个作为`HashMap`的键`K`类型, 第二个作为`HashMap`的值`V`类型, 因为`HashMap`存储的是键值对

传入的**键类型实参**, 必须是**实现了`Hashable`和`Equatable<T>`接口的类型**; 值类型实参, 则无所谓

`HashMap`添加或修改元素, 一定是根据目标键的哈希值确定位置的

创建实例, 可以调用`HashMap`的构造函数进行构造, 同样也可以通过 其他同元素类型的`Collection`进行构造

#### 访问`HashMap`成员

> 当需要对`HashMap`的所有元素进行访问时, 可以使用`for-in`循环遍历`HashMap`的所有元素
> 
> 需要注意的是, `HashMap`并**不保证按插入元素的顺序排列**, 因此遍历的顺序和插入的顺序可能不同
> 
> ```cangjie
> import std.collection.*
> 
> main() {
>     let map = HashMap<String, Int64>([("a", 0), ("b", 1), ("c", 2)])
>     for ((k, v) in map) {
>         println("The key is ${k}, the value is ${v}")
>     }
> }
> ```
> 
> 编译并执行上面的代码, 有可能会输出: 
> 
> ```text
> The key is a, the value is 0
> The key is b, the value is 1
> The key is c, the value is 2
> ```
> 
> 当需要知道某个`HashMap`包含的元素个数时, 可以使用`size`属性获得对应信息
> 
> ```cangjie
> import std.collection.*
> 
> main() {
>     let map = HashMap<String, Int64>([("a", 0), ("b", 1), ("c", 2)])
>     if (map.size == 0) {
>         println("This is an empty hashmap")
>     } else {
>         println("The size of hashmap is ${map.size}")
>     }
> }
> ```
> 
> 编译并执行上面的代码, 会输出: 
> 
> ```text
> The size of hashmap is 3
> ```
> 
> 当想判断`HashMap`中是否包含某个键时, 可以使用`contains`函数
> 
> 如果该键存在会返回`true`, 否则返回`false`
> 
> ```cangjie
> let map = HashMap<String, Int64>([("a", 0), ("b", 1), ("c", 2)])
> let a = map.contains("a")                                         // a == true
> let b = map.contains("d")                                         // b == false
> ```
> 
> 当想**访问指定键对应的元素**时, 可以使用下标语法访问(下标的类型必须是键类型)
> 
> **使用不存在的键作为索引会触发运行时异常**
> 
> ```cangjie
> let map = HashMap<String, Int64>([("a", 0), ("b", 1), ("c", 2)])
> let a = map["a"]                                                  // a == 0
> let b = map["b"]                                                  // b == 1
> let c = map["d"]                                                  // 运行时异常
> ```

`HashMap`可以通过下标语法, 使用确认存在的键, 访问目标键的值

也可以遍历访问元素

同样提供有`contains()`函数, 用来判断**目标键是否存在**

#### 修改`HashMap`

> `HashMap`是一种可变的引用类型, `HashMap`类型提供了修改元素、添加元素、删除元素的功能
> 
> `HashMap`的可变性是一个非常有用的特征, 可以让同一个`HashMap`实例的所有引用都共享同样的元素, 并且对它们统一进行修改
> 
> 可以**使用下标语法**对某个键对应的值进行修改
> 
> ```cangjie
> let map = HashMap<String, Int64>([("a", 0), ("b", 1), ("c", 2)])
> map["a"] = 3
> ```
> 
> `HashMap`是引用类型, `HashMap`在作为表达式使用时不会拷贝副本, 同一个`HashMap`实例的所有引用都会共享同样的数据
> 
> 因此对`HashMap`元素的修改会影响到该实例的所有引用
> 
> ```cangjie
> let map1 = HashMap<String, Int64>([("a", 0), ("b", 1), ("c", 2)])
> let map2 = map1
> map2["a"] = 3
> // map1 包含元素 ("a", 3), ("b", 1), ("c", 2)
> // map2 包含元素 ("a", 3), ("b", 1), ("c", 2)
> ```
> 
> 如果需要将单个键值对添加到`HashMap`里, 请使用`add`函数
> 
> 如果希望同时添加多个键值对, 可以使用`add(all!: Collection<(K, V)>)`函数
> 
> 当键不存在时, `add`函数会执行添加的操作, 当键存在时, `add`函数会将新的值覆盖旧的值
> 
> ```cangjie
> let map = HashMap<String, Int64>()
> map.add("a", 0)                                           // map 包含元素 ("a", 0)
> map.add("b", 1)                                           // map 包含元素 ("a", 0), ("b", 1)
> let map2 = HashMap<String, Int64>([("c", 2), ("d", 3)])
> map.add(all: map2)                                        // map 包含元素 ("a", 0), ("b", 1), ("c", 2), ("d", 3)
> ```
> 
> 除了使用`add`函数以外, 也可以使用赋值的方式直接将新的键值对添加到`HashMap`
> 
> ```cangjie
> let map = HashMap<String, Int64>([("a", 0), ("b", 1), ("c", 2)])
> map["d"] = 3                                              // map 包含元素 ("a", 0), ("b", 1), ("c", 2), ("d", 3)
> ```
> 
> 从`HashMap`中删除元素, 可以使用`remove`函数, 需要指定删除的键
> 
> ```cangjie
> let map = HashMap<String, Int64>([("a", 0), ("b", 1), ("c", 2), ("d", 3)])
> map.remove("d")                                           // map 包含元素 ("a", 0), ("b", 1), ("c", 2)
> ```

`HashMap`可以通过下标语法访问 指定键的值, 当然也可以通过下标语法 修改指定键的值

```cangjie
hashMap["key"] = value
```

使用下标语法修改指定键的值时, 与只是访问指定键的值不同:

**使用下标语法访问指定键的值, 如果指定键不存在, 则会 触发运行时异常**

**但, 使用下标语法修改指定键的值时, 如果指定键不存在, 则会 新增目标键值对**

除此之外`HashMap`可以使用`add()`函数, 添加一个键值对, 如果目标键已存在, 则会修改键对应的值

`add()`函数, 也可以 传入同元素类型的`HashMap`添加一系列的键值对

`add()`函数, 更可以 传入其他的`Collection`类型, 但 要保证元素类型为`Tuple<K, V>`

`HashMap`可以使用`remove()`函数, 传入单个键, 尝试删除单个目标键值对

也可以传入其他 同键类型的`Collection`类型数据, 尝试删除一系列键值对

> [!TIP]
> 
> 仓颉`HashMap`, 使用`remove()`尝试删除键值对时, 如果键不存在, 不会触发异常
> 
> 而 使用下标语法访问目标键的值时, 如果目标键不存在, 会触发异常

---

`HashSet`和`HashMap`都是通过元素(或`Key`)的哈希值 来确定元素的存储位置的

所以, 元素(或`Key`)类型要满足 实现`Hashable`和`Equatable<K>`接口的要求

本篇文章只是介绍`HashSet`和`HashMap`最基本的使用, 更详细的使用可以阅读标准库`API`的使用或相关源码
