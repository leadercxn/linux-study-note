


# 介绍
* 根文件系统首先是内核启动时所 mount(挂载)的第一个文件系统，内核代码映像文件保存在根文件系统中，而系统引导启动程序会在根文件系统挂载之后从中把一些基本的初始化脚本(例如：rcS ,inittab )和服务等加载到内存中去运行。 根文件系统和 Linux 内核是分开的，单独的 Linux 内核是没法正常工作的，必须要搭配根文件系统。如果不提供根文件系统， Linux 内核在启动的时候就会提示内核崩溃(Kernel panic)的提示，

# 构建根文件系统
1. 修改顶层的Makefile文件， 添加编译器和编译的架构
2. 修改 libbb/printable_string.c 和 /libbb/unicode.c 让其支持中文
3. make menuconfig
4. make install CONFIG_PREFIX=/home/cxn/linux/nfs/rootfs  //安装文件系统，方便网络挂载文件系统
5. 添加lib库文件，向在ubuntu安装好的rtoofs文件夹里，新建文件夹 lib，从编译器的libc/lib中获取
    ```
        cd /usr/local/arm/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/arm-linux-gnueabihf/libc/lib  //添加动态库
        cp *so* *.a /home/cxn/linux/rootfs/lib/ -d
        cd /usr/local/arm/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/arm-linux-gnueabihf/lib       //添加静态库
        cp *so* *.a /home/cxn/linux/rootfs/lib/ -d
    ```
6. 特俗的操作，把库文件中的 ld-linux-armhf.so.3
7. 在usr文件夹里新增lib文件夹，并把编译器的动态文件库拷贝到新增的lib文件夹里
    ```
        cd /usr/local/arm/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/arm-linux-gnueabihf/libc/usr/lib
        cp *so* *.a /home/cxn/linux/rootfs/usr/lib -d
    ```
8. 创建其他文件夹dev,proc,mnt,sys,root,tmp
9. 重启开发板，在uboot的模式下，根据 Documentation/filesystems/nfs/nfsroot.txt  里面对环境变量root的格式说明，重新设置环境变量
``` 千万别忘记 rw ，不然挂载的文件系统只读
    setenv bootargs 'console=ttymxc0,115200  root=/dev/nfs rw nfsroot=192.168.18.101:/home/cxn/linux/nfs/rootfs ip=192.168.18.103:192.168.18.101:192.168.18.1:255.255.255.0::eth0:off'
    saveenv
```
10. 完善文件系统（缺少rcS文件），在 rootfs 中创建/etc/init.d/rcS 的脚本文件，   rcS作用：规定启动哪些文件，即一些程序的自启
```
#!/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin:$PATH
LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/lib:/usr/lib
export PATH LD_LIBRARY_PATH
mount -a
mkdir /dev/pts
mount -t devpts devpts /dev/pts
echo /sbin/mdev > /proc/sys/kernel/hotplug
mdev -s

#开机自起启                                 #！！！！！！！！！！！！！！！！可在rsC文件添加响应的命令，来让一些程序开机后自启
cd /drivers
./test &
cd /


```
11. 创建/etc/fstab 文件，      fstab作用：配置哪些需要自动挂载的分区
```
#<file system> <mount point> <type> <options> <dump> <pass>
proc /proc proc defaults 0 0
tmpfs /tmp tmpfs defaults 0 0
sysfs /sys sysfs defaults 0 0
```
12. 创建/etc/inittab 文件     inittab作用：一些命令
```
#etc/inittab
::sysinit:/etc/init.d/rcS
console::askfirst:-/bin/sh
::restart:/sbin/init
::ctrlaltdel:/sbin/reboot
::shutdown:/bin/umount -a -r
::shutdown:/sbin/swapoff -a
```

13. 创建/etc/resolv.conf ， 完善网络服务器域名设置
```
nameserver 114.114.114.114
nameserver 192.168.18.1
```











