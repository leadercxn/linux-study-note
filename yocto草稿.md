repo init -u https://github.com/STMicroelectronics/oe-manifest.git -b refs/tags/openstlinux-5.10-dunfell-mp1-21-03-31

DISTRO=fsl-imx-x11 MACHINE=imx6ull14x14evk source fsl-setup-release.sh -b build

DISTRO=openstlinux-weston MACHINE=stm32mp1 source layers/meta-st/scripts/envsetup.sh

bitbake st-image-weston

# yacto

## yocto 构建工程的一般步骤 在ubuntu环境下
1. 安装必要的依赖
    ```shell
        sudo apt-get update
        sudo apt-get upgrade
        sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib build-essential chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev pylint3 pylint xterm
        sudo apt-get install make xsltproc docbook-utils fop dblatex xmlto
        sudo apt-get install libmpc-dev libgmp-dev
        sudo apt-get install libncurses5 libncurses5-dev libncursesw5-dev libssl-dev linux-headers-generic u-boot-tools device-tree-compiler bison flex g++ libyaml-dev libmpc-dev libgmp-dev
        sudo apt-get install coreutils bsdmainutils sed curl bc lrzsz corkscrew cvs subversion mercurial nfs-common nfs-kernel-server libarchive-zip-perl dos2unix texi2html diffstat libxml2-utils

        git config --global user.name "Your Name"
        git config --global user.email "you@example.com"
        git config --list
    ```
    or
    ```shell
        sudo apt-get install gawk wget git-core
        sudo apt-get install diffstat unzip texinfo gcc-multilib
        sudo apt-get install build-essential chrpath socat libsdl1.2-dev
        sudo apt-get install libsdl1.2-dev
        sudo apt-get install xterm sed cvs
        sudo apt-get install subversion coreutils texi2html
        sudo apt-get install docbook-utils python-pysqlite2
        sudo apt-get install help2man make gcc g++
        sudo apt-get install desktop-file-utils
        sudo apt-get install libgl1-mesa-dev libglu1-mesa-dev mercurial
        sudo apt-get install autoconf automake groff
        sudo apt-get install curl lzop asciidoc
        sudo apt-get install u-boot-tools
    ```
2. 安装 repo
    ```
        curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o repo
        chmod +x repo

        vim repo
    ```
        搜索出 “google” 关键字，把对应行改为链接
        ```shell
            REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
        ```
        后修改 .bashrc 文件，把repo路径添加到环境变量中

3. 创建一个本地 yocto 工程目录
    ``` 
        mkdir yocto_project && cd yocto_project
    ```
4. repo init 初始化仓库
    ```shell
        repo init  -u  [github-url]
            example1: repo init -u https://github.com/STMicroelectronics/oe-manifest.git -b refs/tags/openstlinux-5.10-dunfell-mp1-21-03-31
            example2: repo init -u git://git.freescale.com/imx/fsl-arm-yocto-bsp.git -b imx-4.1-krogoth
    ```
5. repo 仓库同步
    ```shell
        repo sync
    ```

* 备注：以上两步对网络要求都比较高，因为有些仓库在国外，所以需要翻墙

6. 设置环境变量
    ```
        DISTRO=<distro name> MACHINE=<machine name> source fsl-setup-release.sh -b <build dir>
            example1:DISTRO=fsl-imx-x11 MACHINE=imx6ulevk source fsl-setup-release.sh -b ./fsl_build_x11
            exampke2:DISTRO=openstlinux-weston MACHINE=stm32mp1 source layers/meta-st/scripts/envsetup.sh
    ```
    具体的 DISTRO 、 MACHINE 视不同Soc官方给定的手册而定
7. 编译
    ```
        bitbake [image]
            example1:bitbake core-image-minimal
            example2:bitbake st-image-weston
    ```
    具体的 image 视不同Soc官方给定的手册而定

## yocto 编译后的目录架构
1. 镜像生成目录
    ```
        <yocto project>/<build dir>/tmp/deploy/images/imx6ulevk
    ```

2. 下载下来的uboot、kernel、根文件系统目录
    1. uboot 路径： 类似这种
        `<yocto project>/<build dir>/tmp/work/imx6ulevk-poky-linux-gnueabi/u-boot-imx/2016.03-r0/git`
    2. Linux 路径： 类似这种
        `<yocto project>/<build dir>/tmp/work-shared/imx6ulevk/kernel-source`
    3. 根文件系统目录
        `<yocto project>/<build dir>/tmp/deploy/images/imx6ulevk/core-image-minimal-imx6ulevk-          20190621012322.rootfs.tar.bz2`
    
    一般工作方法：
    把以上3件套拿出来单独管理