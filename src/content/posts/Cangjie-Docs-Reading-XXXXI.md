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

#### `const` 上下文与 `const` 表达式

> `const` 上下文是指 **`const`变量初始化表达式**，这些表达式始终在编译时求值
> 
> 因此需要对 `const` 上下文中允许的表达式加以限制，避免修改全局状态、`I/O` 等副作用，**确保其可以在编译时求值**
> 
> `const` 表达式具备了可以在编译时求值的能力
> 
> 满足如下规则的表达式是 `const` 表达式：
> 
> 1. 数值类型、`Bool`、`Unit`、`Rune`、`String` 类型的字面量（不包含插值字符串）
> 
> 2. 所有元素都是 `const` 表达式的 `Array` 字面量（不能是 `Array` 类型，可以使用 `VArray` 类型），`tuple` 字面量
> 
> 3. `const` 变量，`const` 函数形参，`const` 函数中的局部变量
> 
> 4. `const` 函数，包含使用 `const` 声明的函数名、符合 `const` 函数要求的 `lambda`、以及这些函数返回的函数表达式
> 
> 5. `const` 函数调用（包含`const` 构造函数），该函数的表达式必须是`const` 表达式，所有实参必须都是 `const` 表达式
> 
> 6. 所有参数都是 `const` 表达式的 `enum` 构造器调用，和无参数的 `enum` 构造器
> 
> 7. 数值类型、`Bool`、`Unit`、`Rune`、`String` 类型的算术表达式、关系表达式、位运算表达式，所有操作数都必须是 `const` 表达式
> 
> 8. `if`、`match`、`try`、`throw`、`return`、`is`、`as`
> 
>     这些表达式内的表达式必须都是 `const` 表达式
> 
> 9. `const` 表达式的成员访问（不包含属性的访问），`tuple` 的索引访问
> 
> 10. `const init` 和 `const` 函数中的 `this` 和 `super` 表达式
> 
> 11. `const` 表达式的 `const` 实例成员函数调用，且所有实参必须都是 `const` 表达式
> 
> > 注意：
> > 
> > 当前编译器实现暂不支持 `throw` 作为 `const` 表达式使用
