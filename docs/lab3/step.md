# 实验步骤

## 1. 完成OLED的驱动开发

### 1.1 解压实验包

&emsp;&emsp;首先将gw3399-driver.tar.gz拷贝到gw3399-sdk所在的目录下，使用tar命令解压。

&emsp;&emsp;可见gw3399-driver目录结构如下：

``` C
- gw3399-driver
    |- module       /* 驱动程序 */
        |- ht16k33      /* 8x8LED点阵屏的驱动 */
        |- ssd1316      /* OLED屏的驱动 */
    |- app          /* 测试程序 */
    |- output       /* 存放输出文件 */
```

### 1.2 编写OLED驱动程序

!!! 提示
    本实验提供了8x8 LED点阵的I2C设备驱动程序，同学们可以参考该驱动程序来编写OLED的I2C设备驱动程序。

&emsp;&emsp;进入ssd1316目录，需要编写ssd1316.c 和Makefile 两个文件

&emsp;&emsp;1. 实现OLED i2c设备驱动中的i2c_driver接口：dev_init()、dev_exit()、dev_i2c_remove()、
&emsp;&emsp;dev_i2c_probe()、dev_i2c_register()，以及i2c_device_id设备ID表。

&emsp;&emsp;2. 实现i2c设备所挂接的OLED读写接口：buffer_read()、buffer_write()

### 1.3 编译驱动及测试程序

&emsp;&emsp;进入ht16k33目录，通过make指令，编译驱动程序，生成ht16k33.ko文件：

``` shell
$ cd ht16k33/
$ make
```

&emsp;&emsp;将生成的.ko文件复制到output输出目录下，然后清理过程文件：

``` shell
$ sudo cp ht16k33.ko ../../output/
$ make clean
```

&emsp;&emsp;进入ssd1316目录，通过make指令，编译驱动程序，生成ssd1316.ko文件：

``` shell
$ cd ../ssd1316/
$ make
```

&emsp;&emsp;将生成的.ko文件复制到output输出目录下，然后清理过程文件：

``` shell
$ sudo cp ssd1316.ko ../../output/
$ make clean
```

&emsp;&emsp;进入app目录，通过make指令，编译驱动程序，生成可执行文件demo和udpfw：

``` shell
$ cd ../../app/
$ make
```

&emsp;&emsp;将生成的可执行文件复制到output输出目录下，然后清理过程文件：

``` shell
$ sudo cp demo ../output/
$ sudo cp udpfw ../output/
$ make clean
```

&emsp;&emsp;最后，通过tar指令将output目录中的输出文件打包，使用MobaXterm tftp协议拷贝windows系统 D:/FirmWare，并清空output目录：

``` shell
$ cd ../output/
$ sudo tar -czvf ../mydriver.tar.gz ./
$ cd ../
$ sudo rm -rf output/*
```

### 1.4 驱动模块的加载与测试

&emsp;&emsp;在实验1中，我们烧录了一个不带桌面的Ubuntu系统。此处为了能方便调试，同学们需要重新烧录一个带桌面的ubuntu系统。将服务器上的Ubuntu_Desktop.7z下载并解压，得到ubuntu-rootfs-20190609.img和相应的parameter.txt。利用实验1种讲述的烧录部分固件的方法，将该文件系统烧录到GW3399，从而获得带桌面的Ubuntu系统。（注意：文件系统比较大，烧录时间较长，请耐心等待。）

&emsp;&emsp;烧录后将自动重启，进入Ubuntu系统。连接校园WIFI：HITSZ，并到浏览器中输入10.248.98.2网址，输入自己的上网账号和密码。

!!! 注意
    为了增强WIFI信号，需要为GW3399接上WIFI天线。

&emsp;&emsp;将GW3399连接到网络后，用图形界面打开终端模拟器（也可通过屏幕键盘点击快捷键Ctrl+Alt+T），如图1所示。

<center><img src="../assets/5-1.png" width = 350></center>
<center>图1 打开终端</center>

&emsp;&emsp;用ping指令检测GW3399与Windows系统的网络是否通畅。

&emsp;&emsp;通过tftp服务，从Windows系统D:/FirmWare，下载mydriver.tar.gz

!!! 注意
    这一步必须确保GW3399与Windows之间网络通畅。

&emsp;&emsp;解压mydriver.tar.gz到~/mydriver目录下：

``` shell
$ sudo tar -xvzf mydriver.tar.gz
```

&emsp;&emsp;因为GW3399平板用户登录后，就把相关的音频设备给占用了，所以为了演示音频测试效果，我们**需要到GW3399的屏幕上做以下操作：**

&emsp;&emsp;加载8x8点阵驱动和OLED屏驱动，然后运行测试程序：

（**注意：以下操作需要在GW3399平板上进行**）

``` shell
$ sudo insmod ht16k33.ko         # 加载8x8点阵屏驱动
$ sudo insmod ssd1316.ko         # 加载OLED屏驱动
$ sudo ./demo                    # 运行测试程序
```

&emsp;&emsp;运行成功后，4个LED来回闪烁、OLED屏上滚动显示当前TVOC测量值：

<center><img src="../assets/5-2.png" width = 300></center>
<center>图1 测试OLED</center>

&emsp;&emsp;接下来，可通过两个按键，切换工作状态。若OLED可以正常显示字符，则驱动成功。

## 2. display应用及客户端开发

&emsp;&emsp;编写mask_detection.py，该程序主要是socket客户端。客户端接收服务器的口罩识别结果，并调用display程序进行显示。

&emsp;&emsp;补全实验包中的display.c源码文件，使得其能够根据客户端的接收到的数据，分别在8*8 LED点阵和OLED显示屏上显示识别结果。

&emsp;&emsp;要求：

&emsp;&emsp;（1）识别到已佩戴口罩时，OLED需显示“has mask”字样，且LED点阵需显示一张笑脸；

&emsp;&emsp;（2）识别到未佩戴口罩时，OLED需显示“no mask”字样，且LED点阵需显示一张哭脸；

&emsp;&emsp;（3）未识别到人脸时，OLED需显示“unknown”字样，且LED点阵需显示一个问号。

!!! 注意
    此处的socket仅作为一种参考方案，即服务器和客户端之间的通信方式不作限制，同学们可自行发挥。

    此外，客户端并非必须要使用Python实现，同学们也可自行发挥。

## 3. 口罩识别技能开发

### 3.1 登录华为IAM账户

&emsp;&emsp;打开浏览器，访问[华为云IAM用户的登录页面](https://auth.huaweicloud.com/authui/login.html?service=https%3A%2F%2Fconsole.huaweicloud.com%2Fhilens%2F#/login)。在登录框的第一行处输入“t2_test1”，在第二行处输入“embsysX”（X = 组号 - 1，即如果组号为7，那么X为6），初始密码是“HITSZ2021”，如图3所示。

<center><img src="../assets/5-3.png" width = 350></center>
<center>图3 登录IAM账户</center>

&emsp;&emsp;登录成功后，需设置初始密码。

!!! 友情提醒
    请务必记住所设置的密码。若忘记密码，可通过登录页面的“忘记密码”找回密码。若找回失败，请及时联系老师重置密码。

&emsp;&emsp;设置新密码后，将服务器区切换到“华北-北京四”，然后同意HiLens相关的授权。

&emsp;&emsp;然后，点击页面左侧中间的小箭头，展开菜单，如图4所示。

<center><img src="../assets/5-4.png" width = 200></center>
<center>图4 展开控制台菜单</center>

&emsp;&emsp;接着，在控制台菜单的顶部，把当前的工作空间切换到workspace_embsysX（X = 组号 - 1），如图5所示。

<center><img src="../assets/5-5.png" width = 220></center>
<center>图5 切换工作空间</center>

### 3.2 HiLens激活

&emsp;&emsp;切换工作空间后，打开HiLens管理控制台的页面，点击“设备管理”->“设备列表”，查看列表中的HiLens设备，如图6所示。

<center><img src="../assets/5-6.png" width = 550></center>
<center>图6 查看HiLens设备</center>

&emsp;&emsp;若“基础技能权限”处显示红字“未激活”，则需要先激活设备才能进行后续的开发；若显示绿字“已激活”，则可以跳过激活的步骤，直接开始技能开发。

&emsp;&emsp;欲激活HiLens，只需点击“权限激活”->“测试样机激活”，然后输入HiLens背部标签上的20位SN编码即可，如图7、图8所示。

<center><img src="../assets/5-7.png"></center>
<center>图7 激活HiLens</center>

<center><img src="../assets/5-8.png" width = 500></center>
<center>图8 查看HiLens的SN编号</center>

### 3.2 HiLens技能开发

&emsp;&emsp;登录账号后，打开HiLens管理控制台的页面，展开菜单栏下的“技能开发”，点击“HiLens Studio”，如图9所示。

<center><img src="../assets/5-9.png"></center>
<center>图9 点击“技能开发”下的“HiLens Studio”</center>

&emsp;&emsp;随后将弹出如图10所示的HiLens Studio指引的页面，等候其自动完成即可。

<center><img src="../assets/5-10.png"></center>
<center>图10 等待页面</center>

&emsp;&emsp;接下来将进入HiLens Studio页面。此时，点击File->New Project，或点击“Getting Started”页面下的“New Project”以新建技能，如图11所示。

<center><img src="../assets/5-11.png"></center>
<center>图11 新建技能</center>

&emsp;&emsp;随后将弹出一个单独的页面，选择口罩识别即可，如图12所示。

<center><img src="../assets/5-12.png" width = 500></center>
<center>图12 新建口罩识别技能</center>

&emsp;&emsp;根据需要修改左侧的工程基本信息，确认无误后点击“确定”按钮，如图13所示。

<center><img src="../assets/5-13.png"></center>
<center>图13 查看和编辑工程的基本信息</center>

&emsp;&emsp;打开src文件夹下的main.py和utils.py，修改相关代码。

!!! Tips
    HiLens Studio会自动保存修改过的源文件。

&emsp;&emsp;技能编写完成后，可对其进行调试。

&emsp;&emsp;调试前，先从菜单栏选择输入的视频数据源，如图14所示。

<center><img src="../assets/5-14.png"></center>
<center>图14 选择调试的数据源</center>

&emsp;&emsp;调试时，可利用菜单栏的Debug菜单或菜单栏右侧的快捷按钮，如图15所示。

<center><img src="../assets/5-15.png"></center>
<center>图15 调试入口</center>

### 3.3 HiLens技能管理

&emsp;&emsp;技能就绪后，打开右侧的HiLens Widget页面，在HiLens上安装技能，如图16所示。

<center><img src="../assets/5-16.png"></center>
<center>图16 安装技能到HiLens</center>

&emsp;&emsp;安装完成后，点击“Start”启动技能，如图17所示。

<center><img src="../assets/5-17.png" width = 300></center>
<center>图17 启动安装好的技能</center>

&emsp;&emsp;技能启动后，既可以在HiLens Studio中重启、停止技能，也可以在HiLens管理控制台的“设备管理”->“设备列表”中查看和操作技能的运行状态，如图18所示。

<center><img src="../assets/5-18.png"></center>
<center>图18 查看和管理技能</center>
