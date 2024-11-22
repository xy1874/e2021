# 实验概述

## 实验目的

&emsp;&emsp;1. 了解I2C的通信机制，掌握I2C设备驱动程序的开发

&emsp;&emsp;2. 进一步熟悉Socket编程

&emsp;&emsp;3. 了解使用HiLens在端设备进行智能应用开发的基本流程

## 实验内容

&emsp;&emsp;实验内容包括2部分：

&emsp;&emsp;1. 在Linux上编写OLED显示屏的I2C设备驱动程序：

&emsp;&emsp;（1）实现I2C设备驱动中的i2c_driver接口，包含dev_init()、dev_exit()、dev_i2c_remove()、
&emsp;&emsp;dev_i2c_probe()、dev_i2c_register()函数以及i2c_device_id设备ID表；

&emsp;&emsp;（2）实现I2C设备所挂接的OLED读写操作。i2c_driver只是实现设备与总线的挂接，而挂接
&emsp;&emsp;在I2C总线上的设备则千差万别。本实验要求使用Page addressing mode来写OLED屏，因此
&emsp;&emsp;需要实现基于buffer块的OLED读/写操作函数。

&emsp;&emsp;2. 使用HiLens Studio新建口罩识别技能，并开发嵌入式服务器和客户端，使GW3399能够通过Socket获取HiLens输出的数据，并将口罩识别的结果显示在ARM扩展板的8x8 LED点阵和OLED屏幕上。
