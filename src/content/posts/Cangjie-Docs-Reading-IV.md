---
title: "仓颉文档阅读-语言规约II: 类型(III)"
published: 2025-09-23
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

## 类型

> 仓颉编程语言是一种 **静态类型(`statically typed`)** 语言: 大部分保证程序安全的类型检查发生在编译期
>
> 同时, 仓颉编程语言是一种 **强类型(`strongly typed`)** 语言: 每个表达式都有类型, 并且表达式的类型决定了它的取值空间和它支持的操作
>
> 静态类型和强类型机制可以帮助程序员在编译阶段发现大量由类型引发的程序错误

### 类型转换

> 作为一种强类型(`strongly typed`)语言, 仓颉编程语言**仅支持显式类型转换(亦称强制类型转换)**
>
> 对于值类型, 支持使用`valueType(expr)`实现将表达式`expr`的类型转换成`valueType`
>
> 对于`class`和`interface`, 通过使用`as`操作符实现静态类型转换
>
> 对于值类型, 我们说`valueTypeA`到`valueTypeB`是可转换的, 是指定义了将`valueTypeA`转换成`valueTypeB`的转换规则, 对于没有定义转换规则的两个值类型, 我们称它们是不可转换的
>
> 对于`class`和`interface`, 如果两个类型在**继承关系图上存在父子类型关系**, 那么**这两个类型之间就是可转换的**(当然, 转换的结果也有可能是失败的), 否则, 这两个类型之间就是不可转换的
>
> 对于其它未提及的类型, 仓颉不支持通过上述两种方式实现它们之间的类型转换

仓颉作为强类型语言, **只支持强制类型转换**:

1. 值类型, 支持`Type(expr)`强制类型转换

    与C语言的强制类型转换类似

2. 对于`class`和`interface`, 则支持通过`as`操作符, 静态类型转换

#### `Value Type`之间的类型转换

> 对于数值类型, 支持如下类型转换**(未列出即表示不支持)**:
>
> 1. `Rune`类型到`UInt32`类型的转换
> 2. 整数类型(包括`Int8`, `Int16`, `Int32`, `Int64`, `IntNative`, `UInt8`, `UInt16`, `UInt32`, `UInt64`, `UIntNative`)到`Rune`类型的转换
> 3. 所有数值类型(包括`Int8`, `Int16`, `Int32`, `Int64`, `IntNative`, `UInt8`, `UInt16`, `UInt32`, `UInt64`, `UIntNative`, `Float16`, `Float32`, `Float64`)之间的双向转换
>
> `Rune`到`UInt32`的转换使用`UInt32(e)`的方式, 其中`e`是一个`Rune`类型的表达式, `UInt32(e)`的结果是`e`的 Unicode scalar value 对应的`UInt32`类型的整数值
>
> 整数类型到`Rune`的转换使用`Rune(num)`的方式, 其中`num`的类型可以是任意的整数类型, 且仅当`num`的值落在`[0x0000, 0xD7FF]`或`[0xE000, 0x10FFFF]`(即 Unicode scalar value)中时, 返回对应的 Unicode scalar value 表示的字符, 否则, 编译报错(编译时可确定`num`的值)或运行时抛异常
>
> ```cangjie
> main(){
>     var c: Rune = 'a'
>     var num: UInt32 = 0
>     num = UInt32(c)    // num = 97
>     num -= 32           // num = 65
>     c = Rune(num)      // c =`A`
>
>     return 0
> }
> ```

仓颉值类型的强制类型转换, 只支持:

1. 整型和浮点型之间的转换
2. `Rune`到`UInt32`的转换
3. 整型到`Rune`有规则限制的转换

如果把`Rune`单纯看作C/C++中的`char`, 那与C中的整型与浮点型之间的强制类型转换还是比较相似的

但`Rune`可表示字符的范围, 要比C/C++中的`char`要大多了, 所以整型到`Rune`的转换才要限制

> 为了保证类型安全, 仓颉编程语言不支持数值类型之间的隐式类型转换(数值字面量的类型由上下文推断得到, 这种情形并不是隐式类型转换), 要实现一种数值类型到另外一种数值类型的转换, 必须使用显式的方式:`NumericType(expr)`, 表示将`expr`的类型强制转换为`NumericType`类型(`NumericType`表示任意一种数值类型), 如转换成功, 会返回一个新的从`expr`构造而来的类型为`NumericType`的值
>
> 数值类型转换的语法定义为:
>
> ```
> numericTypeConvExpr
>     : numericTypes '(' expression ')'
>        ;
>
> numericTypes
>        : 'Int8' | 'Int16' | 'Int32' | 'Int64' | 'IntNative' | 'UInt8' | 'UInt16' | 'UInt32' | 'UInt64' | 'UIntNative' | 'Float16' | 'Float32' | 'Float64'
>        ;
> ```

> 如果根据数值类型所占`bit`数来定义数值类型间的"大小关系"(所占`bit`数越多, 类型越"大", 所占`bit`数越少, 类型越"小"), 则仓颉编程语言支持以下类型转换:
>
> a)**有符号整数类型之间的双向转换**: 小转大时数值结果不变, **大转小时 若超出小类型的表示范围, 则根据上下文中的属性宏确定溢出处理策略(默认使用抛出异常的策略)**, 不同溢出策略详见[算术表达式]
>
> 下面以`Int8`和`Int16`之间的转换为例进行说明(溢出时, 使用抛异常的处理策略):
>
> ```cangjie
> main(){
>     var i8Number: Int8 = 127
>     var i16Number: Int16 = 0
>
>     i16Number = Int16(i8Number)        // ok: i16Number = 127
>     i8Number = Int8(i16Number)        // ok: i8Number = 127
>     i16Number = 128
>     i8Number = Int8(i16Number)        // throw an ArithmeticException
>
>     return 0
> }
> ```
>
>b)**无符号整数类型之间的双向转换**: 规则同上
>
>以`UInt16`和`UInt32`之间的转换为例进行说明(其他情况遵循一样的规则):
>
>```cangjie
> main(){
>     var u16Number: UInt16 = 65535
>     var u32Number: UInt32 = 0
>
>     u32Number = UInt32(u16Number)        // ok: u32Number = 65535
>     u16Number = UInt16(u32Number)        // ok: u16Number = 65535
>     u32Number = 65536
>     u16Number = UInt16(u32Number)        // throw an ArithmeticException
>
>     return 0
> }
>```

仓颉这一点就与C/C++很不一样!

同符号的整型整型之间的转换, 溢出默认抛异常!!!!

我勒个强类型语言啊, C/C++如果这样就能少很多不容易定位的BUG了

> c)浮点类型之间的双向转换: 使用`round-to-nearest`模式
>
> 举例说明`Float32`和`Float64`之间的转换:
>
> ```cangjie
> main(){
>     var f32Number: Float32 = 1.1
>     var f64Number: Float64 = 0.0
>
>     f64Number = Float64(f32Number)        // f64Number = 1.100000023841858
>     f32Number = Float32(f64Number)        // f32Number = 1.1
>
>     f64Number = 1.123456789
>     f32Number = Float32(f64Number)        // f32Number = 1.1234568
>
>     f32Number = 4.4E38                    // f32Number = POSITIVE_INFINITY
>     f64Number = Float64(f32Number)        // f64Number = POSITIVE_INFINITY
>
>     f64Number = 4.4E38
>     f32Number = Float32(f64Number)        // f32Number = POSITIVE_INFINITY
>
>     f64Number = Float64(f32Number * 0.0)
>     f32Number = Float32(f64Number)        // f32Number = NaN
>
>     return 0
> }
> ```

从示例来看, `round-to-nearest`好像是 就近原则?

查了一下资料: **四舍六入五取偶: 溢出位`<=4`舍去, `>=6`进位, `==5`看前一位的奇偶, 偶就进位, 奇就舍去**

1. **高精度转低精度, 如果超出范围, 会变成无穷**
2. 无穷只能转成无穷
3. `NaN`也只能转成`NaN`

> d)**有符号整数类型和无符号整数类型之间的双向转换**:
>
> 因为任何有符号整数类型的表示范围 均不能 包含长度相同的无符号整数类型的表示范围(反之亦然), 因此它们之间进行转换时, 只要 待转换表达式的值落在目标整数类型的表示范围之内则转换成功, 否则根据上下文中的属性宏确定溢出处理策略(默认使用抛出异常的策略), 不同溢出策略详见[算术表达式]
>
> 下面以`Int8`和`UInt8`之间的转换为例进行说明(溢出时, 使用抛异常的处理策略):
>
> ```cangjie
> main(){
>     var i8Number: Int8 = 127
>     var u8Number: UInt8 = 0
>
>     u8Number = UInt8(i8Number)    // ok: u8Number = 127
>     u8Number = 100
>     i8Number = Int8(u8Number)    // ok: i8Number= 100
>     i8Number= -100
>     u8Number = UInt8(i8Number)    // throw an ArithmeticException
>     u8Number = 255
>     i8Number = Int8(u8Number)    // throw an ArithmeticException
>
>     return 0
> }
> ```

有符号和无符号整型之间转换, 如果没有溢出, 就获得最终值

如果存在溢出, 直接抛异常

> e)整数转换为浮点数: 结果为尽可能接近原整数的浮点数
>
> 超出目标类型的表示范围时, 返回`POSITIVE_INFINITY`或`NEGTIVE_INFINITY`
>
> f)浮点数转换为整数: 浮点类型到有符号整数类型的转换使用`round-toward-zero`模式, 即**保留整数部分舍弃小数部分**
>
> 当整数部分超出目标整数类型的表示范围, 则根据上下文中的整数溢出策略处理
>
> 如果是`throwing`策略, 那么抛出异常;
>
> 否则按如下规则转换:
>
> - `NaN`返回`0`
> - 小于整数取值范围下界时(包括负无穷), 返回整数的取值范围下界
> - 大于整数取值范围上界时(包括正无穷), 返回整数的取值范围上界
>
> ```cangjie
> main(){
>     var i32Number: Int32 = 1024
>     var f16Number: Float16 = 0.0
>     var f32Number: Float32 = 0.0
>
>     f16Number = Float16(i32Number)        // ok: f16Number = 1024.0
>     f32Number = Float32(i32Number)        // ok: f32Number = 1024.0
>     i32Number = 2147483647
>     f16Number = Float16(i32Number)        // f16Number = POSITIVE_INFINITY    正无穷
>     f32Number = Float32(i32Number)        // precision lost: f32Number = 2.14748365E9    精度丢失
>
>     f32Number = 1024.1024
>     i32Number = Int32(f32Number)        // ok: i32Number = 1024
>
>     f32Number = 1024e10
>     i32Number = Int32(f32Number)        // throw an Exception
>
>     f32Number = 3.4e40                    // f32Number = POSITIVE_INFINITY
>     i32Number = Int32(f32Number)        // throw an Exception
>
>     f32Number = 3.4e40 * 0.0            // f32Number = NaN
>     i32Number = Int32(f32Number)        // throw an Exception
>
>     return 0
> }
> ```

从示例来看:

1. 整数->浮点数

    如果超出了浮点数可表示的最大范围, 浮点数会变为无穷

    如果没有超出范围, 但是超出了精度范围, 就会丢失一定的精度

2. 浮点数->整数

    如果整数部分没有超出有效范围, 就取整

    如果整数部分超出有效范围, 就是溢出, 会抛异常

    如果浮点数是`NaN`或无穷, 也会抛异常

#### `class/interface`之间的类型转换

> 对于一个`class/interface`类型的实例`obj`, 如果需要将它的(静态)类型转换到另一个`class/interface`类型`TargetType`, 可使用:`obj as TargetType`
>
> 关于`as`操作符的使用, 以及`class/interface`之间的类型转换规则, 参见[as 操作符]

虽然没有介绍, 但是推测应该与继承关系 有关系

### 类型别名

> 当某个类型的名字比较复杂或者在特定场景中不够直观时, 可以选择使用类型别名的方式为此类型取一个简单并且直观的别名. 定义类型别名的语法为:
>
> ```
> typeAlias
>     : typeModifier?`type`identifier typeParameters?`=`type
>     ;
> ```
>
> 其中, `typeModifier`是可选的可访问性修饰符(即`public`), `type`是关键字, `identifier`是任意的合法标识符, `type`是任意的在`top-level`可见的类型, `identifier`和`type`之间使用`=`进行连接
>
> 另外, 也可通过在`identifier`之后添加类型参数(`typeParameters`)的方式定义泛型别名
>
> 通过以上声明, 即为类型`type`定义了一个名字为`identifier`的别名, 并且`identifier`和`type`被视作同一种类型
>
> 例如:
>
> ```
> type Point2D = (Float64, Float64)
> type Point3D = (Float64, Float64, Float64)
>
> let point1: Point2D = (0.5, 0.8)
> let point2: Point3D = (0.5, 0.8, 1.1)
> ```
>
> 上述`type`定义并不会定义一个新的类型, 它的作用仅仅是为某个已有类型定义另外一个名字而已, 别名和原类型被视作同一个类型, 并且别名不会对原类型的使用带来任何影响

仓颉中的`type`关键字, 应该和C/C++中的`typedef`类似, 与C++中的`using`的一部分功能也类似

使用比较简单:

```cangjie
type 别名(<泛型类型参数>)= 实际类型(可以是泛型)
```

#### 类型别名定义的规则

> 1. 类型别名的定义只能出现在`top-level`
>
>     ```cangjie
>     func test(){
>         type Point2D = (Float64, Float64)                // error: type alias can only be defined at top-level
>         type Point3D = (Float64, Float64, Float64)        // error: type alias can only be defined at top-level
>     }
>     ```
>
> 2. 用`type`定义类型别名时, 原类型必须在`type`定义的位置可见
>
>     ```cangjie
>     class LongNameClassA {}
>     type ClassB = LongNameClassB            // error: use of undeclared type 'LongNameClassB'
>     ```
>
> 3. 定义泛型别名时, 如果泛型别名中引入了原类型中没有使用的泛型参数, 则编译器会告警
>
>     ```cangjie
>     type Class1<V> = GenericClassA<Int64, V>             // ok. ClassA is a generic class
>     type Class2<Value, V> = GenericClassB<Int64, V>     // warning: the type parameter 'Value' in 'Class2<Value, V>' is not used in 'GenericClassB<Int64, V>'
>     type Int<T> = Int32                                 // warning: the type parameter 'T' in 'Int<T>' is not used in`Int32`
>     ```

仓颉中的`type`与C语言中的`typedef`有不同点

`type`只能用在顶层作用域, 不能出现在`{}`代码块中

而C/C++中的`typedef`可以出现在任何地方

> 4. 定义泛型别名时, 不允许为别名和原类型中的类型参数添加泛型约束, 在用到泛型别名时, 可以按需为其添加泛型约束
>
>     另外, 原类型中已有的泛型约束会"传递"至别名
>
>     ```cangjie
>     type Class1<V> where V <: MyTrait = GenericClassA<Int64, V>     // error: generic constraints are not allowed here
>     type Class2<V> = GenericClassB<Int64, V> where V <: MyTrait     // error: generic constraints are not allowed here
>
>     type Class3<V> = GenericClassC<Int64, V>
>     func foo<V> (p: Class3<V>)where V <: MyTrait {                 // add generic constraints when 'Class3<V>' is used
>         functionBody
>     }
>
>     class ClassWithLongName<T> where T<:MyTrait {
>         classBody
>     }
>     type Class<T> = ClassWithLongName<T>                           // Class<T> also has the constraint 'where T<:MyTrait'
>     ```

别名只是起别名, 定义别名 不能额外添加原类型不存在的功能

但是使用时, 与原类型保持一致

> 5. 一个(或多个)`type`定义中禁止出现循环引用(无论是直接的或是间接的)
>
>     其中, 判断循环引用的方式是通过名字判断是否存在循环引用, 并不是使用类型展开的方式
>
>     ```cangjie
>     type Value = GenericClassAp<Int64, Value>        // error: 'Value' references itself
>     type Type1 = (Int64)->Type1                    // error: 'Type1' references itself
>     type Type2 = (Int64, Type2)                    // error: 'Type2' references itself
>     type Type3 = Type4                            // error: 'Type3' indirectlly references itself
>     type Type4 = Type3
>     ```

> 6. 类型别名被视为与原类型等价的类型
>
>     例如, 在下面的例子中, 可以将`Int`类型的参数和`Int32`类型的参数直接相加(`Int`定义为`Int32`的别名)
>
>     注意, 不能通过使用别名达到函数重载的目的:
>
>     ```cangjie
>     type Int = Int32
>     let numOne: Int32 = 10
>     let numTwo: Int = 20
>     let numThree = numOne + numTwo
>
>     func add(left: Int, right: Int32): Int { left + right }
>
>     func add(left: Int32, right: Int32): Int32 { left + right }     // error: invalid redeclaration of 'add : (Int32, Int32)->Int32'
>     ```

> 7. `type`定义的默认可见性为`default`
>
>     如果需要 **在其他`package`内使用本`package`中定义的类型别名**, 需要同时满足:
>
>     (1)原类型在本`package`中的可见性修饰符为`public`
>
>     (2)`type`定义使用`public`修饰符
>
>     另外, 需要注意的是: **别名可以与原类型拥有不同的可见范围, 但是别名的的可见范围不能大于原类型的可见范围**
>
>     ```cangjie
>     // a.cj
>     package A
>     public class ClassWithLongNameA {
>
>     }
>     class ClassWithLongNameB {
>
>     }
>
>     public type classA = ClassWithLongNameA            // ok
>     type classAInter = ClassWithLongNameA            // ok
>
>     /* error: classB can not be declared with modifier 'public', as 'ClassWithLongNameB' is internal */
>     public type classB = ClassWithLongNameB
>
>     // b.cj
>     package B
>     import A.*
>     let myClassA: A.classA = ClassWithLongNameA()
>     ```

仓颉中`type`定义类型别名, 可以用可见性修饰符进行修饰

#### 类型别名的使用

> 类型别名可以用在任何等号右手边它指向的原类型能够使用的位置:
>
> 1. 作为类型使用, 例如:
>
>     ```cangjie
>     type A = B
>     class B {}
>     var a: A = B()// Use typealias A as type B
>     ```
>
> 2. 当类型别名实际指向的类型为`class`、`struct`时, **可以作为构造器名称使用**
>
>     ```cangjie
>     type A = B
>     class B {}
>     func foo(){ A()}  // Use type alias A as constructor of B
>     ```
>
> 3. 当类型别名实际指向的类型为`class`、`interface`、`struct`时, 可以**作为访问内部静态成员变量或函数的类型名**
>
>     ```cangjie
>     type A = B
>     class B {
>         static var b : Int32 = 0;
>         static func foo(){}
>     }
>     func foo(){
>         A.foo()// Use A to access static method in class B
>         A.b
>     }
>     ```
>
> 4. 当类型别名实际指向的类型为`enum`时, 可以作为`enum`声明的构造器的类型名
>
>     ```cangjie
>     enum TimeUnit {
>         Day | Month | Year
>     }
>     type Time = TimeUnit
>     var a = Time.Day
>     var b = Time.Month   // Use type alias Time to access constructors in TimeUnit
>     ```

### 类型间的关系

> 类型间的关系有两种: 相等和子类型

C/C++中, 只有存在继承关系的类之间又一些关系

看来仓颉的类型之间存在许多的关系

#### 类型相等

> 对于任意两个类型`T1`和`T2`, 如果它们满足以下任一条件, 则称`T1`和`T2`相等(记为`T1 === T2`):
>
> - 存在类型别名定义`type T1 = T2`;
> - 在`class`定义的内部和`class`的`extend`内部, `T1`是`class`的名字, `T2`是`This`;
> - `T1`和`T2`的名字完全相同(自反性);
> - `T2 === T1`(对称性);
> - 存在类型`Tk`, 满足`T1 === Tk`且`Tk === T2`(传递性);

上面的所有情况的结论都是, `T1 === T2`, 也就是说每条之后都要加一句`T1 === T2`

第一眼只看每条句子, 一下子没看懂

#### 子类型

> 对于任意两个类型`T1`和`T2`, 如果它们满足以下任一条件, 则称`T1`是`T2`的子类型(记为`T1 <: T2`):
>
> - `T1 === T2`;
> - `T1`是`Nothing`类型;
> - `T1`和`T2`均是`Tuple`类型, 并且`T1`每个位置处的类型都是`T2`对应位置处类型的子类型;
> - `T1`和`T2`均是`Function`类型, 并且`T2`的参数类型是`T1`参数类型的子类型, `T1`的返回类型是`T2`返回类型的子类型;
> - `T1`是任意`class`类型, `T2`是`Object`类型;
> - `T1`和`T2`均是`interface`类型, 并且`T1`继承了`T2`;
> - `T1`和`T2`均是`class`类型, 并且`T1`继承了`T2`;
> - `T2`是`interface`类型, 并且`T1`实现了`T2`;
> - 存在类型`Tk`, 满足`T1 <: Tk`且`Tk <: T2`(传递性)

只从说明可以看出, 仓颉中:

1. `Nothing`是所有类型的子类型

2. `Object`是所有`class`的子类型

3. 存在继承关系的`class`和`interface`类型, 被继承的类型(C++中的基类)是父类型, 子类是子类型

    即, 如果`T1`继承了`T2`, `T1`属于子类型

4. 如果两个`interface`, `T1`实现了`T2`, `T1`是`T2`的子类型

    可以看作是`T2`生出了`T1`

#### 最小公共父类型

> 在有子类型的类型系统里, 有时会遇到需要求两个类型的最小公共父类型的情形, 例如 **`if`表达式的类型便是其两个分支的类型的最小公共父类型**, `match`表达式类似
>
> 两个类型的最小公共父类型, 是其公共父类型中**最小**的一个
>
> **最小意味着它是其他所有公共父类型的子类型**
>
> 最小公共父类型定义如下:
>
> 对于任意两个类型`T1`和`T2`, 如果类型`LUB`满足如下规则, 则`LUB`是`T1`和`T2`的最小公共父类型:
>
> - 对于同时满足`T1 <: T`和`T2 <: T`的任意类型`T`, `LUB <: T`也成立
>
> 注意, 如果公共父类型中的某个类型不比其他类型大, 它只是极小的, 并不一定是最小的

参考数学中的集合

#### 最大公共子类型

> 因为子类型关系中存在逆变(定义参考泛型章节下的类型型变)的情形, 如函数类型的参数类型是逆变的, 此时会需要求两个类型的最大公共子类型
>
> 两个类型的最大公共子类型, 是其公共子类型中**最大**的一个
>
> **最大意味着它是其他所有公共子类型的父类型**
>
> 最大公共子类型定义如下: 对于任意两个类型`T1`和`T2`, 如果类型`GLB`满足如下规则, 则`GLB`是`T1`和`T2`的最大公共子类型:
>
> - 对于同时满足`T <: T1`和`T <: T2`的任意类型`T`, `T <: GLB`也成立
>
> 注意, 如果公共子类型中的某个类型不比其他类型小, 它只是极大的, 并不一定是最大的

同样参考数学中的集合

### 类型安全

> **在没有数据竞争的情况下, 编译器保证内存安全和类型安全**

> 下面是一个类型安全和内存安全都 **得不到保证** 的例子:
>
> ```cangjie
> class C {
>     var x = 1
>     var y = 2
>     var z = 3
> }
>
> enum E {
>     A(Int64)| B(C)
> }
>
> var e = A(1)
>
> main(){
>     spawn {
>         while (true){
>             e = B(C())     // writing to`e`
>             e = A(0)
>         }
>     }
>     while (true){
>         match (e){         // reading from`e`
>             case A(n)=>
>                 println(n+1)
>             case B(c)=>
>                 c.x = 2
>                 c.x = 3
>                 c.x = 4
>         }
>     }
> }
> ```
>
> 不保证是因为 对变量`e`赋值的线程和读取该变量的线程之间存在**数据竞争**
>
> 有关数据竞争的更多信息, 请参见第 15 章 "并发"
