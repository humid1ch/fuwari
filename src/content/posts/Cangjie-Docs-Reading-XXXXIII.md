---
title: "仓颉文档阅读-开发指南VI: 枚举类型和模式匹配(I) - 枚举类型 和 Option类型"
published: 2025-11-17 10:31:22
description: '仓颉文档阅读的开发指南部分, 本篇文章介绍仓颉语言的枚举与模式匹配相关内容'
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

## 枚举类型和模式匹配

### 枚举类型

> 本节介绍仓颉中的`enum`类型
> 
> `enum`类型提供了通过列举一个类型的所有可能取值来定义此类型的方式
> 
> 在很多语言中都有`enum`类型(或者称枚举类型), 但是不同语言中的`enum`类型的使用方式和表达能力均有所差异, 仓颉中的`enum`类型可以理解为函数式编程语言中的代数数据类型(Algebraic Data Types)

什么是枚举类型呢?

`enum`类型, 一般将一个数据可取值的所有值定义为枚举类型

比如, 一周内的星期 只有星期一到星期日, 可穷举 就可以将星期定义为一个枚举类型, 可取值即为星期一到星期日

```cangjie
enum Week {
    Monday |
    Tuesday | 
    Wednesday |
    Thurday |
    Friday |
    Saturday |
    Sunday
}
```

#### `enum`的定义

> 定义`enum`时需要把它**所有可能的取值一一列出**, 称这些值为 **`enum`的构造器**(或者`constructor`)
> 
> `enum`类型的定义以关键字`enum`开头, 接着是`enum`的名字, 之后是定义在一对花括号中的`enum`体, `enum`体中定义了若干构造器, 多个构造器之间使用`|`进行分隔(第一个构造器之前的`|`是可选的)
> 
> 构造器可以是有名字的, 也可以是**没有名字的`...`**
> 
> 每个`enum`中至少存在一个有名字的构造器
> 
> 有名字的构造器可以没有参数(即"无参构造器"), 也可以携带若干个参数(即"有参构造器")
> 
> 如下示例代码定义了一个名为`RGBColor`的`enum`类型, 它有`3`个构造器: `Red`、`Green`和`Blue`, 分别表示`RGB`色彩模式中的红色、绿色和蓝色
> 
> 每个构造器有一个`UInt8`类型的参数, 用来表示每个颜色的亮度级别
> 
> ```cangjie
> enum RGBColor {
>     | Red(UInt8) | Green(UInt8) | Blue(UInt8)
> }
> ```

`enum`就是枚举类型

定义`enum`类型时, 一般将所有可能的取值定义为构造器, **此类型的取值只能是定义的构造器**, 构造器之间使用`|`分割(在每个构造器之前, 第一个构造器之前的`|`是可选的)

每个构造器表示一个此类型`enum`取值, 但, 无名构造器`...`无法作为取值

仓颉中的`enum`是可以携带数据的, C/C++中的`enum`就不能携带数据

> 仓颉支持同一个`enum`中定义多个同名构造器, 但是要求这些构造器的参数个数不同(认为没有参数的构造器的参数个数等于 0), 例如: 
> 
> ```cangjie
> enum RGBColor {
>     | Red | Green | Blue
>     | Red(UInt8) | Green(UInt8) | Blue(UInt8)
> }
> ```
> 
> 每个`enum`中最多只有一个没有名字的`...`构造器, 且`...`只能是最后一个构造器
> 
> 拥有`...`构造器的`enum`称为`non-exhaustive enum`
> 
> 由于没有名字, 这个构造器不能被直接匹配, 在解构时, 需要使用可匹配所有构造器的模式, 如通配符模式`_`或绑定模式, 具体可参见`match`表达式的定义
> 
> 例如: 
> 
> ```cangjie
> enum t {
>     | Red | Green | Blue | ...
> }
> ```

仓颉`enum`的可以定义多个同名构造器, 要求同名构造器之间的参数个数不同, 即 携带数据的个数不同

仓颉`enum`定义`...(无名构造器)`时, 只能作为最后一个构造器

> `enum`支持递归定义, 例如, 下面的例子中使用`enum`定义了一种表达式(即`Expr`), 此表达式只能有 3 种形式: 
> 
> 1. 单独的一个数字`Num`(携带一个`Int64`类型的参数)
> 
> 2. 加法表达式`Add`(携带两个`Expr`类型的参数)
> 
> 3. 减法表达式`Sub`(携带两个`Expr`类型的参数)
> 
> 对于`Add`和`Sub`这两个构造器, 其参数中递归地使用到了`Expr`自身
> 
> ```cangjie
> enum Expr {
>     | Num(Int64)
>     | Add(Expr, Expr)
>     | Sub(Expr, Expr)
> }
> ```

仓颉`enum`的构造器可以递归定义, 即 构造器参数可以是`enum`类型本身

就比如文档中的例子, 可以这样使用:

```cangjie
enum Expr {
    | Num(Int64)
    | Add(Expr, Expr)
    | Sub(Expr, Expr)
}

func exprRes(expr: Expr) : Int64 {
    match (expr) {
        case Num(num) => num
        case Add(left, right) => exprRes(left) + exprRes(right)
        case Sub(left, right) => exprRes(left) - exprRes(right)
    }
}

main() {
    let num1 = Expr.Num(1)
    let num2 = Expr.Num(2)
    let addExpr = Expr.Add(num1, num2)
    let subExpr = Expr.Sub(num1, num2)

    println(exprRes(num1))
    println(exprRes(num2))
    println(exprRes(addExpr))
    println(exprRes(subExpr))

    return 0
}
```

可以使用模式匹配, 匹配对应的构造器实现执行对应的代码

> 另外, 在`enum`体中还可以定义一系列成员函数、操作符函数(详见操作符重载)和成员属性(详见属性), 但是要求**构造器、成员函数、成员属性之间不能重名**
> 
> 例如, 下面的例子在`RGBColor`中定义了一个名为`printType`的函数, 它会输出字符串`RGBColor`: 
> 
> ```cangjie
> enum RGBColor {
>     | Red | Green | Blue
> 
>     public static func printType() {
>         print("RGBColor")
>     }
> }
> ```
> 
> > 注意: 
> > 
> > `enum`只能定义在源文件的顶层作用域

仓颉`enum`中可以定义一些列的成员函数、成员属性等, 但不能存在成员变量

当然也可以实现接口, 比如:

```cangjie
enum Week <: ToString {
    | Monday
    | Tuesday
    | Wednesday
    | Thurday
    | Friday
    | Saturday
    | Sunday

    public func toString(): String {
        match (this) {
            case Monday => "Monday"
            case Wednesday => "Wednesday"
            case Thurday => "Thurday"
            case Friday => "Friday"
            case Saturday => "Saturday"
            case Sunday => "Sunday"
            case _ => "None"
        }
    }
}

main() {
    let theDay = Week.Monday
    println(theDay)
}
```

这段代码将会输出:

```text
Monday
```

#### `enum`的使用

> 定义了`enum`类型之后, 就可以创建此类型的实例(即`enum`值), `enum`值只能取`enum`类型定义中的一个构造器
> 
> `enum`没有构造函数, 可以通过`类型名.构造器`, 或者直接使用构造器的方式来构造一个`enum`值(对于有参构造器, 需要传实参)
> 
> 下例中, `RGBColor`中定义了三个构造器, 其中有两个无参构造器(`Red`和`Green`)和一个有参构造器(`Blue(UInt8)`), `main`中定义了三个`RGBColor`类型的变量`r`, `g`和`b`
> 
> 其中, `r`的值使用`RGBColor.Red`进行初始化, `g`的值直接使用`Green`进行初始化, `b`的值使用`Blue(100)`进行初始化: 
> 
> ```cangjie
> enum RGBColor {
>     | Red | Green | Blue(UInt8)
> }
> 
> main() {
>     let r = RGBColor.Red
>     let g = Green
>     let b = Blue(100)
> }
> ```
> 
> 当省略类型名时, `enum`构造器的名字可能和类型名、变量名、函数名发生冲突
> 
> 此时必须加上`enum`类型名来使用`enum`构造器, 否则只会选择同名的类型、变量、函数定义
> 
> 下面的例子中, 只有构造器`Blue(UInt8)`可以不带类型名使用, `Red`和`Green(UInt8)`皆会因为名字冲突而不能直接使用, 必须加上类型名`RGBColor`
> 
> ```cangjie
> let Red = 1
> 
> func Green(g: UInt8) {
>     return g
> }
> 
> enum RGBColor {
>     | Red | Green(UInt8) | Blue(UInt8)
> }
> 
> let r1 = Red                 // Will choose 'let Red'
> let r2 = RGBColor.Red        // Ok: constructed by enum type name
> 
> let g1 = Green(100)          // Will choose 'func Green'
> let g2 = RGBColor.Green(100) // Ok: constructed by enum type name
> 
> let b = Blue(100)            // Ok: can be uniquely identified as an enum constructor
> ```
> 
> 如下的例子中, 只有构造器`Blue`会因为名称冲突而不能直接使用, 必须加上类型名`RGBColor`
> 
> ```cangjie
> class Blue {}
> 
> enum RGBColor {
>     | Red | Green(UInt8) | Blue(UInt8)
> }
> 
> let r = Red                 // Ok: constructed by enum type name
> 
> let g = Green(100)          // Ok: constructed by enum type name
> 
> let b = Blue(100)           // Will choose constructor of 'class Blue' and report an error
> ```

仓颉 可以使用`enum`定义的其中一个构造器作为`enum`变量的值

**`enum`没有构造函数**:

1. 可以通过`类型名.构造器`的方式, 构造一个`enum`值(对于有参构造器, 需要传实参)

    比如: `RGBColor.Red`或`RGBColor.Green(100)`

2. 直接使用构造器的方式, 构造一个`enum`值(对于有参构造器, 需要传实参)

    比如: `Red`或`Green(100)`

    但, **当构造器名与其他可见标识符存在冲突时, 就只能通过指定`enum`类型名来构造`enum`值**

### `Option`类型

> `Option`类型使用`enum`定义, 它包含两个构造器: `Some`和`None`
> 
> 其中, **`Some`会携带一个参数, 表示有值**; **`None`不带参数, 表示无值**
> 
> 当需要表示**某个类型可能有值, 也可能没有值**时, 可以选择使用`Option`类型
> 
> `Option`类型被定义为一个泛型`enum`类型, 定义如下(这里仅需要知道尖括号中的`T`是一个类型形参, 当`T`为不同类型时会得到不同的`Option`类型即可
> 
> 关于泛型的详细介绍, 可参见[泛型]: 
> 
> ```cangjie
> enum Option<T> {
>     | Some(T)
>     | None
> }
> ```
> 
> 其中, `Some`构造器的参数类型就是类型形参`T`, 当`T`被实例化为不同的类型时, 会得到不同的`Option`类型, 例如: `Option<Int64>`、`Option<String>`等

`Option`是仓颉库中实现的一个`enum`, 通常用于表示某个类型可能有值, 也可能没值

举个例子:

```cangjie
func divBy10(num: Int64): Option<Int64> {
    if (num != 0) {
        return 10 / num
    }

    return None
}

main() {
    println(divBy10(2))
    println(divBy10(0))

    return 0
}
```

定义函数 返回值类型为`Option<Int64>`, `0`不能当除数, 所以当函数实参为`0`时, 返回值当为`None`, 否则计算`10 / num`并返回结果

这段代码的输出为:

```text
Some(5)
None
```

被`Some()`携带, 表示有值, `None`表示无值

这种`Option`方式, 比C/C++中 可能返回空指针更加安全

因为`Option`提供有安全访问机制, 如果不通过安全访问机制 尝试访问`Option<T>`的数据, 无法通过语法检查

> `Option`类型还有一种简单的写法: 在类型名前加`?`
> 
> 也就是说, 对于任意类型`Ty`, `?Ty`等价于`Option<Ty>`
> 
> 例如, `?Int64`等价于`Option<Int64>`, `?String`等价于`Option<String>`等等
> 
> 下面的例子展示了如何定义`Option`类型的变量: 
> 
> ```cangjie
> let a: Option<Int64> = Some(100)
> let b: ?Int64 = Some(100)
> let c: Option<String> = Some("Hello")
> let d: ?String = None
> ```
> 
> 另外, 虽然`T`和`Option<T>`是不同的类型, 但是当明确知道某个位置需要的是`Option<T>`类型的值时, 可以**直接传一个`T`类型的值**, 编译器会用`Option<T>`类型的`Some`构造器**将`T`类型的值封装成`Option<T>`类型的值**(注意: 这里并**不是类型转换**)
> 
> 例如, 下面的定义是合法的(等价于上例中变量`a`, `b`和`c`的定义): 
> 
> ```cangjie
> let a: Option<Int64> = 100
> let b: ?Int64 = 100
> let c: Option<String> = "100"
> ```
> 
> 在上下文没有明确的类型要求时, 无法使用`None`直接构造出想要的类型, 此时应使用`None<T>`这样的语法来构造`Option<T>`类型的数据, 例如: 
> 
> ```cangjie
> let a = None<Int64>           // a: Option<Int64>
> let b = None<Bool>            // b: Option<Bool>
> ```

对于`Option<T>`类型, 存在一种更加快速的定义语法: `?T`, 即`Option<T>`等价于`?T`

另外, 对于`Option<T>`类型, 可以直接使用`T`类型数据赋值, 编译器会自动使用`Option<T>.Some()`构造器进行封装

如果上下文中没有显式指定类型, 要使用`Option<T>.None`赋值时, 可以使用`Option<T>.None`和`None<T>`, 比如: `Option<Int64>.None`、`None<Int64>`

> [!TIP]
> 
> 仓颉文档, 将`Option`的使用放在了[异常处理-使用`Option`]章节, 所以本文不做进一步分析
