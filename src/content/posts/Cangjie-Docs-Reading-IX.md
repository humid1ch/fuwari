---
title: "仓颉文档阅读-语言规约IV: 表达式(IV)"
published: 2025-09-29 13:42:13
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

## 表达式

> 表达式通常由一个或多个操作数(`operand`)构成, 多个操作数之间由操作符(`operator`)连接, 每个表达式都有一个类型, 计算表达式值的过程称为对表达式的求值(`evaluation`)
>
> 在仓颉编程语言中, 表达式几乎无处不在, 有表示各种计算的表达式(如算术表达式、逻辑表达式等), 也有表示分支和循环的表达式(如`if`表达式、循环表达式等)
>
> 对于包含多个操作符的表达式, 必须明确每个操作符的优先级、结合性以及操作数的求值顺序
>
> 优先级和结合性规定了操作数与操作符的结合方式, 操作数的求值顺序规定了二元和三元操作符的操作数求值顺序, 它们都会对表达式的值产生影响
>
> **注: 本章中对于各操作符的操作数类型的规定, 均建立在操作符没有被重载的前提下**

### 流表达式

> 流表达式是包含流操作符的表达式
>
> 流操作符包括两种: 表示数据流向的中缀操作符`|>`(称为`pipeline`)和表示函数组合的中缀操作符`~>`(称为`composition`)
>
> **`|>`和`~>`的优先级相同, 并介于`||`和赋值操作符`=`之间**
>
> **`|>`和`~>`的结合性均为左结合**, 详情参考下文
>
> 流表达式的语法定义为:
>
> ```
> flowExpression
>     : logicDisjunctionExpression (flowOperator logicDisjunctionExpression)*
>     ;
>
> flowOperator
>     : '|>' | '~>'
>     ;
> ```

C++中也存在流, 只不过与仓颉中的流 好像并不是同一种东西?

这里两个操作符的优先级比较重要, 当然对于自己不确定的优先级, 我更喜欢使用`()`来进行分明

#### `pipeline`操作符

> `pipeline`表达式是单个参数函数调用的语法糖, 即`e1 |> e2`是`let v = e1; e2(v)`的语法糖(即先对`|>`操作符左边的`e1`求值)
>
> 这里`e2`是函数类型的表达式, `e1`的类型是`e2`的参数类型的子类型
>
> 或者`e2`的类型重载了函数调用操作符`()`(参见 [可以被重载的操作符])
>
> 注意: 这里的`f`不能是`init`和`super`构造函数
>
> ```cangjie
> func f(x: Int32): Int32 { x + 1 }
>
> let a: Int32 = 1
> var res = a |> f // ok
> var res1 = a |> {x: Int32 => x + 1} // ok
>
> func h(b: Bool) { b }
> let res3 = a < 0 || a > 10  |> h // Equivalence: (a < 0 || a > 10)  |> h
>
> func g<T>(x: T): T { x }
> var res4 = a |> g<Int32> // ok
>
> class A {
>     let a: Int32
>     let b: Int32
>     init(x: Int32) {
>         a = x
>         b = 0
>     }
>     init(x: Int32, y: Int32) {
>         x |> init // error:`init`is not a valid expression
>         b = y
>     }
> }
>
> // PIPELINE with operator`()`overloading
> class A {
>     operator func ()(x: Int32) {
>         x
>     }
> }
> let obj = A()
> let a: Int32 = 1
> let res = a |> obj // Equivalence: obj(a)
> ```

从此例子中看, `|>`可以将左边的操作数作为参数传递给右边的函数、lambda作为参数

不过右边的函数必须是单参数的

#### `composition`操作符

> `composition`表达式表示两个单参函数的组合
>
> 也就是说, `composition`表达式`e1 ~> e2`是`let f = e1; let g = e2; {x => g(f(x))}`的语法糖(即先对`~>`操作符左边的`e1`求值)
>
> 这里的`f`, `g`均为函数类型的表达式或者其类型重载了单参的函数调用操作符`()`(参见 [可以被重载的操作符]), 则会有以下四种情况:
>
> | `e1 ~> e2`                                                   | 对应的`lambda`表达式                                         |
> | :----------------------------------------------------------- | :----------------------------------------------------------- |
> | `e1`和`e2`是函数类型, 且`e1`的返回值类型是`e2`的参数类型的子类型 | `let f = e1; let g = e2;`<br />`{x => g(f(x))}`              |
> | 类型`f`实现了单参数操作符`()`的重载函数, 而`g`是一个函数类型, `f.operator()`的返回值类型是`g`的参数类型的子类型 | `let f = e1; let g = e2;`<br />`{x => g(f.operator()(x))}`   |
> | `f`是一种函数类型, 而`g`的类型实现了单参数操作符`()`的重载函数, 且`f`的返回值类型是`g.operator()`参数类型的子类型 | `let f = e1; let g = e2;`<br />`{x => g.operator()(f(x))}`   |
> | `f`和`g`的类型都实现了单参数运算符`()`的重载函数, 且`f.operator()`的返回值类型是`g.operator()`参数类型的子类型 | `let f = e1; let g = e2;`<br />`{x => g.operator()(f.operator()(x))}` |
>
> 注意: 这里的`e1`, `e2`求值后不能是`init`和`super`构造函数
>
> ```cangjie
> func f(x: Int32): Float32 { Float32(x) }
> func g(x: Float32): Int32 { Int32(x) }
>
> var fg = f ~> g                         // 等价于: {x: Int32 => g(f(x))}
>
> let lambdaComp = {x: Int32 => x} ~> f     // ok
>
> func h1<T>(x: T): T { x }
> func h2<T>(x: T): T { x }
> var hh = h1<Int32> ~> h2<Int32>         // ok
>
> class A {
>     operator func ()(x: Int32): Int32 {
>         x
>     }
> }
> class B {
>     operator func ()(x: Float32): Float32 {
>         x
>     }
> }
> let objA = A()
> let objB = B()
> let af = objA ~> f // ok
> let fb = f ~> objB // ok
> let aa = objA ~> objA // ok
> ```

`composition`操作符`~>`, 作用是将两个单参数函数组合成一个新`lambda`表达式

**如果存在`e1`和`e2`均为单参数函数, 那么`e1 ~> e2`, 就是组合成一个 将`e1`执行返回值作为`e2`的参数的新`lambda`, `e1`的返回值类型需要时`e2`参数的子类型**

### 赋值表达式

> 赋值表达式是包含赋值操作符的表达式, 用于将左操作数的值修改为右操作数的值, 要求右操作数的类型是左操作数类型的子类型
>
> 对赋值表达式求值时, 总是先计算`=`右边的表达式, 再计算`=`左边的表达式, 最后进行赋值
>
> 对于复合赋值表达式(`+=``-=`...)求值时, 总是先计算`=`左边的表达式的左值, 然后根据这个左值取右值, 然后将该右值与`=`右边的表达式做计算(若有短路规则会继续遵循短路规则), 最后赋值
>
> 除了子类型允许的赋值外, 如果右操作数是字符串字面量, 而左操作数的类型是`Byte`或`Rune`, 则字符串值将分别被强制赋值为`Byte`或`Rune`, 并对强制赋值进行赋值
>
> 赋值操作符分为普通赋值操作符和复合赋值操作符, 赋值表达式的语法定义为:
>
> ```
> assignmentExpression
>     : leftValueExpressionWithoutWildCard assignmentOperator flowExpression
>     | leftValueExpression '=' flowExpression
>     | tupleLeftValueExpression`=`flowExpression
>     | flowExpression
>     ;
>
> tupleLeftValueExpression
>     :`(`(leftValueExpression | tupleLeftValueExpression) (`, `(leftValueExpression | tupleLeftValueExpression))+`, `?`)`
>     ;
>
> leftValueExpression
>     : leftValueExpressionWithoutWildCard
>     | '_'
>     ;
>
> leftValueExpressionWithoutWildCard
>     : identifier
>     | leftAuxExpression '?'? assignableSuffix
>     ;
>
> leftAuxExpression
>     : identifier typeArguments?
>     | type
>     | thisSuperExpression
>     | leftAuxExpression ('?')? '.' identifier typeArguments?
>     | leftAuxExpression ('?')? callSuffix
>     | leftAuxExpression ('?')? indexAccess
>     ;
>
> assignableSuffix
>     : fieldAccess
>     | indexAccess
>     ;
>
> fieldAccess
>     : '.' identifier
>     ;
>
> assignmentOperator
>     : '=' | '+=' | '-=' | '**=' | '*=' | '/=' | '%=' | '&&=' | '||='
>     | '&=' | '|=' | '^=' | '<<=' | '>>='
>     ;
> ```

赋值表达式就是`=`作为赋值时相关的表达式

C++与仓颉都拥有的赋值运算符, 使用上大致都相同, 但`&&=`和`||=`是C++中没有的, 这两个是操作符左右需要为`Bool`类型, 且遵循短路规则

其次, 文档中提到的如果左操作数是`Rune`或`Byte`, 右操作数是字符串字面量, 字符串字面量会强制赋值为`Rune`或`Byte`, 然后再赋值. 这里提到的字符串字面量, 其实只是**单字符字符串字面量**, 如果是多字符字符串, 是不允许复制的

> 出现在(复合)赋值操作符左侧的表达式称为*左值表达式*(上述定义中的`leftValueExpression`)
>
> 语法上, *左值表达式*可以是一个`identifier`或`_`, 或者一个`leftAuxExpression`后接`assignableSuffix`(包含`fieldAccess`和`indexAccess`两类), `leftAuxExpression`和`assignableSuffix`之间可以有可选的`?`操作符(对 Option Type 的实例进行赋值的语法糖)
>
> `leftAuxExpression`可以是以下语法形式:
>
> 1. 一个包含可选类型实参(`typeArguments`)的`identifier`
> 2. `this`或`super`
> 3. 一个`leftAuxExpression`后接一个`.`(二者之间可以有可选的`?`操作符)和一个存在可选类型实参的`identifier`
> 4. 一个`leftAuxExpression`后接一个函数调用后缀`callSuffix`或索引访问后缀`indexAccess`(`callSuffix`或`indexAccess`之前可以有可选的`?`操作符)
>
> 语义上, *左值表达式*只能是如下形式的表达式:
>
> 1. `identifier`表示的变量(参见 [变量名和函数名](https://www.humid1ch.cn/blog/cangjie-docs-reading-vi/#heading-2))
> 2. 通配符`_`, 意味着忽略`=`右侧表达式的求值结果(复合赋值表达式禁止使用通配符)
> 3. 成员访问表达式`e1.a`或者`e2?.a`(参见[成员访问表达式](https://www.humid1ch.cn/blog/cangjie-docs-reading-vii#heading-18))
> 4. 索引访问表达式`e1[a]`或者`e2?[a]`(参见[索引访问表达式](https://www.humid1ch.cn/blog/cangjie-docs-reading-vii#heading-20))
>
> > 注: 其中`e1`和`e2`必须是满足`leftAuxExpression`语法的表达式
>
> *左值表达式*是否合法, 取决于*左值表达式*是否是可变的: 仅当*左值表达式*可变时, 它才是合法的
>
> 关于上述表达式的可变性, 可参见对应章节

文档已经将左值表达式的形式列出来了, 其实就是**普通变量、通配符`_`还有索引访问和成员访问**

> **赋值表达式的类型是`Unit`, 值是`()`**, 这样做的好处是可以避免类似于错误地将赋值表达式当做判等表达式使用等问题的发生
>
> 在下面的例子中, 如果先执行`(a = b)`, 则返回值是`()`, 而`()`不能出现在`=`的左侧, 所以执行`()=0`时就会报错
>
> 同样地, 由于`if`之后的表达式必须为`Bool`类型, 所以下例中的`if`表达式也会报错
>
> 另外, `=`是非结合的, 所以类似于`a = b = 0`这样的同一个表达式中同时包含两个以上`=`的表达式是被禁止的(无法通过语法检查)
>
> ```cangjie
> main(): Int64 {
>     var a = 1
>     var b = 1
>     a = (b = 0)     // semantics error
>     if (a = 5) {      // semantics error
>     }
>     a = b = 0       // syntax error
>
>     return 0
> }
> ```

仓颉中规定 赋值表达式类型恒为`Unit`且值恒为`()`

而且, 仓颉中的`=`是非结合的, 即 **禁止连续赋值**, 从语法上是禁止的

> 复合赋值表达式`a op= b`不能简单看做赋值表达式与其他二元操作符的组合`a = a op b`(其中`op`可以是算术操作符、逻辑操作符和位操作符中的任意二元操作符, 操作数`a`和`b`的类型为操作符`op`所要求的类型)
>
> 在仓颉语言中, `a op= b`中的`a`只会被求值一次(副作用也只发生一次), 而`a = a op b`中的`a`会被求值两次(副作用也发生两次)
>
> 因为复合赋值表达式也是一个赋值表达式, 所以复合赋值操作符也是非结合的
>
> 复合赋值表达式同样要求两个操作数的类型相同
>
> 下面举例说明复合赋值表达式的使用:
>
> ```cangjie
> a **= b
> a *= b
> a /= b
> a %= b
> a += b
> a -= b
> a <<= b
> a >>= b
> a &&= b
> a ||= b
> a &= b
> a ^= b
> a |= b
> ```

从语言特性来看, `a op= b`是比`a = a op b`更优的

不过编译器可能会进行优化

> 最后, 如果用户重载了`**`、`*`、`/`、`%`、`+`、`-`、`<<`、`>>`、`&`、`^`、`|`操作符, 那么仓颉语言会提供其对应的复合赋值操作符`**=`、`*=`、`/=`、`%=`、`+=`、`-=`、`<<=`、`>>=`、`&=`、`^=`、`|=`的默认实现
>
> 但有些额外的要求, 否则无法为`a = a op b`提供赋值语义:
>
> 1. 重载后的操作符的返回类型需要与左操作数的类型一致或是其子类型, 即对于`a op= b`中的`a`, `b`, `op`, 它们需要能通过`a = a op b`的类型检查
>
>     例如 当有子类型关系`A <: B <: C`时, 若用户重载的`+`的类型是`(B, Int64) -> B`或`(B, Int64) -> A`, 则仓颉语言可以提供默认实现
>
>     若用户重载的`+`的类型是`(B, Int64) -> C`, 则仓颉语言不会为其提供默认实现
>
> 2. 要求`a op= b`中的`a`必须是可被赋值的, 例如 是一个变量

**仓颉会根据重载的原始操作符, 提供对应的复合赋值操作符**

**但需要满足类型检查**

> 多赋值表达式是一种特殊的赋值表达式, 多赋值表达式等号左边必须是一个`tuple`, 这个`tuple` 里面的元素必须都是左值, 等号右边的表达式也必须是`tuple`类型, 右边`tuple`每个元素的类型必须是对应位置左值类型的子类型
>
> 注意: 当左侧`tuple`中出现`_`时, 表示忽略等号右侧`tuple`对应位置处的求值结果(意味着这个位置处的类型检查总是可以通过的)
>
> 多赋值表达式可以将右边的`tuple`类型的值, 一次性赋值给左边`tuple`内的对应左值, 省去逐个赋值的代码
>
> ```cangjie
> main(): Int64 {
>     var a: Int64
>     var b: Int64
>
>     (a, b) = (1, 2) // a == 1, b == 2
>     (a, b) = (b, a) // swap, a == 2, b == 1
>     (a, _) = (3, 4) // a == 3
>     (_, _) = (5, 6) // no assignment
>
>     return 0
> }
> ```
>
> 多赋值表达式可以看成是如下形式的语法糖
>
> 赋值表达式右侧的表达式会优先求值, 再对左值部分从左往右逐个赋值
>
> ```cangjie
> main(): Int64 {
>     var a: Int64
>     var b: Int64
>     (a, b) = (1, 2)
>
>     // desugar
>     let temp = (1, 2)
>     a = temp[0]
>     b = temp[1]
>
>     return 0
> }
> ```

### `Lambda`表达式

> `Lambda`表达式是函数类型的值, 详见第 5 章函数

### `Quote`表达式

> `Quote`表达式用于引用代码, 并将其表示为可操作的数据对象, 主要用于元编程, 详见第 14 章元编程

### 宏调用表达式

> 宏调用表达式用于调用仓颉定义的宏, 主要用于元编程, 详见第 14 章元编程

### 引用传值表达式

> 引用传值表达式只可用于 C 互操作中调用`CFunc`场景中, 详见第 13 章互操作中`inout`参数一节

原来引用传值表达式 适用来与 C 互操作的

### 操作符的优先级和结合性

> 对于包含两个或两个以上操作符的表达式, 它的值由操作符和操作数的分组结合方式决定, 而分组结合方式取决于操作符的优先级和结合性
>
> 简单来讲, 优先级规定了不同操作符的求值顺序, 结合性规定了具有相同优先级的操作符的求值顺序
>
> 如果一个表达式中包含多个不同优先级的操作符, 那么它的计算顺序是: 先计算包含高优先级操作符的子表达式, 再计算包含低优先级操作符的子表达式
>
> 在包含多个同一优先级操作符的子表达式中, 计算次序由操作符的结合性决定
>
> 下表列出了各操作符的优先级、结合性、功能描述、用法以及表达式的类型
>
> 其中**越靠近表格顶部, 操作符的优先级越高**:
>
> | 操作符  | 结合性 | 描述           | 用法                   | 表达式类型                                                   |
> | :------ | :----- | :------------- | :--------------------- | :----------------------------------------------------------- |
> | `@`     | 右结合 | 宏调用表达式   | `@expr1 @expr2`        | Unit                                                         |
> | `.`     | 左结合 | 成员访问       | `Name.name`            | 成员`name`的类型                                             |
> | `[]`    |        | 索引访问       | `varName[expr]`        | `varName`中元素的类型                                        |
> | `()`    |        | 函数调用       | `funcName(expr)`       | `funcName`返回值的类型                                       |
> | `++`    | None   | 后缀自增       | `varName++`            | Unit                                                         |
> | `--`    |        | 后缀自减       | `varName--`            | Unit                                                         |
> | `?`     |        | 问号           | `expr1?.expr2 etc.`    | `Option<T>``T`是`expr2`的类型                                |
> | `!`     | 右结合 | 按位逻辑非     | `!expr`                | `expr`的类型                                                 |
> | `-`     |        | 一元 负        | `-expr`                |                                                              |
> | `**`    | 右结合 | 求幂           | `expr1 ** expr2`       | `expr1的类型`                                                |
> | `*`     | 左结合 | 乘法           | `expr1 * expr2`        | `expr1`或`expr2`的类型, 因为`expr1`和`expr2`类型相同         |
> | `/`     |        | 除法           | `expr1 / expr2`        |                                                              |
> | `%`     |        | 取余           | `expr1 % expr2`        |                                                              |
> | `+`     | 左结合 | 加             | `expr1 + expr2`        | `expr1`或`expr2`的类型, 因为`expr1`和`expr2`类型相同         |
> | `-`     |        | 减             | `expr1 - expr2`        |                                                              |
> | `<<`    | 左结合 | 按位左移       | `expr1 << expr2`       | `expr1`的类型, 其中`expr1`和`expr2`可以有不同的类型          |
> | `>>`    |        | 按位右移       | `expr1 >> expr2`       |                                                              |
> | `..`    | None   | 范围操作符     | `expr1..expr2:expr3`   | `Range`类型                                                  |
> | `..=`   |        |                | `expr1..=expr2:expr3`  |                                                              |
> | `<`     | None   | 小于           | `expr1 < expr2`        | 除了`expr as userType`的类型是`Option<userType>`<br />其他表达式具有`Bool`类型 |
> | `<=`    |        | 小于等于       | `expr1 <= expr2`       |                                                              |
> | `>`     |        | 大于           | `expr1 > expr2`        |                                                              |
> | `>=`    |        | 大于等于       | `expr1 >= expr2`       |                                                              |
> | `is`    |        | 类型检查(判断) | `expr is type`         |                                                              |
> | `as`    |        | 类型转换       | `expr as userType`     |                                                              |
> | `==`    | None   | 判等           | `expr1 == expr2`       | Bool                                                         |
> | `!=`    |        | 判不等         | `expr1 != expr2`       |                                                              |
> | `&`     | 左结合 | 按位与         | `expr1 & expr2`        | `expr1`或`expr2`的类型, 因为`expr1`和`expr2`类型相同         |
> | `^`     | 左结合 | 按位异或       | `expr1 ^ expr2`        | `expr1`或`expr2`的类型, 因为`expr1`和`expr2`类型相同         |
> | `\|`    | 左结合 | 按位或         | `expr1 \| expr2`       | `expr1`或`expr2`的类型, 因为`expr1`和`expr2`类型相同         |
> | `&&`    | 左结合 | 逻辑与         | `expr1 && expr2`       | Bool                                                         |
> | `\|\|`  | 左结合 | 逻辑或         | `expr1 \|\| expr2`     | Bool                                                         |
> | `??`    | 右结合 | coalescing     | `expr1 ?? expr2`       | `expr2`的类型                                                |
> | `\|>`   | 左结合 | Pipeline       | `expr1 \|> expr2`      |                                                              |
> | `~>`    |        | Composition    | `expr1 ~> expr2`       | `expr1 ~> expr2`的类型是 lambda 表达式`{x=>expr2(expr1(x))}`的类型 |
> | `=`     | None   | 赋值           | `leftValue = expr`     | Unit                                                         |
> | `**=`   |        | 复合赋值       | `leftValue **= expr`   |                                                              |
> | `*=`    |        |                | `leftValue *= expr`    |                                                              |
> | `/=`    |        |                | `leftValue /= expr`    |                                                              |
> | `%=`    |        |                | `leftValue %= expr`    |                                                              |
> | `+=`    |        |                | `leftValue += expr`    |                                                              |
> | `-=`    |        |                | `leftValue -= expr`    |                                                              |
> | `<<=`   |        |                | `leftValue <<= expr`   |                                                              |
> | `>>=`   |        |                | `leftValue >>= expr`   |                                                              |
> | `&=`    |        |                | `leftValue &= expr`    |                                                              |
> | `^=`    |        |                | `leftValue ^= expr`    |                                                              |
> | `\|=`   |        |                | `leftValue \|= expr`   |                                                              |
> | `&&=`   |        |                | `leftValue &&= expr`   |                                                              |
> | `\|\|=` |        |                | `leftValue \|\|= expr` |                                                              |
>
> > 注:`?`和`.`、`()`、`{}`、`[]`一起使用时, 是一种语法糖形式, 不会严格按照它们固有的优先级和结合性进行求值, 详见[问号操作符](https://www.humid1ch.cn/blog/cangjie-docs-reading-vii#heading-21)

### 表达式求值顺序

> 表达式的求值顺序规定了计算操作数的值的顺序, 显然只有包含二元操作符的表达式才存在求值顺序的概念
>
> 仓颉编程语言的默认求值顺序为:
>
> 1. 对于包含逻辑与(`&&`)、逻辑或(`||`)和`coalescing`(`??`)的表达式, 仅当操作符的右操作数的值会影响整个表达式的值时, 才计算右操作数的值, 否则只计算左操作数的值
>
>     因此, **`&&`、`||`和`??`的求值顺序为: 先计算左操作数的值, 再计算右操作数的值**
>
> 2. 对于 optional chaining 表达式, 其中的`?`会将表达式分隔成若干子项, 按从左到右的顺序对各子项依次求值(子项内按使用到的操作符的求值顺序进行求值)
>
> 3. 对于其他表达式(如算术表达式、关系表达式、位运算表达式等), 同样按从左往右的顺序求值
