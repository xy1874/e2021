# 验收与提交

## 实验报告要求

&emsp;&emsp;实验报告需按照实验包中的报告模板认真撰写。

## 验收与提交要求

&emsp;&emsp;验收时，需对系统进行现场演示。

&emsp;&emsp;提交时，首先新建"<font color=green>**学号_姓名_EmbSys实验3**</font>"文件夹，并在其中放入下列文件/文件夹：

``` C++
- 学号_姓名_EmbSys实验3
    |- XXX.doc/pdf      /* 0. 实验报告 */
    |- ssd1316          /* 1. OLED设备驱动 */
    |   |- ssd1316.c        /* OLED驱动的源文件 */
    |   |- Makefile         /* 修改过的Makefile */
    |- mask_app         /* 2. 修改过or自行编写的口罩识别应用相关的源文件及ELF文件 */
    |   |- display.c        /* 根据命令行参数，在LED点阵和OLED上显示识别结果 */
    |   |- display          /* 编译后的display程序的ELF文件 */
    |   |- Makefile         /* 修改过的Makefile */
    |   |- ssd1316.ko       /* OLED屏的显示驱动 */
    |   |- mask_detection.py /* 客户端源文件(不要求必须使用Python编写) */
    |- hilens_skill     /* 3. 修改过or自行编写的相关的HiLens技能源文件 */
        |- main.py
        |- utils.py
```

&emsp;&emsp;然后，将"学号_姓名_EmbSys实验3"文件夹压缩<font color=orange>**打包成.zip**</font>，提交到作业系统。

!!! Tips
    实验报告的撰写可参考《[无人车大赛冠军分享：华为云ModelArts和HiLens平台的联合使用](https://mp.weixin.qq.com/s/41KY-DIvikq1S3wVSD_UNQ)》

&emsp;&emsp;报告提交<font color=red>**Deadline**</font>：查看作业系统。
