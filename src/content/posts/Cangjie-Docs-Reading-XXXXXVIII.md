---
title: "仓颉文档阅读-开发指南X: Collection 类型(III) - Iterable 和 Collections"
published: 2025-11-27 17:46:41
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

### `Iterable`和`Collections`

> 前面已经了解过`Range`、`Array`、`ArrayList`, 它们都可以使用`for-in`进行遍历操作
> 
> 对于开发者自定义的类型, 也能实现类似的遍历操作
> 
> `Range`、`Array`、`ArrayList`都是通过`Iterable`来支持`for-in`语法的
> 
> `Iterable`是如下形式(只展示了核心代码)的一个内置`interface`
> 
> ```cangjie
> interface Iterable<T> {
>     func iterator(): Iterator<T>
>     ...
> }
> ```
> 
> `iterator`函数要求返回的`Iterator`类型是如下形式(只展示了核心代码)的另一个内置`interface`
> 
> ```cangjie
> interface Iterator<T> <: Iterable<T> {
>     mut func next(): Option<T>
>     ...
> }
> ```
> 
> 可以使用`for-in`语法来遍历任何一个实现了`Iterable`接口类型的实例
> 
> 假设有这样一段`for-in`代码, 如下所示
> 
> ```cangjie
> let list = [1, 2, 3]
> for (i in list) {
>     println(i)
> }
> ```
> 
> 它等价于如下形式的`while`代码
> 
> ```cangjie
> let list = [1, 2, 3]
> var it = list.iterator()
> while (true) {
>     match (it.next()) {
>         case Some(i) => println(i)
>         case None => break
>     }
> }
> ```
> 
> 另外一种常见的遍历`Iterable`类型的方法是在`while`表达式的条件中使用模式匹配, 比如上面`while`代码的另一种等价写法是: 
> 
> ```cangjie
> let list = [1, 2, 3]
> var it = list.iterator()
> while (let Some(i) <- it.next()) {
>     println(i)
> }
> ```
> 
> `Array`、`ArrayList`、`HashSet`、`HashMap`类型都实现了`Iterable`, 因此可以将其用在`for-in`或者`while`中

`Iterable`接口, 是`Collection`类型能够使用`for-in`遍历其元素的原因

`Iterable`这个名字意为可迭代的, 不由让人联想到C++容器中的迭代器

仓颉`Iterable`实际可以类比C++容器中的迭代器

`Iterable`接口拥有`iterator()`函数, 此函数返回一个`Iterator<T>`类型

`Iterator<T>`类型拥有`next()`函数, 返回`Option<T>`类型数据

这就是`Collection`可被`for-in`等循环遍历访问元素的关键

这意味着, 只要实现了`Iterable`的类型, 其实都可以使用`for-in`等语法循环迭代遍历
