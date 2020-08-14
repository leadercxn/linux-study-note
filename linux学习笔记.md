# 以 管理员身份 运行VMware
+ 方便修改 虚拟网络编辑器  
+ 主机能无线上网（关闭防火墙，禁用其他网卡），ubuntu能和主机互ping，主机ipv4的DHCP设为自动IP，cmd命令ipconfig查看主机的ip地址和网段，例如主机ip：192.168.18.100  掩码：255.255.255.0 ，网关192.168.18.1 ，虚拟机网络IPv4设为手动，ip地址设为192.168.18.101，掩码：255.255.255.0  网关192.168.18.1  ，其他虚拟网络编辑器 VMnet0设为桥接模式，网卡选主机的无线网卡，然后就可以在虚拟机上上网。不知为何，`ps:用主机接手机的热点，这方法不可用`

# sd卡问题
+ 重装VMware_tool
    ```
       cd VM_Tool/vmware-tools-distrib/
        sudo ./vmware-install,pl
    ```
+ 用读卡器插上，关闭ubuntu，虚拟机设置、USB控制器，选择USB2.0，再开启ubuntu
+ 选择 虚拟机、可移动设备、Realtek usb2.0
+ ls /dev/sd   就会看到挂载的sdb 读卡器

# 我的代码管理
+ 路径： /mnt/hgjs/vmware_share/code/register_code/1_leds
+ 烧录工具： imxdownload
+ 烧录命令: sudo ./imxdownload  xx.bin /dev/sdb

# 编译器 
+ 路径：/usr/local/arm/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf
+ 添加到环境变量： 
    ```
    sudo vim /etc/profile
    export PATH=$PATH:/usr/local/arm/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/bin
    sudo apt-get install lsb-core lib32stdc++6
    重启ubuntu

    ```
+ 验证： `在一般用户下`$ arm-linux-gnueabihf-gcc -v



# uboot
+ 路径: /home/cxn/linux/uboot
+ 编译: ` 512MB + 8G EMMC`  , 因为我的开发板是EMMC版本
    ```
        make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- distclean                              //消除工程
        make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- mx6ull_14x14_ddr512_emmc_defconfig     //配置uboot
        make V=1 ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j12                               //编译
    ```
+ uboot目录下的 compile.sh 就是把前面3个命令合在一起的脚本
+ uboot目录下的 imxdownload 是正点原子转为I.MX6U做的 把程序烧录到sd卡上的工具
    ```
    ./imxdownload  u-boot.bin  /dev/sdb
    ```
+ 烧录完成，插上SD卡，sd卡形式启动，打开串口看输出
    ```
    串口配置：115200   流量检查： XON/XOFF
    ```

# 进程的操作
* ps 可以查看当前后台的进程
* ./xxx & 的格式让进程在后台进行
* kill -9 pid 可以结束进程



# nfs文件系统的挂载
* 卡死在
    ```
        gpio_dvfs: disabling
        can-3v3: disabling
        ALSA device list:
        #0: wm8960-audio
    ```
    代码里
    * 原因：挂载不了文件系统
    * 解决方法： 修改ubuntu下的 /etc/exports 文件, 添加如下代码，打开nfs功能，同时格式要规范 '*'号前面的空格符不能省
    ```
        /home/cxn/linux/nfs *(rw,sync,no_root_squash)
    ```
* 挂载好的文件系统只读
    * 原因： 设置uboot的环境变量 bootargs 没有添加rw
    * 解决方法：
    ```
    setenv bootargs 'console=ttymxc0,115200  root=/dev/nfs rw nfsroot=192.168.18.101:/home/cxn/linux/nfs/rootfs ip=192.168.18.103:192.168.18.101:192.168.18.1:255.255.255.0::eth0:off'
    saveenv
    ```


# 内核代码加入自己创建代码目录
1. 在内核顶层目录的Makefile，参考net-y的方式，新建一个 dev-y指向自己代码文件夹（demo:my_dev）
2. 自己的代码文件夹（my_dev）下的Makefile 参考samples目录下的子Makefile
3. 要添加的源代码文件夹为common，然后common目录下的makefile参考sample子文件夹的makefile文件
4. 在自己的源代码中，添加符号导出 ，EXPORT_SYMBOL 方可在其他.c文件引用相关的函数
    + kernel_dir
        + arch
        .
        .
        + sample
        + my_dev
            * makefile
            + common
                * makefile
                * .c .h
        * Makefile


