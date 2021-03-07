# 制作ubuntu
* 源码
    1. 链接[ubuntu16.04](http://cdimage.ubuntu.com/ubuntu-base/releases/16.04.5/release/)
    2. 解压到自建的文件夹里 ubuntu_rootfs

* 依赖
    1. 安装qemu
        + sudo apt-get install qemu-user-static
        + sudo cp /usr/bin/qemu-arm-static  ~/linux/nfs/ubuntu_rootfs/usr/bin/
    2. 设置软件源
        + sudo cp /etc/resolv.conf ./etc/resolv.conf    
        + vim  ./etc/apt/sources.list
            ```
                deb http://mirrors.ustc.edu.cn/ubuntu-ports/ xenial main multiverse restricted universe
                deb http://mirrors.ustc.edu.cn/ubuntu-ports/ xenial-backports main multiverse restricted universe
                deb http://mirrors.ustc.edu.cn/ubuntu-ports/ xenial-proposed main multiverse restricted universe
                deb http://mirrors.ustc.edu.cn/ubuntu-ports/ xenial-security main multiverse restricted universe
                deb http://mirrors.ustc.edu.cn/ubuntu-ports/ xenial-updates main multiverse restricted universe
                deb-src http://mirrors.ustc.edu.cn/ubuntu-ports/ xenial main multiverse restricted universe
                deb-src http://mirrors.ustc.edu.cn/ubuntu-ports/ xenial-backports main multiverse restricted universe
                deb-src http://mirrors.ustc.edu.cn/ubuntu-ports/ xenial-proposed main multiverse restricted universe
                deb-src http://mirrors.ustc.edu.cn/ubuntu-ports/ xenial-security main multiverse restricted universe
                deb-src http://mirrors.ustc.edu.cn/ubuntu-ports/ xenial-updates main multiverse restricted universe
            ```
    
* 完善unbuntu文件系统
    1. 编写挂载 && 卸载脚本 && 运行挂载脚本 ./mount.sh
        挂载脚本
        ```
            #!/bin/bash

            echo "MOUNTING"
            sudo mount -t proc  /proc       /home/cxn/linux/nfs/ubuntu_rootfs/proc
            sudo mount -t sysfs /sys        /home/cxn/linux/nfs/ubuntu_rootfs/sys
            sudo mount -o bind  /dev        /home/cxn/linux/nfs/ubuntu_rootfs/dev
            sudo mount -o bind  /dev/pts    /home/cxn/linux/nfs/ubuntu_rootfs/dev/pts
            sudo chroot /home/cxn/linux/nfs/ubuntu_rootfs
        ```

        卸载脚本
        ```
            #!/bin/bash

            echo "UNMOUNTING"
            sudo umount     /home/cxn/linux/nfs/ubuntu_rootfs/proc
            sudo umount     /home/cxn/linux/nfs/ubuntu_rootfs/sys
            sudo umount     /home/cxn/linux/nfs/ubuntu_rootfs/dev
            sudo umount     /home/cxn/linux/nfs/ubuntu_rootfs/dev/pts
        ```
    2. 安装常用的命令和软件
        1. 踩过一些坑
            坑：GPG error:Could not execute 'apt-key' to verify signature (is gnupg installed?)
            解决方案：
                chmod 777 /tmp
                apt-get update --allow-unauthenticated
        2. 安装工具
        ```
            apt update
            apt install sudo
            apt install vim
            apt install kmod
            apt install net-tools
            apt install ethtool
            apt install ifupdown
            apt install language-pack-en-base
            apt install rsyslog
            apt install htop

            apt install iputils-ping
        ```
    3. 设置root密码
        ```
            passwd root //设置 root 用户密码
        ```
        + 个人使用了“123456”作为密码
    4. 设置本机名称和IP地址
        ```
            echo "cxn">/etc/hostname
            echo "127.0.0.1 localhost">>/etc/hosts
            echo "127.0.0.1 cxn">>/etc/hosts
        ```
    5. 设置串口终端
        ```
            ln -s /lib/systemd/system/getty@.service /etc/systemd/system/getty.target.wants/getty@ttymxc0.service
        ```
    6. 退出
        exit
        ./unmount.sh
    7. 烧录4件套 内核，设备树，uboot，文件系统（ubuntu）到开发版上

* 板上ubuntu系统的完善(以下操作可以在开发版上完成，也建议在ubuntu mount挂载之后来完善文件系统，再烧录到开发版)
    1. 新增普通用户
        制作的 ubuntu 根文件系统默认只有一个 root 用户，默认都是用 root 用户登录的，使用 root 用户的话可能会因为操作失误导致系统被破坏。因此最好创建一个普通用户，默认使用普通用户。
        ```
            adduser cxn
        ```
        密码采用惯用的：123456
    2. 重启开发版，登录的时候输入用户：cxn，启动开发版后就是处于普通用户
    3. 让普通用户账户支持 sudo 命令
        ```
            su //切换回 root 用户

            chmod u+w /etc/sudoers      #让sudoers文件可读可写

            vim /etc/sudoers
                新增一行： cxn ALL=(ALL:ALL) ALL

            chmod u-w /etc/sudoers
        ```
    4. 网络DHCP配置
        ```
            su //切换到 root 用户
            echo auto eth0 > /etc/network/interfaces.d/eth0
            echo iface eth0 inet dhcp >> /etc/network/interfaces.d/eth0
            /etc/init.d/networking restart
        ```
    5. 搭建 FTP 服务器
        1. 参照 ubuntu 下搭建 FTP服务器的方式
        2. 安装包安装（比较麻烦）
            ```
                cd vsftpd-3.0.3 //进入到 vsftpd 源码目录

                # 修改Makefile
                CC = arm-linux-gnueabihf-gcc //CC 为交叉编译器

                make


                sudo cp vsftpd      /home/cxn/linux/nfs/ubuntu_rootfs/usr/sbin/    //拷贝 vsftpd
                sudo cp vsftpd.conf /home/cxn/linux/nfs/ubuntu_rootfs/etc/         //拷贝 vsftpd.conf

                # 设置权限
                chmod +x /usr/sbin/vsftpd
                chown root:root /etc/vsftpd.conf

                # 修改配置
                sudo vi /etc/vsftpd.conf

                    local_enable=YES
                    write_enable=YES

                # 添加新用户
                sudo cp /etc/passwd /home/cxn/linux/nfs/ubuntu_rootfs/etc/ -f
                sudo cp /etc/group  /home/cxn/linux/nfs/ubuntu_rootfs/etc/ -f
                    再修改：    group文件只保留“root”这一项。 passwd 文件“root”行最后改为“/bin/sh”，

                # 创建文件（有的话，不用理，没的话，自行创建）
                    mkdir /home
                    mkdir /usr/share/empty -p
                    mkdir /var/log -p
                    touch /var/log/vsftpd.log

                adduser ftp //创建 ftp 用户
                adduser nobody //创建 nobody 用户
                adduser cxn //创建登录要用户

                # 启动
                vsftpd & //启动 vsftpd
            ```
    6. 安装 make , gcc
    ```
        sudo apt-get install make
        sudo apt-get install gcc
    ```

* OpenSSH 移植与使用
    * 客户端电脑通过ssh登录 服务器，客户端命令
        ```
            ssh root@服务器IP地址

            输入服务器密码
        ```

    * 移植相关库
        1. 安装 zlib 库
            ```
                cd zlib-1.2.11/ //进去 zlib 源码
                # 配置
                CC=arm-linux-gnueabihf-gcc LD=arm-linux-gnueabihf-ld AD=arm-linux-gnueabihfas ./configure --prefix=/home/cxn/tool/third_lib/zlib 

                # 编译
                make

                # 安装
                make install

                # 拷贝库到文件系统中
                sudo  cp  ~/tool/third_lib/zlib/lib/*  ～/linux/nfs/ubuntu_rootfs/lib/  -rfa
            ```
        2. 移植 openssl 库
            ```
                # 进入安装路径
                cd  openssl-1.1.1d/

                # 配置
                ./Configure linux-armv4 shared no-asm --prefix=/home/cxn/tool/third_lib/openssl  CROSS_COMPILE=arm-linux-gnueabihf-

                # 编译
                make

                # 安装
                make install

                # 拷贝到文件系统中
                sudo cp lib/libcrypto.so* /home/zuozhongkai/linux/nfs/ubuntu_rootfs/lib/ -af
                sudo cp lib/libssl.so* /home/zuozhongkai/linux/nfs/ubuntu_rootfs/lib/ -af

            ```
        3. 移植 openssh 库
            ```
                # 进入目录
                cd openssh-8.2p1/

                # 配置
                ./configure --host=arm-linux-gnueabihf  --with-libs  --with-zlib=/home/cxn/tool/third_lib/zlib  --with-ssl-dir=/home/cxn/tool/third_lib/openssl  --disable-etcdefault-login CC=arm-linux-gnueabihf-gcc AR=arm-linux-gnueabihf-ar

                # 编译
                make

            ```


        * openssh 设置[ 个人使用这种方法，出现错误：sshd: no hostkeys available — exiting ]
            1. 添加密钥
                ```
                    ssh-keygen -t rsa -f ssh_host_rsa_key -N ""
                    ssh-keygen -t dsa -f ssh_host_dsa_key -N ""
                    ssh-keygen -t ecdsa -f ssh_host_ecdsa_key -N ""
                    ssh-keygen -t ed25519 -f ssh_host_ed25519_key -N ""
                ```
            2. 启动ssh服务
                ```
                    /sbin/sshd //启动 sshd 服务

                    也可以在/etc/init.d/rcS 文件中加入如下命令，实现 ssh 服务开机自启动：
                    /sbin/sshd &
                ```

        * 个人的处理方式是在开发版上
            ```
                sudo apt-get install openssh-client
                sudo apt-get install openssh-server

                sudo /usr/sbin/service ssh start
            ```