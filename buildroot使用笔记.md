# buildroot 笔记

## buildroot的编译
* 操作流程
1. 解压 buildroot 压缩包
2. make menuconfig
    1. 配置Target options
    ```
        Target options
        -> Target Architecture = ARM (little endian)
        -> Target Binary Format = ELF
        -> Target Architecture Variant = cortex-A7
        -> Target ABI = EABIhf
        -> Floating point strategy = NEON/VFPv4
        -> ARM instruction set = ARM
    ```
    2. 配置 Toolchain
    ```
        Toolchain
        -> Toolchain type = External toolchain
        -> Toolchain = Custom toolchain                     /用户自己的交叉编译器
        -> Toolchain origin = Pre-installed toolchain       //预装的编译器
        -> Toolchain path =/usr/local/arm/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf   //自己编译器的路径
        -> Toolchain prefix = $(ARCH)-linux-gnueabihf       //前缀 !!!!!!!!!!!!!!!!!!!!!!!!
        -> External toolchain gcc version = 4.9.x
        -> External toolchain kernel headers series = 4.1.x
        -> External toolchain C library = glibc/eglibc
        -> [*] Toolchain has SSP support? (NEW)             //选中
        -> [*] Toolchain has RPC support? (NEW)             //选中
        -> [*] Toolchain has C++ support?                   //选中
        -> [*] Enable MMU support (NEW)                     //选中
    ```
    3. 配置 System configuration
    ```
        System configuration
        -> System hostname = cxn_sys //平台名字，自行设置
        -> System banner = Welcome to cxn system //欢迎语
        -> Init system = BusyBox //使用 busybox
        -> /dev management = Dynamic using devtmpfs + mdev //使用 mdev
        -> [*] Enable root login with password (NEW) //使能登录密码
        -> Root password = 123456 //登录密码为 123456
    ```
    4. 配置 Filesystem images
    ```
        -> Filesystem images
        -> [*] ext2/3/4 root filesystem //如果是 EMMC 或 SD 卡的话就用 ext3/ext4
        -> ext2/3/4 variant = ext4 //选择 ext4 格式
        -> [*] ubi image containing an ubifs root filesystem //如果使用 NAND 的话就用 ubifs
    ```
    5. 禁止编译 Linux 内核和 uboot
    ```
        -> Kernel
            -> [ ] Linux Kernel //不要选择编译 Linux Kernel 选项！

        -> Bootloaders
            -> [ ] U-Boot //不要选择编译 U-Boot 选项！
    ```
    6. 配置第三方软件和库的配置
    ```
        Target packages
            -> Libraries
                -> Audio/Sound
                    -> -*- alsa-lib ---> 此配置项下的文件全部选中

        Target packages
            -> Audio and video applications
                -> alsa-utils 此目录下的软件全部选中
    ```
3. (编译过程出现.stamp_configured' failed)时的处理
    ```
        vim /home/cxn/tool/arm-linux-gnueabihf/gcc-linaro/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/arm-linux-gnueabihf/libc/usr/include/linux/version.h

        把 LINUX_VERSION_CODE为262144，此值为10进制，转换为16进制为40000，对应的linux内核版本为4.0.x，在此要把该值改为与buildroot中配置的一致，即为4.1.x，转换为16进制为40100，对应的十进制为262400，将文件中改为此值保存如下
    ```
3. sudo make
4. 把 images/rootfs.tar 拷贝到 nfs 的 buildroot 目录下解压，开发版uboot修改变量，挂载到该目录下的文件系统


## buildroot 下的 busybox 配置
* 操作流程



