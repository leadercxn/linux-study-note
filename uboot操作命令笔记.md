# uboot 进入命令模式
+ 在系统(uboot)启动时，有倒计时，此时在终端随便按键盘，让板子进入uboot的命令模式
+ pri 命令查看给中环境变量
+ 通过uboot的日志打印，可以看到开发板的相关信息

# uboot中非常重要的两个环境变量
+ bootcmd 和 bootargs

# setenv 设置环境变量的值   &&  saveenv 保存环境变量
    ```
    demo：
        setenv bootdelay 5
        saveenv
    ```
+ 修改环境变量带有空格 ,用单引号''
    ```
        setenv bootcmd 'console=ttymxc0,115200 root=/dev/mmcblk1p2 rootwait rw '
        seveenv
    ```
+ 新建环境变量 
    ```
    格式： setenv 变量名  变量值
    demo： setenv author cxn    
    ```
+ 删除环境变量 
    ```
    格式： setenv 变量名   //值为空
    demo： setenv author     
    ```


# 内存的操作
+ 查I.MX6U嵌入式Linux驱动开发指南V1.0.pdf 的 30.4.3
+ 常用 md.b  addr  cnt  ,从内存addr地址处读取cnt个数据 ，addr 和 cnt 都是十六进制数据

# 网络操作
+ 设置开发板网络参数
    ```
        setenv ipaddr 192.168.18.102            `设置开发板ip`
        setenv ethaddr 00:04:9f:04:d2:35       `设置开发板MAC地址`
        setenv gatewayip 192.168.18.1            `设置网关`
        setenv netmask 255.255.255.0              `设置掩码`
        setenv serverip 192.168.18.101         `设置服务器ip,同一路由下的其他PC或者虚拟机ubuntu`
        saveenv
        ping 192.168.18.101
    ```

# FTP 服务器
+ 操作
    1. 下载安装
        ```
            sudo apt-get install vsftpd
        ```
    2. 编辑配置文件
        ```
            sudo vi /etc/vsftpd.conf

                local_enable=YES
                write_enable=YES
        ```
        
    3. sudo /etc/init.d/vsftpd restart

# nfs && SSH 搭建
+ ubuntu安装nfs服务，共享资源
    ```
    sudo apt-get install nfs-kernel-server portmap
    vim /etc/exports 
        添加一行：
            /home/cxn/linux/nfs *(rw,sync,no_root_squash)       //  /home/cxn/linux/nfs 该路径是用户自己新建的文件夹,该命令的作用就是赋予某个文件夹有nfs功能
    sudo /etc/init.d/nfs-kernel-server restart                  //  重启NFS服务
    ```
+ ubuntu安装SSH服务
    ```
        sudo apt-get install openssh-server                     // 配置文件为 /etc/ssh/sshd_config
        sudo apt-get install openssh-client
        sudo service ssh start
    ```

+ 开发板执行命令
    1.开发板ping通ubuntu
    2.nfs 80800000 192.168.18.101:/home/cxn/linux/nfs/zImage   //该命令让开发板通过局域网下载服务器上的镜像文件到DRAM

# tftp
+ ubuntu虚拟机
    1. 安装包 tftp-hpa、 tftpd-hpa
        ```
            sudo apt-get install tftp-hpa tftpd-hpa
        ```
    2. 新建一个tftp共享文件夹,且需要给该文件夹权限
        ```
            /home/cxn/linux/tftpboot
            chmod 777  /home/cxn/linux/tftpboot
        ```
    3. 新建路径 和 文件 , 并写入配置
        ```
            vim /etc/xinetd.d/tftp

            server tftp
            {
                socket_type = dgram
                protocol = udp
                wait = yes
                user = root
                server = /usr/sbin/in.tftpd
                server_args = -s /home/cxn/linux/tftpboot/
                disable = no
                per_source = 11
                cps = 100 2
                flags = IPv4
            }
        ```
    4. 启动tftp 
        ```
            sudo service tftpd-hpa start
        ```
    5. 修改配置文件  /etc/default/tftpd-hpa
    ```
        # /etc/default/tftpd-hpa
        
        TFTP_USERNAME="tftp"
        TFTP_DIRECTORY="/home/cxn/linux/tftpboot"
        TFTP_ADDRESS=":69"
        TFTP_OPTIONS="-l -c -s"
    ```
    6. 重启tftp [这样，ubuntu通过tftp，和开发板相连]
    ```
        sudo service tftpd-hpa restart
    ```
    7. 把镜像拷贝到tftpboot文件夹中
    ```
        cp ../nfs/zImage  .
        chmod 777 zImage
    ```
    8. 开发板上命令，把镜像文件zImage下载到开发板DRAM上
    ```
        tftp 80800000 zImage
    ```

# EMMC && SD
+ 具体参考 《I.MX6U嵌入式linux驱动开发指南V1.3》30.4.5小节

+ mmc list 命令 , 查看当前开发板一共有几个 MMC 设备
    ```
        FSL_SDHC:0 (SD)
        FSL_SDHC:1 
    ```
+ mmc dev 命令
    ```
        mmc dev 0   //切换到SD
        mmc dev 1 0  //切换到EMMC，分区0
    ```
+ `mmc read `命令 , 用于读取 mmc 设备的数据
    + 格式：
        ```
            mmc read addr blk# cnt        //从扇区blk#连续读取cnt个块的数据，到DRAM的addr地址处 ,一个块是512字节

            demo:
                mmc dev 1 0               //切换到 MMC 分区 0
                mmc read 80800000 600 10  //读取数据
                md.b 80800000 2000        //查看数据
        ```
+ `mmc write` 命令 , 将数据写到 MMC 设备里面
    + 格式:
    ```
        mmc write addr blk# cnt             //从DRAM的addr地址处，把数据写到MMC的起始扇区blk#，cnt个块

        demo:
            mmc dev 0  //切换到 SD 卡
            version    //查看SD中的ubuntu版本号
            // tftp 80800000 u-boot.imx or nfs 80800000 192.168.18.101:/home/cxn/linux/nfs/zImage //下载新的uboot到SD卡的80800000地址处，算出uboot.bin的大小，除以512，算出占用几个块
            mmc dev 0 0
            mmc write 80800000 2 32E
    ```
    + 过程理解
        DRAM  -->  MMC 中    ，因为MMC（SD卡）中存在uboot，通过`mmc arite`,把下载到DRAM的uboot.bin的数据，写到MMC中，这样，下次启动板子，板子启动从MMC中读取的uboot就是覆盖好了的数据
+ `mmc erase`命令
    + 格式：
    ```
        mmc erase blk# cnt
    ```



# FAT 文件系统（并非所有的文件系统）
+ 具体参考 《I.MX6U嵌入式linux驱动开发指南V1.3》30.4.6小节

# EXT 文件系统（并非所有的文件系统）
+ 具体参考 《I.MX6U嵌入式linux驱动开发指南V1.3》30.4.7小节

# NAND的操作
+ nand erase 用于擦除NAND Flash  (有点类似于flash的操作)
    + 格式：
    ```
        nand erase[.spread] [clean] off size    //从指定地址开始(off)开始，擦除指定大小(size)的区域。
        nand erase.part [clean] partition       //擦除指定的分区
        nand erase.chip [clean]                 //全篇擦除
    ```
+ nand write 命令 , 用于向 NAND 指定地址写入指定的数据，一般和“nand erase”命令配置使用来更新NAND 中的 uboot、 linux kernel 或设备树等文件
    + 格式：
    ```
        nand write addr off size        addr 是要写入的数据首地址[DRAM]， off 是 NAND 中的目的地址， size 是要写入的数据大小
    ```
    + 使用环境：
        1. 在 uboot 里面使用“nand write”命令烧写 kernel 和 dtb
        2. 开发板的分区
        ```
            0x000000000000-0x000004000000 : "boot"
            0x000004000000-0x000006000000 : "kernel"
            0x000006000000-0x000007000000 : "dtb"
            0x000007000000-0x000020000000 : "rootfs"
        ```
+ nand read 命令,用于从 NAND 中的指定地址读取指定大小的数据到 DRAM 中
    + 格式：
    ```
        nand read addr off size         addr 是目的地址[DRAM]， off 是要读取的 NAND 中的数据源地址， size 是要读取的数据大小
    ```

# BOOT操作命令 ，对uboot启动的一些操作
+ `bootz` 用于启动 `zImage` 镜像文件
    + 格式:
    ```
        bootz [addr [initrd[:size]] [fdt]]          // addr 是 Linux 镜像文件在 DRAM 中的位置， initrd 是 initrd 文件在DRAM 中的地址，如果不使用 initrd 的话使用‘-’代替即可， fdt 就是设备树文件在 DRAM 中的地址
    ```
    + demo ，通过网络启动linux，其实就是通过网络拉去镜像文件和设备树文件到开发板的DRAM上，然后通过bootz启动
    ```
        tftp 80800000 zImage
        tftp 83000000 imx6ull-14x14-nand-4.3-800x480-c.dtb          //xxx.dtb文件拷到ubuntu的tftp文件夹中，通过tftp放到DRAM中 
        bootz 80800000 – 83000000 
    ```

+ `bootm`  用于启动 `uImage` 镜像文件
    + 格式：
    ```
        bootm addr                                  // addr 是 uImage 镜像在 DRAM 中的首地址。

        bootm [addr [initrd[:size]] [fdt]]          //  其中 addr 是 uImage 在 DRAM 中的首地址， initrd 是 initrd 的地址， fdt 是设备树(.dtb)文件在 DRAM 中的首地址，如果 initrd 为空的话，同样是用“-”来替代。
    ```
    + demo ,参考上例。

+ `boot` 也是用来启动 Linux 系统的，只是 boot 会读取环境变量 bootcmd 来启动 Linux 系统， `bootcmd` 是一个很重要的环境变量
    + demo 
    ```
        setenv bootcmd 'tftp 80800000 zImage;  tftp 83000000 imx6ull-14x14-nand-4.3-800x480-c.dtb;  bootz 80800000 - 83000000' //先设置环境变量bootcmd
        saveenv             // 保存环境变量
        boot                // 运行boot
    ```
    + 理解： `bootcmd`相当于脚本，运行镜像时，带着脚本运行 
    + 加深理解 `bootcmd` , uboot 倒计时结束以后就会启动 Linux 系统，其实就是执行的 bootcmd 中的启动命令。只要不修改 bootcmd 中的内容，以后每次开机 uboot 倒计时结束以后都会使用 tftp 命令从网络下载 zImage 和 imx6ull-alientek-emmc.dtb，然后启动 Linux。`setenv bootcmd`时，记得带单引号``

+ 现象，按照以上操作之后，串口打印出现 “Kernel panic – not Syncing: VFS: Unable to mount root fs on unknown-block(0,0)” ，那是因为uboot没有设置bootargs环境变量。

# 其他命令
+ `go` 跳到指定的DRAM地址处执行应用
    + 格式
    ```
        go addr [arg ...]               // addr 是应用在 DRAM 中的首地址
    ```
    + demo
    ```
        tftp 87800000 printf.bin       // printf.bin 在ubuntu中，利用编译器 arm-linux-gnueabihf- ，执行make ，生成的
        go 87800000
    ```
+ `run` 运行环境变量中定义的命令,一般用来运行我们自定义的环境变量。
    + demo
    ```
        setenv mybootemmc ' xxxxxx '
        saveenv
        run mybootemmc
    ```











