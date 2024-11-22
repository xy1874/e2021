# U-Boot命令实现及驱动模型

## 1. U-Boot命令实现方式

### 1.1 命令行的处理

&emsp;&emsp;系统上电后，RK3399芯片先从BootRom获取SPL代码，然后加载U-Boot到内存中运行（详见“Bootloader启动流程分析”）。在所有初始化程序执行完成之后，U-Boot调用run_main_loop函数以进入主循环，并在主循环中进入命令行模式，以获取串口数据，从而解析和执行用户输入的命令行命令。

``` C++
static int run_main_loop(void)
{
#ifdef CONFIG_SANDBOX
    sandbox_main_loop_init();
#endif
    /* main_loop() can return to retry autoboot, if so just run it again */
    for (;;)
        main_loop();
    return 0;
}
```

&emsp;&emsp;U-Boot把所有命令的数据结构都放在命令表中。命令表的每一行都代表一个命令，其数据类型是cmd_tbl_t。这些命令在链接时会被链接到指定的段中，因此也可以认为命令表是被定义在这个段中。

### 1.2 U_BOOT_CMD宏定义

&emsp;&emsp;U-Boot命令在common目录下实现，且该目录下的单个C文件通常会实现一条或多条命令。如果要增加一条新命令，则首先需要在common目录下新建相应的C文件，然后在Makefile中指定编译该文件。

&emsp;&emsp;定义新的U-Boot命令时，需要使用U_BOOT_CMD的宏定义。

<!-- 可参考common/console.c文件编写led-test.c。 -->

&emsp;&emsp;U_BOOT_CMD是定义在include/command.h中的宏：

``` C++
#define U_BOOT_CMD(_name, _maxargs, _rep, _cmd, _usage, _help)              \
    U_BOOT_CMD_COMPLETE(_name, _maxargs, _rep, _cmd, _usage, _help, NULL)
```

&emsp;&emsp;而U_BOOT_CMD_COMPLETE也是一个宏定义：

``` C++
#define U_BOOT_CMD_COMPLETE(_name, _maxargs, _rep, _cmd, _usage, _help, _comp) \
    ll_entry_declare(cmd_tbl_t, _name, cmd) =                               \
        U_BOOT_CMD_MKENT_COMPLETE(_name, _maxargs, _rep, _cmd,              \
                                    _usage, _help, _comp);
```

&emsp;&emsp;由上述代码可知，U_BOOT_CMD_COMPLETE被定义为一个赋值语句。

&emsp;&emsp;赋值语句右侧是另一个宏定义U_BOOT_CMD_MKENT_COMPLETE：

```c++
#define U_BOOT_CMD_MKENT_COMPLETE(_name, _maxargs, _rep, _cmd,              \
                                    _usage, _help, _comp)                   \
        { #_name, _maxargs, _rep, _cmd, _usage,                             \
            _CMD_HELP(_help) _CMD_COMPLETE(_comp) }
```

!!! 说明
    \#_name是将\_name传递的值字符串化，\_CMD_HELP和\_CMD_COMPLETE就是取本身。
    
    具体宏定义见include/command.h。

&emsp;&emsp;赋值语句左侧的ll_entry_declare也是一个宏定义（详见include/linker_lists.h）：

```c++
#define ll_entry_declare(_type, _name, _list)                               \
    _type _u_boot_list_2_##_list##_2_##_name __aligned(4)                   \
        __attribute__((unused, section(".u_boot_list_2_"#_list"_2_"#_name)))
```

!!! 说明
    “##”和“#”都是预编译操作符，“##”有字符串连接的功能，而“#”则表示后面紧接着的是一个字符串。

    \_type：传递变量类型

    \_name，\_list：传递组成变量名称的字符串，并将该变量放在section中

    _aligned(4)：指定定义的变量4字节对齐

    _attribute：选择未使用的section。

&emsp;&emsp;此外，U_BOOT_CMD_COMPLETE中定义了一个类型为cmd_tbl_t的结构体并对其赋值。

&emsp;&emsp;在U-Boot中，每个命令都要用这样的一个结构体来描述，其定义如下：

``` C++
typedef struct cmd_tbl_s cmd_tbl_t;
struct cmd_tbl_s {
    char *name;         /* Command Name */
    int maxargs;        /* maximum number of arguments */
    int repeatable;     /* autorepeat allowed? */
    /* Implementation function */
    int (*cmd)(struct cmd_tbl_s*, int, int, char* const[]);
    char *usage;        /* Usage message (short) */

#ifdef CONFIG_SYS_LONGHELP
    char *help;         /* Help message (long) */
#endif

#ifdef CONFIG_AUTO_COMPLETE
    /* do auto completion on the arguments */
    int (*complete)(int argc, char* const argv[], char last_char, int maxv, char* cmdv[]);
#endif
};
```

!!! 说明
    成员cmd是一个函数指针，指向该命令对应的处理函数。

    所有命令的处理函数的接口都是一致的。

&emsp;&emsp;将上述宏定义全部展开，可得如下结构体：

``` C++
struct cmd_tbl_s _u_boot_list_2_cmd_2_##_name = {
    #_name, _maxargs, _rep, _cmd, _usage, _help, NULL
};
```

&emsp;&emsp;综上所述，U_BOOT_CMD就是存放在U-Boot中的空闲section内的一个结构体变量。

&emsp;&emsp;U-Boot编译完成后，可在arch/arm/cpu/armv8/u-boot-spl.lds和u-boot.map中看到映射的地址：

``` lds
.u_boot_list : {
    . = ALIGN(8);
    KEEP(*(SORT(.u_boot_list*)));
} >.sram

...

.rela.u_boot_list_2_cmd_2_led-test
                0x00000000002b61b8       0x60 arch/arm/cpu/armv8/start.o
```

&emsp;&emsp;从u-boot.map可以看到，led-test（需将编写的led-test.c放到common目录下，修改Makefile文件，编译后才能看到led-test段）已经放到u_boot_list_2段中。

&emsp;&emsp;运行时，U-Boot从串口接收用户的命令，然后从u_boot_list_2段中依次查找每个cmd_tbl_t，根据成员name比较是否和用户命令匹配。如果匹配成功，则执行第三个成员cmd指向的函数，否则查找下一个cmd_tbl_t。具体实现可以参考find_cmd函数。

## 2. U-Boot驱动模型

&emsp;&emsp;U-Boot的驱动模型（Driver Model，简称DM）为驱动的定义和访问接口提供了统一的方法，提高了驱动之间的兼容性以及访问的标准。

!!! 说明
    DM的详细说明见u-boot/doc/driver-model/README.txt。

&emsp;&emsp;DM中包含三个基本概念：Device、Driver和Uclass。

- Device

&emsp;&emsp;Device指设备对象。每个Device绑定到一个具体的端口或外设，在README中的说明如下：

> Device - an instance of a driver, tied to a particular port or peripheral.

- Driver

&emsp;&emsp;Driver指设备驱动。Driver实现和底层硬件设备的通信，并且为设备提供上层访问接口，在README中的说明如下：

> Driver - some code which talks to a peripheral and presents a higher-level interface to it.

!!! 举个栗子
    RK3399的GPIO Driver提供了gpio_direction_input、gpio_direction_output和gpio_get_value等接口。

- Uclass

&emsp;&emsp;Uclass为同一组类型的设备提供统一接口，在README中的说明如下：

> Uclass - a group of devices which operate in the same way. A uclass provides a way of accessing individual devices within the group, but always using the same interface. For example a GPIO uclass provides operations for get/set value. An I2C uclass may have 10 I2C ports, 4 with one driver, and 6 with another.

!!! 举个栗子
    GPIO Uclass提供了gpio_request和gpio_free接口

## 3. U-Boot中的GPIO驱动模型

&emsp;&emsp;GPIO的驱动模型主要包括GPIO Core、GPIO Uclass、GPIO Udevice和GPIO Udriver四部分，如下图所示。

<center><img src="../assets/2-3.svg" width = 500></center>

- GPIO Core（实现：u-boot/drivers/gpio/gpio-uclass.c）

&emsp;&emsp;GPIO Core为上层提供GPIO设备的访问接口。GPIO Core从设备树arch/arm/dts/rk3399.dtsi中获取GPIO属性，并从GPIO Uclass的设备链表中获取相应的Udevice设备。

- GPIO Uclass（实现：u-boot/drivers/gpio/gpio-uclass.c）

&emsp;&emsp;GPIO Uclass为GPIO Udevice的驱动提供统一的操作集接口，并链接属于该Uclass的所有GPIO Udevice设备。

- GPIO Udevice（实现：u-boot/drivers/gpio/rk_gpio.c）

&emsp;&emsp;RK3399共有5组GPIO bank，每组包含32个IO接口。其中，每个GPIO bank都有一个GPIO Udevice与之对应。

```c++
enum {
    ROCKCHIP_GPIOS_PER_BANK = 32,
};

static int rockchip_gpio_probe(struct udevice *dev)
{
    struct gpio_dev_priv *uc_priv = dev_get_uclass_priv(dev);
    struct rockchip_gpio_priv *priv = dev_get_priv(dev);
    char *end;
    int ret;

    priv->regs = dev_read_addr_ptr(dev);
    ret = uclass_first_device_err(UCLASS_PINCTRL, &priv->pinctrl);
    if (ret) return ret;

    uc_priv->gpio_count = ROCKCHIP_GPIOS_PER_BANK;
    end = strrchr(dev->name, '@');
    priv->bank = trailing_strtoln(dev->name, end);
    priv->name[0] = 'A' + priv->bank;
    uc_priv->bank_name = priv->name;

    return 0;
}
```

``` C++
U_BOOT_DRIVER(gpio_rockchip) = {
    .name = "gpio_rockchip",
    .id = UCLASS_GPIO,
    .of_match = rockchip_gpio_ids,
    .ops = &gpio_rockchip_ops,
    .priv_auto_alloc_size = sizeof(struct rockchip_gpio_priv),
    .probe = rockchip_gpio_probe,
};
```

- GPIO Udriver（实现：u-boot/drivers/gpio/rk_gpio.c）

&emsp;&emsp;已知RK3399共有5组GPIO bank，每组包含32个IO接口。GPIO驱动模型使用bank内偏移来表示具体的GPIO编号。GPIO Udriver根据bank编号和bank内偏移找到对应的GPIO，并对相应寄存器的相应比特位进行操作。
