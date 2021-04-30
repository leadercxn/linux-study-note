# Makefile 笔记
* $ 搭配
    $@      目标文件 
    $^      所有的依赖文件
    $<      第一个依赖文件
    $?      所有比目标新的依赖目标的集合。以空格分隔。
    $(@F)   表示"$@"的文件部分，如果"$@"值是"dir/foo.o"，那么"$(@F)"就是"foo.o"，"$(@F)"相当于函数"$(notdir $@)"
    $(@D)   $@的目录部分

    ```demo

        有main.c  test.c  test1.c  test2.c 四个源文件

        例子1：
        %.o : %.c
        gcc  -c  $<  -o  $@
        把所以的c文件编译生成对应的o文件，$<代表每次取的c文件，$@代表每次c文件对应的目标文件


        例子2：
        main ： main.o  test.o  test1.o  test2.o
        gcc  -o  $@  $^
        把所有的o文件编译生成可执行的main文件，$^代表所以的依赖文件集合（main.o  test.o  test1.o  test2.o），@代表目标文件（main）

        例子3：
        lib : test.o  test1.o  test2.
        ar r lib $?
        把有更新的依赖文件重新打包到库lib中， 如果只有test1.o更新，则$?代表test1.o， 如果test.o  test1.o都有更新，则$?代表test.o  test1.o的集合。
    ```

* 关键字
    1. `notdir`   去除所有的目录信息，SRC里的文件名列表将只有文件名。
    ```demo
        PROJNAME := $(notdir $(shell pwd)/a.c)

        $(shell pwd) ==> /Application/sensoro/vs_pro/git_hub/sss_alarm
        $(notdir $(shell pwd)/a.c) ==> a.c
        最后返回最后一级路径
    ```

    2. `wildcard` 搜索当前目录及./foo/下所有以.c结尾的文件，生成一个以空格间隔的文件名列表，并赋值给SRC.当前目录文件只有文件名，子目录下的文件名包含路径信息，比如./foor/bar.c。
    ```
        $(wildcard src/*)
        输出：得到src目录下的文件列表

        $(wildcard src/*.c)
        输出：得到src目录下的所有的.c文件 而生成的列表
    ```

    3. `addprefix` 增加前缀 
        格式:   $(addprefix <prefix>, <name1 name2 ...>)
    ```demo
        result = $(addprefix %., c cpp)
        test:
            @echo $(result)
        输出:   %.c %.cpp
    ```

    4. `addsuffix` 增加后缀 ，用法和 addprefix 一样

    5. `subst` 字符串替换函数
    用法:   $(subst FROM,TO,TEXT),即将TEXT中的东西从FROM变为TO
    ```demo
        $(subst a,the,There is a big tree)
        
        把“There is a big tree”中的“a”替换成“the”，返回结果是“There is the big tree”。
    ```

    6. `abspath`
    用法： $(abspath names)  , 该函数主要用于将names中的各路径转换成绝对路径，并将转换后的结果返回
    ```demo
        C_SOURCE_FILES += \
                $(abspath ./app/main.c) \
    ```

    7. `patsubst` 替换
    格式：$(patsubst pattern, replacement, text)
    功能：查找< text >中的单词（单词以“空格”、“Tab”或“回车”“换行”分隔）是否符合模式< pattern >，如果匹配的话，则以< replacement >替换。这里，< pattern >可以包括通配符“%”，表示任意长度的字串。如果< replacement >中也包含“%”，那么，< replacement >中的这个“%”将是< pattern >中的那个“%”所代表的字串。（可以用“/”来转义，以“/%”来表示真实含义的“%”字符）

    8. `foreach` 遍历
    格式：$(foreach <var>,<list>,<text>)
    功能： 把参数<list>;中的单词逐一取出放到参数<var>;所指定的变量中，然后再执行< text>;所包含的表达式。每一次<text>;会返回一个字符串，循环过程中，<text>;的所返回的每个字符串会以空格分隔，最后当整个循环结束时，<text>;所返回的每个字符串所组成的整个字符串（以空格分隔）将会是foreach函数的返回值。所以，<var>;最好是一个变量名，<list>;可以是一个表达式，而<text>;中一般会使用<var>;这个参数来依次枚举<list>;
    ```demo
        names := a b c d
        files := $(foreach n,$(names),$(n).o)

        输出：$(files)的值是"a.o b.o c.o d.o"
    ```

    9. `dir`取目录函数
    格式： $(dir name)
    ```demo
        $(dir src/foo.c  a.c)
        输出： src/ ./
    ```

    10. `call` 回调、引用自定义函数
    格式： $(call VARIABLE,PARAM,PARAM,...)
    g说明： 在执行时,将它的参数"PARAM"依次赋给临时变量"$(1)","$(2)".call对参数的数目没有限制，也可以没有参数值。最后再对VARIABLE展开后的表达式进行处理.
    函数返回值:VARIABLE展开后的表达式的值

    注意：call函数中对VARIABLE的调用,直接给函数或变量名就好了，不要用"$";多个PARAM使用逗号分割开,且逗号和PARAM之间不能有空格，否则会导致解析异常
    ```demo
        #不带参
        define FUNC1
        $(info echo 3-"hello")
        endef
        $(call FUNC1)
        all:
            @echo Done

        #带参
        define FUNC1
        $(info echo 4-$(1) $(2))
        endef

        $(call FUNC1,hello,wolrd)

        all:
            @echo Done    
    ```

    11. `strip ` 去空格函数
    格式： $(strip string)
    ```C
        str1 := abc  asd
        str2 := a b  c
        str3 := a     b     c

        all:
            @echo $(strip $(str1))
            @echo $(strip $(str2))
            @echo $(strip $(str3))

        输出结果：
        abc asd
        a b c
        a b c
    ```

    12. `firstword ` 首单词函数
    格式： $(firstword NAME1 NAME2) ,返回NAME1
    ```demo
        $(firstword foo bar) 

        返回值为“foo”
    ```

    13. `filter` 过滤函数
    格式： $(filter <pattern...>,<text> )
    说明： 在text中找出 符合pattern的字符串
    ```demo
        source := a.s b.c c.cpp
        $(filter %.c %.s, $(source))    # 空格分开

        返回的是a.s b.c
    ```

    14. `filter-out` 反过滤函数，和“filter”函数实现的功能相反
    格式：$(filter-out PATTERN…,TEXT)
    说明：过滤掉字串“TEXT”中所有符合模式“PATTERN”的单词，保留所有不符合此模式的单词。可以有多个模式。存在多个模式时，模式表达式之间使用空格分割。
    ```demo
        objects=main1.o foo.o main2.o bar.o 

        mains=main1.o main2.o

        $(filter-out$(mains),$(objects))

        实现了去除变量“objects”中“mains”定义的字串（文件名）功能。它的返回值为“foo.o bar.o”。
    ```

    15. `sort` 排序函数 兼 去字符串中的重复单词
    格式： $(sort foo bar lose foo)
    ```C
        $(sort foo bar lose foo) 

        $(sort foo bar lose foo) 
    ```

    16. `vpath` 指定某类型文件的搜索路径
    ```demo
        vpath %.h include    //指定.h类型文件的搜索路径是include
    ```

    17. `word ` 取单词函数
    格式： $(word N,TEXT) 
    函数功能：取字串“TEXT”中第“N”个单词（“N”的值从 1开始）。 
    返回值：返回字串“TEXT”中第“N”个单词。 
    函数说明：如果“N”值大于字串“TEXT”中单词的数目，返回空字符串。如果“N”为 0，出错！ 
    ```C
    $(word 2, foo bar baz) 

    返回值为“bar”
    ```

    18. `export` 将被声明的变量添加到该工程的环境变量中
    在 GNU make 中，实现此功能的指示符是 export。当一个变量使用 export 进行声明后，变量和它的值将被加入到当前工作的环境变量中，以后在 make 执行的所有规则的命令都可以使用这个变量。

    而当没有使用指示符 export 对任何变量进行声明的情况下，上层 make 只将那些已经初始化的环境变量（在执行 make 之前已经存在的环境变量）和使用命令行指定的变量（如命令make CFLAGS +=-g 或者 make –e CFLAGS +=-g）传递给子 make 过程
    ```demo
        KCONFIG_CONFIG = .config
        KCONFIG_AUTOHEADER = application/kconfig.h
        KCONFIG_AUTOCONFIG = build/include/config/auto.conf
        KCONFIG_TRISTATE = build/include/config/tristate.conf

        export KCONFIG_CONFIG KCONFIG_AUTOHEADER KCONFIG_AUTOCONFIG KCONFIG_TRISTATE
    ```

    19. `realpath` 用于获取指定目录或文件的绝对路径
    Shell 脚本中，通常会使用相对路径来指明文件，但有时候，我们需要用到绝对路径，此时可以使用 realpath 来获取。
    ```demo
        realpath [OPTIONS] FILES

        realpath ./hello.tgz

        返回:   /data/test/src/hello.tgz
    ```

* 可选参数

    `-C`    -C $(KDIR) 指明跳转到源码目录下读取那里的Makefile；
    `M`     M=$(PWD) 表明然后返回到当前目录继续读入、执行当前的Makefile
    `-f`    指定特定的Makefile 。 make -f file
    `-e`    环境变量覆盖Makefile文件

    ```demo
        gcc -o hello hello.c -I /home/hello/include -L /home/hello/lib -lworld
    ```
    `-I`     -I /home/hello/include表示将/home/hello/include目录作为第一个寻找头文件的目录,寻找的顺序是：/home/hello/include-->/usr/include-->/usr/local/include
    `-L`     -L /home/hello/lib表示将/home/hello/lib目录作为第一个寻找库文件的目录，寻找的顺序是：/home/hello/lib-->/lib-->/usr/lib-->/usr/local/lib
            -lworld 表示在上面的lib的路径中寻找libworld.so动态库文件（如果gcc编译选项中加入了“-static”表示寻找libworld.a静态库文件）

    `-c`      -c 编译和汇编，但不要链接。

    `-l`(小写L)     #lc 是link libc
                    #lm 是link libm	-lm选项告诉编译器，我们程序中用到的数学函数要到这个库文件里找
                    #lz 是link libz 
                    #-lpthread posix线程 

* Makefile的内嵌变量
    1. CURDIR   Makefile的内嵌变量，自动设置为当前目录



# 脚本
* 关键字
    1. `dirname` 表示提取参数里的目录
        ```sh
            dirname "modules/tools/planning_traj_plot/run.s"
                => modules/tools/planning_traj_plot
        ```

    2. `${BASH_SOURCE[0]}` 表示bash脚本的第一个参数（如果第一个参数是bash，表明这是要执行bash脚本，这时"${BASH_SOURCE[0]}"自动转换为第二个参数
        ```sh
            bash modules/tools/planning_traj_plot/run.sh 

            "${BASH_SOURCE[0]}"代表的是modules/tools/planning_traj_plot/run.sh

            cd "$( dirname "${BASH_SOURCE[0]}" )" # 表示切换到刚才提取的目录

            DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )" #导出脚本的路径
        ```
 