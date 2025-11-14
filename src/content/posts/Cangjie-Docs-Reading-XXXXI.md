---
title: "仓颉文档阅读-开发指南IV: 函数(VI) - const 函数和常量求值"
published: 2025-11-13 16:18:22
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

### `const`函数和常量求值

> 常量求值 允许某些特定形式的表达式 **在编译时求值**，可以减少程序运行时需要的计算
> 
> 本章主要介绍常量求值的使用方法与规则

常量求值主要的作用就是 编译器求值, 减少运行时消耗

#### `const`上下文与`const`表达式

> `const`上下文是指 **`const`变量初始化表达式**，这些表达式始终在编译时求值
> 
> 因此需要对`const`上下文中允许的表达式加以限制，避免修改全局状态、`I/O`等副作用，**确保其可以在编译时求值**
> 
> `const`表达式具备了可以在编译时求值的能力
> 
> 满足如下规则的表达式是`const`表达式：
> 
> 1. 数值类型、`Bool`、`Unit`、`Rune`、`String`类型的字面量（不包含插值字符串）
> 
> 2. 所有元素都是`const`表达式的`Array`字面量（不能是`Array`类型，可以使用`VArray`类型），`tuple`字面量
> 
> 3. `const`变量，`const`函数形参，`const`函数中的局部变量
> 
> 4. `const`函数，包含使用`const`声明的函数名、符合`const`函数要求的`lambda`、以及这些函数返回的函数表达式
> 
> 5. `const`函数调用（包含`const`构造函数），该函数的表达式必须是`const`表达式，所有实参必须都是`const`表达式
> 
> 6. 所有参数都是`const`表达式的`enum`构造器调用，和无参数的`enum`构造器
> 
> 7. 数值类型、`Bool`、`Unit`、`Rune`、`String`类型的算术表达式、关系表达式、位运算表达式，所有操作数都必须是`const`表达式
> 
> 8. `if`、`match`、`try`、`throw`、`return`、`is`、`as`
> 
>     这些表达式内的表达式必须都是`const`表达式
> 
> 9. `const`表达式的成员访问（不包含属性的访问），`tuple`的索引访问
> 
> 10. `const init`和`const`函数中的`this`和`super`表达式
> 
> 11. `const`表达式的`const`实例成员函数调用，且所有实参必须都是`const`表达式
> 
> > 注意：
> > 
> > 当前编译器实现暂不支持`throw`作为`const`表达式使用

仓颉中, 存在`const`上下文的概念

什么是`const`上下文呢?

即, **给`const`变量初始化 所用的表达式**

但, 仓颉中 并不是所有表达式都可以给`const`变量做初始化, 只有`const`表达式, 才能给`const`变量做初始化

这也就是说`const`上下文中, 实际只能出现`const`表达式, 只有这样才能保证编译时求值

仓颉中对`const`表达式的定义在文档中已经列出了

有 常用数据类型的字面量, `const`变量, `const`函数的形参以及函数体内的局部变量, `const`函数等等

#### `const`函数

> `const`函数是一类特殊的函数，这些函数**具备了可以在编译时求值的能力**
> 
> **在`const`上下文中调用**这种函数时，这些函数会在**编译时执行计算**
> 
> 而在其他非`const`上下文，`const`函数会和普通函数一样在运行时执行
> 
> 下例是一个计算平面上两点距离的`const`函数，`distance`中使用`let`定义了两个局部变量`dx`和`dy`：
> 
> ```cangjie
> struct Point {
>     const Point(let x: Float64, let y: Float64) {}
> }
> 
> const func distance(a: Point, b: Point) {
>     let dx = a.x - b.x
>     let dy = a.y - b.y
>     (dx**2 + dy**2)**0.5
> }
> 
> main() {
>     const a = Point(3.0, 0.0)
>     const b = Point(0.0, 4.0)
>     const d = distance(a, b)
>     println(d)
> }
> ```
> 
> 编译运行输出：
> 
> ```text
> 5.000000
> ```

仓颉中的`const`函数, 在`const`上下文中调用时, 编译时执行运算

在非`const`上下文中调用时, `const`函数就被当作普通函数调用

> 需要注意：
> 
> 1. `const`函数声明必须使用`const`修饰
> 
> 2. 全局`const`函数和`static const`函数中**只能访问`const`声明的外部变量**，包含`const`全局变量、`const`静态成员变量，其他外部变量都不可访问
> 
>     `const init`函数和`const`实例成员函数除了能访问`const`声明的外部变量，还可以访问当前类型的实例成员变量
> 
> 3. `const`函数中的表达式都必须是`const`表达式，`const init`函数除外
> 
> 4. `const`函数中可以使用`let`、`const`声明新的局部变量
> 
>     但**不支持`var`**
> 
> 5. `const`函数中的参数类型和返回类型没有特殊规定
> 
>     如果该函数调用的实参不符合`const`表达式要求，那这个函数调用不能作为`const`表达式使用，但仍然可以作为普通表达式使用
> 
> 6. `const`函数不一定都会在编译时执行，例如可以在非`const`函数中运行时调用
> 
> 7. **`const`函数与非`const`函数重载规则一致**
> 
> 8. 数值类型、`Bool`、`Unit`、`Rune`、`String`类型 和`enum`支持定义`const`实例成员函数
> 
> 9. 对于`struct`和`class`，**只有定义了`const init`才能定义`const`实例成员函数**
> 
>     `class`中的`const`实例成员函数不能是`open`的
> 
>     `struct`中的`const`实例成员函数不能是`mut`的
> 
> 另外，接口中也可以定义`const`函数，但会受到以下规则限制：
> 
> 1. 接口中的`const`函数，实现类型必须也用`const`函数才算实现接口
> 
> 2. 接口中的非`const`函数，实现类型使用`const`或非`const`函数都算实现接口
> 
> 3. 接口中的`const`函数与接口的`static`函数一样，只有在该接口作为泛型约束的时候，受约束的泛型变元或变量才能使用这些`const`函数
> 
> 在下面的例子中，在接口`I`里定义了两个`const`函数，类`A`实现了接口`I`，泛型函数`g`的形参类型上界是`I`
> 
> ```cangjie
> interface I {
>     const func f(): Int64
>     const static func f2(): Int64
> }
> 
> class A <: I {
>     public const func f() { 0 }
>     public const static func f2() { 1 }
>     const init() {}
> }
> 
> const func g<T>(i: T) where T <: I {
>     return i.f() + T.f2()
> }
> 
> main() {
>     println(g(A()))
> }
> ```
> 
> 编译执行上述代码，输出结果为：
> 
> ```text
> 1
> ```

仓颉中, `const`函数的就是被`const`修饰的函数:

```cangjie
const func function() {}
```

不过, `const`函数的定义存在许多的限制, 具体限制不用太多介绍, 文档中都有列出


#### `const init`

> 如果一个`struct`或`class`定义了`const`构造器，那么这个`struct/class`实例可以用在`const`表达式中
> 
> 1. 如果当前类型是`class`，则**不能具有`var`声明的实例成员变量**，否则不允许定义`const init`
> 
>     如果当前类型具有父类，当前的`const init`必须调用父类的`const init`可以显式调用或者隐式调用无参`const init`），**如果父类没有`const init`则报错**
> 
> 2. 当前类型的实例成员变量如果有初始值，初始值必须要是`const`表达式，否则不允许定义`const init`
> 
> 3. `const init`内可以使用赋值表达式对实例成员变量赋值，除此以外不能有其他赋值表达式
> 
> `const init`与`const`函数的区别是`const init`内允许对实例成员变量进行赋值（需要使用赋值表达式）

`const init`就是类或结构体的`const`构造函数

如果要定义`const init`, 那么:

1. 不允许存在`var`成员变量

2. 父类必须也存在`const init`

3. 所有实例成员变量的初始值, 必须是`const`表达式
