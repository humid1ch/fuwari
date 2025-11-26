---
title: "仓颉文档阅读-开发指南VIV: 扩展(II) - 扩展访问规则"
published: 2025-11-26 16:27:23
description: '仓颉文档阅读的开发指南部分, 本篇文章介绍仓颉语言的 扩展访问规则 的相关内容'
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

## 扩展

### 访问规则

扩展成员的可访问性, 并不完全与原类型成员的可访问性保持一致

#### 扩展的修饰符

> **扩展本身不能使用修饰符修饰**
> 
> 例如，下面的例子中对 `A` 的直接扩展前使用了 `public` 修饰，将编译报错
> 
> ```cangjie
> public class A {}
> 
> public extend A {}            // Error, 在 extend 之前不该存在修饰符
> ```
> 
> 扩展成员可使用的修饰符有：`static`、`public`、`protected`、`internal`、`private`、`mut`
> 
> - 使用 **`private`** 修饰的成员 **只能在本扩展内使用**，外部不可见
> 
> - 使用 **`internal`** 修饰的成员 **可以在当前包及子包（包括子包的子包）内使用**，这是默认行为
> 
> - 使用 **`protected`** 修饰的成员 **在本模块内可以被访问（受导出规则限制）**
> 
>     当被扩展类型是 `class` 时，该 **`class`的子类定义体也能访问**
> 
> - 使用 **`static`** 修饰的成员，**只能通过类型名访问**，不能通过实例对象访问
> 
> - 对 **`struct`** 类型的扩展 **可以定义`mut`函数**
> 
> ```cangjie
> package p1
> 
> public open class A {}
> 
> extend A {
>     public func f1() {}
>     protected func f2() {}
>     private func f3() {}
>     static func f4() {}
> }
> 
> main() {
>     A.f4()
>     var a = A()
>     a.f1()
>     a.f2()
> }
> ```
> 
> 扩展内的成员定义不支持使用 `open`、`override`、`redef` 修饰
> 
> ```cangjie
> class Foo {
>     public open func f() {}
>     static func h() {}
> }
> 
> extend Foo {
>     public override func f() {}       // Error
>     public open func g() {}           // Error
>     redef static func h() {}          // Error
> }
> ```

扩展定义本身不允许用任何修饰符修饰

但扩展的成员可以使用 可访问性修饰符: `private`、`internal`、`protected`和`public`, 以及`static`和`mut`(仅`struct`)修饰

同时, 扩展的成员不允许使用`open`、`override`和`redef`修饰, 即扩展不能被继承
