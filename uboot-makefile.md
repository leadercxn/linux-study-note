# 开发板对 uboot根目录 的信息关注
+ /arch/arm/imx-common  &&  /arch/arm/cpu/armv7
+ /board/freescale/mx6ul_14x14_evk  
+ /config/mx6ull_14x14_ddr512_emmc_defconfig
+ xxx.cmd
+ Makefile

# u-boot.xxx 文件
+ u-boot.xxx 同样也是一系列文件，包括 u-boot、u-boot.bin、u-boot.cfg、u-boot.imx、u-boot.lds、u-boot.map、 u-boot.srec、 u-boot.sym 和 u-boot-nodtb.bin，这些文件的含义如下：
    1. uu-boot：编译出来的 ELF 格式的 uboot 镜像文件。
    2. uu-boot.bin：编译出来的二进制格式的 uboot 可执行镜像文件。
    3. uu-boot.cfg： uboot 的另外一种配置文件。
    4. uu-boot.imx： u-boot.bin 添加头部信息以后的文件， NXP 的 CPU 专用文件。
    5. uu-boot.lds：链接脚本。
    6. uu-boot.map： uboot 映射文件，通过查看此文件可以知道某个函数被链接到了哪个地址上。
    7. uu-boot.srec： S-Record 格式的镜像文件。
    8. uu-boot.sym： uboot 符号文件。
    9. uu-boot-nodtb.bin：和 u-boot.bin 一样， u-boot.bin 就是 u-boot-nodtb.bin 的复制文件。

#  .config 文件
+ .config 文件中都是以“CONFIG_”开始的配置项，这些配置项就是 Makefile 中的变量，因此后面都跟有相应的值， uboot 的顶层 Makefile 或子 Makefile 会调用这些变量值。在.config 中会有大量的变量值为‘y’，这些为‘y’的变量一般用于控制某项功能是否使能，为‘y’的话就表示功能使能


# 其他细节
+ 在windows 和 ubuntu 共享文件夹下，进行uboot-xxxxxx.tar.bz2 的解压和uboot目录的编译，都会出错
+ 新建 settings.json 可以设置vscode，从而方便工程管理代码



# Makefile
+ `MAKEFLAGS` && `SHELL` 值会自动传递给 子make
    + demo
    ```
        MAKEFLAGS += -rR --include-dir=$(CURDIR)
        “+=”来给变量 MAKEFLAGS 追加了一些值，“-rR”表示禁止使用内置的隐含规则和变量定义，“--include-dir”指明搜索路径， ”$(CURDIR)”表示当前目录
    ```
+ `$(origin <variable>)`origin 用于告诉你变量是哪来的,变量 V 是在命令行定义的那么它的来源就是"command line"，这样"$(origin V)"和"commandline"就相等了
    + demo
    ```
        ifeq ("$(origin V)", "command line")
    ```
+ 日志打印
    @ 不会打印日志到终端 
    quiet = NULL，整个命令都会输出。
    quiet =quiet_，仅输出短版本。
    quiet =silent_，整个命令都不会输出



# 图形化体验 make menuconfig
+ 安装工具
    ```
        sudo apt-get install build-essential
        sudo apt-get install libncurses5-dev

        make menuconfig
    ```

+ 原理
    1. Kconfig 用来生成图形界面的配置项
    2. .config 是图形界面配置后更新参数到里面


# uboot的移植（important）
1. 取原厂的uboot `D:\正点原子linux\阿尔法Linux开发板光盘资料（A盘）-资料盘\4、NXP官方原版Uboot和Linux`
2. 拷贝到个人目录作为ubootfac `/home/cxn/linux/ubootfac/uboot-imx-rel_imx_4.1.15_2.1.0_ga`
3. cd `configs` 找出与自己开发板相似的   `defconfig文件` ，拷贝，重命名，修改 ，   `开发板默认配置文件`
    ```
        cp  mx6ull_14x14_evk_emmc_defconfig     mx6ull_alientek_cxn_emmc_defconfig

        CONFIG_SYS_EXTRA_OPTIONS="IMX_CONFIG=board/freescale/mx6ull_alientek_cxn_emmc/imximage.cfg,MX6ULL_EVK_EMMC_REWORK"  //修改
        CONFIG_ARM=y
        CONFIG_ARCH_MX6=y
        CONFIG_TARGET_MX6ULL_ALIENTEK_CXN_EMMC=y        //修改
        CONFIG_CMD_GPIO=y
    ```
5. cd `include/configs`,找到与开发板相似的`头文件`，拷贝重命名，修改 , 该文件作用 ，`配置和裁剪uboot`
    ```
        cp mx6ullevk.h   mx6ull_alientek_cxn_emmc.h
    ```
6. 添加对应的`板级文件夹` cd `board/freescale/mx6ullevk` 
    ```
        cp  mx6ullevk/  -r   mx6ull_alientek_cxn_emmc
    ```
    以及修改其文件夹里面一些文件内容 `Makefile` ,`imximage.cfg`,`Kconfig`,`MAINTAINERS`
7. 修改uboot的图形配置，`加入自己的开发板选项`, vim ` arch/arm/cpu/armv7/mx6/Kconfig`
    ```
        config TARGET_MX6ULL_ALIENTEK_EMMC                          # 207行加入
        bool "Support mx6ull_alientek_cxn_emmc"
        select MX6ULL
        select DM
        select DM_THERMAL

        source "board/freescale/mx6ull_alientek_cxn_emmc/Kconfig"   # 最后一行source.....
    ```
8. 修改显示屏参数
    1. board/freescale/mx6ull_alientek_cxn_emmc.c/display_info_t const displays[],根据自己的显示屏参数修改结构体参数
    ``` 
        struct display_info_t const displays[] = {{
            .bus = MX6UL_LCDIF1_BASE_ADDR,
            .addr = 0,
            .pixfmt = 24,
            .detect = NULL,
            .enable	= do_enable_parallel_lcd,
            .mode	= {
                .name			= "TFT7084", //"TFT43AB"
                .xres           = 800,
                .yres           = 480,
                .pixclock       = 30030,        //计算方法： (1/像素时钟sclk)*10^12
                .left_margin    = 46,           //HBP
                .right_margin   = 210,          //HFP
                .upper_margin   = 23,           //VBP
                .lower_margin   = 22,           //VFP
                .hsync_len      = 1,            //HSPW
                .vsync_len      = 1,            //VSPW
                .sync           = 0,
                .vmode          = FB_VMODE_NONINTERLACED
        } } };
        size_t display_count = ARRAY_SIZE(displays);
    ```
    2. include/configs/mx6ull_alientek_cxn_emmc.h,全修改panel 的名字
    ```
        demo:
            ···"panel=TFT7084\0" \ ···
    ```
    3. uboot启动之后，修改环境变量的名字
    ```
        setenv panel TFT7016
        saveenv
    ```
9. 网络参数
    1. include/configs/mx6ull_alientek_cxn_emmc.h/#define CONFIG_FEC_ENET_DEV  1 来选定用哪个网络设备（假如板载有两个设备）
    2. 修改地址 #define CONFIG_FEC_MXC_PHYADDR  0x0		//根据芯片的物理地址选择
    3. 芯片厂商的选择
        ``` 
            #define CONFIG_PHYLIB
            #define CONFIG_PHY_SMSC				//网络驱动芯片厂商的选择
            #endif
        ```
    4. board/freescale/mx6ull_alientek_cxn_emmc.c中修改一些IO;drivers/net/phy/phy.c/genphy_update_link() 在该函数添加自己的代码
10. 其他修改
    1. board/freescale/mx6ull_alientek_cxn_emmc.c 的checkboard()函数修改板子的名字
    2. include/configs/mx6ull_alientek_cxn_emmc.h 的CONFIG_EXTRA_ENV_SETTINGS来设置环境变量bootcmd和bootargs

# uboot的环境变量 bootcmd 和 bootargs
+ bootargs ： 板级头文件 include/configs/mx6ull_alientek_cxn_emmc.h 的 CONFIG_EXTRA_ENV_SETTINGS,bootargs 环境变量是由 mmcargs 设置的，保存着uboot传递给linux内核的参数，例如设置 控制台使用哪个串口，波特率console，根文件系统存放的位置root，文件系统类型rootfstype


+ bootcmd 是uboot启动的命令  include/configs/mx6ull_alientek_cxn_emmc.h ：CONFIG_BOOTCOMMAND,bootcmd很好的解释了uboot启动内核的过程，理解下
```
#define CONFIG_BOOTCOMMAND \                            
	   "run findfdt;" \                                     //寻找设备树文件，findfdt是NXP自行添加的环境变量
	   "mmc dev ${mmcdev};" \                               //切换 mmc设备
	   "mmc dev ${mmcdev}; if mmc rescan; then " \          //扫描有没有mmc设备或者SD卡
		   "if run loadbootscript; then " \                 //如果存在xxx脚本
			   "run bootscript; " \                         
		   "else " \
			   "if run loadimage; then " \                  //运行命令，该命令会导入zImage内核文件
				   "run mmcboot; " \                        //运行环境变量mmcboot
			   "else run netboot; " \
			   "fi; " \
		   "fi; " \
	   "else run netboot; fi"
#endif

==> 最后bootcmd运行的命令总结如下

mmc dev 1                                           //切换到 EMMC
fatload mmc 1:1 0x80800000 zImage                   //读取 zImage 到 0x80800000 处
fatload mmc 1:1 0x83000000 imx6ull-14x14-evk.dtb    //读取设备树到 0x83000000 处
bootz 0x80800000 - 0x83000000                       //启动 Linux

```

## uboot启动linux
1. 从emmc中导入内核和设备树，设置环境变量,注意后面 bootz 80800000 - 83000000; 的格式，不然出错启动不了
    ```
        ls mmc 1:1

        setenv bootargs 'console=ttymxc0,115200 root=/dev/mmcblk1p2 rootwait rw' 
        setenv bootcmd 'mmc dev 1; fatload mmc 1:1 80800000 zimage;fatload mmc 1:1 83000000 imx6ull-14x14-emmc-4.3-800x480-c.dtb;bootz 80800000 - 83000000;'
        saveenv
    ```
