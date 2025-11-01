---
title: "仓颉文档阅读-语言规约XII: 异常"
published: 2025-10-16 14:57:04"
description: "一直对仓颉挺感兴趣的, 但是一直没有去读一下文档, 慢慢看一看, 了解一下"
image: 'https://humid1ch.oss-cn-shanghai.aliyuncs.com/20250929154944807.webp'
categories:
    - Blogs
tags:
    - 开发语言
    - 仓颉
---

import Info from "../../components/mdx/Info.astro";

import Warning from "../../components/mdx/Warning.astro";

import Question from "../../components/mdx/Question.astro";

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

## 异常

> 在编写软件系统时，检测和处理程序中的错误行为往往是十分困难的，为了保证系统的正确性和健壮性，很多软件系统中都包含大量的代码用于错误检测和错误处理
>
> 异常, 是一类特殊的**可以被程序员捕获并处理**的错误，是程序执行时出现的一系列不正常行为的统称，例如，索引越界、除零错误、计算溢出、非法输入等
>
> 异常不属于程序的正常功能，**一旦发生异常，要求程序必须立即处理**，即 将程序的控制权从正常功能的执行处转移至处理异常的部分
>
> 仓颉编程语言提供异常处理机制, 用于处理程序运行时 可能出现的各种异常情况
>
> 异常处理主要涉及：
>
> - `try` 表达式（`try expression`），包括普通 `try` 表达式和 `try-with-resources` 表达式
>
> - `throw` 表达式（`throw expression`）由关键字 `throw` 以及尾随的表达式组成
>
>     尾随表达式的类型必须继承于 `Exception` 或 `Error` 类
>
> 下面将分别介绍 `try` 表达式和 `throw` 表达式

经典的`try`和`throw`异常处理机制, C++中当然也存在这样的机制

但是细节还未了解, 只知道异常是用来做什么的

### `Try`表达式

> 根据是否涉及资源的自动管理，将 `try` 表达式分为两类：**不涉及资源自动管理**的普通 `try` 表达式，以及**会进行资源自动管理**的 `try-with-resources` 表达式
>
> `try` 表达式的语法定义为：
>
> ```
> tryExpression
>     : 'try' block 'finally' block
>     | 'try' block ('catch' '(' catchPattern ')' block)+ ('finally' block)?
>     | 'try' '(' ResourceSpecifications ')' block ('catch' '(' catchPattern ')' block)* ('finally' block)?
>     ;
> catchPattern
>     : wildcardPattern
>     | exceptionTypePattern
>     ;
> exceptionTypePattern
>     : ('_' | identifier) ':' type ('|' type)*
>     ;
> ResourceSpecifications
>     : ResourceSpecification (',' ResourceSpecification)*
>     ;
>
> ResourceSpecification
>     : identifier (':' classType)? '=' expression
>     ;
> ```
>
> 接下来分别对普通 `try` 表达式和 `try-with-resources` 表达式进行介绍

仓颉因为涉及资源自动管理, 所以将`try`分为了两种, 而C++几乎所有非栈资源均为手动管理, 所以没有区分

#### 普通`Try`表达式

> 普通 `try` 表达式（本小节提到的 `try` 表达式特指普通的 `try` 表达式）包括三个部分：`try` 块，`catch` 块和 `finally` 块
>
> - `Try` 块，以**关键字 `try`** 开始，后面紧跟一个由 表达式与声明 组成的块（用一对花括号括起来，定义了新的局部作用域，可以包含任意表达式和声明，后简称"块"），`try` 后面的块内可以**抛出异常**，并被紧随的 `catch` 块所捕获并处理（如果不存在 `catch` 块或未被捕获，则在执行完 `finally` 块后，继续抛出至调用它的函数）
>
> - `Catch` 块，一个 `try` 表达式可以包含零个或多个 `catch` 块 **(当没有 `catch` 块时必须有 `finally` 块)**
>
>     每个 `catch` 块以 **关键字`catch`** 开头，后跟一条(`catchPattern`)和一个 表达式与声明 组成的块，`catchPattern` 通过 **模式匹配** 的方式匹配待捕获的异常，一旦匹配成功，则交由表达式与声明组成的块进行处理，并且 **忽略它后面的所有 `catch` 块**
>
>     当某个 `catch` 块可捕获的异常类型 均可被定义在它前面的某个 `catch` 块所捕获时，会在此 `catch` 块处报"`catch` 块不可达"的 `warning`
>
> - `Finally` 块，以**关键字 `finally`** 开始，后面紧跟一个用花括号括起来的 表达式与声明 组成的块
>
>     原则上，`finally` 块中主要实现一些"善后"的工作，如释放资源等，且要尽量**避免在 `finally` 块中再抛异常**
>
>     并且无论异常是否发生（即无论 `try` 块中是否抛出异常），`finally` 块内的内容都会被执行（若异常未被处理，执行完 `finally` 块后，继续向外抛出异常）
>
>     另外，一个 `try` 表达式中可以包含一个 `finally` 块，也可以不包含 `finally` 块（但此时必须至少包含一个 `catch` 块）

熟悉的`try-catch`, 这是C++中拥有的, 仓颉还有一个`finally`

`catch`和`finally`可以共存, 且`catch`和`finally`至少有一个

仓颉中的`catch`是按照模式匹配去捕获异常的

如果存在`finally`, 则 此块是**必执行**的 无论`try`是否抛出异常

> 其中 `catchPattern` 有两种模式：
>
> - ***通配符模式（"`_`"）***：可以捕获同级 `try` 块内抛出的任意类型的异常，等价于类型模式中的 `e: Exception`，即捕获 `Exception` 及其子类所定义的异常
>
>     示例：
>
>     ```cangjie
>     // 使用通配符模式 捕获异常
>     let arrayTest: Array<Int64> = Array<Int64>([0, 1, 2])
>     try {
>         let lastElement = arrayTest[3]
>     } catch (_) {
>         print("catch an exception!")
>     }
>     ```
>
> - ***类型模式***：可以捕获指定类型（或其子类型）的异常，语法上主要有两种格式：
>
>     **`identifier` : `ExceptionClass`**
>
>     此格式可以捕获 **类型为 `ExceptionClass` 及其子类的异常**，并将捕获到的异常实例转换成 `ExceptionClass`，然后 **与 `identifier` 定义的变量进行绑定**，接着就可以在 `catch` 块中通过 `identifier` 定义的变量访问捕获到的异常实例
>
>     **`identifier : ExceptionClass_1 | ExceptionClass_2 | ... | ExceptionClass_n`**
>
>     此格式可以通过连接符`|` **将多个异常类进行拼接**，连接符 `|` 表示"或"的关系：可以捕获 类型为 `ExceptionClass_1` 及其子类的异常，或者捕获类型为 `ExceptionClass_2` 及其子类的异常，依次类推，或捕获类型为 `ExceptionClass_n` 及其子类的异常（假设 n 大于 1）
>
>     **当待捕获异常的类型 属于上述"或"关系中的任一类型或其子类型时，此异常将被捕获**
>
>     但是由于无法静态地确定被捕获异常的类型，所以会 **将被捕获异常的类型转换成由`|`连接的所有类型的最小公共父类**，并将异常实例与 `identifier` 定义的变量进行绑定
>
>     因此在此类模式下，`catch` 块内只能通过 `identifier` 定义的变量访问 `ExceptionClass_i（1 <= i <= n）` 的 **最小公共父类中的成员变量和成员函数**
>
>     当然，也可以使用通配符代替类型模式中的 `identifier`，差别仅在于不会进行绑定操作
>
>     关于类型模式用法的示例如下：
>
>     ```cangjie
>     // 第一种情况
>     main() {
>         try {
>             throw ArithmeticException()
>         } catch (e: Exception) {                                          // 捕获
>             print("Exception and its subtypes can be caught here")
>         }
>     }
>     ```
>
>     ```cangjie
>     // 第二种情况
>     // 用户定义的异常
>     open class Father <: Exception {
>         var father: Int64 = 0
>         func whatFather() {
>             print("I am Father")
>         }
>     }
>     class ChildOne <: Father {
>         var childOne: Int64 = 1
>         func whatChildOne() {
>             print("I am ChildOne")
>         }
>         func whatChild() {
>             print("I am method in ChildOne")
>         }
>     }
>     class ChildTwo <: Father {
>         var childTwo: Int64 = 2
>         func whatChildTwo() {
>             print("I am ChildTwo")
>         }
>         func whatChild() {
>             print("I am method in ChildTwo")
>         }
>     }
>
>     // main 函数
>     main() {
>         var a = 1
>         func throwE() {
>             if (a == 1) {
>                 ChildOne()
>             } else {
>                 ChildTwo()
>             }
>         }
>         try {
>             throwE()
>         } catch (e: ChildOne | ChildTwo) {
>             e.whatFather()                    // ok: e 是一个 Father 对象
>             //e.whatChildOne()                // error: e 是一个 Father 对象
>             //e.whatChild()                   // error: e 是一个 Father 对象
>             print(e.father)                   // ok: e 是一个 Father 对象
>             //print(e.childOne)               // error: e 是一个 Father 对象
>             //print(e.childOTwo)              // error: e 是一个 Father 对象
>         }
>
>         return 0
>     }
>     ```
>
> 使用 `finally` 块的例子如下：
>
> ```cangjie
> // 使用异常类型模式 捕获
> try {
>     throw IndexOutOfBoundsException()
> } catch (e: ArithmeticException | IndexOutOfBoundsException) {
>     print("exception info: " + e.toString())
> } catch (e: Exception) {
>     print("neither ArithmeticException nor IndexOutOfBoundsException, exception info: " + e.toString())
> } finally {
>     print("the finally block is executed")
> }
> ```

异常捕获, `catch`使用模式匹配, **总是匹配目标异常类及其子类**

但`catch`只会执行一个, `finally`如果存在总会被执行

一个`catch`可以匹配多个不同的异常类, 每个异常类使用`|`分隔, 但此时 **如果捕获到目标异常, 此异常将会被转换为 所有异常类的最小公共父类 然后再绑定到标识符**

使用时, 也只能访问最小公共父类的成员

猜测, 仓颉中所有异常类 均需要有一个祖先类: `Exception`

##### `Try`表达式的类型

> 类似于 `if` 表达式:
>
> - 如果 `try` 表达式的值 没有被读取或者返回，那么整个 `try` 表达式的类型为 `Unit`，`try` 块和 `catch` 块不要求存在公共父类型；否则，按如下规则检查;
>
> - 在上下文没有明确的类型要求时，要求 `try` 块和所有 `catch` 块（如果存在）拥有最小公共父类型
>
>     整个 `try` 表达式的类型就是该最小公共父类型;
>
> - 在上下文有明确的类型要求时，`try` 块和任一 `catch` 块（如果存在）的类型都必须是上下文所要求的类型的子类型，但此时不要求它们拥有最小公共父类型
>
>
> 需要注意的是，虽然 `finally` 块会在 `try` 块和 `catch` 之后执行，但是它不会对整个 `try` 表达式的类型产生影响，并且 `finally` 块的类型始终是 `Unit`（即使 `finally` 块内表达式的类型不是 `Unit`）

首先, `finally`表达式的类型恒为`Unit`, 且不会对整个`try`表达式的类型有任何影响

`try`表达式的值 如果没有被读取或返回, 就是`Unit`类型

否则, 需要根据上下文进行推断, 如果上下文中指定了类型, 那么`try`和`catch`的类型必须是指定类型的子类型, 如果没有指定 那么`try`和`catch`必须存在最小公共父类型

和`if`表达式的类型和值是类似的

##### `Try`表达式的求值顺序

> 关于 `try {e1} catch (catchPattern) {e2} finally {e3}` 表达式执行规则的额外规定：
>
> - 若进入 `finally` 块前执行到 `return e` 表达式，则会将 `e` 求值至 `v`，然后立刻执行 `finally block`；若进入 `finally` 块前执行到 `break` 或 `continue` 表达式，则会立刻执行 `finally block`
>
>     - 若 `finally` 块中无 `return` 表达式，则处理完 `finally` 块后会将缓存的结果 `v` 返回（或抛出异常）
>
>         也就是说，即使 `finally` 块中有对上述 `e` 中引用变量的赋值，也不会影响已经求值的结果 `v`，举例如下：
>
>         ```cangjie
>         func f(): Int64 {
>             var x = 1;
>             try {
>                 return x + x;                                           // 返回 2
>             } catch (e: Exception) {                                    // 捕获
>                 print("Exception and its subtypes can be caught here")
>             } finally {
>                 x = 2;
>             }  // 返回值会是 2 而不是 4
>         }
>         ```
>
>     - 若在 `finally` 块中执行到 `return e2` 或 `throw e2`，则会对 `e2` 求值至结果 `v2`，并立即返回或抛出 `v2`；若在 `finally` 块中执行到 `break` 或 `continue`，则会终止 `finally` 的执行并立即跳出循环
>
>         举例如下：
>
>         ```cangjie
>         func f(): Int64 {
>             var x = 1;
>             try {
>                 return x + x;                                           // 返回 2
>             } catch (e: Exception) {                                    // 捕获
>                 print("Exception and its subtypes can be caught here")
>             } finally {
>                 x = 2;
>                 return x + x;                                           // 返回 4
>             } // 返回值会是 4 而不是 2
>         }
>         ```
>
> 总之，`finally` 块一定会被执行
>
> 如果 `finally` 块中有任何控制转移表达式，都会覆盖 进入`finally`之前的控制转移表达式
>
> `Try` 表达式中 `throw` 的处理更为复杂，具体请参考下一小节

仓颉中, `try`表达式的块内可以执行控制转移表达式: `return` `break` `continue`

在`try`表达式的块内, 如果执行了`return`, `try`表达式执行完之后 会**直接返回整个函数**

但返回值是多少, 则 根据规则计算

1. 如果在进入`finally`之前执行了`return`、`break`或`continue`

    则, 会直接进入`finally`, 去执行`finally`, 且会**隐含的记录`return`的表达式值 作为返回值**

2. 如果进入`finally`之后, 又执行了`return`、`break`或`continue`

    则 **新的`return`值会覆盖已经记录的值 作为返回值**, 或 结束循环 或 继续下一次循环

    如果没有执行`return`, 那么返回值就 依旧是之前记录的值

##### `Try`表达式处理异常的逻辑

> 关于 `try {e1} catch (catchPattern) {e2} finally {e3}` 表达式执行时抛出异常的规定：
>
> 1. 若执行 `e1` 的过程中没有抛出异常（这种情况下不会执行 `e2`）:
>
>     - 当执行 `e3` 的过程中亦无异常抛出，那么整个 `try` 表达式不会抛出异常;
>
>     - 当执行 `e3` 的过程中抛出异常 `E3`，那么整个 `try` 表达式抛出异常 `E3`;
>
> 2. 若执行 `e1` 的过程中抛出异常 `E1`，且执行 `e3` 的过程中抛出异常 `E3`，那么整个 `try` 表达式抛出异常 `E3`（无论 `E1` 是否被 `catch` 块所捕获）;
>
> 3. 若执行 `e1` 的过程中抛出异常 `E1` ，且执行 `e3` 的过程中无异常抛出，那么：
>
>     - 当 `E1` 可以被 `catch` 捕获且执行 `e2` 的过程中无异常抛出时，整个 `try` 表达式无异常抛出；
>
>     - 当 `E1` 可以被 `catch` 捕获且执行 `e2` 的过程中抛出异常 `E2` 时，整个 `try` 表达式抛出异常 `E2`；
>
>     - 当 `E1` 未能被 `catch` 捕获时，整个 `try` 表达式抛出异常 `E1`

如果`finally`抛出异常`E3`, 那么整个`try`表达式抛出异常`E3`

如果`finally`没有抛异常, 那么还要分情况:

1. 如果`try`抛出异常`E1`, 且没有`catch`捕获, 那么`try`表达式抛出异常`E1`

2. 如果`try`抛出异常`E1`, 被`catch`捕获了, 且执行时 异常被处理了 没有再抛出异常, 那么`try`表达式不抛异常

3. 如果`try`抛出异常`E1`, 被`catch`捕获了, 但执行时 又抛出了异常`E2`, 那么`try`表达式抛出异常`E2`

总之, 要看执行时 `try`中的异常是否被处理, 以及 执行到的最后抛出异常的语句 来结合判断`try`表达式的异常情况

#### `Try-With-Resources`表达式

> `Try-with-resources` 表达式主要是为了自动释放**非内存资源**
>
> 不同于普通 `try` 表达式，`try-with-resources` 表达式中的 `catch` 块和 `finally` 块均是可选的，并且 `try` 关键字其后的块之间可以**插入一个或者多个 `ResourceSpecification`**用来申请一系列的资源（`ResourceSpecification` 并不会影响整个 `try` 表达式的类型)
>
> 这里所讲的资源对应到语言层面即指 **对象**，因此 `ResourceSpecification` 其实就是实例化一系列的对象（多个实例化之间使用`,`分隔）

有些迷糊了, 什么是`ResourceSpecification`, 暂时没有具体的涉及到 仓颉中的相关内容

但, 非内存资源猜测可能是: 文件描述符? 网络套接字? 数据库句柄? 之类的

不过 **`finally`和`catch`都可以不存在**

> 使用 `try-with-resources` 表达式的例子如下所示：
>
> ```cangjie
> class MyResource <: Resource {
>     var flag = false
>     public func isClosed() { flag }
>     public func close() { flag = true }
>     public func hasNextLine() { false }
>     public func readLine() { "line" }
>     public func writeLine(_: String) {}
> }
>
> main() {
>     try (input = MyResource(),
>         output = MyResource()) {
>         while (input.hasNextLine()) {
>             let lineString = input.readLine()
>             output.writeLine(lineString)
>         }
>     } catch (e: Exception) {
>         print("Exception happened when executing the try-with-resources expression")
>     } finally {
>         print("end of the try-with-resources expression")
>     }
> }
> ```
>
> 在 `try-with-resources` 表达式中的 `ResourceSpecification` 的类型必须实现 `Resource` 接口：
>
> ```cangjie
> interface Resource {
>     func isClosed(): Bool
>     func close(): Unit
> }
> ```

看起来, 应该是在`try`中创建资源, 然后出`try`块之后, 会自动尝试调用`close()`接口自动释放资源?

主要的应该依靠 实现`Resource`接口

> `Try-with-resources` 表达式会首先（依声明顺序）执行实例化对应的一系列资源申请（上例中，先实例化 `input` 对象，再实例化 `output` 对象），在此过程中若某个资源申请失败（例如，`output` 实例化失败），则在它之前申请成功的资源（如 `input` 对象）会被全部释放（释放过程中若抛出异常，会被忽略），并抛出申请此资源（`output` 对象）失败的异常
>
> 如果所有资源均申请成功，则继续执行 `try` 之后紧跟的块
>
> 在执行块的过程中，无论是否发生异常，之后均会按照资源申请时的逆序依次自动释放资源（上例中，先释放 `output` 对象，再释放 `input` 对象）
>
> 在释放资源的过程中，若某个资源在被释放时发生异常，并**不会影响其它资源的继续释放**，并且整个 `try` 表达式抛出的异常遵循如下原则：
>
> 1. 如果 `try` 之后紧跟的块中有异常抛出，则 释放资源的过程中抛出的异常 会被**忽略**；
>
> 2. 如果 `try` 之后紧跟的块中没有抛出异常，释放资源的过程中抛出的首个异常将会被抛出（后续释放资源过程中抛出的异常均会被忽略）

资源会根据声明顺序进行申请, 如果存在一个资源申请失败, 则会抛出申请失败的异常 并 释放已经成功申请的资源(如果释放时抛异常, 此异常会被忽略)

资源释放时抛异常, 并不影响其他资源的释放

`Try-with-resources` 中, 如果`try`块抛异常, 则 实际会抛出第一个被抛出的异常, **其他异常会被忽略**(实际上 其他异常可能只有释放资源时抛出的异常)

但我有一点不明白, 为什么在执行块的**过程中**, 申请的资源就会被逆序自动释放? 而不是块执行完毕?

怎么想都应该是 块里执行完毕之后在释放比较合理

> 需要说明的是，`try-with-resources` 表达式中一般没有必要再包含 `catch` 块和 `finally` 块，也**不建议用户再手动释放资源**
>
> 因为 `try` 块执行的过程中 无论是否发生异常，所有申请的资源都会被自动释放，并且执行过程中产生的异常均会被向外抛出
>
> 但是，如果需要显式地捕获 `try` 块 或 资源申请和释放过程中可能抛出的异常 并处理，仍可在 `try-with-resources` 表达式中包含 `catch` 块和 `finally` 块：
>
> ```cangjie
> try (input = MyResource(),
>     output = MyResource()) {
>     while (input.hasNextLine()) {
>         let lineString = input.readLine()
>         output.writeLine(lineString)
>     }
> } catch (e: Exception) {
>     print("Exception happened when executing the try-with-resources expression")
> } finally {
>     print("end of the try-with-resources expression")
> }
> ```
>
> 事实上，上述 `try-with-resources` 表达式等价于下述普通 `try` 表达式：
>
> ```cangjie
> try {
>     var freshExc = None<Exception>                    // 一个可以存储任何异常的新变量
>     let input = MyResource()
>     try {
>         var freshExc = None<Exception>
>         let output = MyResource()
>         try {
>             while (input.hasNextLine()) {
>                 let lineString = input.readLine()
>                 output.writeLine(lineString)
>             }
>         } catch (e: Exception) {
>             freshExc = e
>         } finally {
>             try {
>                 if (!output.isClosed()) {
>                     output.close()
>                 }
>             } catch (e: Exception) {
>                 match (freshExc) {
>                     case Some(v) => throw v           // 用户代码中引发的异常将被抛出
>                     case None => throw e
>                 }
>             }
>             match (freshExc) {
>                 case Some(v) => throw v
>                 case None => ()
>             }
>         }
>     } catch (e: Exception) {
>         freshExc = e
>     } finally {
>         try {
>             if (!input.isClosed()) {
>                 input.close()
>             }
>         } catch (e: Exception) {
>             match (freshExc) {
>                 case Some(v) => throw v
>                 case None => throw e
>             }
>         }
>         match (freshExc) {
>             case Some(v) => throw v
>             case None => ()
>         }
>     }
> } catch (e: Exception) {
>     print("Exception happened when executing the try-with-resources expression")
> } finally {
>     print("end of the try-with-resources expression")
> }
> ```
>
> 可以看到，`try` 块（即用户代码）中 若抛出的异常会被记录在 `freshExc` 变量中，并最终被层层向外抛出，其优先级高于 释放资源的过程中可能出现的异常
>
> `try-with-resources` 表达式的类型是 `Unit`

这应该是个典型代码了, 可以认真分析一下

```
try {
    var freshExc = None<Exception>                  // 一个可以存储任何异常的新变量
    let input = MyResource()
    try {
        var freshExc = None<Exception>
        let output = MyResource()
        try {                                       // 用户代码
            while (input.hasNextLine()) {
                let lineString = input.readLine()
                output.writeLine(lineString)
            }
        } catch (e: Exception) {                    // 尝试捕获 用户代码异常
            freshExc = e                            // 捕获到了, 记录 用户代码异常
        } finally {
            try {
                if (!output.isClosed()) {           // 尝试释放资源
                    output.close()
                }
            } catch (e: Exception) {                // 尝试捕获 释放资源异常
                match (freshExc) {                  // 如果捕获到了, 就模式匹配 用户代码异常
                    case Some(v) => throw v         // 匹配到 用户代码异常存在, 则 抛出用户代码异常
                    case None => throw e            // 匹配到 None, 即 用户代码异常不存在, 则 抛出资源释放异常
                }
            }
            match (freshExc) {                      // 如果没有捕获到资源释放异常, 模式匹配 用户代码异常
                case Some(v) => throw v             // 匹配到 用户代码异常存在, 则 抛出用户代码异常
                case None => ()                     // 匹配到 None, 即 用户代码异常不存在, 模式匹配值为()
            }
        }
    } catch (e: Exception) {                        // 尝试捕获内层代码中的异常
        freshExc = e                                // 捕获到 则记录 内层异常
    } finally {
        try {
            if (!input.isClosed()) {                // 尝试释放资源
                input.close()
            }
        } catch (e: Exception) {                    // 尝试捕获 释放资源异常
            match (freshExc) {                      // 如果捕获到了释放资源异常, 模式匹配 内层异常
                case Some(v) => throw v             // 模式匹配 内层异常存在, 则抛出内层异常
                case None => throw e                // 模式匹配 None, 即内层异常不存在, 则抛出资源释放异常
            }
        }
        match (freshExc) {                          // 如果资源释放没有异常, 则模式匹配 内层异常
            case Some(v) => throw v                 // 模式匹配 内层确实抛出异常, 则 抛出内层异常
            case None => ()                         // 模式匹配 None, 则 内层异常不存在, 整个`try`表达式无异常
        }
    }
} catch (e: Exception) {
    print("Exception happened when executing the try-with-resources expression")
} finally {
    print("end of the try-with-resources expression")
}
```

其实也不是很复杂, 严格匹配异常并处理

### `Throw`表达式

> `throw` 表达式的语法定义为：
>
> ```
> throwExpression
>     : 'throw' expression
>     ;
> ```
>
> `Throw` 表达式由关键字 `throw` 和一条表达式组成，用于抛出异常
>
> `Throw` 表达式的类型是 `Nothing`
>
> 需要注意的是，关键字 `throw` 之后的表达式只能是一个**继承于 `Exception` 或 `Error` 的类型**的对象
>
> `Throw` 表达式会改变程序的执行逻辑：`throw` 表达式在执行时会抛出一个异常，**捕获此异常的代码块将被执行**，而非 `throw` 后剩余的表达式
>
> 使用 `throw` 表达式的例子如下：
>
> ```cangjie
> // 使用 异常类型模 捕获异常
> let listTest = [0, 1, 2]
> try {
>     throw ArithmeticException()
>     let temp = listTest[0] + 1                                  // 永远不会被执行
> } catch (e: ArithmeticException) {
>     print("an arithmeticException happened: " + e.toString())
> } finally {
>     print("the finally block is executed")
> }
> ```

`Throw`表达式, 使用`throw`关键字 抛出异常或`Error`

异常是继承于`Exception`的类对象, `Error`可能是另外一种相同的作用的类

执行过`throw`之后, 块中之后的代码就不会再执行了, 直到异常被捕获或`finally`

> 当 `throw` 表达式抛出一个异常后，**必须要能够将其捕获并处理**
>
> 搜寻异常捕获代码的顺序是函数调用链的逆序：当一个异常被抛出时，首先在抛出异常的函数内搜索匹配的 `catch` 块，如果未找到，则**终止此函数的执行**，并在调用这个函数的函数内继续寻找相匹配的 `catch` 块，若仍未找到，该函数同样需要终止，并继续搜索调用它的函数，**以此类推**，直到找到相匹配的 `catch` 块
>
> 但是，如果在调用链内的所有函数中**均未找到合适的 `catch` 块**，则程序跳转到 `Exception` 中 `terminate` 函数内执行，使得程序非正常退出
>
> 下面的例子展示了异常在不同位置被捕获的场景：
>
> ```cangjie
> // 被 catchE() 捕获
> func catchE() {
>     let listTest = [0, 1, 2]
>     try {
>         throwE()                                                           // 被 catchE() 捕获
>     } catch (e: IndexOutOfBoundsException) {
>         print("an IndexOutOfBoundsException happened: " + e.toString())
>     }
> }
>
> // Terminate 函数被执行
> func notCatchE() {
>     let listTest = [0, 1, 2]
>     throwE()
> }
>
> // 函数 throwE()
> func throwE() {
>     throw IndexOutOfBoundsException()
> }
> ```

当程序抛出异常之后, 会在函数内寻找匹配的`catch`, 如果找不到, 函数终止, 然后逐层出栈 直到找到匹配的`catch`, 或 直到始终找不到 执行`Exception.terminate()` 非正常退出程序
