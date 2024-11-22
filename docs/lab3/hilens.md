# 基于HiLens的口罩识别

## 1. HiLens简介

&emsp;&emsp;HiLens是华为开发的一个端云协同的多模态AI开发应用平台，提供简单易用的开发框架、开箱即用的开发环境、丰富的AI技能市场和云上管理平台，对接多种端侧计算设备，支持视觉及听觉AI应用开发、AI应用在线部署、海量设备管理等。

&emsp;&emsp;HiLens云侧提供开发框架HiLens Framework、开发环境HiLens Studio、管理平台和多模态AI技能，供用户在云侧选购或开发技能，用户可以安装技能到端侧设备，使端侧设备具备AI技能，也可以发布技能至技能市场，共享给其他用户，如图1所示。

<center><img src="../assets/4-1.png"></center>
<center>图1 Hilens云侧架构图</center>

&emsp;&emsp;更多关于HiLens的信息，请访问[HiLens的官方网站](https://www.huaweicloud.com/product/hilens.html)。

## 2. 系统概述

&emsp;&emsp;本实验要求设计基于HiLens的口罩识别系统，该系统包括HiLens智能摄像头和GW3399嵌入式平台。其中，HiLens和GW3399通过局域网进行通信，且GW3399通过总线连接ARM扩展板并在其上显示识别结果，如图2所示。

<center><img src="../assets/4-2.jpg" width = 500></center>
<center>图2 实物连接</center>

&emsp;&emsp;基于HiLens的口罩识别系统主要由HiLens智能摄像头和GW3399嵌入式硬件平台2部分组成。其中，HiLens通过摄像头采集人脸图像数据，并通过运行于其上的口罩识别技能判断摄像头前的人是否佩戴口罩。

&emsp;&emsp;口罩的识别结果需通过Socket传输到作为客户端的GW3399平台上。口罩的识别结果有三种：“has_mask”、“no_mask”和“unknown”，分别表示戴口罩、未戴口罩、未检测到人脸。

&emsp;&emsp;GW3399接收到识别结果后，调用一个称为display的应用程序来在ARM扩展板上显示口罩识别结果。

&emsp;&emsp;display的作用是根据识别结果，通过驱动程序将相应的显示数据输出到OLED显示屏和8*8的LED点阵。

&emsp;&emsp;若识别结果为“has_mask”，则OLED需显示“has mask”字样，且LED点阵需显示一张笑脸，如图3所示。

<center><img src="../assets/4-3.jpg" width = 400></center>
<center>图3 检测到戴口罩</center>

&emsp;&emsp;同理，若识别结果为“no_mask”，则OLED需显示“no mask”字样，且LED点阵需显示一张哭脸；若识别结果为“unknown”，则OLED需显示“unknown”字样，且LED点阵需显示一个问号，如图2所示。
