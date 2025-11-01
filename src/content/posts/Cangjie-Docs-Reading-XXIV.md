---
title: "仓颉文档阅读-语言规约XV: 元编程"
published: 2025-10-20 15:22:21
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

## 元编程

> 元编程允许把代码表示成可操作的数据对象, 可以对其增、删、改、查
>
> 为此, 仓颉编程语言提供了编译标记用于元编程
>
> 编译标记分为两类: **宏和注解**
>
> 宏机制支持基于`Tokens`类型的**编译期代码变换**, 支持将代码转为数据、拼接代码等基础操作

关于元编程, 博主只简单地接触过`QT`的`moc`, 还有C/C++的宏

### `quote`表达式和`Tokens`类型

> 仓颉通过`quote`表达式引用具体代码, 表示成可操作的数据对象
>
> `quote`表达式的语法定义为:
>
> ```
> quoteExpression
>     : 'quote' quoteExpr
>     ;
>
> quoteExpr
>     : '(' quoteParameters ')'
>     ;
>
> quoteParameters
>     : (quoteToken | quoteInterpolate | macroExpression)+
>     ;
>
> quoteToken
>     : '.' | ',' | '(' | ')' | '[' | ']' | '{' | '}' | '**' | '*' | '%' | '/' | '+' | '-'
>     | '|>' | '~>'
>     | '++' | '--' | '&&' | '||' | '!' | '&' | '|' | '^' | '<<' | '>>' | ':' | ';'
>     | '=' | '+=' | '-=' | '*=' | '**=' | '/=' | '%='
>     | '&&=' | '||=' | '&=' | '|=' | '^=' | '<<=' | '>>='
>     | '->' | '=>' | '...' | '..=' | '..' | '#' | '@' | '?' | '<:' | '<' | '>' | '<=' | '>='
>     | '!=' | '==' | '_' | '\\' | '`' | '$'
>     | 'Int8' | 'Int16' | 'Int32' | 'Int64' | 'UInt8' | 'UInt16' | 'UInt32' | 'UInt64' | 'Float16'
>     | 'Float32' | 'Float64' | 'Rune' | 'Bool' | 'Unit' | 'Nothing' | 'struct' | 'enum' | 'This'
>     | 'package' | 'import' | 'class' | 'interface' |'func' | 'let' | 'var' | 'type'
>     | 'init' | 'this' | 'super' | 'if' | 'else' | 'case' | 'try' | 'catch' | 'finally'
>     | 'for' | 'do' | 'while' | 'throw' | 'return' | 'continue' | 'break' | 'as' | 'in' | '!in'
>     | 'match' | 'from' | 'where' | 'extend' | 'spawn' | 'synchronized' | 'macro' | 'quote' | 'true' | 'false'
>     | 'sealed' | 'static' | 'public' | 'private' | 'protected'
>     | 'override' | 'abstract' | 'open' | 'operator' | 'foreign'
>     | Identifier | DollarIdentifier
>     | literalConstant
>     ;
>
> quoteInterpolate
>     : '$' '(' expression ')'
>     ;
> ```
>
> `quote`表达式的语法规则总结如下:
>
> - 通过关键字`quote`定义`quote`表达式
>
> - `quote`表达式由`()`括起, 内部引用的可以是代码(`Token`)、代码插值、宏调用表达式
>
> - `quote`表达式为**`Tokens`类型**
>
>     `Tokens`是仓颉标准库提供的类型, 是由词法单元(`token`)组成的序列
>
> 上述语法定义中的`quoteToken`是指仓颉编译器支持的任意合法`token`, 注释、终结符等不被`Lexer`解析的`token`除外
>
> 但当换行符作为两条表达式分隔符时, 不会被忽略, 它会被解析成一个`quoteToken`
>
> 如下的例子, 只有两条赋值表达式之间的换行符会被解析成`quoteToken`, `quote`表达式里的其他换行符都会被忽略
>
> ```cangjie
> // 赋值表达式之间的换行字符被保留, 其他字符被忽略
> quote(
>
> var a = 3
> var b = 4
>
> )
> ```

从文档中看, `quote`表达式应该是这样的:

```cangjie
quote ( 任意的合法仓颉表达式(包括代码插值、宏调用表达式) 或 任意合法的Token )
```

关于这个合法的`Token`, 好像只要是文档中列出来的都可以:`quote (.)``quote (,)``quote ([)`等甚至都是合法的

具体要用到, 才更能深入了解

`quote`表达式就是`Tokens`类型

> 不允许未经过代码插值的连续`quote`, 关于代码插值, 详见下一节
>
> 当`quote`表达式引用如下`token`时, 需要进行转义:
>
> - `quote`表达式中不允许不匹配的小括号, 但是通过`\`转义的小括号, 不计入匹配规则
>
> - 当`@`表示普通`token`, 而**非宏调用表达式**时, 需要通过`\`进行转义
>
> - 当`$`表示普通`token`, 而**非代码插值**时, 需要通过`\`进行转义
>
> 注: 本章所提到的`token`(字母均小写), 指仓颉`Lexer`解析出来的词法单元
>
> 下面是一些以`Tokens`或代码插值为参数的`quote`表达式用例:
>
> ```cangjie
> let a0: Tokens = quote(==)                                      // ok
> let a1: Tokens = quote(2+3)                                     // ok
> let a2: Tokens = quote(2) + quote(+) + quote(3)                 // ok
> let a3: Tokens = quote(main(): Int64 {
>     0
> })                                                              // ok
> let a4: Tokens = quote(select * from Users where id=100086)     // ok
>
> let b0: Tokens = quote(()                                       // error: 不匹配的`(`(需要转义或配对)
> let b1: Tokens = quote(quote(x))                                // ok --`b1.size == 4`
> ```
>
> `quote`表达式还可以引用宏调用表达式, 例如:
>
> ```cangjie
> quote(@SayHi("say hi"))                     // 一个quoted, 未展开的宏 -- 宏展开会在稍后进行
> ```
>
> 仓颉编程语言提供`Tokens`类型表示代码从原始文本信息解析后的`token`序列
>
> `Tokens`支持通过`operator +`将两个对象的`token`结果拼接到新对象
>
> 下面例子里`a1`和`a2`基本等价
>
> ```cangjie
> let a1: Tokens = quote(2 + 3)                     // ok
> let a2: Tokens = quote(2) + quote(+) + quote(3)   // ok
> ```

不知道是否能把`quote`表达式, 看作可以在仓颉中用于拼接的代码单元?

#### 代码插值

> 在`quote`表达式里 使用`$`作为代码插值运算符, 运算符后面接表达式, 表示将**该表达式的值转成`tokens`**
>
> 该运算符为一元前缀运算符, **只能用在`quote`表达式中**, 优先级比其他运算符高
>
> 代码插值可作用于所有仓颉合法表达式
>
> 代码插值本身不是表达式, 不具备类型
>
> ```cangjie
> var rightOp: Tokens = quote(3)
> quote(2 + $rightOp)               // quote(2 + 3)
> ```
>
> 能被代码插值的表达式类型必须实现`ToTokens`接口
>
> `ToTokens`接口形式如下:
>
> ```cangjie
> interface ToTokens  {
>     func toTokens(): Tokens
> }
> ```
>
> - 仓颉中大部分的值类型: 数值类型、`Rune`类型、`Bool`类型、`String`类型已有默认实现
>
> - `Tokens`类型默认实现`ToTokens`接口, `toTokens`返回它自己
>
> - 用户自定义数据结构须主动实现`ToTokens`接口
>
>     仓颉标准库提供了丰富的生成`Tokens`的接口
>
>         例如, 对于`String`类型的变量`str`, 用户可以直接使用`str.toTokens()`获取`str`对应的`Tokens`

代码插值, 是在`quote`表达式里对表达式使用的, `$(expression)`表示将表达式的值转为`tokens`

要能使用`$`, 就必须实现`ToTokens`接口

#### `quote`表达式求值规则

> `quote`表达式对括号内的引用规则总结如下:
>
> - 代码插值: 取被插值的表达式`.toTokens()`结果
>
> - 普通`tokens`: 取对应的`Tokens`结果
>
> `quote`表达式根据以上情况再按照表达式出现的顺序拼接成更大的`Tokens`
>
> 下面是一些`quote`表达式求值示例, 注释里表示程序执行返回的`Tokens`结果
>
> ```cangjie
> var x: Int64 = 2 + 3
>
> // quote 中的 tokens 是从相应的 Tokens 中获得的
> quote(x + 2)                                  // quote(x + 2)
>
> // Tokens 类型只能与 Tokens 类型相加
> quote(x) + 1                                  // error! quote(x) 是 Tokens 类型, 不能与整型相加
>
> // quote 中代码插值的值 等于 插值表达式的值对应的 tokens 结果
> quote($x)                                     // quote(5)
> quote($x + 2)                                 // quote(5 + 2)
> quote($x + (2 + 3))                           // quote(5 + (2 + 3))
> quote(1 + ($x + 1) * 2)                       // quote(1 + (5 + 1) * 2)
> quote(1 + $(x + 1) * 2)                       // quote(1 + 6 * 2)
>
> var t: Tokens = quote(x)                      // quote(x)
>
> // 不使用插值, `t`是token`t`
> quote(t)                                      // quote(t)
>
> // 使用插值时, `t`将被求值, 并期望实现 ToTokens 接口
> quote($t)                                     // quote(x)
> quote($t+1)                                   // quote(x+1)
>
> // quote 表达式可以在插值内部使用, 并取消
> quote($(quote(t)))                            // quote(t)
> quote($(quote($t)))                           // quote(x)
>
> quote($(t+1))                                 // error! t 是 Tokens 类型, 不能与整型相加
>
> public macro PlusOne(input: Tokens): Tokens {
>     quote(($input + 1))
> }
>
> // 宏调用被保留, 未展开, 直到宏展开器将其重新插入代码中
> // 然而, 插值仍然发生, 包括在宏的参数之间
> quote(@PlusOne(x))                            // quote(@PlusOne(x))
> quote(@PlusOne($x))                           // quote(@PlusOne(5))
> quote(@PlusOne(2+3))                          // quote(@PlusOne(2+3))
> quote(1 + @PlusOne(x) * 2)                    // quote(1 + @PlusOne(x) * 2)
>
> // 当宏调用在quote之外时, 宏扩展会提前发生
> var y: Int64 = @PlusOne(x)                    // 5 + 1
> quote(1 + $y * 2 )                            // quote(1 + 6 * 2)
> ```

### 宏

> 宏是实现元编程的重要技术
>
> 宏和函数的相同点在于都可以被调用, 具有输入和输出
>
> 不同点在于宏代码**在编译期进行展开**, 仓颉将宏调用表达式替换为展开后的代码
>
> 用户可通过宏编写代码模板(即生成代码的代码)
>
> 另外, 宏提供自定义文法的能力
>
> 用户可基于宏灵活定义自己的`DSL`文法
>
> 最后, 仓颉提供丰富的代码操作接口供用户方便地完成代码变换
>
> 宏分为**无属性宏**和**有属性宏两类**
>
> 相对于无属性宏, 有属性宏可根据属性不同对宏的输入进行相应的分析变换得到不同的输出结果

仓颉的宏和C/C++的宏, 思想是相似的, 即 在编译期对代码进行展开、替换

但, C/C++的宏是纯文本的替换, 不知道仓颉具体是如何实现的, 但猜测与上面的`Tokens`相关

#### 宏定义

> 仓颉编程语言中, 宏定义遵循以下语法规则:
>
> ```
> macroDefinition
>     : 'public' 'macro' identifier
>     (macroWithoutAttrParam | macroWithAttrParam)
>     (':' identifier)?
>     ('=' expression | block)
>     ;
>
> macroWithoutAttrParam
>     : '(' macroInputDecl ')'
>     ;
>
> macroWithAttrParam
>     : '(' macroAttrDecl ',' macroInputDecl ')'
>     ;
>
> macroInputDecl
>     : identifier ':' identifier
>     ;
>
> macroAttrDecl
>     : identifier ':' identifier
>     ;
> ```
>
> 总结如下:
>
> - 通过关键字`macro`定义一个宏
>
> - `macro`前需要有`public`修饰, 表示对包外部可见
>
> - `macro`后需要带有宏的名字
>
> - 宏的参数由`()`括起
>
>     无属性宏固定 1 个`Tokens`类型参数对应宏的输入, 有属性宏固定 2 个`Tokens`类型依次对应宏的属性和输入
>
> - 可缺省的函数返回类型固定为`Tokens`
>
> 下面是无属性宏的示例
>
> 它包含了宏定义具备的所有要素:`public`表示对包外部可见;`macro`关键字;`foo`是宏的标识符; 形参`x`和它的类型`Tokens`; 返回值和输入同值、同类型
>
> ```cangjie
> public macro foo(x: Tokens): Tokens { x }
>
> public macro bar(x: Tokens): Tokens {
>     return quote($x)                      // 或者只`return x`
> }
> ```
>
> 下面是有属性宏的示例
>
> 和无属性宏相比, 多一个`Tokens`类型的输入, 宏定义体内可以进行更灵活的操作
>
> ```cangjie
> public macro foo(attr: Tokens, x: Tokens): Tokens { attr + x }
>
> public macro bar(attr: Tokens, x: Tokens): Tokens {
>     return quote($attr + $x)
> }
> ```
> 宏一旦被定义, **宏名字不能再被赋值**
>
> 另外, 宏对参数类型、参数个数有严格要求

仓颉中的宏定义语法, 更类似函数的定义

但宏必须用`public`修饰, `macro`是定义宏的关键字

仓颉中的宏, 至少有一个参数, 还缺省一个输出返回值, 类型均恒为`Tokens`

仓颉宏的替换结果, 应该就是宏调用的返回值

#### 宏调用

> 宏调用表达式遵循以下语法规则:
>
> ```
> macroExpression
>     : '@' identifier macroAttrExpr?
>     (macroInputExprWithoutParens | macroInputExprWithParens)
>     ;
>
> macroAttrExpr
>     : '[' quoteToken* ']'
>     ;
>
> macroInputExprWithoutParens
>     : functionDefinition
>     | operatorFunctionDefinition
>     | staticInit
>     | structDefinition
>     | structPrimaryInit
>     | structInit
>     | enumDefinition
>     | caseBody
>     | classDefinition
>     | classPrimaryInit
>     | classInit
>     | interfaceDefinition
>     | variableDeclaration
>     | propertyDefinition
>     | extendDefinition
>     | macroExpression
>     ;
>
> macroInputExprWithParens
>     : '(' macroTokens ')'
>     ;
>
> macroTokens
>     : (quoteToken | macroExpression)*
>     ;
> ```
>
> 宏调用表达式规则总结如下:
>
> - 通过关键字`@`定义宏调用表达式
>
> - 宏调用使用`()`括起来
>
>     括号里面可以是任意合法`tokens`, 不可以是空
>
> - 调用有属性宏时, 宏属性使用`[]`括起来
>
>     里面可以是任意合法`tokens`, 不可以是空
>
> 当宏调用表达式引用如下`token`时, 需要转义:
>
> - 宏调用表达式参数中不允许不匹配的小括号, 例如`@ABC(we have two (( and one ))`
>
>     但通过`\`转义的小括号, 不计入匹配规则
>
> - 有属性宏的中括号内不允许不匹配的中括号, 例如`@ABC[we have two [[ and one ]]()`
>
>     但通过`\`进行转义的中括号, 不计入匹配规则
>
> - 宏调用表达式参数中引用`@`时, 需要通过`\`转义
>
> 宏调用用于某些声明或表达式前面的时候可省略括号, 其语义和不带括号的宏调用有所不同
>
> 带括号的宏调用, 其参数可以是任意合法`tokens`; 不带括号的宏调用, 其参数必须是以下声明或表达式中的一种
>
> - 函数声明
>
> - `struct`声明
>
> - `enum`声明
>
> - `enum constructor`
>
> - 类声明
>
> - 静态初始化器
>
> - 类和`struct`构造函数和主构造函数
>
> - 接口声明
>
> - 变量声明
>
> - 属性声明
>
> - 扩展声明
>
> - 宏调用表达式
>
> 另外, 对于不带括号的宏调用, 其参数还必须满足:
>
> - 如果参数是声明, 那么该宏调用只能出现在该声明允许出现的位置
>
> 下面是无属性宏、有属性宏、不带括号的宏调用的示例
>
> ```cangjie
> func foo() {
>     print("In foo\n")
> }
>
> // 无属性宏
> public macro Twice(input: Tokens): Tokens {
>     print("Compiling the macro`Twice`...\n")
>     quote($input; $input)
> }
>
> @Twice(foo())                         // 宏展开后: foo(); foo()
> @Twice()                              // error, 宏调用中的参数不能为空
>
> // 带属性宏
> public macro Joint(attr: Tokens, input: Tokens): Tokens {
>     print("Compiling the macro`Joint`...\n")
>     quote($attr; $input)
> }
>
> @Joint[foo()](foo())                  // 宏展开后: foo(); foo()
> @Joint[foo()]()                       // error, 宏调用中的参数不能为空
> @Joint[](foo())                       // error, 宏调用中的属性不能为空
>
> // 无属性宏
> public macro MacroWithoutParens(input: Tokens): Tokens {
>     print("Compiling the macro`MacroWithoutParens`...\n")
>     quote(func foo() { $input })
> }
>
> @MacroWithoutParens
> var a: Int64 = 0                      // 宏展开后: func foo() { var a: Int64 = 0 }
>
> public macro echo(input: Tokens) {
>     return input
> }
>
> @echo class A {}                      // ok, class 只能在顶层中定义, 当前宏调用也是如此
>
> func goo() {
>     @echo func tmp() {}               // ok, 函数可以在另一个函数体内定义, 当前宏调用也是如此
>     @echo class B {}                  // error, class 只能在顶层中定义, 当前宏调用也是如此
> }
> ```

仓颉中宏调用的语法稍微比宏定义要复杂一些

宏调用:

```cangjie
// 带()的调用
@宏标识符[属性参数](输入参数)

// 不带()的调用
@宏标识符[属性参数]
输入参数
```

不带`()`的调用, 对参数还有一些额外的限制, 即 限制只能是声明, 具体见上面的描述

但两种调用都有一点: 输入参数不能为空, `[属性参数]`可以缺省(具体由定义决定)

属性不允许匹配`[]`

参数不允许匹配`()`


> 宏调用作为一种表达式, 它可以出现在任意表达式允许出现的位置上
>
> 另外宏调用还可以出现在如下声明可以出现的位置: 函数声明、`struct`声明、`enum`声明、类声明、接口声明、变量声明(函数参数除外)、属性声明、扩展声明
>
> **宏只在编译期可见**
>
> 宏调用表达式在编译期会**被宏展开后的代码替换**
>
> **宏展开**是指 在代码编译时, 宏定义会被执行, 执行后的结果会被解析成仓颉语法树, 替换宏调用表达式
>
> 替换后的节点经过语义分析之后拥有相应的类型信息, 该类型可以认为是宏调用表达式的类型
>
> 例如上述例子中, 当用户执行`@Twice`, `@Joint`, `@MacroWithoutParens`这些宏调用的时候, 宏定义中的代码会被编译并执行, 打印出`Compiling the macro ...`等字样, 执行后的结果作为新的语法树节点, 替换掉原来的宏调用表达式
>
> 当宏调用出现在不同的上下文时, 需要满足以下规则:
>
> 1. 如果宏调用出现在期望表达式或者声明的上下文里, 那么宏调用会在编译期被展开成程序文本
>
> 2. 如果宏调用出现在期望`token`序列的上下文里(作为宏调用表达式或`quote`表达式的参数), 那么宏调用会在该`token`序列被求值的时刻调用, 而且宏调用的返回值(`token`序列)会被直接使用而不展开成程序文本

仓颉中宏的替换, 与C/C++中不同

C/C++中的宏, 就只是文本替换

但仓颉中的宏, 会根据调用时上下文的期望, 做出两种不同的替换

1. 出现在期望 **表达式或者声明** 的上下文里

    宏调用会在编译期被展开成程序文本, 即 编译期 进行文本替换

2. 出现在期望 **`token`序列** 的上下文里(宏调用表达式的参数、`quote`表达式的参数)

    宏调用会在该`token`序列被求值的时刻调用, 而且宏调用的返回值(`token`序列)会被直接使用而不展开成程序文本

因为暂时没有用过, 所以没有深入理解 这两种具体会有什么差别

#### 宏作用域和包的导入

> 宏必须在源文件顶层定义, 作用域是整个`package`
>
> 宏定义所在的`package`, 需使用`macro package`来声明, 被`macro package`限定的包, 仅允许宏定义声明对外可见, 其他声明仅包内可见, **其他声明被修饰为对外可见时将报错**
>
> ```cangjie
> //define.cj
> macro package define                      // 使用 macro 修饰包
> import std.ast.*
>
> // public func A(){}                      // error: func A 不能被`public`修饰
> public macro M(input:Tokens): Tokens {    // 只有宏可以被`public`修饰
>    return input
> }
>
> // call.cj
> package call
> import define.*
> main(){
>    @M()
>    return 0
> }
> ```

宏定义所在的包, 声明时要使用`macro`修饰, 且包内只有宏定义可以使用`public`修饰

> 宏的导入遵守仓颉通用的包导入规则
>
> 需要特殊说明的是, `macro package`仅允许被`macro package`重导出
>
> ```cangjie
> // A.cj
> macro package A
>
> public macro M1(input:Tokens): Tokens {
>     return input
> }
>
> // B.cj
> package B
> //public import A.*                   // error: 宏包只能通过`public import`在宏包中导入
>
> public func F1(input:Int64): Int64 {
>     return input
> }
>
> // C.cj
> macro package
> public import A.*                     // 只有宏包可以 public import 到宏包中
> // public import B.*                  // error: 普通包不能在宏包中 public import
> import B.*
>
> public macro M2(input:Tokens): Tokens {
>     return @M1(input) + Token(TokenKind.ADD) + quote($(F1(1)))
> }
> ```
>
> 在同一个`package`中, 宏对于同名的约定 和函数一致, 属性宏和非属性宏的规则如下表所示:
>
> | Same name, Same package | attribute macros | non-attribute macros |
> | ----------------------- | ---------------- | -------------------- |
> | `attribute macros` | `NO` | `YES` |
> | `non-attribute macros` | `YES` | `NO` |

宏包, 只能在宏包中`public import`导入, 能够重导出

且, 在宏包中, 只有宏包能`public import`导入

且, 同一个包中, 不允许出现同名的无属性宏 和 同名的带属性宏

#### 嵌套宏和递归宏

> 宏允许嵌套调用其他宏
>
> 宏调用表达式包含宏调用时, 如`@Outer @Inner(2+3)`, **优先执行内层的宏, 再执行外层的宏**
>
> 内层宏返回的`Tokens`结果和其他`Tokens`拼接一起交给外层宏调用
>
> 内层宏和外层宏可以是不同的宏, 也可以是同一个宏
>
> 内层宏可以调用库函数`AssertParentContext`来保证 内层宏调用 一定嵌套在特定的外层宏调用中
>
> 如果内层宏调用这个函数时 没有嵌套在给定的外层宏调用中, 该函数将抛出一个错误
>
> 第二个函数`InsideParentContext`只在内层宏调用 嵌套在给定的外层宏调用中时 返回`true`
>
> ```cangjie
> public macro Inner(input: Tokens): Tokens {
>     AssertParentContext("Outer")
>     // ...or...
>     if (InsideParentContext("Outer")) {
>         // ...
>     }
> }
> ```
>
> 内层宏也可以通过发送键/值对的方式**与外层宏通信**
>
> 当内层宏执行时, 通过调用标准库函数`SetItem`发送信息; 随后, 当外层宏执行时, 调用标准库函数`GetChildMessages`接收每一个内层宏发送的信息(一组键/值对映射)
>
> ```cangjie
> public macro Inner(input: Tokens): Tokens {
>     AssertParentContext("Outer")
>     SetItem("key1", "value1")
>     SetItem("key2", "value2")
>     // ...
> }
>
> public macro Outer(input: Tokens): Tokens {
>     let messages = GetChildMessages("Inner")
>     for (m in messages) {
>         let value1 = m.getString("key1")
>         let value2 = m.getString("key2")
>         // ...
>     }
> }
> ```
>
> 宏定义体包含宏调用时, 如果宏调用出现在**期望`token`序列的上下文**里, 则两个宏可以是不同的宏或**同一个宏(即支持递归)**, 否则, 被嵌套调用的宏不能和调用的宏相同
>
> 具体的使用可以参考下面两个例子
>
> 下面例子里, 嵌套调用的宏出现在`quote`表达式, 则支持递归调用:
>
> ```cangjie
> public macro A(input: Tokens): Tokens {
>     print("Compiling the macro A ...\n")
>     let tmp = A_part_0(input)
>     if cond {
>         return quote($tmp)
>     }
>     let bb: Tokens = quote(@A(quote($tmp)))  // ok
>     A_part_1()
> }
>
> main():Int64 {
>     var res: Int64 = @A(2+3)  // ok, @A will be treated as Int64 after macro expand
>     return res
> }
> ```
>
> 在这个例子中, 当宏`A`未被外部调用时, 宏`A`不会被执行(即使内部调用了自己), 即不会打印`Compiling the macro A ...`
>
> `if cond`是递归的终止条件
>
> 注意: 宏递归和函数递归有类似的约束, 需要有终止条件, 否则会陷入死循环, 导致编译无法停止
>
> 下面例子里, 嵌套调用的宏出现的地方不在`quote`表达式, 则不支持递归调用:
>
> ```cangjie
> public macro A(input: Tokens): Tokens {
>     let tmp = A_part_0(input)
>     if cond {
>         return quote($tmp)
>     }
>     let bb: Tokens = @A(quote($tmp))  // error, recursive macro expression not in quote
>     A_part_1()
> }
>
> main():Int64 {
>     var res: Int64 = @A(2+3)  // error, type mismatch
>     return res
> }
> ```
>
> 总结下宏嵌套调用和递归调用规则:
>
> - 宏调用表达式允许嵌套调用
>
> - 宏定义允许嵌套调用其他宏, 但只允许在`quote`表达式中递归调用自己

仓颉中的宏, 允许嵌套调用, 即可以调用时 再调用其他宏

且, 如果调用上下文是`quote`表达式中时, 还可以自己调用自己(递归调用)

并且, 可以在宏定义时, 通过函数限制宏只能 作为指定某个宏的内层

仓颉宏嵌套调用时, 内层宏可调用`SetItem("key", "value")`与外层宏进行通信

内层宏调用完成之后, 外层宏可以调用`GetChildMessages("宏标识符")`一次性取出内层宏发送的所有消息

#### 限制

> - 宏有条件地支持递归调用自己
>
>     具体情况请参考前面小节说明
>
> - 除了宏递归调用外, 宏定义和宏调用**必须位于不同的`package`**
>
>     宏调用的地方必须`import`宏定义所在的`package`, 保证宏定义比宏调用点先编译
>
>     不支持宏的循环依赖导入
>
>     例如下面的用法是非法的:`pkgA`导入`pkgB`, `pkgB`又导入`pkgA`, 存在循环依赖导入
>
>     ```cangjie
>     // ======= file A.cj
>     macro package pkgA
>     import pkgB.*
>     public macro A(..) {
>         @B(..)                  // error
>     }
>
>     // ======= file B.cj
>     macro package pkgB
>     import pkgA.*
>     public macro B(..) {
>         @A(..)                  // error
>     }
>     ```
>
> - 宏允许嵌套调用其他宏
>
>     被调用的宏也要求定义点和调用点必须位于不同的`package`

仓颉中, 宏的调用位置 和 定义位置必须位于不同的包

仓颉宏禁止循环依赖

#### 内置编译标记**

##### 源码位置

> 仓颉提供了几个内置编译标记, 用于在编译时获取源代码的位置
>
> - `@sourcePackage()`展开后是一个`String`类型的字面量, 内容为当前宏所在的源码的**包名**
>
> - `@sourceFile()`展开后是一个`String`类型的字面量, 内容为当前宏所在的源码的**文件名**
>
> - `@sourceLine()`展开后是一个`Int64`类型的字面量, 内容为当前宏所在的源码的**代码行**
>
> 这几个编译标记可以在任意表达式内部使用, 只要能符合类型检查规则即可
>
> 示例如下:
>
> ```cangjie
> func test1() {
>     let s: String = @sourceFile()                         //`s`的值是当前源文件名
> }
>
> func test2(n!: Int64 = @sourceLine()) { /* 在第五行 */
>     //`n`的默认值是`test2`定义的源文件行号
>     println(n)                                            // 打印 5
> }
> ```

这三个宏的作用再明显不过了, 最常在日志、调试使用

##### `@Intrinsic`


> `@Intrinsic`标记可用于修饰全局函数
>
> `@Intrinsic`修饰的函数是**由编译器提供的特殊函数**
>
> - `@Intrinsic`修饰的函数不允许写函数体
>
> - `@Intrinsic`修饰的函数, 是编译器决定的一份名单, 跟随标准库发布
>
>     名单之外的任何其它函数用`@Intrinsic`修饰都会报错
>
> - `@Intrinsic`标记只在`std module`内可见, `std module`外用户可以定义自己的叫做`Intrinsic`的宏或者注解, 不会发生名字冲突
>
> - `@Intrinsic`修饰的函数, 不允许作为一等公民使用
>
> 示例代码如下:
> ```cangjie
> @Intrinsic
> func invokeGC(heavy: Bool): Unit
>
> public func GC(heavy!: Bool = false): Unit {
>     unsafe { return invokeGC (heavy) }                // CJ_MCC_InvokeGC(heavy)
> }
> ```

`@Intrinsic`修饰的函数是由编译器提供的, 是跟随标准库发布的一个名单, 用户不允许使用`@Intinsic`修饰函数

不过`Intrinsic`这个词是可以使用的, 因为`@Intrinsic`只在`std module`内可见

##### `@FastNative`

> 为了提升与 C 语言互操作的性能, 仓颉提供`@FastNative`用于优化对 C 函数的调用
>
> 值得注意的是`@FastNative`只能用于`foreign`声明的函数
>
> 开发者在使用`@FastNative`修饰`foreign`函数时, 应确保对应的 C 函数满足以下两点要求:
>
> - 函数的整体执行时间不宜太长
>
>     例如: 不允许函数内部存在很大的循环; 不允许函数内部产生阻塞行为, 如, 调用`sleep`、`wait`等函数
>
> - 函数内部不能调用仓颉方法

`@FastNative`是用来修饰C函数的, 并且 是纯粹的C函数, 内部不允许调用仓颉函数

并且, 要被`@FastNative`修饰, C函数内部不允许存在大循环 和 可能产生的阻塞行为

##### 条件编译

> 条件编译是一种在程序代码中根据特定条件选择性地编译不同代码段的技术
>
> 条件编译的作用主要体现在以下几个方面:
>
> - 平台适应: 支持根据当前的编译环境选择性地编译代码, 用于实现跨平台的兼容性
>
> - 功能选择: 支持根据不同的需求选择性地启用或禁用某些功能, 用于实现功能的灵活配置
>
>     例如, 选择性地编译包含或排除某些功能的代码
>
> - 调试支持: 支持调试模式下编译相关代码, 用于提高程序的性能和安全性
>
>     例如, 在调试模式下编译调试信息或记录日志相关的代码, 而在发布版本中将其排除
>
> - 性能优化: 支持根据预定义的条件选择性地编译代码, 用于提高程序的性能
>
> 条件编译遵循以下语法规则:
>
> `{{conditionalCompilationDefinition|pretty}}`
>
> 条件编译语法规则总结如下:
>
> - 内置标记`@When`只能用于声明节点和导入节点
>
> - 编译条件使用`[]`括起来, `[]`内支持输入一组或多组编译条件(请参见[多条件编译])
>
> `@When[...]`作为一种内置编译标记, 在导入前处理, 由宏展开生成的代码中含有`@When[...]`会编译报错

仓颉提供`@When[...]`实现条件编译?

文档中没有说 是否还有其他标记

###### 编译条件

编译条件主要由关系表达式或逻辑表达式组成, 编译器根据编译条件评估**获取一个布尔值**, 从而决定编译哪段代码

条件变量是编译器计算编译条件的基准, 根据条件变量是否由编译器提供可将条件变量分为**内置条件变量**和**用户自定义条件变量**

同时, 条件变量的作用域仅限于`@When`的`[]`内, 其他作用域内使用 未定义的条件变量标识符 会触发未定义错误

###### 编译器内置条件变量

> 编译器为条件编译提供了**五个内置条件变量:`os`、`backend`、`cjc_version`、`debug`和`test`** 用于获取当前构建环境中对应的值, 内置条件变量支持比较运算和逻辑运算
>
> 其中, `os`、`backend`和`cjc_version`三个变量支持比较运算, 在比较运算中条件变量只能作为二元操作符的左操作数, 二元操作符的右操作数必须是一个`String`类型的字面量值; 逻辑运算仅适用于条件变量`debug`和`test`
>
> - `os`表示当前编译环境所在的操作系统
>
>     它用于实时获取编译器所在的具体操作系统类型, 进而与目标操作系统的字面量值构成一个编译条件
>
>     如果想为`Windows`操作系统生成代码, 可以将变量`os`与字面量值`"Windows"`进行比较判断
>
>     类似地, 如果想为`Linux`生成代码, 可以将`os`与字面量值`"Linux"`判等
>
>     ```cangjie
>     @When[os == "Linux"]
>     func foo() {
>         print("Linux")
>     }
>
>     @When[os == "Windows"]
>     func foo() {
>         print("Windows")
>     }
>
>     main() {
>         foo()                       // 编译并运行代码将在 Linux 或 Windows 上打印"Linux"或"Windows"
>         return 0
>     }
>     ```
>
> - `backend`表示当前编译器所使用的后端
>
>     它用于实时获取编译器当前所使用的后端类型, 进而与目标后端的字面量值构成一个编译条件
>
>     如果想为`cjnative`后端编译代码, 可以将`backend`与字面量值`"cjnative"`进行比较判断
>
>     支持的后端有:`cjnative`、`cjnative-x86`、`cjnative-x86_64`、`cjnative-arm`、`cjnative-aarch64`、`cjvm`、`cjvm-x86`、`cjvm-x86_64`、`cjvm-arm`、`cjvm-aarch64`
>
>     ```cangjie
>     @When[backend == "cjnative"]
>     func foo() {
>         print("cjnative backend")
>     }
>
>     @When[backend == "cjvm"]
>     func foo() {
>         print("cjvm backend")
>     }
>
>     main() {
>         foo()                       // 使用 cjnative 和 cjvm 后端版本编译并执行, 分别打印"llvm 后端"和"cjvm 后端"
>     }
>     ```
>
> - `cjc_version`表示当前编译器的版本号
>
>     版本号是一个`String`类型的字面量值, 采用`X.Y.Z`格式, 其中`X`、`Y`和`Z`为非负的整数且不允许包含前导零, 例如`"0.18.8"`
>
>     它用于实时获取编译器当前的版本号, 进而与目标版本号比较, 确保编译器能够基于特定的版本编译代码
>
>     共支持六种类型`==`、`!=`、`>`、`<`、`>=`、`<=`的比较操作
>
>     条件的结果由从左到右依次比较这些字段时的第一个差异决定
>
>     例如, `0.18.8 < 0.18.11`, `0.18.8 == 0.18.8`
>
> - `debug`是一个条件编译标识符, 表示当前编译的代码是否是一个调试版本
>
>     用于在编译代码时进行调试和发布版本之间的切换
>
>     使用`@When[debug]`修饰的代码只会在调试版本中编译; 使用`@When[!debug]`修饰的代码只会在发布版本中编译
>
> - `test`是一个条件编译标识符, 用于标记测试代码
>
>     测试代码通常与普通源码位于相同的文件中, 使用`@When[test]`修饰的代码, 只会在使用`--test`选项时才会被编译, 正常的构建时该部分代码将会被排除

仓颉编译器内置的一些条件变量, 还是比较经常使用的: 选择系统平台, 调试模式, 测试代码 等

###### 用户自定义条件变量

> 用户自定义条件变量是用户根据自己的需要定义条件变量, 进而在代码中使用这些条件变量来控制代码的编译
>
> 用户自定义条件变量与内置变量**本质上相似**, 唯一不同点是用户自定义条件变量的值由用户自身配置设定, 而内置变量的值由编译器根据当前编译环境决定
>
> 用户自定义条件变量遵循如下规则:
>
> - 自定义条件变量必须是一个合法的[标识符]
>
> - 自定义条件变量仅支持等于和不等于两种关系运算符的比较运算
>
> - 用户自定义的编译条件变量**必须是`String`类型**
>
> 一个用户自定义条件变量的示例如下:
>
> ```cangjie
> //source.cj
> @When[feature == "lion"]              // "feature"是用户自定义的条件变量
> func foo() {
>     print("feature lion, ")
> }
>
> @When[platform == "dsp"]              // "platform"是用户自定义的条件变量
> func fee() {
>     println("platform dsp")
> }
>
> main() {
>     foo()
>     fee()
> }
> ```

用户条件变量, 应该是在编译时 添加编译选项, 然后实现控制条件编译

###### 配置自定义条件变量

> 配置自定义条件变量的方式有两种: **编译选项和配置文件**
>
> 编译选项`--cfg`允许接收字符串用于配置自定义条件变量的值, 具体规则如下:
>
> - 允许多次使用`--cfg`
>
> - 允许使用多个条件赋值语句, 用逗号`(,)`分隔
>
> - 不允许多次指定条件变量的值
>
> - `=`操作符的左边必须是一个合法的标识符
>
> - `=`操作符的右边是一个字面量
>
> 使用`--cfg`对用户自定义条件变量`feature`和`platform`进行配置
>
> ```bash
> cjc --cfg "feature = lion, platform = dsp" source.cj              // ok
> ```
>
> ```bash
> cjc --cfg "feature = lion" --cfg "platform = dsp" source.cj       // ok
> ```
>
> ```bash
> cjc --cfg "feature = lion, feature = dsp" source.cj               // error
> ```
>
> 采用`.toml`文件格式作为用户自定条件变量的配置文件, 配置文件命名为`cfg.toml`
>
> - 采用键值对的方式对自定义条件变量配置, 键值对由`=`分隔, 每个键值对单独占一行
>
> - 健名是一个合法的[标识符]
>
> - 键值是一个双引号括起来的字面量值
>
> 默认以当前目录作为条件编译配置文件的搜索路径, 支持使用`--cfg`设置配置文件搜索路径
>
> ```bash
> cjc --cfg "/home/cangjie/conditon/compile" source.cj              // ok
> ```

仓颉用户自定义的条件变量, 需要在进行编译时, 使用`--cfg cfg_file_path`或`--cfg "条件变量标识符 = 值, 条件变量标识符 = 值, ..."`添加条件编译选项

##### 多条件编译

> 允许使用逻辑与`(&&)`、逻辑或`(||)`组合多个编译条件, 其优先级和结合性与逻辑表达式一致(请参见[逻辑表达式])
>
> 允许使用圆括号将编译条件括起来, 圆括号括起来的编译条件被视作一个单独的计算单元被优先计算
>
> ```cangjie
> @When[(backend == "cjnative" || os == "Linux") && cjc_version >= "0.40.1"]
> func foo() {
>     print("This is Multi-conditional compilation")
> }
>
> main() {
>     foo()                 // 编译源代码的条件: cjc_version 大于 0.40.1; 编译器后端是 cjnative 或操作系统是 Linux
>     return 0
> }
> ```

`@When[]`中的条件变量, 可以使用`&&`和`||`连接多个, 实现多条件编译
