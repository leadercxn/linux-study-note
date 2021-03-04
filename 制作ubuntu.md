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
