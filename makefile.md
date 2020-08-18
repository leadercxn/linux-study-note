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
        PROJNAME := $(notdir $(shell pwd))

        $(shell pwd) ==> /Application/sensoro/vs_pro/git_hub/sss_alarm
        $(notdir $(shell pwd)) ==> sss_alarm
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







