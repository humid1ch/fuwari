---
title: "[C++] 理解 using namespace std;"
published: 2022-05-19
description: "本篇文章要涉及的内容, 就是理解 C++ 中 using namespace std; 的含义"
image: https://humid1ch.oss-cn-shanghai.aliyuncs.com/20251215223758900.webp
category: Blogs
tags:
    - C++
---

## 引言

标题已经点明了本篇文章要涉及的内容, 就是理解C++中`using namespace std;`的含义

在C语言中,  同一个作用域中 定义变量或初始化变量, 变量名是不可以相同的, 即 **不可以重定义变量 多次初始化变量**


以下面代码为例:

```c
int gVal = 0;
int gVal = 1;

int main() {
    int lVal = 0;
    int lVal = 1;

    return 0;
}
```

如果尝试编译, 结果为:

![](https://humid1ch.oss-cn-shanghai.aliyuncs.com/20251215194528446.webp)

第2行`gVal`和第6行`lVal`重复定义

这是C语言中的规定, 这个规定有些死板

因为一个项目在开发时, 可能由多个小组负责, 不可能保证在定义某些函数、接口、结构的时候一定不会重名

C语言这个规定, 会造成: 如果存在命名相同, 那就只能留一个, 其他人都需要修改

这在一般由多组负责的一个项目中, 是非常不合理的

那么在C++中, 这个问题通过一个新的概念, 得到比较完美的解决: `namespace`

下面就介绍一下, C++中的`namespace`是如何解决**重复定义**这个问题的

## `namespace`

相信许多人在刚开始学习C++的时候, 一定遇到过 而且一定很纳闷这是个什么东西:

![](https://humid1ch.oss-cn-shanghai.aliyuncs.com/20251215195147065.webp)

`using namespace std;`, 许多教学中存在着个语句

这究竟是什么意思: `using`是什么意思? `namespace` 是什么意思? `std` 又是什么意思? 

这个语句的核心其实与`namespace`有关

### 什么是`namespace`

`namespace`在C++中被提出来, 用来解决C语言不能重复定义的问题

`namespace`被称为**命名空间**, 使用时可以将其**认为是一块单独开辟出的空间**

这块空间内, 可以随意定义 变量、函数等

### `namespace`(命名空间)定义及作用

定义一个命名空间非常的简单:

```cpp
#include <iostream>

namespace humid1ch {
    int val1 = 10;
    int val2 = 11;
}

int main() {

    return 0;
}
```

这样就已经定义了一个命名空间了, 编译也不会出错

但是命名空间有什么作用呢? 

编译以下下面的代码:

```cpp
#include <iostream>

namespace humid1ch1 {
    int val1 = 10;
    int val2 = 11;
}

namespace humid1ch2 {
    int val1 = 20;
    int val2 = 21;
}

int val1 = 30;
int val2 = 31;

int main() {

    return 0;
}
```

![ ](https://humid1ch.oss-cn-shanghai.aliyuncs.com/20250711181051864.webp)

编译也是没有错误的

这可以说明一件事情, **一个命名空间内的变量 是不会与 此命名空间外的其他同名变量 冲突的**

这样就解决了, 在小组分工时可能存在的重定义的问题

只需要不同的小组 按需 开辟命名空间 就可以了

> 命名空间内是可以嵌套命名空间的:
>
> ```cpp
> namespace humid1ch {
>     int val1 = 10;
>     int val2 = 11;
> 
>     namespace humi {
>         int val1 = 12;
>         int val2 = 13;
> 
>         namespace d1ch {
>             int val1 = 14;
>             int val2 = 15;
>         }
>     }
> }
> ```
>
> 不同的命名空间内的变量都是相互隔离的.
>
> PS: 同一个项目中的 同名命名空间, 编译时编译器认为其为同一个命名空间, 即 会检查重复命名情况

其实简单来看, `namespace`命名空间作用就是将变量 或 函数等规划到了不同的作用域, 这样就起到了 将变量隔离的效果

### `namespace`(命名空间) 使用

命名空间内容的使用, 有很多种方法:

#### 方法一

```cpp
#include <iostream>

namespace humid1ch {
    int val1 = 10;
    int val2 = 11;

    namespace humi {
        int val1 = 12;
        int val2 = 13;

        namespace d1ch {
            int val1 = 14;
            int val2 = 15;
        }
    }
}

int main() {
    std::cout << humid1ch::val1 << std::endl;
    std::cout << humid1ch::humi::val1 << std::endl;
    std::cout << humid1ch::humi::d1ch::val1 << std::endl;

    return 0;
}
```

![](https://humid1ch.oss-cn-shanghai.aliyuncs.com/20251215220814136.webp)

`命名空间`+`::`+`变量名`, 就是 使用命名空间内变量的最简单的用法

使用嵌套的命名空间中的变量, 是这样的: `命名空间`+`::`+`(命名空间::)+变量名`

> `::`作用域限定符, 用于访问命名空间内的成员

#### 方法二

除了在变量名之前添加`命名空间`, 还可以直接将命名空间释放出来:

```cpp
#include <iostream>

namespace humid1ch {
    int val1 = 10;
    int val2 = 11;

    namespace humi {
        int val1 = 12;
        int val2 = 13;
    }
}

using namespace humid1ch;

int main() {
    std::cout << val1 << std::endl;
    std::cout << humi::val1 << std::endl;

    return 0;
}
```
![](https://humid1ch.oss-cn-shanghai.aliyuncs.com/20251215221335726.webp)

`using namespace humid1ch;` 之后, 使用`humid1ch`内的变量就不需要再变量前 加`humid1ch::`

但, 此时`humid1ch`命名空间的隔离作用就会失效, **相当于取消了`humid1ch`命名空间**

---

文章读到这里, 再回头看`using namespace std;`

实际就是将`namespace std`释放出来了, 所以直接通过`cout`访问`std::cout`, 实现输出:

```cpp
#include <iostream>

using namespace std;

int main() {
    cout << "Hello World" << endl;
    std::cout << "std Hello World" << std::endl;

    return 0;
}
```

如果不使用`using namespace std;`, 要访问C++标准库提供函数、容器等, 就需要通过`std::`前缀进行访问

实际的开发中, `using namespace std;`一般不允许使用, 因为会对项目造成污染
