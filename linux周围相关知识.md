# NorFlash 、 NandFlash、eMMC 的区别
+ NorFlash , 经典代表W25QXX系列的ROM芯片，较长的抹写时间
+ NandFlash ,要外部加主控和电路设计。。NAND Flash具有较快的抹写时间, 而且每个存储单元的面积也较小，这让NAND Flash相较于NOR Flash具有较高的存储密度与较低的每比特成本。同时它的可抹除次数也高出NOR Flash十倍 . 因为多数微处理器与微控制器要求字节等级的随机存取，所以 `NAND Flash不适合取代那些用以装载程序的ROM`。从这样的角度看来，`NAND Flash比较像光盘、硬盘这类的次级存储设备`。NAND Flash非常适合用于储存卡之类的大量存储设备。
+ eMMC , eMMC (Embedded Multi Media Card) ,`eMMC 相当于 NandFlash+主控IC` ，对外的接口协议与`SD、TF`卡一样,eMMC的一个明显优势是在封装中集成了`一个控制器`，它提供标准接口并管理闪存，


# zImage 和 uImage 的关系
+ `vmlinux`  编译出来的最原始的内核文件，未压缩。
+ `zImage`   是vmlinux经过gzip压缩后的文件。
+ `bzImage` bz表示“big zImage”，不是用bzip2压缩的。两者的不同之处在于，zImage解压缩内核到低端内存(第一个640K)，bzImage解压缩内核到高端内存(1M以上)。如果内核比较小，那么采用zImage或bzImage都行，如果比较大应该用bzImage。
+ `uImage`   U-boot专用的映像文件，它是在zImage之前加上一个长度为0x40的tag。
+ `vmlinuz`  是bzImage/zImage文件的拷贝或指向bzImage/zImage的链接。
+ `initrd`   是“initial ramdisk”的简写。一般被用来临时的引导硬件到实际内核vmlinuz能够接管并继续引导的状态

+ 说明：
    vmlinux是内核文件，`zImage`是一般情况下默认的压缩内核映像文件，压缩`vmlinux`，加上一段解压启动代码得到。而`uImage`则是使用工具`mkimage`对普通的压缩内核映像文件（zImage）加工而得。`uImage`是uboot专用的映像文件，它是在zImage之前加上一个长度为64字节的“头tag”，说明这个内核的版本、加载位置、生成时间、大小等信息；其0x40之后与zImage没区别。其实就是一个自动跟手动的区别,有了uImage头部的描述,u-boot就知道对应Image的信息,如果没有头部则需要自己手动去搞那些参数。如何生成uImage文件？首先在uboot的/tools目录下寻找mkimage文件，把其copy到系统/usr/local/bin目录下，这样就完成制作工具。然后在内核目录下运行make uImage，如果成功，便可以在arch/arm/boot/目录下发现uImage文件，其大小比 zImage多64个字节。此外，平时调试用uImage，不用去管调整了哪些东西；zImage则是一切OK后直接烧0x0,开机就运行。  `zImage + 压缩 => uImage`

# 相关I.MX6U => uboot.imx与uboot.bin的关系
+ u-boot.imx与u-boot.bin文件的主要关系是：`u-boot.imx`是在`u-boot.bin`的前面附加上一个image header，主要包含IVT header、 Boot data、DCD header；整个header的大小限制为3Kbyte。
+ .imx 构成 .bin文件
|  eMMC       |.imx |
|  1K offset  |     |
|  IVT Header |     |
|  Boot data  |     |
|  DCD Header |     |
|  U-boot.bin |     |

# IMX6ULL平台资源介绍
1. Cortex-A7架构，主频792MHz
    1. Cortex-A7 MPcore 处理器在一个处理器上选配 1~4 个内核
















