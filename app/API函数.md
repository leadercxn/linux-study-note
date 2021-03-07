# 接口函数

## getopt 短、长选项
1. [getopt()和getopt_long()](https://www.cnblogs.com/chenliyang/p/6633739.html)
    变量说明
    ```C
        #if 0
            optarg —— 指向当前选项参数(如果有)的指针。
            optind —— 再次调用 getopt() 时的下一个 argv指针的 索引。
            optopt —— 最后一个未知选项。
            opterr ­—— 如果不希望getopt()打印出错信息，则只要将全域变量opterr设为0即可。
        #endif
    ```


