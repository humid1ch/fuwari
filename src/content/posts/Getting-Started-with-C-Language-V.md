---
title: '从零接触C语言(初览)-V: 条件 和 循环'
description: 'C语言中, 语句是程序执行的基本单元, 本篇文章将介绍C语言中的条件分支和循环语句'
published: 2025-07-25
image: 'https://humid1ch.oss-cn-shanghai.aliyuncs.com/20250710215328038.webp'
category: Blogs
tags:
    - C
    - 从零开始接触C语言
---

> [!TIP]
>
> 如果你从未接触过C语言, 那么我建议你先阅读前面的文章:
>
> [📌从零开始接触C语言](https://www.humid1ch.cn/blog/tag/从零开始接触C语言)

## 语句

C语言中, 语句是程序执行的基本单元, 以`;`结尾

前面的文章中, 已经遇见过最基础两种句式的:

1. 表达式语句:

    ```c
    age = 20;
    age1 + age2;
    printf("Hello World\n");
    ```

2. 声明语句:

    ```c
    int age;
    int age1 = 20;                                     // 特殊的声明语句, 定义
    char str2[] = { 'H', 'e', 'l', 'l', 'o', '\0' }; // 特殊的声明语句, 定义
    ```

除此之外, C语言还规定有其他句式:

1. 空语句:

    ```c
    ;
    // 只一个 ; 分号, 单独为一个语句
    ```

2. 控制语句:

    - 条件分支语句:

        ```c
        // if-elseif-else
        if (a > 0) {
            // 其他语句
        }
        else if (a == 0) {
            // 其他语句
        }
        else {
            // 其他语句
        }

        // switch-case
        switch (a) {
        case 0:
            // 其他语句
            break;
        case 1:
            break;
        default:
            break;
        }
        ```

    - 循环语句:

        ```c
        // for 循环
        for (i = 0; i < 10; i = i + 1) {
            // 其他语句
        }

        // while 循环
        while (true) {
            // 其他语句
        }

        // do-while 循环
        do {
            // 其他语句
        } while (true);
        ```

    - 跳转语句:

        ```c
        break; // 直接结束循环、switch-case

        continue; // 直接跳过当前循环, 进入下一轮循环

        goto label; // 跳转到 label标签语句处

        return; // 从函数中返回
        ```

3. 标签语句:

    ```c
    label:
        // 其他语句
    // 用于 goto 跳转
    ```

本片文章要重点介绍的是**控制语句**中的**条件分支和循环语句**

### 条件分支语句

条件分支语句, 是**根据条件来选择执行分支的语句**

什么是根据条件来选择执行分支呢?

意思是, **满足哪个条件, 就执行哪个条件对应的分支**, 更书面一点表述: **哪个条件为真, 则执行此条件对应的分支**

C语言中有两种条件分支语句``if-else if-else`和`switch-case`

但在具体介绍这两种条件分支语句之前, 还先要了解 C语言中 的条件是什么, 条件的满足是怎么样判断的? 即, **如何判断 条件的真假**

#### 条件 与 条件的真假

C语言提供有关系运算符和逻辑运算符来实现条件的判断:

1. 关系运算符:

    | 符号 | 使用     | 功能                   |
    | ---- | -------- | ---------------------- |
    | `==` | `a == b` | 判断`a`和`b`是否相等   |
    | `!=` | `a != b` | 判断`a`和`b`是否不相等 |
    | `>`  | `a > b`  | 判断`a`是否大于`b`     |
    | `<`  | `a < b`  | 判断`a`是否小于`b`     |
    | `>=` | `a >= b` | 判断`a`是否大于等于`b` |
    | `<=` | `a <= b` | 判断`a`是否小于等于`b` |

    **关系运算符, 用于判断变量或常量之间的关系是否成立, 成立则为真, 不成立则为假**

2. 逻辑运算符

    | 符号         | 使用                 | 功能                                   |
    | ------------ | -------------------- | -------------------------------------- |
    | `&&(逻辑与)` | `表达式1 && 表达式2` | 两个表达式必须全为真, 此表达式才为真   |
    | `\|\|(逻辑或)` | `表达式1 \|\| 表达式2` | 两个表达式只要有一个为真, 此表达式为真 |
    | `!(逻辑非)`  | `! 表达式`           | 根据所作用表达式的真假, 转换真假       |

    **逻辑运算符, 用于组成复杂逻辑的条件判断表达式, 作用于其他表达式**

    需要额外说明的是:

    1. **`&&(逻辑与)`运算符, 只有左边的表达式为真时, 才会执行右边的表达式进行判断**
    2. **`||(逻辑或)`运算符, 如果左边的表达式为真, 右边的表达式将不会被执行**

    **这两个性质非常的重要**

C语言提供了这么多运算符用于条件和逻辑判断, 实际上表达式的真假却很简单

**C语言中, 表达式为非零值(正数或负数)时即为真, 表达式为0时即为假**

并且, C语言中, **当条件表达式为真时, 此表达式的值为1, 为假时, 此表达式的值为0**

```c
#include <stdio.h>

int main() {
    printf("10 > 0: %d\n", 10 > 0);
    printf("10 < 0: %d\n", 10 < 0);

    return 0;
}
```

![](https://humid1ch.oss-cn-shanghai.aliyuncs.com/20250729153355512.webp)

#### `if-elseif-else`

`if`条件分支的语法是这样的:

```c
/* 单分支 */
if (条件) {
    // 条件满足 会执行此分支
}

/* 双分支 */
if (条件) {
    // 条件满足 会执行此分支
}
else {
    // 条件不满足 会执行此分支
}

/* 多分支 */
if (条件1) {
    // 条件1 对应的分支
}
else if (条件2) {
    // 条件2 对应的分支
}
else if (条件3) {
    // 条件3 对应的分支
}
else if (条件n) {
    // 条件n 对应的分支
}
else {
    // 如果以上条件 都不满足, 会执行的分支
}
```

`if`条件分支的使用很简单, 但需要注意:

**同一个条件语句块中, 条件分支是从上到下依次判断的, 且每一个条件分支的执行是互斥的, 即使存在多个条件满足的情况, 也只会执行第一个条件满足的分支**

以代码为例:

```c
#include <stdio.h>

int main() {
    int num = 10;
    if (num < 0) {
        printf("num < 0, num: %d\n", num);
    }
    else if (num > 0) {
        printf("num > 0, num: %d\n", num);
    }
    else if (num == 10) {
        printf("num == 10\n");
    }
    else if (num > 10) {
        printf("num > 10, num: %d\n", num);
    }
    else {
        printf("num == 0\n");
    }

    return 0;
}
```

这段代码的执行结果为:

![](https://humid1ch.oss-cn-shanghai.aliyuncs.com/20250729153016253.webp)

虽然`num = 10`同时满足了`num > 0`和`num == 10`两个条件

但是, 只执行了`num > 0`条件的分支, 可以证明各分支之间的执行是互斥的

#### `switch-case`

`switch`条件分支的语法是这样的:

```c
switch (整型值) {
case 0:
    // 分支内容
    break;
case 1:
    // 分支内容
    break;
default:
    // 分支内容
    break;
}
```

`if`条件分支的使用 比较灵活, 可以用于任何的条件表达式

而`switch`不同, **`switch`则只用于整型值是否与特定值相等的判断**, 即 **`switch ()`的括号中只能是整型值**

**`switch (整型值)`括号中的整型值, 是与`case 整型值`中的整型值做相等判断的**

如果相等, 则执行对应的分支; 如果不相等, 就不执行

**如果`switch`中的整型值与所有`case`后的整型值均不相等, 且`default`分支存在, 则会去执行`default`分支**

下面以代码为例:

```c
#include <stdio.h>

int main() {
    int num = 3;
    switch (num) {
    case 0:
        printf("num : 0\n");
        break;
    case 1:
        printf("num : 1\n");
        break;
    case 2:
        printf("num : 2\n");
        break;
    case 3:
        printf("num : 3\n");
        break;
    default:
        printf("default num : %d\n", num);
        break;
    }

    return 0;
}
```

![](https://humid1ch.oss-cn-shanghai.aliyuncs.com/20250729162630533.webp)

可以看到, `num = 3`就执行了`switch (num)-case 3`的分支

`switch`正确的使用并不复杂, 不过有几个点需要特别注意一下:

1. **`switch`括号中只能是一个整型值, 不支持其他类型的数据**

2. **`case`后的整型值必须是整型常量, 且在同一个`switch`条件分支必须唯一**

3. **`break;`通常是必须的, 通常用于执行完匹配的`case`分支后跳出整个`switch`结构**

    如果不使用`break;`, 程序会继续执行后续的`case`分支, 直到遇到`break;`或`switch`结束

    这个现象被称为`case`穿透

    以代码为例:

    ```c
    #include <stdio.h>

    int main() {
        int num = 1;
        switch (num) {
        case 0:
            printf("num : 0\n");
        case 1:
            printf("num : 1\n");
        case 2:
            printf("num : 2\n");
        case 3:
            printf("num : 3\n");
        default:
            printf("default num : %d\n", num);
            break;
        }

        return 0;
    }
    ```

    这段代码的执行结果是:

    ![](https://humid1ch.oss-cn-shanghai.aliyuncs.com/20250729164340113.webp)

    可以看到, 在匹配到`case 1`之后, 在遇到`break;`或`switch`结束之前, 按顺序执行了之间的所有`case`分支

> [!NOTE]
> 
> `break;`关键词只能在`switch`和循环语句中使用, 作用是跳出所处结构

### 循环语句

从字面意思理解, 循环语句可以实现语句的循环执行

事实也确实如此

C语言中, 循环语句有三种`for循环` `while循环`和`do-while 循环`

#### `for`循环

`for`循环的语法是这样的:

```c
for (part1; part2; part3) {
    // 循环体, 即 要循环执行的内容
}
```

首先, 比较容易理解的是循环体, 循环体就是需要循环执行的语句, 没有额外的语法限制

然后就是`for (part1; part2; part3)`的三个`part`:

1. `part1`, 是开始循环之前会自动执行一次的语句

    即, **首次循环之前, 会首先自动执行一次且仅执行一次的语句, 通常用于循环控制变量的初始化**

2. `part2`, 是循环继续执行的条件表达式

    **如果, `part2`这个表达式为真, 循环就继续执行**

    **如果, `part2`这个表达式为加, 循环就结束**

3. `part3`, 是每次循环结束之后, 会自动执行的语句

    **`part3`的语句, 在每次`for`循环完成之后, 都会执行一次**

    **通常用于修改循环控制变量, 以控制`for`循环结束**

用代码最方便理解:

```c
#include <stdio.h>

int main() {
    for (int index = 0; index < 5; index = index + 1) {
        printf("the index: %d\n", index);
    }

    return 0;
}
```

![](https://humid1ch.oss-cn-shanghai.aliyuncs.com/20250729170345731.webp)

可以看到, 循环体一共执行了`5`次

从`int index = 0(part1)`开始, 每次执行循环体之后, 执行`index = index + 1(part3)`, 直到`index = 5`时, 不满足`index < 5(part2)`循环结束

#### `while`循环

`while`循环的语法是这样的:

```c
while (条件) {
    // 循环体
}
```

`while`循环相对比较简单, 只要条件成立, 循环体就执行

需要注意的时, **循环控制变量要在`while`循环之前就定义好, 循环控制操作通常在循环体内执行**

举个简单的例子:

```c
#include <stdio.h>

int main() {
    int index = 0;
    while (index < 5) {
        printf("the index: %d\n", index);
        index = index + 1;
    }

    return 0;
}
```

执行结果为:

![](https://humid1ch.oss-cn-shanghai.aliyuncs.com/20250729171204495.webp)

与上面`for`循环的例子执行结果是一样的

实际上, 这两个例子(`for`和`while`循环的例子)可以看作是等价的

#### `do-while`循环

`do-while`循环与`while`循环很相似, 语法是这样的:

```c
do {
    // 循环体
} while (条件);
```

与`while`循环相同, `do-while`循环, 只要条件为真, 循环就继续执行

与`while`循环不同的是, **`do-while`循环总是先执行一次循环体, 之后再进行条件判断**

同样举一个简单的例子:

```c
#include <stdio.h>

int main() {
    int index = 1;
    do {
        printf("the index: %d\n", index);
    } while (index < 1);

    return 0;
}
```

这段代码的执行结果为:

![](https://humid1ch.oss-cn-shanghai.aliyuncs.com/20250729171823492.webp)

可以看到, 即使`index`初始值为1, 且`do-while`循环中的条件是`index < 1`, 循环体依旧执行了一次

这可以说明, `do-while`循环确实总是先执行一次循环体, 之后再进行条件判断
