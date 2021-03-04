# openwrt 笔记
1. 安装编译所需依赖环境
    ```C
        sudo apt-get install g++ 
        sudo apt-get install libncurses5-dev 
        sudo apt-get install zlib1g-dev 
        sudo apt-get install bison 
        sudo apt-get install flex 
        sudo apt-get install unzip 
        sudo apt-get install autoconf 
        sudo apt-get install gawk 
        sudo apt-get install make 
        sudo apt-get install gettext 
        sudo apt-get install gcc 
        sudo apt-get install binutils 
        sudo apt-get install patch 
        sudo apt-get install bzip2 
        sudo apt-get install libz-dev 
        sudo apt-get install asciidoc 
        sudo apt-get install subversion 
        sudo apt-get install sphinxsearch 
        sudo apt-get install libtool 
        sudo apt-get install sphinx-common

        sudo apt-get install ocaml-nox
        sudo apt-get install libxml-parser-perl
        sudo apt-get install libc6-dev
        sudo apt-get install pkg-config
        sudo apt-get install liblzo2-dev
        sudo apt-get install libacl1-dev
        sudo apt-get install uuid-dev
        sudo apt-get install sharutils
    ```

2. 从git_hub上下载源码 [源码](https://github.com/openwrt/openwrt)

## 编译前的准备
* 拉取相关库
    1. ./scripts/feeds update -a    
        * clone 和 update  feeds.conf.default里面指定的5个插件(package、luci、routing、telephony、freifunk)
        * 会生成一个feeds文件夹，里面存放了5个子文件夹
* 安装库
    1. ./scripts/feeds install -a
        * 会把update 的5个库拷贝到 package/feeds,本质是吧feeds目录下的所有文件夹集中地软链接接到 package/feeds/ 目录下
* 补充
    1. feeds 是从第三方(非官方)拉取的包
    2. package 是官方的包

# 编译
* 操作
    1. make menuconfig 自定义选择编译条件
    2. make -j4 V=s