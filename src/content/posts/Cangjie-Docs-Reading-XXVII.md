---
title: "仓颉文档阅读-语言规约XVIII: 仓颉语法"
published: 2025-10-21 15:56:11
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

## 仓颉语法

本文的内容, 都是仓颉语法的  词法规则, 与语言特性无关

### 词法

#### 注释

```
DelimitedComment
    : '/*' ( DelimitedComment | . )*? '*/'
    ;

LineComment
    : '//' ~[\u000A\u000D]*
    ;
```

```cangjie
/* 多行注释 */

// 单行注释
```

#### 空白和换行

```
WS
    : [\u0020\u0009\u000C]
    ;

NL: '\u000A' | '\u000D' '\u000A' ;
```

```cangjie
// 空格(\u0020)

// 制表符(\u0009)

// 换页符(\u000C)


// Uinx/Linux风格换行 LF(\u000A)

// Windows风格换行 CRLF(\u000D \u000A)

```


#### 符号

```
DOT: '.' ;
COMMA: ',' ;
LPAREN: '(' ;
RPAREN: ')' ;
LSQUARE: '[' ;
RSQUARE: ']' ;
LCURL: '{' ;
RCURL: '}' ;
EXP: '**' ;
MUL: '*' ;
MOD: '%' ;
DIV: '/' ;
ADD: '+' ;
SUB: '-' ;
PIPELINE: '|>' ;
COMPOSITION: '~>' ;
INC: '++' ;
DEC: '--' ;
AND: '&&' ;
OR: '||' ;
NOT: '!' ;
BITAND: '&' ;
BITOR: '|' ;
BITXOR: '^' ;
LSHIFT: '<<' ;
RSHIFT: '>>' ;
COLON: ':' ;
SEMI: ';' ;
ASSIGN: '=' ;
ADD_ASSIGN: '+=' ;
SUB_ASSIGN: '-=' ;
MUL_ASSIGN: '*=' ;
EXP_ASSIGN: '**=' ;
DIV_ASSIGN: '/=' ;
MOD_ASSIGN: '%=' ;
AND_ASSIGN: '&&=' ;
OR_ASSIGN: '||=' ;
BITAND_ASSIGN: '&=' ;
BITOR_ASSIGN: '|=' ;
BITXOR_ASSIGN: '^=' ;
LSHIFT_ASSIGN: '<<=' ;
RSHIFT_ASSIGN: '>>=' ;
ARROW: '->' ;
BACKARROW: '<-' ;
DOUBLE_ARROW: '=>' ;
ELLIPSIS: '...' ;
CLOSEDRANGEOP: '..=' ;
RANGEOP: '..' ;
HASH: '#' ;
AT: '@' ;
QUEST: '?' ;
UPPERBOUND: '<:';
LT: '<' ;
GT: '>' ;
LE: '<=' ;
GE: '>=' ;
NOTEQUAL: '!=' ;
EQUAL: '==' ;
WILDCARD: '_' ;
BACKSLASH: '\\' ;
QUOTESYMBOL: '`';
DOLLAR: '$';
QUOTE_OPEN: '"' ;
TRIPLE_QUOTE_OPEN: '"""' NL;
QUOTE_CLOSE: '"' ;
TRIPLE_QUOTE_CLOSE: '"""' ;
LineStrExprStart: '${' ;
MultiLineStrExprStart: '${' ;
```

#### 关键字

```
INT8: 'Int8' ;
INT16: 'Int16' ;
INT32: 'Int32' ;
INT64: 'Int64' ;
INTNATIVE: 'IntNative' ;
UINT8: 'UInt8' ;
UINT16: 'UInt16' ;
UINT32: 'UInt32' ;
UINT64: 'UInt64' ;
UINTNATIVE: 'UIntNative' ;
FLOAT16: 'Float16' ;
FLOAT32: 'Float32' ;
FLOAT64: 'Float64' ;
RUNE: 'Rune' ;
BOOLEAN: 'Bool' ;
UNIT: 'Unit' ;
Nothing: 'Nothing' ;
STRUCT: 'struct' ;
ENUM: 'enum' ;
THISTYPE: 'This';
PACKAGE: 'package' ;
IMPORT: 'import' ;
CLASS: 'class' ;
INTERFACE: 'interface' ;
FUNC: 'func';
MAIN: 'main';
LET: 'let' ;
VAR: 'var' ;
CONST: 'const' ;
TYPE_ALIAS: 'type' ;
INIT: 'init'  ;
THIS: 'this' ;
SUPER: 'super' ;
IF: 'if' ;
ELSE: 'else' ;
CASE: 'case' ;
TRY: 'try' ;
CATCH: 'catch' ;
FINALLY: 'finally' ;
FOR: 'for' ;
DO: 'do' ;
WHILE: 'while' ;
THROW: 'throw' ;
RETURN: 'return' ;
CONTINUE: 'continue' ;
BREAK: 'break' ;
IS: 'is' ;
AS: 'as' ;
IN: 'in' ;
MATCH: 'match' ;
FROM: 'from' ;
WHERE: 'where';
EXTEND: 'extend';
SPAWN: 'spawn';
SYNCHRONIZED: 'synchronized';
MACRO: 'macro';
QUOTE: 'quote';
TRUE: 'true';
FALSE: 'false';
STATIC: 'static';
PUBLIC: 'public' ;
PRIVATE: 'private' ;
PROTECTED: 'protected' ;
OVERRIDE: 'override' ;
REDEF: 'redef' ;
ABSTRACT: 'abstract' ;
OPEN: 'open' ;
OPERATOR: 'operator' ;
FOREIGN: 'foreign';
INOUT: 'inout';
PROP: 'prop';
MUT: 'mut';
UNSAFE: 'unsafe';
GET: 'get';
SET: 'set';
```

#### 字面量

```
IntegerLiteralSuffix
   : 'i8' |'i16' |'i32' |'i64' |'u8' |'u16' |'u32' | 'u64'
   ;

IntegerLiteral
   : BinaryLiteral IntegerLiteralSuffix?
   | OctalLiteral IntegerLiteralSuffix?
   | DecimalLiteral '_'* IntegerLiteralSuffix?
   | HexadecimalLiteral IntegerLiteralSuffix?
   ;
BinaryLiteral
    : '0' [bB] BinDigit (BinDigit | '_')*
    ;
BinDigit
    : [01]
    ;
OctalLiteral
    : '0' [oO] OctalDigit (OctalDigit | '_')*
    ;
OctalDigit
    : [0-7]
    ;
DecimalLiteral
    : (DecimalDigitWithOutZero (DecimalDigit | '_')*) | DecimalDigit
    ;
fragment DecimalFragment
    : DecimalDigit (DecimalDigit | '_')*
    ;
fragment DecimalDigit
    : [0-9]
    ;
fragment DecimalDigitWithOutZero
    : [1-9]
    ;
HexadecimalLiteral
    : '0' [xX] HexadecimalDigits
    ;

HexadecimalDigits
    : HexadecimalDigit (HexadecimalDigit | '_')*
    ;

HexadecimalDigit
    : [0-9a-fA-F]
    ;

FloatLiteralSuffix
    : 'f16' | 'f32' | 'f64'
    ;

FloatLiteral
    : (DecimalLiteral DecimalExponent | DecimalFraction DecimalExponent? | (DecimalLiteral DecimalFraction) DecimalExponent?)  FloatLiteralSuffix?
    | ( Hexadecimalprefix (HexadecimalDigits | HexadecimalFraction | (HexadecimalDigits HexadecimalFraction)) HexadecimalExponent)
    ;

fragment DecimalFraction : '.' DecimalFragment ;
fragment DecimalExponent : FloatE Sign? DecimalFragment  ;
fragment Sign : [-] ;
fragment Hexadecimalprefix : '0' [xX] ;


DecimalFraction : '.' DecimalLiteral ;
DecimalExponent : FloatE Sign? DecimalLiteral  ;
HexadecimalFraction : '.' HexadecimalDigits ;
HexadecimalExponent : FloatP Sign? DecimalFragment ;
FloatE : [eE] ;
FloatP : [pP] ;
Sign : [-] ;
Hexadecimalprefix : '0' [xX] ;

RuneLiteral
    : '\'' (SingleChar | EscapeSeq ) '\''
    ;

SingleChar
    :	~['\\\r\n]
    ;

EscapeSeq
    : UniCharacterLiteral
    | EscapedIdentifier
    ;

UniCharacterLiteral
    : '\\' 'u' '{' HexadecimalDigit '}'
    | '\\' 'u' '{' HexadecimalDigit HexadecimalDigit '}'
    | '\\' 'u' '{' HexadecimalDigit HexadecimalDigit HexadecimalDigit '}'
    | '\\' 'u' '{' HexadecimalDigit HexadecimalDigit HexadecimalDigit HexadecimalDigit '}'
    | '\\' 'u' '{' HexadecimalDigit HexadecimalDigit HexadecimalDigit HexadecimalDigit HexadecimalDigit '}'
    | '\\' 'u' '{' HexadecimalDigit HexadecimalDigit HexadecimalDigit HexadecimalDigit HexadecimalDigit HexadecimalDigit '}'
    | '\\' 'u' '{' HexadecimalDigit HexadecimalDigit HexadecimalDigit HexadecimalDigit HexadecimalDigit HexadecimalDigit HexadecimalDigit '}'
    | '\\' 'u' '{' HexadecimalDigit HexadecimalDigit HexadecimalDigit HexadecimalDigit HexadecimalDigit HexadecimalDigit HexadecimalDigit HexadecimalDigit '}'
    ;

EscapedIdentifier
    : '\\' ('t' | 'b' | 'r' | 'n' | '\'' | '"' | '\\' | 'f' | 'v' | '0' | '$')
    ;

ByteLiteral
    : 'b' '\'' (SingleCharByte | ByteEscapeSeq) '\''
    ;

ByteEscapeSeq
    : HexCharByte
    | ByteEscapedIdentifier
    ;

SingleCharByte
    // ASCII 0x00~0x7F without \n \r \' \" \\
    // +-------+-----+-----+
    // | Rune  | Hex | Dec |
    // +-------+-----+-----+
    // | \n    |  0A |  10 |
    // | \r    |  0D |  13 |
    // | \"    |  22 |  34 |
    // | \'    |  27 |  39 |
    // | \\    |  5C |  92 |
    // +-------+-----+-----+
    : [\u0000-\u0009\u000B\u000C\u000E-\u0021\u0023-\u0026\u0028-\u005B\u005D-\u007F]
    ;

fragment ByteEscapedIdentifier
    : '\\' ('t' | 'b' | 'r' | 'n' | '\'' | '"' | '\\' | 'f' | 'v' | '0')
    ;

fragment HexCharByte
    : '\\' 'u' '{' HexadecimalDigit '}'
    | '\\' 'u' '{' HexadecimalDigit HexadecimalDigit '}'
    ;

ByteStringArrayLiteral
    : 'b' '"' (SingleCharByte | ByteEscapeSeq)* '"'
    ;

JStringLiteral
    : 'J' '"' (SingleChar | EscapeSeq)* '"'
    ;

LineStrText
    : ~["\\\r\n]
    | EscapeSeq
    ;

TRIPLE_QUOTE_CLOSE
    : MultiLineStringQuote? '"""' ;

MultiLineStringQuote
    : '"'+
    ;

MultiLineStrText
    : ~('\\')
    | EscapeSeq
    ;

MultiLineRawStringLiteral
    : MultiLineRawStringContent
    ;

fragment MultiLineRawStringContent
    : HASH MultiLineRawStringContent HASH
    | HASH '"' .*? '"' HASH
    ;
```

```cangjie
// 整型字面量
// 二进制整型
0b1010
0B1010
// 八进制整型
0o755
0O755
// 十进制整型
42
1_000_000
// 十六进制整型
0xFF
0XFF
// 可加后缀
i8 | i16 | i32 | i64
u8 | u16 | u32 | u64


// 浮点数字面量
// 十进制浮点
3.14
6.02e23
1e-10f64
// 十六进制浮点
0x1.FFFFp0
0x1p-10
// 可加后缀
f16 | f32 | f64


// 字符字面量
// 普通字符
'A'
// 转义字符：
'\n'
'\u{1F600}'
'\''


// 字节字面量
// ASCII字符
b'x'
// 十六进制值
b'\u{78}'
// 转义序列
b'\n'

// 字节字符串字面量
b"Hello\x00World"


// J字符串字面量
J"包含\n换行的字符串"


// 单行字符串字面量
"包含\"引号\"的字符串"
'包含\'引号\'的字符串'


// 多行字符串字面量
"""
春眠不觉晓，
处处闻啼鸟。
"""


// 多行原始字符串字面量
#"
This is a \raw string
\n will NOT be escaped
"#


// 插值字符串
"你好, ${name}"
```

等

#### 标识符

```
identifier
    : Identifier
    | PUBLIC
    | PRIVATE
    | PROTECTED
    | OVERRIDE
    | ABSTRACT
    | OPEN
    | REDEF
    | GET
    | SET
    ;

Identifier
    : '_'* Letter (Letter | '_' | DecimalDigit)*
    | '`' '_'* Letter (Letter | '_' | DecimalDigit)* '`'
    ;

Letter
    : [a-zA-Z]
    ;

DollarIdentifier
    : '$' Identifier
    ;
```

```cangjie
// 普通标识符
age, _count, MAX_1

// 原始标识符
`class`, `_`
`public`

// $标识符
$temp, $_
```

### 语法

#### 编译单元

```
translationUnit
    : NL* preamble end* topLevelObject* (end+ mainDefinition)? NL* (topLevelObject (end+ topLevelObject?)*)? EOF
    ;

end
    : NL | SEMI
    ;
```

```cangjie
// 最小文件
main() {}

// 完整结构
package com.example
import std.io.*

let x = 42

public class A {}

main() {
    println(x)
}

func helper() {}
```

#### 包定义和包导入

```
preamble
    : packageHeader? importList*
    ;

packageHeader
    : PACKAGE NL* packageNameIdentifier end+
    ;

packageNameIdentifier
    : identifier (NL* DOT NL* identifier)*
    ;

importList
    : (FROM NL* identifier)? NL* IMPORT NL* importAllOrSpecified
    (NL* COMMA NL* importAllOrSpecified)* end+
    ;

importAllOrSpecified
    : importAll
    | importSpecified (NL* importAlias)?
    ;

importSpecified
    : (identifier NL* DOT NL*)+ identifier
    ;

importAll
    : (identifier NL* DOT NL*)+ MUL
    ;

importAlias
    : AS NL* identifier
    ;
```

```cangjie
// 包的定义
package com.example         // 包声明后需换行或分号
package com.example.utils;  // 分号结尾

// 包的导入
package com.example

// 基础
import std.io.*
import com.utils.{Timer, Logger}
import net.Http as NetHttp

// 多行
import std.collections.{
    HashMap,
    HashSet as HS
}  // 多行导入需用逗号分隔
```

#### `top-level` 定义

```
topLevelObject
    : classDefinition
    | interfaceDefinition
    | functionDefinition
    | variableDeclaration
    | enumDefinition
    | structDefinition
    | typeAlias
    | extendDefinition
    | foreignDeclaration
    | macroDefinition
    | macroExpression
    ;
```

```cangjie
// classDefinition
public class Rectangle {}
// interfaceDefinition
interface Drawable {}
// functionDefinition
func max(a: Int64, b: Int64): Int64 {}
// variableDeclaration
let PI: Float64 = 3.1415926
var counter: Int64 = 0
// enumDefinition
public enum Color {
    RED, GREEN, BLUE
}
// structDefinition
struct Point {}
// typeAlias
type UserID = String
type Callback = (Int64) -> Unit
// extendDefinition
extend Int64 {}
// foreignDeclaration
foreign func foo(): Unit
// macroDefinition
public macro foo(x: Tokens): Tokens { x }
// macroExpression
@Twice(foo())
```

#### `class` 定义

```
classDefinition
    : (classModifierList NL*)? CLASS NL* identifier
    (NL* typeParameters NL*)?
    (NL* UPPERBOUND NL* superClassOrInterfaces)?
    (NL* genericConstraints)?
    NL* classBody
    ;

superClassOrInterfaces
    : superClass (NL* BITAND NL* superInterfaces)?
    | superInterfaces
    ;

classModifierList
    : classModifier+
    ;

classModifier
    : PUBLIC
    | PROTECTED
    | INTERNAL
    | PRIVATE
    | ABSTRACT
    | OPEN
    ;

typeParameters
    : LT NL* identifier (NL* COMMA NL* identifier)* NL* GT
    ;

superClass
    : classType
    ;

classType
    : (identifier NL* DOT  NL*)*  identifier (NL* typeParameters)?
    ;

typeArguments
    : LT NL* type (NL* COMMA NL* type)* NL* GT
    ;

superInterfaces
    : interfaceType (NL* COMMA NL* interfaceType )*
    ;

interfaceType
    : classType
    ;

genericConstraints
    : WHERE NL* (identifier | THISTYPE) NL* UPPERBOUND NL* upperBounds (NL* COMMA NL* (identifier | THISTYPE) NL* UPPERBOUND NL* upperBounds)*
    ;

upperBounds
    : type (NL* BITAND NL* type)*
    ;

classBody
    : LCURL end*
         classMemberDeclaration* NL*
         classPrimaryInit? NL*
         classMemberDeclaration* end* RCURL
    ;

classMemberDeclaration
    : (classInit
    | staticInit
    | variableDeclaration
    | functionDefinition
    | operatorFunctionDefinition
    | macroExpression
    | propertyDefinition
    ) end*
    ;

classInit
    : (classNonStaticMemberModifier | CONST NL*)? INIT NL* functionParameters NL* block
    ;

staticInit
    : STATIC INIT LPAREN RPAREN
    LCURL
    expressionOrDeclarations?
    RCURL
    ;

classPrimaryInit
    : (classNonStaticMemberModifier | CONST NL*)?  className NL* LPAREN NL*
          classPrimaryInitParamLists
       NL* RPAREN NL*
 	  LCURL NL*
 	       (SUPER callSuffix)? end
 	       expressionOrDeclarations?
 	  NL* RCURL
    ;

className
    : identifier
    ;

classPrimaryInitParamLists
    : unnamedParameterList (NL* COMMA NL* namedParameterList)? (NL* COMMA NL* classNamedInitParamList)?
    | unnamedParameterList (NL* COMMA NL* classUnnamedInitParamList)? (NL* COMMA NL* classNamedInitParamList)?
    | classUnnamedInitParamList (NL* COMMA NL* classNamedInitParamList)?
    | namedParameterList (NL* COMMA NL* classNamedInitParamList)?
    | classNamedInitParamList
    ;

classUnnamedInitParamList
    : classUnnamedInitParam (NL* COMMA NL* classUnnamedInitParam)*
    ;

classNamedInitParamList
    : classNamedInitParam (NL* COMMA NL* classNamedInitParam)*
    ;

classUnnamedInitParam
    : (classNonSMemberModifier NL*)? (LET | VAR) NL* identifier NL* COLON NL* type
    ;

classNamedInitParam
    : (classNonSMemberModifier NL*)? (LET | VAR) NL* identifier NL* NOT NL* COLON NL* type (NL* ASSIGN NL* expression)?
    ;

classNonStaticMemberModifier
    : PUBLIC
    | PRIVATE
    | PROTECTED
    | INTERNAL
    ;
```

```cangjie
public class SimpleClass <: BaseClass & IInterface {
    // 成员变量
    var value: Int64 = 10
    let name: String

    // 主构造函数
    SimpleClass(value: Int64, name!: String) {
        this.value = value
        this.name = name
    }

    init(name!: String) {
        this.name = name
    }

    // 方法
    public func printInfo(): Unit {
        println("${name}: ${value}")
    }

    // 实现接口
    public func print(): Unit {
        println("name: ${this.name}, value: ${this.value}")
    }
}
```

#### `interface` 定义

```
interfaceDefinition
    : (interfaceModifierList NL*)? INTERFACE NL* identifier
    (NL* typeParameters NL*)?
    (NL* UPPERBOUND NL* superInterfaces)?
    (NL* genericConstraints)?
    (NL* interfaceBody)
    ;

interfaceBody
    : LCURL end* interfaceMemberDeclaration* end* RCURL
    ;

interfaceMemberDeclaration
    : (functionDefinition
    | operatorFunctionDefinition
    | macroExpression
    | propertyDefinition) end*
    ;

interfaceModifierList
    : (interfaceModifier NL*)+
    ;

interfaceModifier
    : PUBLIC
    | PROTECTED
    | INTERNAL
    | PRIVATE
    | OPEN
    ;
```

```cangjie
internal interface SimpleInterface {
    // 抽象方法（无实现）
    func doSomething(): Unit

    // 带默认实现的方法
    func log(message: String) {
        println(message)
    }

    // 属性（只有getter）
    prop count: Int64
}
```

#### `function` 定义

```
functionDefinition
    :(functionModifierList NL*)? FUNC
    NL* identifier
    (NL* typeParameters NL*)?
    NL* functionParameters
    (NL* COLON NL* type)?
    (NL* genericConstraints)?
    (NL* block)?
    ;

operatorFunctionDefinition
    : (functionModifierList NL*)? OPERATOR NL* FUNC
    NL* overloadedOperators
    (NL* typeParameters NL*)?
    NL* functionParameters
    (NL* COLON NL* type)?
    (NL* genericConstraints)?
    (NL* block)?
    ;

functionParameters
    : (LPAREN (NL* unnamedParameterList ( NL* COMMA NL* namedParameterList)? )?
        NL* RPAREN NL*)
    | (LPAREN NL* (namedParameterList NL*)? RPAREN NL*)
    ;

nondefaultParameterList
    : unnamedParameter (NL* COMMA NL* unnamedParameter)*
        (NL* COMMA NL*  namedParameter)*
    | namedParameter (NL* COMMA NL* namedParameter)*
    ;

unnamedParameterList
    : unnamedParameter (NL* COMMA NL*  unnamedParameter)*
    ;

unnamedParameter
    : (identifier | WILDCARD) NL* COLON NL* type
    ;

namedParameterList
    : (namedParameter | defaultParameter)
      (NL* COMMA NL* (namedParameter | defaultParameter))*
    ;

namedParameter
    : identifier NL* NOT NL* COLON NL* type
    ;

defaultParameter
    : identifier NL* NOT NL* COLON NL* type NL* ASSIGN NL* expression
    ;

functionModifierList
    : (functionModifier NL*)+
    ;

functionModifier
    : PUBLIC
    | PRIVATE
    | PROTECTED
    | INTERNAL
    | STATIC
    | OPEN
    | OVERRIDE
    | OPERATOR
    | REDEF
    | MUT
    | UNSAFE
    | CONST
    ;
```

```cangjie
// 函数
public func sum( a: Int64, b!: Int64 = 0): Int64 {
    a + b
}

// lambda
let f1: (Int64, Int64)->Int64 = {a: Int64, b: Int64 => a + b}
```

#### 变量定义

```
variableDeclaration
    : variableModifier* NL* (LET | VAR | CONST) NL* patternsMaybeIrrefutable ( (NL* COLON NL* type)? (NL* ASSIGN NL* expression)
                                                                     | (NL* COLON NL* type)
                                                                     )
    ;

variableModifier
    : PUBLIC
    | PRIVATE
    | PROTECTED
    | INTERNAL
    | STATIC
    ;
```

```cangjie
var varVal: Int64 = 20
let letVal = 20
const constVal = 20
static const sConstVal = 20  // static 不能在 top-level、main()

```

#### `enum` 定义

```
enumDefinition
    : (enumModifier NL*)? ENUM NL* identifier (NL* typeParameters NL*)?
    (NL* UPPERBOUND NL* superInterfaces)?
    (NL* genericConstraints)? NL* LCURL end* enumBody end* RCURL
    ;

enumBody
    : (BITOR NL*)? caseBody (NL* BITOR NL* caseBody)*
    (NL*
    ( functionDefinition
    | operatorFunctionDefinition
    | propertyDefinition
    | macroExpression
    ))*
    ;

caseBody
    : identifier ( NL* LPAREN NL* type (NL* COMMA NL* type)* NL* RPAREN)?
    ;

enumModifier
    : PUBLIC
    | PROTECTED
    | INTERNAL
    | PRIVATE
    ;
```

```cangjie

internal enum StatusCode {
    | OK(Int64)
    | NotFound(Int64)
    | ServerError(Int64)

    // 成员方法
    public func isSuccess(): Bool {
        match (this) {
            case OK(_) => true
            case _ => false
        }
    }
}
```

#### `struct` 定义

```
structDefinition
    : (structModifier NL*)? STRUCT NL* identifier (NL* typeParameters NL*)?
    (NL* UPPERBOUND NL* superInterfaces)?
    (NL* genericConstraints)? NL* structBody
    ;

structBody
    : LCURL end*
        structMemberDeclaration* NL*
        structPrimaryInit? NL*
        structMemberDeclaration*
      end* RCURL
    ;

structMemberDeclaration
    : (structInit
    | staticInit
    | variableDeclaration
    | functionDefinition
    | operatorFunctionDefinition
    | macroExpression
    | propertyDefinition
    ) end*
    ;

structInit
    : (structNonStaticMemberModifier | CONST NL*)? INIT NL* functionParameters NL* block
    ;

staticInit
    : STATIC INIT LPAREN RPAREN
    LCURL
    expressionOrDeclarations?
    RCURL
    ;

structPrimaryInit
    : (structNonStaticMemberModifier | CONST NL*)? structName NL* LPAREN NL* structPrimaryInitParamLists? NL* RPAREN NL*
 	  LCURL NL*
 	       expressionOrDeclarations?
 	  NL* RCURL
    ;

structName
    : identifier
    ;

structPrimaryInitParamLists
    : unnamedParameterList (NL* COMMA NL* namedParameterList)? (NL* COMMA NL* structNamedInitParamList)?
    | unnamedParameterList (NL* COMMA NL* structUnnamedInitParamList)? (NL* COMMA NL* structNamedInitParamList)?
    | structUnnamedInitParamList (NL* COMMA NL* structNamedInitParamList)?
    | namedParameterList (NL* COMMA NL* structNamedInitParamList)?
    | structNamedInitParamList
    ;

structUnnamedInitParamList
    : structUnnamedInitParam (NL* COMMA NL* structUnnamedInitParam)*
    ;

structNamedInitParamList
    : structNamedInitParam (NL* COMMA NL*  structNamedInitParam)*
    ;

structUnnamedInitParam
    : (structNonStaticMemberModifier NL*)? (LET | VAR) NL* identifier NL* COLON NL* type
    ;

structNamedInitParam
    : (structNonStaticMemberModifier NL*)? (LET | VAR) NL* identifier NL* NOT NL* COLON NL* type (NL* ASSIGN NL* expression)?
    ;

structModifier
    : PUBLIC
    | PROTECTED
    | INTERNAL
    | PRIVATE
    ;

structNonStaticMemberModifier
    : PUBLIC
    | PROTECTED
    | INTERNAL
    | PRIVATE
    ;
```

```cangjie
internal struct Point {
    // 成员变量
    var y: Int64

    // 主构造函数（简化成员变量初始化）
    public Point(y: Int64, var x: Int64) {
        this.x = x
        this.y = y
    }

    // 次构造函数
    public init(y: Int64) {
        this.x = 0
        this.y = y
    }

    // 静态初始化块
    static init() {
        println("Point initialized")
    }

    // 成员方法
    public mut func move(dx: Int64, dy!: Int64): Unit {
        this.y = y + dy
    }
}
```

#### 类型别名定义

```
typeAlias
    : (typeModifier NL*)? TYPE_ALIAS NL* identifier (NL* typeParameters)? NL* ASSIGN NL* type end*
    ;

typeModifier
    : PUBLIC
    | PROTECTED
    | INTERNAL
    | PRIVATE
    ;
```

```cangjie
internal type UserID = String
```

#### 扩展定义

```
extendDefinition
    : EXTEND NL* extendType
    (NL* UPPERBOUND NL* superInterfaces)? (NL* genericConstraints)?
    NL* extendBody
    ;

extendType
    : (typeParameters)? (identifier NL* DOT  NL*)* identifier (NL* typeArguments)?
    | INT8
    | INT16
    | INT32
    | INT64
    | INTNATIVE
    | UINT8
    | UINT16
    | UINT32
    | UINT64
    | UINTNATIVE
    | FLOAT16
    | FLOAT32
    | FLOAT64
    | RUNE
    | BOOLEAN
    | NOTHING
    | UNIT
    ;

extendBody
    : LCURL end* extendMemberDeclaration* end* RCURL
    ;

extendMemberDeclaration
    : (functionDefinition
    | operatorFunctionDefinition
    | macroExpression
    | propertyDefinition
    ) end*
    ;
```

```cangjie
// 直接扩展
extend String {
    func printSize() {
        print(this.size)
    }
}

// 接口扩展
interface I1 {
    func f1(): Unit
}

interface I2 {
    func f2(): Unit
}

class Foo {}

extend Foo <: I1 & I2 {
    public func f1() {}
    public func f2() {}
}
```

#### `foreign` 声明

```
foreignDeclaration
    : FOREIGN NL* (foreignBody | foreignMemberDeclaration)
    ;

foreignBody
    : LCURL end* foreignMemberDeclaration* end* RCURL
    ;

foreignMemberDeclaration
    : (classDefinition
    | interfaceDefinition
    | functionDefinition
    | macroExpression
    | variableDeclaration) end*
    ;
```

```cangjie
foreign func printf(format: CString, ...): Int32

foreign var errno: Int32

foreign {
    func malloc(size: UIntNative): CPointer<Unit>
    func free(ptr: CPointer<Unit>): Unit
}
```

#### `Annotation`

```
annotationList: annotation+;

annotation
    : AT (identifier NL* DOT)* identifier (LSQUARE NL* annotationArgumentList NL* RSQUARE)?
    ;

annotationArgumentList
    : annotationArgument (NL* COMMA NL* annotationArgument)* NL* COMMA?
    ;

annotationArgument
    : identifier NL* COLON NL* expression
    | expression
    ;
```

```cangjie
@Test1
func myTest() {}

@Test2("Use newFunc() instead")
func oldFunc() {}

@Test3(name: "Alice", email: "alice@example.com")
class MyClass {}

@Test4([READ, WRITE])
class Document {}
```

#### 宏声明

```
macroDefinition
    : PUBLIC NL* MACRO NL* identifier NL*
    (macroWithoutAttrParam | macroWithAttrParam) NL*
    (COLON NL* identifier NL*)?
    (ASSIGN NL* expression | block)
    ;

macroWithoutAttrParam
    : LPAREN NL* macroInputDecl NL* RPAREN
    ;

macroWithAttrParam
    : LPAREN NL* macroAttrDecl NL* COMMA NL* macroInputDecl NL* RPAREN
    ;

macroInputDecl
    : identifier NL* COLON NL* identifier
    ;

macroAttrDecl
    : identifier NL* COLON NL* identifier
    ;
```

```cangjie
macro Empty(input: Tokens) {}

public macro Log(input: Tokens): Tokens {
    quote(println("[LOG] ${$(input)}"))
}

public macro Validate(attr: Tokens, input: Tokens): Tokens {
    quote(attr + x)
}
```

#### 属性定义

```
propertyDefinition
    : propertyModifier* NL* PROP NL* identifier NL* COLON NL* type NL* propertyBody?
    ;

propertyBody
    : LCURL end* propertyMemberDeclaration+ end* RCURL
    ;

propertyMemberDeclaration
    : GET NL* LPAREN RPAREN NL* block end*
    | SET NL* LPAREN identifier RPAREN NL* block end*
    ;

propertyModifier
    : PUBLIC
    | PRIVATE
    | PROTECTED
    | INTERNAL
    | STATIC
    | OPEN
    | OVERRIDE
    | REDEF
    | MUT
    ;
```

```cangjie
class User {
    prop username: String = "guest"
    prop isAdmin: Bool {
        get { username == "admin" }
    }
}
```

#### 程序入口定义

```
mainDefinition
    : MAIN
      NL* functionParameters
      (NL* COLON NL* type)?
      NL* block
    ;
```

```cangjie
main() {}

main(args: Array<String>): Int32 {
    if (args.size == 0) {
        println("Need arguments")
        return 1
    }
    return 0
}
```

### 类型

```
type
    : arrowType
    | tupleType
    | prefixType
    | atomicType
    ;

arrowType
    : arrowParameters NL* ARROW NL* type
    ;

arrowParameters
    : LPAREN NL* (type (NL* COMMA NL* type)* NL*)? RPAREN
    ;

tupleType
    : LPAREN NL* type (NL* COMMA NL* type)+ NL* RPAREN
    ;

prefixType
    : prefixTypeOperator type
    ;

prefixTypeOperator
    : QUEST
    ;

atomicType
    : charLangTypes
    | userType
    | parenthesizedType
    ;

charLangTypes
    : numericTypes
    | RUNE
    | BOOLEAN
    | Nothing
    | UNIT
    | THISTYPE
    ;

numericTypes
    : INT8
    | INT16
    | INT32
    | INT64
    | INTNATIVE
    | UINT8
    | UINT16
    | UINT32
    | UINT64
    | UINTNATIVE
    | FLOAT16
    | FLOAT32
    | FLOAT64
    ;

userType
    : (identifier NL* DOT NL*)* identifier ( NL* typeArguments)?
    ;

parenthesizedType
    : LPAREN NL* type NL* RPAREN
    ;
```

### 表达式语法

```
expression
    : assignmentExpression
    ;

assignmentExpression
    : leftValueExpressionWithoutWildCard NL* assignmentOperator NL* flowExpression
    | leftValueExpression NL* ASSIGN NL* flowExpression
    | tupleLeftValueExpression NL* ASSIGN NL* flowExpression
    | flowExpression
    ;

tupleLeftValueExpression
    : LPAREN NL* (leftValueExpression | tupleLeftValueExpression) (NL* COMMA NL* (leftValueExpression | tupleLeftValueExpression))+ NL* COMMA? NL* RPAREN
    ;

leftValueExpression
    : leftValueExpressionWithoutWildCard
    | WILDCARD
    ;

leftValueExpressionWithoutWildCard
    : identifier
    | leftAuxExpression QUEST? NL* assignableSuffix
    ;

leftAuxExpression
    : identifier (NL* typeArguments)?
    | type
    | thisSuperExpression
    | leftAuxExpression QUEST? NL* DOT NL* identifier (NL* typeArguments)?
    | leftAuxExpression QUEST? callSuffix
    | leftAuxExpression QUEST? indexAccess
    ;

assignableSuffix
    : fieldAccess
    | indexAccess
    ;

fieldAccess
    : NL* DOT NL* identifier
    ;

flowExpression
    : coalescingExpression (NL* flowOperator NL* coalescingExpression)*
    ;

coalescingExpression
    : logicDisjunctionExpression (NL* QUEST QUEST NL* logicDisjunctionExpression)*
    ;

logicDisjunctionExpression
    : logicConjunctionExpression (NL* OR NL* logicConjunctionExpression)*
    ;

logicConjunctionExpression
    : rangeExpression (NL* AND NL* rangeExpression)*
    ;

rangeExpression
    : bitwiseDisjunctionExpression NL* (CLOSEDRANGEOP | RANGEOP) NL* bitwiseDisjunctionExpression (NL* COLON NL* bitwiseDisjunctionExpression)?
    | bitwiseDisjunctionExpression
    ;

bitwiseDisjunctionExpression
    : bitwiseXorExpression (NL* BITOR NL* bitwiseXorExpression)*
    ;

bitwiseXorExpression
    : bitwiseConjunctionExpression (NL* BITXOR NL* bitwiseConjunctionExpression)*
    ;

bitwiseConjunctionExpression
    : equalityComparisonExpression (NL* BITAND NL* equalityComparisonExpression)*
    ;

equalityComparisonExpression
    : comparisonOrTypeExpression (NL* equalityOperator NL* comparisonOrTypeExpression)?
    ;

comparisonOrTypeExpression
    : shiftingExpression (NL* comparisonOperator NL* shiftingExpression)?
    | shiftingExpression (NL* IS NL* type)?
    | shiftingExpression (NL* AS NL* type)?
    ;

shiftingExpression
    : additiveExpression (NL* shiftingOperator NL* additiveExpression)*
    ;

additiveExpression
    : multiplicativeExpression (NL* additiveOperator NL* multiplicativeExpression)*
    ;

multiplicativeExpression
    : exponentExpression (NL* multiplicativeOperator NL* exponentExpression)*
    ;

exponentExpression
    : prefixUnaryExpression (NL* exponentOperator NL* prefixUnaryExpression)*
    ;

prefixUnaryExpression
    : prefixUnaryOperator* incAndDecExpression
    ;

incAndDecExpression
    : postfixExpression (INC | DEC )?
    ;

postfixExpression
     : atomicExpression
     | type NL* DOT NL* identifier
     | postfixExpression NL* DOT NL* identifier (NL* typeArguments)?
     | postfixExpression callSuffix
     | postfixExpression indexAccess
     | postfixExpression NL* DOT NL* identifier callSuffix? trailingLambdaExpression
     | identifier callSuffix? trailingLambdaExpression
     | postfixExpression (QUEST questSeperatedItems)+
     ;

questSeperatedItems
     : questSeperatedItem+
     ;

questSeperatedItem
     : itemAfterQuest (callSuffix | callSuffix? trailingLambdaExpression | indexAccess)?
     ;

itemAfterQuest
     : DOT identifier (NL* typeArguments)?
     | callSuffix
     | indexAccess
     | trailingLambdaExpression
     ;

callSuffix
    : LPAREN NL* (valueArgument (NL* COMMA NL* valueArgument)* NL*)? RPAREN
    ;

valueArgument
    : identifier NL* COLON NL* expression
    | expression
    | refTransferExpression
    ;

refTransferExpression
    : INOUT (expression DOT)? identifier
    ;

indexAccess
    : LSQUARE NL* (expression | rangeElement) NL* RSQUARE
    ;

rangeElement
    :  RANGEOP
    | ( CLOSEDRANGEOP | RANGEOP ) NL* expression
    | expression NL* RANGEOP
    ;

atomicExpression
    : literalConstant
    | collectionLiteral
    | tupleLiteral
    | identifier (NL* typeArguments)?
    | unitLiteral
    | ifExpression
    | matchExpression
    | loopExpression
    | tryExpression
    | jumpExpression
    | numericTypeConvExpr
    | thisSuperExpression
    | spawnExpression
    | synchronizedExpression
    | parenthesizedExpression
    | lambdaExpression
    | quoteExpression
    | macroExpression
    | unsafeExpression
    ;

literalConstant
    : IntegerLiteral
    | FloatLiteral
    | RuneLiteral
    | ByteLiteral
    | booleanLiteral
    | stringLiteral
    | ByteStringArrayLiteral
    | unitLiteral
    ;

booleanLiteral
    : TRUE
    | FALSE
    ;

stringLiteral
    : lineStringLiteral
    | multiLineStringLiteral
    | MultiLineRawStringLiteral
    ;

lineStringContent
    :  LineStrText
    ;

lineStringLiteral
    : QUOTE_OPEN (lineStringExpression | lineStringContent)* QUOTE_CLOSE
    ;

lineStringExpression
    : LineStrExprStart SEMI* (expressionOrDeclaration (SEMI+ expressionOrDeclaration?)*) SEMI* RCURL
    ;

multiLineStringContent
    : MultiLineStrText
    ;

multiLineStringLiteral
    : TRIPLE_QUOTE_OPEN (multiLineStringExpression | multiLineStringContent)* TRIPLE_QUOTE_CLOSE
    ;

multiLineStringExpression
    : MultiLineStrExprStart end* (expressionOrDeclaration (end+ expressionOrDeclaration?)*) end* RCURL
    ;

collectionLiteral
    : arrayLiteral
    ;

arrayLiteral
    : LSQUARE (NL* elements)? NL* RSQUARE
    ;

elements
    : element ( NL* COMMA NL* element )*
    ;

element
    : expressionElement
    | spreadElement
    ;

expressionElement
    : expression
    ;

spreadElement
    : MUL expression
    ;

tupleLiteral
    : LPAREN NL* expression (NL* COMMA NL*expression)+ NL* RPAREN
    ;

unitLiteral
    : LPAREN NL* RPAREN
    ;

ifExpression
    : IF NL* LPAREN NL* (LET NL* deconstructPattern NL* BACKARROW NL*)? expression NL* RPAREN NL* block
    (NL* ELSE (NL* ifExpression | NL* block))?
    ;

deconstructPattern
    : constantPattern
    | wildcardPattern
    | varBindingPattern
    | tuplePattern
    | enumPattern
    ;

matchExpression
    : MATCH NL* LPAREN NL* expression NL* RPAREN NL* LCURL NL* matchCase+ NL* RCURL
    | MATCH NL* LCURL NL* (CASE NL* (expression | WILDCARD) NL* DOUBLE_ARROW NL* expressionOrDeclaration (end+ expressionOrDeclaration?)*)+ NL* RCURL
    ;

matchCase
    : CASE NL* pattern NL* patternGuard? NL* DOUBLE_ARROW NL* expressionOrDeclaration (end+ expressionOrDeclaration?)*
    ;

patternGuard
    : WHERE NL* expression
    ;

pattern
   : constantPattern
   | wildcardPattern
   | varBindingPattern
   | tuplePattern
   | typePattern
   | enumPattern
   ;

constantPattern
   : literalConstant NL* ( NL* BITOR NL* literalConstant)*
   ;

wildcardPattern
   : WILDCARD
   ;

varBindingPattern
   : identifier
   ;

tuplePattern
   : LPAREN NL* pattern (NL* COMMA NL* pattern)+ NL* RPAREN
   ;

typePattern
   : (WILDCARD | identifier) NL* COLON NL* type
   ;

enumPattern
   : NL* ((userType NL* DOT NL*)? identifier enumPatternParameters?) (NL* BITOR NL* ((userType NL* DOT NL*)? identifier enumPatternParameters?))*
   ;

enumPatternParameters
   : LPAREN NL* pattern (NL* COMMA NL* pattern)* NL* RPAREN
   ;

loopExpression
    : forInExpression
    | whileExpression
    | doWhileExpression
    ;

forInExpression
    : FOR NL* LPAREN NL* patternsMaybeIrrefutable NL* IN NL* expression NL* patternGuard? NL* RPAREN NL* block
    ;

patternsMaybeIrrefutable
    : wildcardPattern
    | varBindingPattern
    | tuplePattern
    | enumPattern
    ;

whileExpression
    : WHILE NL* LPAREN NL* (LET NL* deconstructPattern NL* BACKARROW NL*)? expression NL* RPAREN NL* block
    ;

doWhileExpression
    : DO NL* block NL* WHILE NL* LPAREN NL* expression NL* RPAREN
    ;

tryExpression
    : TRY NL* block NL* FINALLY NL* block
    | TRY NL* block (NL* CATCH NL* LPAREN NL* catchPattern NL* RPAREN NL* block)+ (NL* FINALLY NL* block)?
    | TRY NL* LPAREN NL* resourceSpecifications NL* RPAREN NL* block
    (NL* CATCH NL* LPAREN NL* catchPattern NL* RPAREN NL* block)* (NL* FINALLY NL* block)?
    ;

catchPattern
    : wildcardPattern
    | exceptionTypePattern
    ;

exceptionTypePattern
    : (WILDCARD | identifier) NL* COLON NL* type (NL* BITOR NL* type)*
    ;

resourceSpecifications
    : resourceSpecification (NL* COMMA NL* resourceSpecification)*
    ;

resourceSpecification
    : identifier (NL* COLON NL* classType)? NL* ASSIGN NL* expression
    ;

jumpExpression
    : THROW NL* expression
    | RETURN (NL* expression)?
    | CONTINUE
    | BREAK
    ;

numericTypeConvExpr
    : numericTypes LPAREN NL* expression NL* RPAREN
    ;

thisSuperExpression
    : THIS
    | SUPER
    ;

lambdaExpression
    : LCURL NL* lambdaParameters? NL* DOUBLE_ARROW NL* expressionOrDeclarations? RCURL
    ;

trailingLambdaExpression
    : LCURL NL* (lambdaParameters? NL* DOUBLE_ARROW NL*)? expressionOrDeclarations? RCURL
    ;

lambdaParameters
    : lambdaParameter (NL* COMMA NL* lambdaParameter)*
    ;

lambdaParameter
    : (identifier | WILDCARD) (NL* COLON NL* type)?
    ;

spawnExpression
    : SPAWN (LPAREN NL* expression NL* RPAREN)? NL* trailingLambdaExpression
    ;

synchronizedExpression
    : SYNCHRONIZED LPAREN NL* expression NL* RPAREN NL* block
    ;

parenthesizedExpression
    : LPAREN NL* expression NL* RPAREN
    ;

block
    : LCURL expressionOrDeclarations RCURL
    ;

unsafeExpression
    : UNSAFE NL* block
    ;

expressionOrDeclarations
    : end* (expressionOrDeclaration (end+ expressionOrDeclaration?)*)?
    ;

expressionOrDeclaration
    : expression
    | varOrfuncDeclaration
    ;

varOrfuncDeclaration
    : functionDefinition
    | variableDeclaration
    ;

quoteExpression
    : QUOTE quoteExpr
    ;

quoteExpr
    : LPAREN NL* quoteParameters NL* RPAREN
    ;

quoteParameters
    : (NL* quoteToken | NL* quoteInterpolate | NL* macroExpression)+
    ;

quoteToken
    : DOT | COMMA | LPAREN | RPAREN | LSQUARE | RSQUARE | LCURL | RCURL | EXP | MUL | MOD | DIV | ADD | SUB
    | PIPELINE | COMPOSITION
    | INC | DEC | AND | OR | NOT | BITAND | BITOR | BITXOR | LSHIFT | RSHIFT | COLON | SEMI
    | ASSIGN | ADD_ASSIGN | SUB_ASSIGN | MUL_ASSIGN | EXP_ASSIGN | DIV_ASSIGN | MOD_ASSIGN
    | AND_ASSIGN | OR_ASSIGN | BITAND_ASSIGN | BITOR_ASSIGN | BITXOR_ASSIGN | LSHIFT_ASSIGN | RSHIFT_ASSIGN
    | ARROW | BACKARROW | DOUBLE_ARROW | ELLIPSIS | CLOSEDRANGEOP | RANGEOP | HASH | AT | QUEST | UPPERBOUND | LT | GT | LE | GE
    | NOTEQUAL | EQUAL | WILDCARD | BACKSLASH | QUOTESYMBOL | DOLLAR
    | INT8 | INT16 | INT32 | INT64 | INTNATIVE | UINT8 | UINT16 | UINT32 | UINT64 | UINTNATIVE | FLOAT16
    | FLOAT32 | FLOAT64 | RUNE | BOOL | UNIT | NOTHING | STRUCT | ENUM | THIS
    | PACKAGE | IMPORT | CLASS | INTERFACE | FUNC | LET | VAR | CONST | TYPE
    | INIT | THIS | SUPER | IF | ELSE | CASE | TRY | CATCH | FINALLY
    | FOR | DO | WHILE | THROW | RETURN | CONTINUE | BREAK | AS | IN
    | MATCH | FROM | WHERE | EXTEND | SPAWN | SYNCHRONIZED | MACRO | QUOTE | TRUE | FALSE
    | STATIC | PUBLIC | PRIVATE | PROTECTED
    | OVERRIDE | ABSTRACT | OPEN | OPERATOR | FOREIGN
    | Identifier | DollarIdentifier
    | literalConstant
    ;

quoteInterpolate
    : DOLLAR LPAREN NL* expression NL* RPAREN
    ;

macroExpression
    : AT Identifier macroAttrExpr? NL* (macroInputExprWithoutParens | macroInputExprWithParens)
    ;

macroAttrExpr
    : LSQUARE NL* quoteToken* NL* RSQUARE
    ;

macroInputExprWithoutParens
    : functionDefinition
    | operatorFunctionDefinition
    | staticInit
    | structDefinition
    | structPrimaryInit
    | structInit
    | enumDefinition
    | caseBody
    | classDefinition
    | classPrimaryInit
    | classInit
    | interfaceDefinition
    | variableDeclaration
    | propertyDefinition
    | extendDefinition
    | macroExpression
    ;

macroInputExprWithParens
    : LPAREN NL* macroTokens NL* RPAREN
    ;

macroTokens
    : (quoteToken | macroExpression)*
    ;

assignmentOperator
    : ASSIGN
    | ADD_ASSIGN
    | SUB_ASSIGN
    | EXP_ASSIGN
    | MUL_ASSIGN
    | DIV_ASSIGN
    | MOD_ASSIGN
    | AND_ASSIGN
    | OR_ASSIGN
    | BITAND_ASSIGN
    | BITOR_ASSIGN
    | BITXOR_ASSIGN
    | LSHIFT_ASSIGN
    | RSHIFT_ASSIGN
    ;

equalityOperator
    : NOTEQUAL
    | EQUAL
    ;

comparisonOperator
    : LT
    | GT
    | LE
    | GE
    ;

shiftingOperator
    : LSHIFT | RSHIFT
    ;

flowOperator
    : PIPELINE | COMPOSITION
    ;

additiveOperator
    : ADD | SUB
    ;

exponentOperator
    : EXP
    ;

multiplicativeOperator
    : MUL
    | DIV
    | MOD
    ;

prefixUnaryOperator
    : SUB
    | NOT
    ;

overloadedOperators
    : LSQUARE RSQUARE
    | NOT
    | ADD
    | SUB
    | EXP
    | MUL
    | DIV
    | MOD
    | LSHIFT
    | RSHIFT
    | LT
    | GT
    | LE
    | GE
    | EQUAL
    | NOTEQUAL
    | BITAND
    | BITXOR
    | BITOR
    ;
```
