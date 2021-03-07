# MfgTool工具  (NXP给I.MX系列CPU烧写系统的软件)

## 原理
0. 操作步骤：开发板要选USB启动，拨码开关要选择，TF卡要拔出
1. 将 firmware 目录中的 uboot、 linux kernel 和.dtb(设备树)，没有文件系统，然后通过 USB OTG 将这个文件下载到开发板的 DDR 中，目的就是在 DDR 中启动 Linux 系统，为后面的烧写做准备
2. 经过第1步的操作，此时 Linux 系统已经运行起来了，系统运行起来以后就可以很方便的完成对 EMMC 的格式化、分区等操作。 EMMC 分区建立好以后就可以从 files 中读取要烧写的 uboot、 linux kernel、 .dtb(设备树)和 rootfs 这 4 个文件，然后将其烧写到 EMMC 中。

## 相关工具的配置文件说明
* L4.1.15_2.0.0-ga_mfg-tools\mfgtools-with-rootfs\mfgtools\cfg.ini 保存一些变量，
* L4.1.15_2.0.0-ga_mfg-tools\mfgtools-with-rootfs\mfgtools\Profiles\Linux\OS Firmware\ucl2.xml 操作文件，选定型号，选定储存的设备,针对烧录到emmc的产品，修改如下
    ```xml
        <LIST name="eMMC" desc="Choose eMMC as media">
            <CMD state="BootStrap" type="boot" body="BootStrap" file ="firmware/u-boot.imx" ifdev="MX6ULL">Loading U-boot</CMD>

            <CMD state="BootStrap" type="load" file="firmware/zImage" address="0x80800000"
                loadSection="OTH" setSection="OTH" HasFlashHeader="FALSE" ifdev="MX6SL MX6SX MX7D MX6UL MX6ULL">Loading Kernel.</CMD>

            <CMD state="BootStrap" type="load" file="firmware/%initramfs%" address="0x83800000"
                loadSection="OTH" setSection="OTH" HasFlashHeader="FALSE" ifdev="MX6SL MX6SX MX7D MX6UL MX6ULL">Loading Initramfs.</CMD>

            <CMD state="BootStrap" type="load" file="firmware/imx6ull_emmc_cxn.dtb" address="0x83000000"
                loadSection="OTH" setSection="OTH" HasFlashHeader="FALSE" ifdev="MX6ULL">Loading device tree.</CMD>

            <CMD state="BootStrap" type="jump" > Jumping to OS image. </CMD>


            <!-- create partition -->
            <CMD state="Updater" type="push" body="send" file="mksdcard.sh.tar">Sending partition shell</CMD>
            <CMD state="Updater" type="push" body="$ tar xf $FILE "> Partitioning...</CMD>
            <CMD state="Updater" type="push" body="$ sh mksdcard.sh /dev/mmcblk%mmc%"> Partitioning...</CMD>

            <!-- burn uboot -->
            <CMD state="Updater" type="push" body="$ dd if=/dev/zero of=/dev/mmcblk%mmc% bs=1k seek=768 conv=fsync count=8">clear u-boot arg</CMD>
            <!-- access boot partition -->
            <CMD state="Updater" type="push" body="$ echo 0 > /sys/block/mmcblk%mmc%boot0/force_ro">access boot partition 1</CMD>
            <CMD state="Updater" type="push" body="send" file="files/u-boot.imx" ifdev="MX6ULL">Sending u-boot.bin</CMD>
            <CMD state="Updater" type="push" body="$ dd if=$FILE of=/dev/mmcblk%mmc%boot0 bs=512 seek=2">write U-Boot to sd card</CMD>
            <CMD state="Updater" type="push" body="$ echo 1 > /sys/block/mmcblk%mmc%boot0/force_ro"> re-enable read-only access </CMD>
            <CMD state="Updater" type="push" body="$ mmc bootpart enable 1 1 /dev/mmcblk%mmc%">enable boot partion 1 to boot</CMD>

            <!-- create fat partition -->
            <CMD state="Updater" type="push" body="$ while [ ! -e /dev/mmcblk%mmc%p1 ]; do sleep 1; echo \"waiting...\"; done ">Waiting for the partition ready</CMD>
            <CMD state="Updater" type="push" body="$ mkfs.vfat /dev/mmcblk%mmc%p1">Formatting rootfs partition</CMD>
            <CMD state="Updater" type="push" body="$ mkdir -p /mnt/mmcblk%mmc%p1"/>
            <CMD state="Updater" type="push" body="$ mount -t vfat /dev/mmcblk%mmc%p1 /mnt/mmcblk%mmc%p1"/>

            <!-- burn zImage -->
            <CMD state="Updater" type="push" body="send" file="files/zImage">Sending kernel zImage</CMD>
            <CMD state="Updater" type="push" body="$ cp $FILE /mnt/mmcblk%mmc%p1/zImage">write kernel image to sd card</CMD>

            <!-- burn dtb -->
            <CMD state="Updater" type="push" body="send" file="files/imx6ull_emmc_cxn.dtb" ifdev="MX6ULL">Sending Device Tree file</CMD>

            <CMD state="Updater" type="push" body="$ cp $FILE /mnt/mmcblk%mmc%p1/imx6ull_emmc_cxn.dtb" ifdev="MX6ULL">write device tree to sd card</CMD>

            <CMD state="Updater" type="push" body="$ umount /mnt/mmcblk%mmc%p1">Unmounting vfat partition</CMD>

            <!-- burn rootfs -->
            <CMD state="Updater" type="push" body="$ mkfs.ext3 -F -E nodiscard /dev/mmcblk%mmc%p2">Formatting rootfs partition</CMD>
            <CMD state="Updater" type="push" body="$ mkdir -p /mnt/mmcblk%mmc%p2"/>
            <CMD state="Updater" type="push" body="$ mount -t ext3 /dev/mmcblk%mmc%p2 /mnt/mmcblk%mmc%p2"/>
            <CMD state="Updater" type="push" body="pipe tar -jxv -C /mnt/mmcblk%mmc%p2" file="files/rootfs.tar.bz2" ifdev="MX6UL MX7D MX6ULL">Sending and writting rootfs</CMD>
            <CMD state="Updater" type="push" body="frf">Finishing rootfs write</CMD>
            <CMD state="Updater" type="push" body="$ umount /mnt/mmcblk%mmc%p2">Unmounting rootfs partition</CMD>
            <CMD state="Updater" type="push" body="$ echo Update Complete!">Done</CMD>
        </LIST>
    ```
* L4.1.15_2.0.0-ga_mfg-tools\mfgtools-with-rootfs\mfgtools\xxxx.vbs 运行mfgtool2.exe,并附带一堆参数来运行


## 制作自己的系统，根据 ucl2.xml 文件规定好的文件名
* 自己编译的uboot，生成的   u-boot.imx 
* 自己编译生成内核镜像文件， zImage
* 自己编译生成设备树文件，   imx6ull_emmc_cxn.dtb
* 自己生成文件文件系统，压缩成压缩包   rootfs.tar.bz2 （tar -vcjf rootfs.tar.bz2 *）
* 把上面四件套放到 mfgtools\Profiles\Linux\OS Firmware\files , mfgtools\Profiles\Linux\OS Firmware\firmware两个文件夹下
* 运行 mfgtool2-yocto-mx-evk-emmc.vbs文件

# 生成自己的系统之后
1. ifconfig -a 查看网络设备
2. ifconfig eth0 up 打开eth0
3. 分配ip
    * 板子链接路由器，路由器自动分配ip
        udhcpc -i eth0 
    * 板子链接电脑 或者 路由器 ，可以将该命令添加到/etc/init.d/rcS来自启动
        ifconfig eth0 192.168.18.102     netmask 255.255.255.0
        route add default gw 192.168.18.1       //网关


# 修改MfgTool的脚本配置
* 4件套 uboot.imx ，zImage(内核)，dtb ,rootfs 的准备
* 直接拷贝重命名.vbs文件
* 可通过ucl2.xml来指定自己烧写的文件 ， ucl2.xml控制烧写和升级
    ```
        demo:
            <CMD state="BootStrap" type="boot" body="BootStrap" file ="firmware/u-boot-imx6ul%lite%%6uluboot%_emmc.imx" ifdev="MX6ULL">Loading U-boot</CMD>
            <CMD state="BootStrap" type="boot" body="BootStrap" file ="firmware/u-boot-emmc.imx" ifdev="MX6ULL">Loading U-boot</CMD>    
    ```
* 如果修改了设备树名称，会导致开机不了，要在uboot阶段修改bootcmd来dtb文件。 又或者在 uboot的源代码中，修改 include/configs/mx6ull_alientek_emmc.h 中的 findfdt 变量来修改启动选用的设备树文件名称
    ```
        "findfdt="\
			"if test $fdt_file = undefined; then " \
				"if test $board_name = EVK && test $board_rev = 9X9; then " \
					"setenv fdt_file imx6ull-9x9-evk.dtb; fi; " \
				"if test $board_name = EVK && test $board_rev = 14X14; then " \     //再次修改设备树名称
					"setenv fdt_file imx6ull-14x14-evk.dtb; fi; " \
				"if test $fdt_file = undefined; then " \
					"echo WARNING: Could not determine dtb to use; fi; " \
			"fi;\0" \
            改为：
        "findfdt="\
			"if test $fdt_file = undefined; then " \
				"setenv fdt_file imx6ull_emmc_cxn.dtb; " \//改为自己的设备树文件名称
			"fi;\0" \
    ```


## 卡死在 UTP Waiting for device to appear
* 解决方法 make menuconfig 选中 Device Drivers -> USB support -> USB announce new devicesDevice
* 解决方法 make menuconfig 选中 General setup ---> [*] Initial RAM filesystem and RAM disk (initramfs/initrd) support，生成自己的defconfig
* 我在make menuconfig中找不到这两个参数 USB_MASS_STORAGE ，CONFIG_FSL_UTP在哪配置，只能参考官方的 imx_v7_mfg_deconfig，在该文件里找到上述两个参数配置，拷贝到自己的defconfig文件中，重新编译kernel。就行了



