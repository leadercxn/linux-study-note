# gdb调试
* 安装移植
    * 方法一:
        1. 移植，百度

        * 大概讲解一下
            1. 下载gdb源码包 [链接](http://www.gnu.org/software/gdb/download/),并解压
            2. PC端 编译，并安装
                ```shell
                    cd gdb-9.9/
                    mkdir
                    mkdir build && cd build
                    ../configure --target=arm-linux-gnueabihf --prefix=/home/zuozhongkai/linux/IMX6ULL/tool/gdb
                            //配置 gdb。配置完成以后会在 build 目录下生成 Makefile 文件
                    make
                    make install
                ```
                编译完成以后PC端运行的gdb工具就会安装到gdb/bin目录下，名字为arm-linux-gnueabihfgdb。
            3. 移植 gdbserver 。 gdbserver 源码保存在 gdb-9.1/gdb/gdbserver 目录下，进入此目录
                ```shell
                    cd gdb-9.1/gdb/gdbserver //进入到 gdbserver 目录
                    ./configure --target=arm-linux-gnueabihf --host=arm-linux-gnueabihf //配置
                    make CC=arm-linux-gnueabihf-gcc //交叉编译 gdbserver
                ```
                编译完成会生成一个 gdbserver 的文件

    * 方法二:(该方法要求安装编译器是用源码包进行安装，不能用 apt-get install 命令来安装)
        1. 在ubuntu上，找到自己自己编译器的安装路径。例如我的是:/home/cxn/tool/arm-linux-gnueabihf/gcc-linaro/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/bin
        2. 把编译器bin路径下的 arm-linux-gnueabihf-gdb 和 gdbserver 两个文件拷贝到板子系统根目录下的bin文件里 /bin

* 调试
    * 要求: 
        1. 板子和ubuntu能进行网络通信，互ping通
        2. 在ubuntu上编译生成的程序 demo，需带 -g , ex: arm-linux-gnueabihf-gcc demo.c -o demo -g
    * 操作
        1. 在开发版上输入命令
            gdbserver 192.168.1.101:2001  demo   ==>注释: ubuntu调试主机的IP地址, 2001为端口号，端口号可随意
        2. 在ubuntu上启动gdb调试工具
            arm-linux-gnueabihf-gdb demo
            然后输入:
                target remote 192.168.1.192:2001    ==>注释: 开发板子的IP地址，2001端口号和开发版输入的端口号对应
        * gdb调试的命令
            1. l - list 命令
                列出所有源代码
            2. b - break 命令
                设置断点。 后面可以跟具体的函数或者行号
                ```shell
                    b main
                    or
                    break main
                    or
                    break 11
                ```
            3. c 
                运行到断点处，再按c，运行到下一个断点
            4. s - step 命令
                单步运行,此命令会进入到函数里面
            5. n - next 命令
                单步运行,但不会进入到函数
            6. p - print 命令
                用于打印某个变量值
            7. q - quit 命令
                退出调试。开发版上的gdbserver也会停止




