# linux -- kernel
* 链接网址：https://www.kernel.org
* 安装 lzop 库
    ```
        sudo apt-get install lzop
    ```

# 编译
* 写操作脚本
```
    #!/bin/sh
        make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- distclean
        make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- imx_v7_defconfig
        make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
        make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- all -j16
```
* 生成镜像文件位置 设备树位置
    1. 内核镜像 arch/arm/boot/zImage
    2. 设备树 arch/arm/boot/dts/imx6ull-14x14-evk.dts 和 imx6ull-14x14-evk.dtb 文件





* vscode 屏蔽设置
```
{
    "search.exclude": 
    {
        "**/node_modules": true,
        "**/bower_components": true,
        "**/*.o":true,
        "**/*.su":true,
        "**/*.cmd":true,
        "Documentation":true,

        /* 屏蔽不用的架构相关的文件 */
        "arch/alpha":true,
        "arch/arc":true,
        "arch/arm64":true,
        "arch/avr32":true,
        "arch/[b-z]*":true,
        "arch/arm/plat*":true,
        "arch/arm/mach-[a-h]*":true,
        "arch/arm/mach-[n-z]*":true,
        "arch/arm/mach-i[n-z]*":true,
        "arch/arm/mach-m[e-v]*":true,
        "arch/arm/mach-k*":true,
        "arch/arm/mach-l*":true,

        /* 屏蔽排除不用的配置文件 */
        "arch/arm/configs/[a-h]*":true,
        "arch/arm/configs/[j-z]*":true,
        "arch/arm/configs/imo*":true,
        "arch/arm/configs/in*":true,
        "arch/arm/configs/io*":true,
        "arch/arm/configs/ix*":true,
        /* 屏蔽掉不用的 DTB 文件 */
        "arch/arm/boot/dts/[a-h]*":true,
        "arch/arm/boot/dts/[k-z]*":true,
        "arch/arm/boot/dts/in*":true,
        "arch/arm/boot/dts/imx1*":true,
        "arch/arm/boot/dts/imx7*":true,
        "arch/arm/boot/dts/imx2*":true,
        "arch/arm/boot/dts/imx3*":true,
        "arch/arm/boot/dts/imx5*":true,
        "arch/arm/boot/dts/imx6d*":true,
        "arch/arm/boot/dts/imx6q*":true,
        "arch/arm/boot/dts/imx6s*":true,
        "arch/arm/boot/dts/imx6ul-*":true,
        "arch/arm/boot/dts/imx6ull-9x9*":true,
        "arch/arm/boot/dts/imx6ull-14x14-ddr*":true,
    },
    "files.exclude": 
    {
        "**/.git": true,
        "**/.svn": true,
        "**/.hg": true,
        "**/CVS": true,
        "**/.DS_Store": true,
        "**/*.o":true,
        "**/*.su":true,
        "**/*.cmd":true,
        "Documentation":true,
        /* 屏蔽不用的架构相关的文件 */
        "arch/alpha":true,
        "arch/arc":true,
        "arch/arm64":true,
        "arch/avr32":true,
        "arch/[b-z]*":true,
        "arch/arm/plat*":true,
        "arch/arm/mach-[a-h]*":true,
        "arch/arm/mach-[n-z]*":true,
        "arch/arm/mach-i[n-z]*":true,
        "arch/arm/mach-m[e-v]*":true,
        "arch/arm/mach-k*":true,
        "arch/arm/mach-l*":true,
        /* 屏蔽排除不用的配置文件 */
        "arch/arm/configs/[a-h]*":true,
        "arch/arm/configs/[j-z]*":true,
        "arch/arm/configs/imo*":true,
        "arch/arm/configs/in*":true,
        "arch/arm/configs/io*":true,
        "arch/arm/configs/ix*":true,
        /* 屏蔽掉不用的 DTB 文件 */
        "arch/arm/boot/dts/[a-h]*":true,
        "arch/arm/boot/dts/[k-z]*":true,
        "arch/arm/boot/dts/in*":true,
        "arch/arm/boot/dts/imx1*":true,
        "arch/arm/boot/dts/imx7*":true,
        "arch/arm/boot/dts/imx2*":true,
        "arch/arm/boot/dts/imx3*":true,
        "arch/arm/boot/dts/imx5*":true,
        "arch/arm/boot/dts/imx6d*":true,
        "arch/arm/boot/dts/imx6q*":true,
        "arch/arm/boot/dts/imx6s*":true,
        "arch/arm/boot/dts/imx6ul-*":true,
        "arch/arm/boot/dts/imx6ull-9x9*":true,
        "arch/arm/boot/dts/imx6ull-14x14-ddr*":true,
    }
}



```

# vmlinux,Image,zImage,uImage
* vmlinux 是elf格式文件，是编译出来的最原始`内核文件`，是未压缩的，略大 ，约16M
* Image是linux`内核镜像文件`,仅包含可执行的`二进制文件`,也是未压缩的，是把vmlinux取消一些信息，保存在 arch/arm/boot ，约12M
* zImage 是把Image经gzip压缩生成的，约6M，经常使用。使用命令"make"、"make all"、"make zImage"生成,`常用`
* uImage 是在zImage前面加一个长度为64字节的 “头”， 该头信息描述 镜像文件的类型，加载位置，生成时间，大小等信息。是老版本uboot专用的镜像文件，很少用到; 新uboot已经支持zImage启动。


# Linux内核启动[大致讲解]
* linux内核的链接脚本文件 arch/arm/kernel/vmlinux.lds  ENTRY(stext) 指明了linux内核入口，入口为stext  ==> arch/arm/kernel/head.S  ENTRY(stext) 
* init/main.c 初始化内核  start_kernel(void)
```
    start_kernel(void);
    rest_init(void);
    kernel_init(void *unused);  //是init进程具体的工作
```

* linuxn内核启动过于复杂，后期自己有空再分析

# initrd 的理解
* 链接
    [什么是initrd?](https://blog.csdn.net/weixin_34237596/article/details/94293148?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EsearchFromBaidu%7Edefault-2.pc_relevant_baidujshouduan&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EsearchFromBaidu%7Edefault-2.pc_relevant_baidujshouduan)



# 内核移植
1. 添加开发板默认配置文件
    * `arch/arm/config/`imx_v7_defconfig 
        ```
            cp arch/arm/configs/imx_v7_defconfig  imx_v7_emmc_cxn_defconfig imx_v7_emmc_cxn_defconfig
        ```
2. 添加对应的设备树文件 ( .dts 编译生成.dtb )
    * `arch/arm/boot/dts` ，可以直接在设备树文件夹里，命令： make dtbs 来编译设备树
        ```
            cp arch/arm/boot/dts/imx6ull-14x14-evk-emmc.dts  imx6ull_emmc_cxn.dts
        ```
    * 修改该目录下的Makefile，增加对应的.dtb文件
        ```
            dtb-$(CONFIG_SOC_IMX6ULL) += \
            ···
            imx6ull_emmc_cxn.dtb \
            ···
        ```
3. 修改主频
    * `arch/arm/config/`imx_v7_emmc_cxn_defconfig 
    ```
        #CONFIG_CPU_FREQ_DEFAULT_GOV_ONDEMAND=y
        CONFIG_CPU_FREQ_GOV_ONDEMAND=y
    ```
    * 启动内核之后，cat /sys/devices/system/cpu/cpu0/cpufreq/cpuinfo_cur_freq 查看当前cpu主频，cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor 查看当前调频策略， performance为高性能模式 , ondemand 按需响应模式，也可以通过make menuconfig图形配置
4. 修改自己的设备树文件，修改emmc驱动为8线模式
5. 修改设备树文件，修改网络驱动，IO
    * 另外还需修改 drivers/net/ethernet/freescale/fec_main.c
    ```
        fec_probe 中加入设置 MX6UL_PAD_ENET1_TX_CLK 和 MX6UL_PAD_ENET2_TX_CLK，这两个 IO 的复用寄存器的 SION 位为 1。
    ```
    * 修改 drivers/net/phy/smsc.c  文件的static int smsc_phy_reset(struct phy_device *phydev)函数
    * make menuconfig  来修改支持使能 LAN8720 驱动
6. 通过make menuconfig 生成的 .config的保存方法
    1. make menuconfig 生成.config后，重命名为 imx_v7_emmc_cxn_defconfig , 然后保存到 arch/arm/configs 的目录下
    2. 在make menuconfig 图形界面，修改好选项后，选择 save ，然后在弹出来的框上添上路径和保存为文件的名字，即可生成



100. 在uboot下 download ubuntu的文件命令
```
    tftp 80800000 zImage
    tftp 83000000 imx6ull_emmc_cxn.dtb
    bootz 80800000 - 83000000
``` 
也可以在uboot下设置环境变量，让其开机的时候执行执行环境变量，直接从网络下载内核镜像和设备树文件
```
setenv bootcmd 'tftp 80800000 zImage;tftp 83000000 imx6ull_emmc_cxn.dtb;bootz 80800000 - 83000000'
setenv bootcmd 'tftp 80800000 zImage;tftp 83000000 imx6ull-14x14-evk-emmc.dtb;bootz 80800000 - 83000000'
```



