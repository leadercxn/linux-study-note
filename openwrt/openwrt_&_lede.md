# OpenWrt/Lede

* 硬件组成
    1. RAM
        SDRAM、DDR、DDR2、DDR3
    2. ROM
        SPI Flash、NOR Flash、NAND Flash

* bootloader 和 firmware 都是存放在ROM里面，运行的时候，都在RAM里面

## 佐大学习视频
* 佐大学习视频的lede源码链接
    1. 拉取源码 [LeDe](git clone https://git.openwrt.org/openwrt/openwrt.git lede)
    2. 自己在git_hub上拉取的 [lede源码库](https://github.com/coolsnowwolf/lede)
    3. 安装依赖
    ```
        github的demo给的依赖安装
        sudo apt-get -y install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3 python2.7 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf wget curl swig rsync

        个人使用：
        sudo apt-get -y install build-essential asciidoc binutils bzip2 gawk gettext libncurses5-dev libz-dev patch unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf wget curl swig rsync
    ```
    4. feeds
    ```
        ./scripts/feeds update -a
        ./scripts/feeds install -a
        make menuconfig
    ```
    5. 下载dl库
        make -j8 download V=s
    6. 编译
        make -j1 V=s

* 踩坑
    1. 在执行 `./script/feeds update -a`时会出现 报错 gnutls_handshake() failed: Error in the pull function.
        + 解决方案：[解决链接](https://www.cnblogs.com/zidingliu/p/11496009.html)
        ```
            sudo apt-get update
            sudo apt-get install build-essential fakeroot dpkg-dev libcurl4-openssl-dev
            sudo apt-get build-dep git
            mkdir ~/git-openssl
            cd ~/git-openssl
            sudo apt-get source git
            sudo dpkg-source -x git_1.9.2-1.dsc
            cd git-1.9.2
            #注意上面的1.9.2要按照你自己的版本文件名进行修改
            vi debian/control
            找到libcurl4-gnutls-dev，替换为libcurl4-openssl-dev
            sudo dpkg-buildpackage -rfakeroot -b -us -uc
                -us    Do not sign the source package.
                -uc    Do not sign the .changes file.
            如果fail on test，可以将文件debian/rules中的TEST=test注释掉
            最后sudo dpkg -i ../git_1.9.2-1_amd64.deb安装git即可。


            未满足的构建依赖关系： libcurl4-openssl-dev
            需再次安装  sudo apt-get libcurl4-openssl-dev
            Git bash Unknown SSL protocol error in connection to github.com:443

            git config http.sslVerify "false"
            不行多试几次,就可以了.
        ```
    2. 后面又遇到`RPC failed; curl 56 GnuTLS recv error (-9): A TLS packet with unexpected length was received`
        ```
            sudo apt-get purge git

            sudo apt-get install git
        ```
    3. git clone https相关链接经常失败
        ```
            git config --global http.sslverify "false"
        ```