---
title: "仓颉文档阅读-语言规约VI: 类和接口(I)"
published: 2025-10-04 00:59:16
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

## 类和接口

任何面向对象的编程语言, 基本都存在类这个概念, C++不例外, 仓颉当然也不例外

但, C++ 中不存在接口这个东西, 接口这个词, 在业务方面听起来不陌生, 但是语言中的接口具体情况并不是非常了解

### 类

#### 类的定义

> 在仓颉编程语言中, 通过关键字`class`定义一个类, 类定义包含以下几个部分:
>
> - 可选的修饰符
>
> -`class`关键字
>
> - 类名, 必须是合法的`identifier`
>
> - 可选的类型参数
>
> - 可选的指定父类或者父接口(写在`<:`后面, 使用`&`分隔), 如果存在父类, 父类必须写在第一个, 否则编译报错;
>
> - 可选的泛型约束
>
> - 类体
>
> 类只能定义在源文件顶层, 类定义的语法如下:
>
> ```
> classDefinition
>      : classModifierList? 'class' identifier
>      typeParameters?
>      ('<:' superClassOrInterfaces)?
>      genericConstraints?
>      classBody
>      ;
> superClassOrInterfaces
>     : classType ('&' superInterfaces)?
>     | superInterfaces
>     ;
> ```
>
> 类定义的例子如下:
>
> ```cangjie
> interface I1<X> {}
> interface I2 {}
> open class A{}
>
> // 下面的类 B 继承了类 A, 并实现了接口 I2
> open class B <: A & I2 {}
>
> /* 以下类 C 声明了 1 个类型参数 U. 它继承自 B, 并实现了接口 I1<U>和 I2. 此外, 它的类型参数有约束 U <: A */
> class C<U> <: B & I1<U> & I2 where U <: A {}
> ```

文档中说: 如果继承, 且继承存在父类, 则父类必须写在`<:`之后的第一个, 否则编译报错

那么也就可以推测, 仓颉的`class`类是禁止多继承的, 只能单继承

但是, `class`可以实现多个接口, 所以之后可以使用`&`将所有实现的接口继承下来, 但是如果继承类 只能单继承

##### 类的修饰符

> 在此小节中, 我们将介绍定义类可以使用的修饰符

###### 访问修饰符

> 类可以被所有访问修饰符修饰, 默认的可访问性为`internal`
>
> 详细内容请参考包和模块管理章节 访问修饰符

访问修饰符还是要在 包和模块管理 章节了解

###### 继承性修饰符

> -`open`: 类用`open`修饰, 表示允许其他类从这个类继承
>
>    ```cangjie
>     /* 以下类使用修饰符`open`声明, 表示它可以被继承 */
>     open class C1 {
>         func foo(): Unit {
>             return
>         }
>     }
>
>     class C2 <: C1 {}
>    ```
> 需要注意的是:
>
> > 没有用`open`修饰的非抽象类不能被任何类继承

仓颉中的普通类, 默认是禁止被继承的, 使用`open`修饰, 表示此类可以被继承

> -`sealed`: 修饰符表示只能在`class`定义所在的包内 继承该`class`
>
>     -`sealed`已经蕴含了`public`的语义, 所以定义`sealed class`时的`public`修饰符是可选的
>
>     -`sealed`的子类可以不是`sealed`类, 仍可被`open/sealed`修饰, 或不使用任何继承性修饰符
>
>       若`sealed`类的子类同时被`public`和`open`修饰, 则其子类可在包外被继承
>
>     -`sealed`的子类可以不被`public`修饰
>
>    ```cangjie
>     // package A
>     package A
>     public sealed class C1 {}         // OK
>     sealed class C2 {}                // OK, 使用 'sealed' 时, 'public' 是可选的
>
>     class S1 <: C1 {}  // OK
>     public open class S2 <: C1 {}     // OK
>     public sealed class S3 <: C1 {}   // OK
>     open class S4 <: C1 {}            // OK
>
>     // package B
>     package B
>     import A.*
>
>     class SS1 <: S2 {}                // OK
>     class SS2 <: S3 {}                // Error, S3 是一个密封类, 不能在此处继承
>    ```
>
> 需要注意的是:
>
> > 没有用`open`或`sealed`修饰的非抽象类不能被任何类继承

仓颉中, `class`默认禁止被继承, 如果用`open`修饰, 则表示可以被继承, 只要能看到这个类; 如果用`sealed`修饰, 则表示可以在所在包中被继承, 不能在其他包中被继承, 即使其他包中可以看到这个类

`sealed`的子类, 可以被`open`修饰, 可以将子类暴露在包外, 在包外进行继承

<Info>

经测试, `sealed`修饰符只能用于抽象类!!!!!

不禁怀疑, Cangjie 0.53.18的语法规约是不是有一定的错误还没有修正?

不过也有可能, 这只是非常简单的语法上介绍的内容, 具体的语义等行为, 要在开发指南中进行具体分析介绍

</Info>

###### 抽象类修饰符

> `abstract`: 该类属于抽象类, 与普通类不同的是, 在抽象类中除了可以定义普通的函数, 还允许声明抽象函数, 只有用该修饰符修饰的类才为抽象类
>
> 如果一个函数没有函数体, 我们称其为抽象函数
>
> 需要注意的是:
>
> - 抽象类`abstract`修饰符已经包含了可被继承的语义, 因此抽象类定义时的`open`修饰符是可选的, 也可以使用`sealed`修饰符修饰抽象类, 表示该抽象类只能在本包被继承
>
> - 抽象类中禁止定义`private`的抽象函数
>
> - 不能为抽象类创建实例
>
> - 允许抽象类的抽象子类不实现父类中的抽象函数
>
> - 抽象类的非抽象子类必须实现父类中的所有抽象函数

仓颉中的抽象类, 要用`abstract`修饰符来修饰, 而不是像C++中: 如果类内定义了纯虚函数, 那么这个类就是抽象类

并且, 仓颉中, **只有抽象类内能够定义抽象函数, 普通类中无法定义抽象函数**

抽象函数类似与纯虚函数, 即 没有函数体实现的函数

##### 类继承

> **类只支持单继承**
>
> 类通过`<: superClass`的方式来指定当前类的直接父类(父类是某个已经定义了的类)
>
> 以下示例展示了类C2继承类C1的语法:
>
> ```cangjie
> open class C1 {}
> class C2 <: C1 {}
> ```
>
> 父类可以是一个泛型类, 只需要在继承时提供合法的类型即可
>
> 例如: 非泛型类C2与泛型类C3继承泛型类C1:
>
> ```cangjie
> open class C1<T> {}
> class C2 <: C1<Int32> {}
> class C3<U> <: C1<U> {}
> ```
>
> 所有类都有一个父类, 对于没有定义父类的类, 默认其父类为`Object`
>
> `Object`例外, 没有父类
>
> ```cangjie
> class Empty {} 	//  隐式继承自 Object:  class Empty <: Object {}
> ```
>
> 类仅支持单继承, 以下示例中的语法将引起编译错误
>
> ```cangjie
> open class C1 {}
> open class C2 {}
> class C3 <: C1 & C2 {}		// Error: 不支持多继承
> ```
>
> 当一个类继承另一个类时, 将被继承的类称为父类, 将产生继承行为的类称为子类
>
> ```cangjie
> open class C1 {}			// C1 is superclass
> class C2 <: C1 {}			// C2 is subclass
> ```
>
> 子类将继承父类的所有成员, **私有成员和构造函数除外**
>
> 子类可以直接访问父类的成员, 但是在覆盖时, 将不能直接通过名字来访问父类被覆盖的实例成员
>
> 这时可以通过`super`来指定(`super`指向的是当前类对象的直接父类)或创建对象并通过对象来访问

仓颉中, 类的继承使用的符号是`<:`

仓颉中的类只支持单继承, 与刚开始的推断是吻合的

其次, 仓颉中所有的类都存在父类, 如果没有定义时没有显式指明, 则隐式继承于`Object`

仓颉中类除了存在`this`表示当前类对象, 还存在`super`表示当前类父类对象

##### 实现接口

> 类支持实现一个或多个接口, 通过`<: I1 & I2 & ... & Ik`的方式声明当前类想要实现的接口, 多个接口之间用`&`分隔
>
> 如果当前类也指定了父类, 则接口需要出现在父类后面
>
> 例如:
>
> ```cangjie
> interface I1 {}
> interface I2 {}
>
> // 类 C1 实现了接口 I1
> open class C1 <: I1 {}
>
> // 类 C2 继承类 C1 并实现接口 I1、I2
> class C2 <: C1 & I1 & I2 {}
> ```
>
> 接口也可以是泛型的, 此时在实现泛型接口时需要给出合法的类型参数
>
> 例如:
>
> ```cangjie
> interface I1<T> {}
> interface I2<U> {}
> class C1 <: I1<Int32> & I2<Bool> {}
> class C2<K, V> <: I1<V> & I2<K> {}
> ```
>
> 当类型实现接口时, 对于非泛型接口不能直接实现多次, 对于泛型接口不能用同样的类型参数直接实现多次
>
> 例如:
>
> ```cangjie
> interface I1 {}
> class C1 <: I1 & I1 {}                    // error
>
> interface I2<T> {}
> class C2 <: I2<Int32> & I2<Int32> {}      // error
> class C3<T> <: I2<T> & I2<Int32> {}       // ok
> ```
>
> 如果上述定义的泛型类C3在使用时被应用了类型参数Int32, 导致重复实现了两个相同类型的接口, 那么编译器会在类型被使用到的位置报错:
>
> ```cangjie
> interface I1<T> {}
> open class C3<T> <: I1<T> & I1<Int32> {}  // ok
> var a: C3<Int32>                          // error
> var b = C3<Int32>()                       // error
> class C4 <: C3<Int32> {}                  // error
> ```
>
> 关于接口的详细描述在章节[接口]

仓颉中, 类实现接口 与 继承类似, 都是使用符号`<:`, 且可以同时进行

但, 只能单继承, 且父类要写在`<:`之后的最开始位置, 之后可以出现若干**不同的接口类型**, 每个接口通过`&`连接

实现的接口可以是泛型接口, 但是 具体的**接口类型不能相同**

##### 类体

> 类体表示当前类包括的内容, 类体由大括号包围, 包含以下内容:
>
> - 可选的静态初始化器
>
> - 可选的主构造函数
>
> - 可选的构造函数
>
> - 可选的成员变量定义
>
> - 可选的成员函数、成员操作符函数定义或声明
>
> - 可选的成员属性定义或声明
>
> - 可选的宏调用表达式
>
> - 可选的类终结器
>
> 类体的语法定义如下:
>
> ```
> classBody
>     : '{'
>            classMemberDeclaration*
>            classPrimaryInit?
>            classMemberDeclaration*
>        '}'
>     ;
>
> classMemberDeclaration
>     : classInit
>     | staticInit
>     | variableDeclaration
>     | functionDefinition
>     | operatorFunctionDefinition
>     | macroExpression
>     | propertyDefinition
>     | classFinalizer
>     ;
> ```
>
> 上述类体的语法定义中, 包含以下内容:
>
> `staticInit`代表一个静态初始化器的定义, 一个类最多只能定义一个静态初始化器;
>
> `classPrimaryInit`指的是主构造函数的定义, 一个类最多只能定义一个;
>
> `classInit`表示`init`构造函数的定义;
>
> `variableDeclaration`表示成员变量的声明;
>
> `operatorFunctionDefinition`表示操作符重载成员函数的定义;
>
> `macroExpression`表示宏调用表达式, 宏展开后依然要符合`classMemberDeclaration`的语法定义;
>
> `propertyDefinition`表示属性的定义;
>
> `classFinalizer`表示类的终结器的定义;
>
> 类体中引入的定义或声明均属于类的成员, 将在[类的成员]章节中详细介绍
