---
title: "仓颉文档阅读-开发指南VII: 类和接口(I) - 类的定义"
published: 2025-11-18 16:26:06
description: '仓颉文档阅读的开发指南部分, 本篇文章介绍仓颉语言的类的相关内容'
image: 'https://humid1ch.oss-cn-shanghai.aliyuncs.com/20250929154944807.webp'
tags: ["开发语言", "仓颉"]
category: Blogs
draft: true
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

## 类和接口

### 类

> `class` 类型是面向对象编程中的经典概念，仓颉中同样支持使用 `class` 来实现面向对象编程
> 
> `class` 与 `struct` 的主要区别在于：`class` 是引用类型，`struct` 是值类型，它们在赋值或传参时行为是不同的
> 
> **`class` 之间可以继承，但 `struct` 之间不能继承**
> 
> 本节依次介绍如何定义 `class` 类型，如何创建对象，以及 `class` 的继承

仓颉的类也是用`class`定义的

具体本篇文章会介绍

#### `class`定义

> `class` 类型的定义以关键字 `class` 开头，后跟 `class` 的名字，接着是定义在一对花括号中的 `class` 定义体
> 
> `class` 定义体中可以定义一系列的成员变量、成员属性（参见 [属性]() ）、静态初始化器、构造函数、成员函数和操作符函数（详见 [操作符重载]() ）
> 
> ```cangjie
> class Rectangle {
>     let width: Int64
>     let height: Int64
> 
>     public init(width: Int64, height: Int64) {
>         this.width = width
>         this.height = height
>     }
> 
>     public func area() {
>         width * height
>     }
> }
> ```
> 
> 上例中定义了名为 `Rectangle` 的 `class` 类型，它有两个 `Int64` 类型的成员变量 `width` 和 `height`，一个有两个 `Int64` 类型参数的构造函数，以及一个成员函数 `area`（返回 `width` 和 `height` 的乘积）
> 
> > 注意：
> > 
> > `class` 只能定义在源文件的顶层作用域

仓颉`class`, 也是可以定义成员、函数等

但与C++不同的是, 仓颉`class`可以定义属性

且 仓颉`class`的成员函数的可访问性, 是使用可访问性修饰符单独修饰的, 而不是使用修饰符修饰范围

> 使用 `abstract` 修饰的类为抽象类，与普通类不同的是，在抽象类中除了可以定义普通的函数，还允许声明抽象函数（没有函数体）
> 
> 抽象类定义时的 `open` 修饰符是可选的，也可以使用 `sealed` 修饰符修饰抽象类，表示该抽象类只能在本包被继承
> 
> 下例中在抽象类 `AbRectangle` 中定义了抽象函数 `foo`
> 
> ```cangjie
> abstract class AbRectangle {
>     public func foo(): Unit
> }
> ```
> 
> > 注意：
> > 
> > - 抽象类中禁止定义 `private` 的抽象函数
> > 
> > - 不能为抽象类创建实例
> > 
> > - 抽象类的非抽象子类必须实现父类中的所有抽象函数

仓颉的抽象类, 使用`abstract`修饰符显式修饰

关于抽象类, 自动包含`open`属性, 可以**使用`sealed`修饰, 限定只能在本包中被继承**

仓颉中, 只有显式使用`abstract`修饰符修饰的类, 才能定义抽象函数(无函数体)

这与C++不同, C++是存在纯虚函数的类 就是抽象类

但抽象类的其他概念是一样的, 不能创建实例, 不能定义`private`抽象函数, 子类必须实现抽象函数

##### `class`成员变量

`class` 成员变量分为**实例成员变量**和**静态成员变量**

**静态成员变量**使用 `static` 修饰符修饰，**没有静态初始化器时必须有初值**，**只能通过类型名访问**，参考如下示例：

```cangjie
class Rectangle {
    let width = 10
    static let height = 20
}

let l = Rectangle.height        // l = 20
```

**实例成员变量**定义时可以不设置初值（但必须标注类型），也可以设置初值，只能通过对象（即类的实例）访问，参考如下示例：

```cangjie
class Rectangle {
    let width = 10
    let height: Int64
    init(h: Int64){
        height = h
    }
}
let rec = Rectangle(20)
let l = rec.height              // l = 20
```
