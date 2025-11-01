---
title: "仓颉文档阅读-语言规约XVII: 注解"
published: 2025-10-21 11:51:11
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

## 注解

> 注解(Annotation)是一种只用在声明上的特殊语法，它用来给编译器或者运行时 提供额外信息 来**实现特定的功能**
>
> 注解最基本的语法示例如下:
>
> ```cangjie
> @Annotation1[arg1, arg2]
> @Annotation2
> class Foo {}
> ```
>
> 注解可以用在不同的声明上
>
> 其语法规格定义为:
>
> ```
> annotationList: annotation+;
>
> annotation
>     : '@' (identifier '.')* identifier ('[' annotationArgumentList ']')?
>     ;
>
> annotationArgumentList
>     : annotationArgument (',' annotationArgument)*  ','?
>     ;
>
> annotationArgument
>     : identifier ':' expression
>     | expression
>     ;
> ```
>
> **注解和宏调用使用了一样的语法**
>
> 由于**宏处理的阶段比注解更早**，编译器在解析时会先尝试将该语法当作宏处理
>
> 如果作为宏不成功则继续尝试将该语法作为注解处理
>
> 如果两种处理方式都不成功则应该编译报错

注解, 又是一个C++原生不支持的语法

对于 算是只接触过C/C++的我来说, 注解的细节基本处于一种未知的状态

只知道, 它可以限制一些功能的实现等

仓颉中, **注解和宏调用使用同一种语法:`@标识符[参数列表]`**

### 自定义注解

> 自定义注解机制用来**让反射获取标注内容**，目的是在类型元数据之外提供更多的有用信息，以支持更复杂的逻辑
>
> 自定义注解可以用在**类型声明**、**成员变量声明**、**成员属性声明**、**成员函数声明**、**构造函数声明**、**构造器声明**、(前面提到的函数里的)**函数参数声明**上
>
> 开发者可以通过 **自定义类型标注`@Annotation`方式**创建自己的自定义注解
>
> `@Annotation`只能修饰`class`，并且不能是`abstract`或`open`或`sealed`修饰的`class`
>
> 当一个`class`声明它标注了`@Annotation`，那么它**必须要提供至少一个`const init`函数**，否则编译器会报错
>
> 下面是一个自定义注解的例子:
>
> ```cangjie
> @Annotation
> public class CustomAnnotation {
>     let name: String
>     let version: Int64
>
>     public const init(name: String, version!: Int64 = 0) {
>         this.name = name
>         this.version = version
>     }
> }
>
> // 使用标注
> @CustomAnnotation["Sample", version: 1]
> class Foo {}
> ```
>
> 注解信息需要在编译时生成信息绑定到类型上，自定义注解在使用时必须使用`const init`构建出合法的实例
>
> 我们规定:
>
> - 注解声明语法与声明宏语法一致，后面的`[]`括号中需要按顺序或命名参数规则传入参数，且参数必须是`const`表达式
>
> - 对于拥有无参构造函数的注解类型，声明时允许省略括号
>
> ```cangjie
> @Annotation
> public class MyAnnotation { ... }
>
> @MyAnnotation                         // ok
> class Foo {}
> ```
>
> 对于同一个注解目标，同一个注解类**不允许声明多次**，即 不可重复
>
> 编译器不承诺生成的多个`Annotation`元数据的顺序跟代码中`Annotation`的标注顺序保持一致
>
> ```cangjie
> @MyAnnotation
> @MyAnnotation                         // error
> class Foo {}
>
> @MyAnnotation
> @MyAnnotation2
> class Bar {}
> ```
>
> `Annotation`不会被继承，因此一个类型的注解元数据只会来自它定义时声明的注解
>
> 如果需要父类型的注解元数据信息，需要开发者自己用反射接口查询
>
> ```cangjie
> @BaseAnno
> open class Base {}
>
> @InterfaceAnno
> interface I {}
>
> @SubAnno
> class Sub <: Base & I {}              // Sub 只有 SubAnno 注解信息
> ```

当单独使用`@Annotation`修饰一个类时, 此类就是一个注解类, 需要拥有一个`const init`, 且此构造函数的参数列表就是注解可以传入的参数列表

自定义了注解类之后, 就可以**通过类名使用自定义注解**了, 不允许多次重复对同一目标使用同一注解

自定义注解不会被继承

自定义注解, 可通过反射机制实现具体功能, 这是一个C++不原生具备的语法

> 自定义注解可以限制自己可以使用的位置，这样可以减少开发者的误用，这类注解需要在声明`@Annotation`时标注`target`参数
>
> `target`参数需要以变长参数的形式传入该自定义注解希望支持的位置，`target`接收的参数是一个`Array<AnnotationKind>`
>
> 可以限制的位置包括:
>
> - 类型声明(`class`、`struct`、`enum`、`interface`)
>
> - 成员函数/构造函数中的参数
>
> - 构造函数声明
>
> - 成员函数声明
>
> - 成员变量声明
>
> - 成员属性声明
>
> ```cangjie
> public enum AnnotaitionKind {
>     | Type
>     | Parameter
>     | Init
>     | MemberProperty
>     | MemberFunction
>     | MemberVariable
> }
>
> @Annotation[target: [Type, MemberFunction]]
> class CustomAnnotation{}
> @Annotation
> class SubCustomAnnotation{}
>
> @Annotation[target: []]
> class MyAnno{}
> ```
>
> 当没有限定`target`的时候，该自定义注解可以用在以上全部位置
>
> 当限定`target`时，只能用在声明的列表中

自定义注解, 可以在定义时通过在参数列表中设置`参数target`的值, 来限制此注解的使用位置

即:`@Annotation[target: [Array<AnnotationKind>]]`, `AnnotationKind`是仓颉内置的一个枚举类型
