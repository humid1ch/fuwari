---
title: "仓颉文档阅读-开发指南I: 初始仓颉语言"
published: 2025-10-21 17:11:11
description: "仓颉文档阅读的开发指南部分, 本篇文章简单介绍仓颉语言的一些特点, 并安装仓颉工具链 运行仓颉程序"
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

---

## 写在开始

前面其实关于仓颉编程语言的文档阅读, 已经有一部分了

但之前阅读的内容是: 仓颉语言的语言规约, 大多是语法规范等内容, 设计语言使用细节、语言特性的分析介绍并不完整, 更多是对语法规范以及仓颉内容的一些了解

而且, 语言规约的甚至是跟随`Cangjie-0.53.18`发布的, 在之后的`1.0.0-STL`版本中, 并没有对语言规约进行更新

所以, 仓颉语言最细节的部分还是 阅读 具体开发指南 [Cangjie-LTS-1.0.3](https://cangjie-lang.cn/docs?url=/1.0.3/index.html) 更加细节的学习仓颉

## 初始仓颉语言

### 初始仓颉语言

> 仓颉编程语言是一种面向全场景应用开发的通用编程语言, 可以兼顾开发效率和运行性能, 并提供良好的编程体验, 主要具有如下特点:
>
> - 多后端支持: 仓颉编程语言支持`CJNative`和`CJVM`两种后端
>
>     其中`CJNative`后端将代码编译为原生二进制代码, 直接在操作系统层面上运行
>
>     `CJVM`后端将代码编译为字节码, 基于`VM(虚拟机)`进行运行
>
>     本次发布仅提供`CJNative`后端`SDK`, `CJVM`后端`SDK`敬请期待
>
> - 语法简明高效: 仓颉编程语言提供了一系列简明高效的语法, 旨在减少冗余书写、提升开发效率, 例如插值字符串、主构造函数、`Flow`表达式、`match`和重导出等语法, 让开发者可以用较少编码表达相关逻辑
>
> - 多范式编程: 仓颉编程语言支持**函数式**、**命令式**和**面向对象**等多范式编程, 融合了高阶函数、代数数据类型、模式匹配、泛型等函数式语言的先进特性, 还有封装、接口、继承、子类型多态等支持模块化开发的面向对象语言特性, 以及值类型、全局函数等简洁高效的命令式语言特性
>
>     开发者可以根据开发偏好或应用场景, 选用不同的编程范式
>
> - 类型安全: 仓颉编程语言是静态强类型语言, 通过编译时类型检查尽早识别程序错误, 降低运行时风险, 也便于代码维护
>
>     同时, 仓颉编译器提供了强大的类型推断能力, 可以减少类型标注工作, 提高开发效率
>
> - 内存安全: 仓颉编程语言支持**自动内存管理**, 并在运行时进行数组下标越界检查、溢出检查等操作, 确保运行时内存安全
>
> - 高效并发: 仓颉编程语言提供了**用户态轻量化线程(原生协程)**, 以及简单易用的并发编程机制, 保证并发场景的高效开发和运行
>
> - 兼容语言生态: 仓颉编程语言支持和 **C 等编程语言的互操作**, 并采用便捷的声明式编程范式, 可实现对其他语言库的高效复用和生态兼容
>
> - 领域易扩展: 仓颉编程语言提供了**基于词法宏的元编程能力**, 支持在编译时变换代码
>
>     此外, 还提供了尾随`lambda`、属性、操作符重载、部分关键字可省略等特性, 开发者可由此深度定制程序的语法和语义, 这有利于内嵌式领域专用语言(Embedded Domain Specific Languages, EDSL)的构建
>
> - 助力 UI 开发: UI 开发是构建端侧应用的重要环节, 基于仓颉编程语言的元编程和尾随`lambda`等特性, 用户可以搭建声明式 UI 开发框架, 提升 UI 开发效率和体验
>
> - 内置库功能丰富: 仓颉编程语言提供了功能丰富的内置库, 涉及数据结构、常用算法、数学计算、正则匹配、系统交互、文件操作、网络通信、数据库访问、日志打印、解压缩、编解码、加解密和序列化等功能

从文档来看, 仓颉编程语言是全场景的通用开发语言, 提供原生后端和虚拟机后端, 但虚拟机后端SDK暂未提供

仓颉提供协程, 仓颉拥有自动内存管理, 仓颉是强类型语言, 仓颉提供元编程能力

这些都是现代语言特性, 与C/C++非常不一样

不过这介绍内容简单看一下就可以了, 具体还是语言细节更重要一些

### 安装仓颉工具链

> 在开发仓颉程序时, 必用的工具之一是仓颉编译器, 它可以将仓颉源代码编译为可运行的二进制文件, 但现代编程语言的配套工具并不止于此, 实际上, 仓颉为开发者提供了**编译器**、**调试器**、**包管理器**、**静态检查工具**、**格式化工具**和**覆盖率统计工具**等一整套仓颉开发工具链, 同时提供了友好的安装和使用方式, 基本能做到"开箱即用"
>
> 目前仓颉工具链已适配部分版本的`Linux`、`macOS`和`Windows`平台, 但是仅针对部分`Linux`发行版做了完整功能测试, 详情可参阅附录`Linux`版本工具链的支持与安装章节, 在暂未进行过完整功能测试的平台上, 仓颉工具链的功能完整性不受到保证
>
> 此外, 当前`Windows`平台上的仓颉编译器基于`MinGW`实现, 相较于`Linux`版本的仓颉编译器, **功能会有部分欠缺**

仓颉的工具链, 目前针对`Linux`、`macOS`和`Windows`三个平台做了适配

但仅在`Linux`中做了完整的功能测试, 且`Windows`平台还有功能欠缺, 所以 本篇文章将 只跟着文档, 在`Linux`平台对仓颉工具链进行配置

但我不禁还想问一句, `HarmonyOS`端上来这么慢吗?

#### `Linux/macOS`

##### 环境准备

###### `Linux`

> `Linux`版仓颉工具链的系统环境要求如下:
>
> | 架构 | 环境要求 |
> | ---- | -------- |
> | `x86_64` | `glibc 2.22`, `Linux Kernel 4.12`或更高版本, 系统安装`libstdc++ 6.0.24`或更高版本 |
> | `aarch64` | `glibc 2.27`, `Linux Kernel 4.15`或更高版本, 系统安装`libstdc++ 6.0.24`或更高版本 |
>
> 除此之外, 对于`Ubuntu 18.04`, 还需要安装相应的依赖软件包:
>
> ```bash
> $ apt-get install binutils libc-dev libc++-dev libgcc-7-dev
> ```
>
> 更多`Linux`发行版的依赖安装命令可以参见附录`Linux`版本工具链的支持与安装章节
>
> 此外, 仓颉工具链还依赖`OpenSSL 3`组件, 由于该组件可能无法从以上发行版的默认软件源直接安装, 因此需要自行手动安装, 安装方式请参见附录[`Linux`版本工具链的支持与安装](https://cangjie-lang.cn/docs?url=%2F1.0.3%2Fuser_manual%2Fsource_zh_cn%2FAppendix%2Flinux_toolchain_install.html)章节

博主之前写了一篇文章, 是为C/C++搭建`WSL`环境, 系统是`openEuler 22.04`, 如果你手上当前没有可用的`Linux`系统, 可以去阅读一下 跟着部署一下`WSL`环境

[接触C语言之前的准备: 环境搭建](https://www.humid1ch.cn/blog/getting-started-with-c-language-before)

当然, 如果嫌麻烦可以跟着文档在`Windows`下配置环境

因为仓颉目前只提供了`VSCode`的相关开发扩展, 无论是直接在`Windows`下, 还是使用`WSL`, 还是直接在`Linux`下进行环境搭建

目前来说, 若想有一个比较舒服的仓颉开发环境, 还是得用`VSCode`, 如果是原生平台就直接用`VSCode`, 如果是`WSL`也需要使用`VSCode`连接`WSL`

###### `macOS`

> `macOS`版仓颉工具链支持在`macOS 12.0`及以上版本(`aarch64`架构)运行
>
> 使用`macOS`版本前需要安装相应的依赖软件包, 可以通过执行以下命令安装:
>
> ```cangjie
> $ brew install libffi
> ```

##### 安装指导


> 首先请前往[仓颉官网](https://cangjie-lang.cn/download/1.0.3), 下载适配平台架构的安装包:
>
> - `cangjie-sdk-linux-x64-x.y.z.tar.gz`: 适用于`x86_64`架构`Linux`系统的仓颉工具链
>
> - `cangjie-sdk-linux-aarch64-x.y.z.tar.gz`: 适用于`aarch64`架构`Linux`系统的仓颉工具链
>
> - `cangjie-sdk-mac-aarch64-x.y.z.tar.gz`: 适用于`aarch64/arm64`架构`macOS`系统的仓颉工具链

下载完成之后, 如果你是`WSL`, 那么还需要将下载好的文件移到`WSL`系统中:

在`Windows`上下载完成之后, 找到你的文件路径, 例如:`"C:\Users\humid1ch\Downloads\cangjie-sdk-linux-x64-1.0.3.tar.gz"`

然后启动你的`WSL`, 可以将工具链压缩包复制到`Linux`的目标路径下:

```bash
cp /mnt/c/Users/humid1ch/Downloads/cangjie-sdk-linux-x64-1.0.3.tar.gz .
# /mnt/c 对应挂载 Windows的C盘
```

![](https://humid1ch.oss-cn-shanghai.aliyuncs.com/20251021211455904.webp)


> 假设这里选择了`cangjie-sdk-linux-x64-x.y.z.tar.gz`, 下载到本地后, 请执行如下命令解压:
>
> ```bash
> tar xvf cangjie-sdk-linux-x64-x.y.z.tar.gz
> ```
>
> 解压完成, 可以在当前工作路径下看到一个名为`cangjie`的目录, 其中存放了仓颉工具链的全部内容, 请执行如下命令完成仓颉工具链的安装配置:
>
> ```bash
> source cangjie/envsetup.sh
> ```
>
> 为了验证是否安装成功, 可以执行如下命令:
>
> ```bash
> cjc -v
> ```
>
> 其中`cjc`是仓颉编译器的可执行文件名, 如果在命令行中看到了仓颉编译器版本信息, 表示已经成功安装了仓颉工具链
>
> 值得说明的是, `envsetup.sh`脚本只是在当前`shell`环境中配置了工具链相关的环境变量, 所以仓颉工具链仅在当前`shell`环境中可用, 在新的`shell`环境中, 需要重新执行`envsetup.sh`脚本配置环境
>
> 若想使仓颉工具链的环境变量配置在`shell`启动时自动生效, 可以在`$HOME/.bashrc`或`$HOME/.zshrc`(依`shell`种类而定)等`shell`初始化配置文件的最后加入以下命令:
>
> 假设仓颉安装包解压在`/home/user/cangjie`中
>
> ```bash
> source /home/user/cangjie/envsetup.sh  # 即 envsetup.sh 的绝对路径
> ```
>
> 配置完成后`shell`启动即可直接使用仓颉编译工具链

执行命令解压仓颉工具链:

![](https://humid1ch.oss-cn-shanghai.aliyuncs.com/20251021212245040.webp)

从解压命令执行过程的输出信息中, 可以看到`cangjie/envsetup.sh`

然后执行命令激活工具链的相关配置:

<Warning>

如果你和我一样, 用的`fish`而不是`bash`或`zsh`, 那么`envsetup.sh`是没有办法直接执行的

```fish
# Copyright Huawei Technologies Co., Ltd. 2020-2023. All rights reserved.
# This script needs to be placed in the output directory of Cangjie compiler.
# ** NOTE: Please use`source' command to execute this script. **

# Get the absolute path of this script
set script_dir (dirname (status -f))
set script_dir (realpath "$script_dir")
set -gx CANGJIE_HOME "$script_dir"

# Get hardware architecture
set hw_arch (uname -m)
if test -z "$hw_arch"
    set hw_arch "x86_64"
end

# Set PATH
set -gx PATH "$CANGJIE_HOME/bin" "$CANGJIE_HOME/tools/bin" "$HOME/.cjpm/bin" $PATH

# Set LD_LIBRARY_PATH
set -gx LD_LIBRARY_PATH "$CANGJIE_HOME/runtime/lib/linux_$hw_arch"_llvm "$CANGJIE_HOME/tools/lib" $LD_LIBRARY_PATH

# Setup auto completion for cjc and cjc-frontend (Fish equivalent)
if command -q cjc
    # Remove existing completions if any
    complete -c cjc --erase

    # Set up generic file completion for cjc (similar to _gnu_generic in bash/zsh)
    complete -c cjc -f -a "(ls)" -d "Cangjie compiler command"
    complete -c cjc -s h -l help -d "Show help"
    complete -c cjc -l version -d "Show version"
    complete -c cjc -s v -l verbose -d "Verbose output"
    complete -c cjc -s o -l output -r -d "Output file"
end

if command -q cjc-frontend
    # Remove existing completions if any
    complete -c cjc-frontend --erase

    # Set up generic file completion for cjc-frontend
    complete -c cjc-frontend -f -a "(ls)" -d "Cangjie frontend command"
    complete -c cjc-frontend -s h -l help -d "Show help"
    complete -c cjc-frontend -l version -d "Show version"
    complete -c cjc-frontend -s v -l verbose -d "Verbose output"
    complete -c cjc-frontend -s o -l output -r -d "Output file"
end

# Clean up temporary variables
set -e hw_arch
set -e script_dir

echo "Cangjie environment set for Fish shell"
echo "CANGJIE_HOME = $CANGJIE_HOME"
echo "Auto-completion enabled for cjc and cjc-frontend"
```

</Warning>

![](https://humid1ch.oss-cn-shanghai.aliyuncs.com/20251021215658432.webp)

如果你是`bash`或`zsh`或兼容`bash`语法的`shell`, `source cangjie/envsetup.sh`就可以了

且, 每次打开终端都需要重新执行, 或者把`source envsetup.sh(绝对路径)`添加到目标`.bashrc`或`.zshrc`中

然后, 执行`cjc -v`查看是否能够执行成功:

![](https://humid1ch.oss-cn-shanghai.aliyuncs.com/20251021220135966.webp)

##### 卸载与更新

> 在`Linux`和`macOS`平台, 删除上述仓颉工具链的安装包目录, 同时移除上述环境变量(最简单的, 可以新开一个`shell`环境), 即可完成卸载
>
> ```bash
> $ rm -rf <path>/<to>/cangjie
> ```
>
> 若需要更新仓颉工具链, 需要**先卸载当前版本**, 然后按上述指导**重新安装最新版本**的仓颉工具链


#### Windows

> 本节以`Windows 10`平台为例, 介绍仓颉工具链的安装方式

##### 安装指导

> 在`Windows`平台上, 仓颉为开发者提供了`exe`和`zip`两种格式的安装包, 请前往仓颉官网, 选择和下载适配平台架构的`Windows`版安装包
>
> - 如果选择`exe`格式的安装包(例如`cangjie-sdk-windows-x64-x.y.z.exe`), 请直接执行此文件, 跟随安装向导点击操作, 即可完成安装
>
> - 如果选择`zip`格式的安装包(例如`cangjie-sdk-windows-x64-x.y.z.zip`), 请将它解压到适当目录
>
>     在安装包中, 仓颉为开发者提供了三种不同格式的安装脚本, 分别是`envsetup.bat`, `envsetup.ps1`和`envsetup.sh`, 可以根据使用习惯及环境配置, 选择一种执行:
>
>     若使用`Windows 命令提示符(CMD)`环境, 请执行:
>
>     ```cmd
>     path\to\cangjie\envsetup.bat
>     ```
>
>     若使用`PowerShell`环境, 请执行:
>
>     ```powershell
>     . path\to\cangjie\envsetup.ps1
>     ```
>
>     若使用`MSYS shell`、`bash`等环境, 请执行:
>
>     ```bash
>     source path/to/cangjie/envsetup.sh
>     ```
>
>     为了验证是否安装成功, 请在以上命令环境中继续执行`cjc -v`命令, 如果输出了仓颉编译器版本信息, 表示已经成功安装了仓颉工具链
>
> **值得注意的是**, 基于`zip`安装包和执行脚本的安装方式, 类似于`Linux`平台, 即`envsetup`脚本所配置的环境变量, 只在**当前命令行环境**中有效, 如果打开新的命令行窗口, 需要重新执行`envsetup`脚本配置环境
>
> 此时, 若想使仓颉工具链的环境变量配置在命令提示符或终端启动时自动生效, 可以对系统进行如下配置:
>
> - 若使用 bash 环境, 可以根据如下步骤操作:
>
>     在`$HOME/.bashrc`初始化配置文件的最后加入以下命令(`$HOME`为当前用户目录的路径):
>
>     ```bash
>     # 假设仓颉安装包解压在 /home/user/cangjie 中
>     source /home/user/cangjie/envsetup.sh  # 即 envsetup.sh 的绝对路径
>     ```
>
>     配置完成后`bash`启动即可直接使用仓颉编译工具链
>
> - 若使用`Windows 命令提示符(CMD)`、`PowerShell`或其他环境, 可以根据如下步骤操作:
>
>     1. 在`Windows`搜索框中, 搜索 "查看高级系统设置" 并打开对应窗口;
>
>     2. 单击 "环境变量" 按钮;
>
>     3. 执行如下操作, 配置`CANGJIE_HOME`变量:
>
>         1. 在 "用户变量"(为当前用户进行配置)或 "系统变量"(为系统所有用户进行配置)区域中, 查看是否已有`CANGJIE_HOME`环境变量
>
>             若没有, 则单击 "新建" 按钮, 并在 "变量名" 字段中输入`CANGJIE_HOME`
>
>             若有, 则说明该环境可能已经进行过仓颉配置, 如果想要继续为当前的仓颉版本进行配置并覆盖原配置, 请点击 "编辑" 按钮, 进入 "编辑系统变量" 窗口
>
>         2. 在 "变量值" 字段中输入仓颉安装包的解压路径, 若原先已经存在路径, 则使用新的路径覆盖原有的路径, 例如仓颉安装包解压在`D:\cangjie`, 则输入`D:\cangjie`
>
>         3. 配置完成后,  "编辑用户变量" 或 "编辑系统变量" 窗口中显示的变量名为`CANGJIE_HOME`、变量值为`D:\cangjie`
>
>             确认路径正确配置后单击 "确定"
>
>     4. 执行如下操作, 配置`Path`变量:
>
>         1. 在 "用户变量"(为当前用户进行配置)或 "系统变量"(为系统所有用户进行配置)区域中, 找到并选择`Path`变量, 单击 "编辑" 按钮, 进入 "编辑环境变量" 窗口
>
>         2. 分别单击 "新建" 按钮, 并分别输入`%CANGJIE_HOME%\bin`、`%CANGJIE_HOME%\tools\bin`、`%CANGJIE_HOME%\tools\lib`、`%CANGJIE_HOME%\runtime\lib\windows_x86_64_llvm`(`%CANGJIE_HOME%`为仓颉安装包的解压路径)
>
>             例如, 仓库安装包解压在`D:\cangjie`, 则新建的环境变量分别为: `D:\cangjie\bin`、`D:\cangjie\tools\bin`、`D:\cangjie\tools\lib`、`D:\cangjie\runtime\lib\windows_x86_64_llvm`
>
>         3. (仅适用于为当前用户设置)单击 "新建" 按钮, 并输入当前用户目录路径, 并在路径后面添加`.cjpm\bin`
>
>             例如用户路径在`C:\Users\bob`, 则输入`C:\Users\bob\.cjpm\bin`
>
>         4. 配置完成后应能在 "编辑环境变量" 窗口中看到配置的路径如下所示
>
>             确认路径正确配置后单击 "确定"
>
>             ```
>             D:\cangjie\bin
>             D:\cangjie\tools\bin
>             D:\cangjie\tools\lib
>             D:\cangjie\runtime\lib\windows_x86_64_llvm
>             C:\Users\bob\.cjpm\bin
>             ```
>
>         5. 单击 "确定" 按钮, 退出 "环境变量" 窗口
>
>         6. 单击 "确定" 按钮, 完成设置
>
>         > 注意:
>         >
>         > 设置完成后可能需要**重启命令行窗口或重启系统**以让设置生效
>
>     配置完成后`Windows 命令提示符(CMD)`或`PowerShell`启动即可直接使用仓颉编译工具链

##### 卸载与更新

> - 如果选择`exe`格式的安装包进行的安装, 运行仓颉安装目录下的`unins000.exe`可执行文件, 跟随卸载向导点击操作, 即可完成卸载
>
> - 如果选择`zip`格式的安装包进行的安装, 删除仓颉工具链的安装包目录, 同时移除上述环境变量设置(若有), 即可完成卸载
>
> 若需要更新仓颉工具链, 需要先卸载当前版本, 然后按上述指导重新安装最新版本的仓颉工具链


### 运行第一个仓颉程序

> 万事俱备, 开始编写和运行第一个仓颉程序吧！
>
> 首先, 请在适当目录下新建一个名为`hello.cj`的文本文件, 并向文件中写入以下仓颉代码:
>
> ```cangjie
> // hello.cj
> main() {
>     println("你好, 仓颉")
> }
> ```
>
> 在这段代码中, 使用了仓颉的注释语法, 可以在`//`符号之后写单行注释, 也可以在一对`/*`和`*/`符号之间写多行注释, 这与 C/C++ 等语言的注释语法相同
>
> 注释内容不影响程序的编译和运行
>
> 然后, 请在此目录下执行如下命令:
>
> ```shell
> cjc hello.cj -o hello
> ```
>
> 这里仓颉编译器会将`hello.cj`中的源代码编译为此平台上的可执行文件`hello`, 在命令行环境中运行此文件, 将看到程序输出了如下内容:
>
> ```shell
> 你好, 仓颉
> ```
>
> > 注意:
> >
> > 以上编译命令是针对`Linux`和`macOS`平台的, 如果使用`Windows`平台, 只需要将编译命令改为`cjc hello.cj -o hello.exe`即可

![](https://humid1ch.oss-cn-shanghai.aliyuncs.com/20251021232849467.webp)
