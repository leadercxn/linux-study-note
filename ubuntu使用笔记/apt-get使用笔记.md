# apt-get 下载后，软件所在路径是
* 安装命令: `apt-get install`
    * /var/cache/apt/archives
    * 视安装的文件类型而定，系统安装软件一般在/usr/share，可执行的文件在/usr/bin，配置文件可能安装到了/etc下等。
        1. 文档一般在 /usr/share
        2. 可执行文件 /usr/bin
        3. 配置文件 /etc
        4. lib文件 /usr/lib
* 卸载命令: `apt-get remove`


# ubuntu中软件安装的方法
1. APT方式
    * 普通安装：apt-get install softname1 softname2 …;
    * 修复安装：apt-get -f install softname1 softname2... ;(-f Atemp to correct broken dependencies)
    * 重新安装：apt-get --reinstall install softname1 softname2...;
2. Dpkg方式
    * 普通安装：dpkg -i package_name.deb
        ```demo
            sudo dpkg -i 文件名.deb
        ```
3. 源码安装（.tar、tar.gz、tar.bz2、tar.Z）
    首先解压缩源码压缩包然后通过tar命令来完成
    a. 解xx.tar.gz：tar zxf xx.tar.gz 
    b. 解xx.tar.Z：tar zxf xx.tar.Z 
    c. 解xx.tgz：tar zxf xx.tgz 
    d. 解xx.bz2：bunzip2 xx.bz2 
    e. 解xx.tar：tar -vxf xx.tar
    然后进入到解压出的目录中，建议先读一下README之类的说明文件，因为此时不同源代码包或者预编译包可能存在差异，然后建议使用ls -F --color或者ls -F命令（实际上我的只需要 l 命令即可）查看一下可执行文件，可执行文件会以*号的尾部标志。
    一般依次执行
            ./configure
            make
            sudo make install
    即可完成安装。

4. 安装了python的
    1. 通过 pip install 命令来安装
        * 可以通过 pip install xxxx==1.9.0 的命令来安装指定的版本号
            ```
                demo:
                    pip install nrfutil==0.5.3
            ```

# 其他应用总结
    apt-cache search # ------(package 搜索包)
    apt-cache show #------(package 获取包的相关信息，如说明、大小、版本等)
    apt-get install # ------(package 安装包)
    apt-get install # -----(package --reinstall 重新安装包)
    apt-get -f install # -----(强制安装, "-f = --fix-missing"当是修复安装吧...)
    apt-get remove #-----(package 删除包)
    apt-get remove --purge # ------(package 删除包，包括删除配置文件等)
    apt-get autoremove --purge # ----(package 删除包及其依赖的软件包+配置文件等（只对6.10有效，强烈推荐）)
    apt-get update #------更新源
    apt-get upgrade #------更新已安装的包
    apt-get dist-upgrade # ---------升级系统
    apt-get dselect-upgrade #------使用 dselect 升级
    apt-cache depends #-------(package 了解使用依赖)
    apt-cache rdepends # ------(package 了解某个具体的依赖,当是查看该包被哪些包依赖吧...)
    apt-get build-dep # ------(package 安装相关的编译环境)
    apt-get source #------(package 下载该包的源代码)
    apt-get clean && apt-get autoclean # --------清理下载文件的存档 && 只清理过时的包
    apt-get check #-------检查是否有损坏的依赖
    dpkg -S filename -----查找filename属于哪个软件包
    apt-file search filename -----查找filename属于哪个软件包
    apt-file list packagename -----列出软件包的内容
    apt-file update --更新apt-file的数据库

    dpkg --info "软件包名" --列出软件包解包后的包名称.
    dpkg -l --列出当前系统中所有的包.可以和参数less一起使用在分屏查看. (类似于rpm -qa)
    dpkg -l |grep -i "软件包名" --查看系统中与"软件包名"相关联的包.
    dpkg -s 查询已安装的包的详细信息.
    dpkg -L 查询系统中已安装的软件包所安装的位置. (类似于rpm -ql)
    dpkg -S 查询系统中某个文件属于哪个软件包. (类似于rpm -qf)
    dpkg -I 查询deb包的详细信息,在一个软件包下载到本地之后看看用不用安装(看一下呗).
    dpkg -i 手动安装软件包(这个命令并不能解决软件包之前的依赖性问题),如果在安装某一个软件包的时候遇到了软件依赖的问题,可以用apt-get -f install在解决信赖性这个问题.
    dpkg -r 卸载软件包.不是完全的卸载,它的配置文件还存在.
    dpkg -P 全部卸载(但是还是不能解决软件包的依赖性的问题)
    dpkg -reconfigure 重新配置







# 遇到的问题 && 解决方法
1. apt_get install
    * 问题:
    ```
        E: Could not get lock /var/lib/dpkg/lock-frontend - open (11: Resource temporarly unavailable)
        E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), is an other process using it?
    ```
    * 解决方法:
        删除锁定文件，行了
        sudo rm /var/lib/dpkg/lock-frontend       
        sudo rm /var/lib/dpkg/lock
2. 安装openocd
    * 问题,编译openocd的时候的源代码出现错误
    ```
        error: this statement may fall through [-Werror=implicit-fallthrough=]
    ```
    * 解决方法
        make CFLAGS='-Wno-implicit-fallthrough'




# 实际应用
1. 安装Jlink驱动
    1. 安装libusb库
    2. 安装readline工具
    3. 安装libstdc++6
    4. 安装lib32stdc++6

2. 安装openocd 
3. 安装cmake
4. 安装ninja的依赖--re2c,再安装ninja
    * 解压.rpm文件 ==>  demo:   rpm2cpio re2c-1.1.1-3.fc31.src.rpm | cpio -idmv
    * 安装.pkg文件 ==>  demo    sudo installer -pkg xxx.pkg  -target  /     (注: 后面的只能是 / ，假如其他路径的话，会出错长度)
5. 安装arm-none-eabi-gcc编译器,编译 ARM 架构的裸机系统（包括 ARM Linux 的 boot、kernel，不适用编译 Linux 应用 Application）
6. 安装ARMCC ,DS-5
    1. 安装依赖库 







