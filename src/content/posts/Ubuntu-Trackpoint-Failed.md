---
title: "关于Ubuntu-24.04 Trackpoint(小红帽)检测不到的解决方案"
published: 2025-10-02
description: "在笔记本上安装了Ubuntu-24.04之后, 发现小红帽没有被识别到, 无法使用, 驱动也没有找到, 最终还是修复了"
image: 'https://humid1ch.oss-cn-shanghai.aliyuncs.com/undefined20251002234243626.webp'
category: Blogs
tags:
    - 设备驱动
    - Linux
    - 设备故障
---

前一段时间把笔记本换上了Ubuntu-24.04系统, 想尝试一下niri

但是, 安装完之后发现系统没有识别到 小红帽

这就让我非常难受了, 笔记本不外接键鼠的时候, 还没有小红帽, 用着就非常难受了

然后查阅各种资料, 发现各种解决方案都没有用

又是各种安装xserver的, 又是各种安装xinput的

然后发现都没有什么用, xinput该检测不到小红帽设备, 还是检测不到

已经开始一度怀疑, 是不是因为硬件问题没有检测到了

然后机缘巧合之下, 了解到一个命令`dmidecode`

可以查看设备的硬件详细信息, 且这些信息是由BIOS/UEFI固件提供的, 和系统没有关系

然后查阅文档, 执行 **`dmidecode -t 21`** 可以查看 设备内置的指针设备

然后我就发现了:

![](https://humid1ch.oss-cn-shanghai.aliyuncs.com/undefined20251002235739891.webp)

可以看到硬件是没有问题的, 更大的可能是系统没有检测到, 没有启动相关的驱动

然后问了AI, 可以尝试在grub配置中, 强制启用PS/2的Trackpoint相关的端口

即, 在 `/etc/default/grub` 文件的 `GRUB_CMDLINE_LINUX_DEFAULT` 行的值中, 添加 `i8042.nopnp=1 i8042.reset i8042.aux=1`

<Info>

参数说明:

 - i8042.aux=1 → 强制启用 PS/2 AUX 端口(TrackPoint 所在)

 - i8042.reset → 重置 PS/2 控制器

 - i8042.nopnp=1 → 忽略 BIOS PnP 信息(避免冲突)

</Info>

修改完之后, 执行:

```bash
sudo update-grub    # 更新grub配置

reboot              # 重启系统
```

重启之后, 就可以使用小红帽了
