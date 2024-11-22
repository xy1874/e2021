# BootLoader概述

## 1. BootLoader的概念

&emsp;&emsp;引导加载程序是系统加电后运行的第一段软件代码。回忆一下PC的体系结构我们可以知道，PC机中的引导加载程序由BIOS（本质是一段固件程序）和位于硬盘MBR中的BootLoader（如LILO、GRUB等）组成。BIOS在完成硬件检测和资源分配后，将硬盘MBR中的BootLoader读到系统的RAM中，然后将控制权交给BootLoader。BootLoader的主要任务是将OS内核镜像从硬盘上读到RAM中，然后跳转到OS内核的入口，也即开始启动操作系统。

&emsp;&emsp;在嵌入式系统中，一般并没有像BIOS那样的固件程序，因此整个系统的加载启动任务就完全由BootLoader来完成。

!!! 注意
    有的嵌入式CPU也会内嵌一段短小的启动程序。

&emsp;&emsp;Bootloader的中文名是启动引导程序，它可以工作在无操作系统的环境下，也可以工作在有操作系统的环境下。在无操作系统环境下通常表现为：与应用程序编译在一起，在应用程序之前运行的一段代码，一般由汇编编写，完成基本硬件的初始化，为应用程序做准备。

&emsp;&emsp;通常说的BootLoader一般特指在操作系统环境下：在操作系统运行之前运行的一段或多段程序。BootLoader的功能是初始化硬件设备、建立系统的内存空间映射图，将系统的软件硬件环境带到一个合适的状态，为调用操作系统内核准备好正确的环境，把操作系统内核映像加载到RAM中，并将系统控制权交给它。

## 2. Bootloader的分类

&emsp;&emsp;按照CPU的不同，BootLoader的要求也不一样

&emsp;&emsp;1. 针对X86上有LILO、GRUB、ntloader等

&emsp;&emsp;2. 针对ARM架构的有u-boot、vivi、armboot等

&emsp;&emsp;3. 针对ppc架构的有ppcboot等
