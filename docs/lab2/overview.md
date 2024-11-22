# 实验概述

## 实验目的

&emsp;&emsp;1. 了解U-Boot的驱动框架

&emsp;&emsp;2. 掌握U-Boot上的编程方法

## 实验内容

&emsp;&emsp;在UBoot中添加一条LED命令用来控制ARM扩展板上LED，即在U-Boot上实现如下命令：

```bash
> led-test on <led>         # 点亮指定LED  
> led-test off <led>        # 熄灭指定LED  
> led-test toggle <led>     # 反转指定LED  
```
