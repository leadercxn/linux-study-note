# kernel接口使用整理

1. BUILD_BUG_ON(condition)
    定义: #define BUILD_BUG_ON(condition) ((void)sizeof(char[1 - 2*!!(condition)]))
    编译的时候如果condition为真，则编译出错

2. #define LIST_HEAD(name)
    一个定义并初始化作用。
    * demo:
    ```C
        static LIST_HEAD(device_list);
    ```

3. module_param(name, type, perm);
    作用:表示向当前模块传入参数.(驱动模块)
    参数:
        name:既是用户看到的参数名，又是模块内接受参数的变量；;
        type:表示参数的类型;
            bool，invbool /*一个布尔型( true 或者 false)值(相关的变量应当是 int 类型). invbool 类型颠倒了值, 所以真值变成 false, 反之亦然.*/
            harp/*一个字符指针值. 内存为用户提供的字串分配, 指针因此设置.*/
            int，long，short
            uint，ulong，ushort /*基本的变长整型值. 以 u 开头的是无符号值.*/
        perm:指定了在sysfs中相应文件的访问权限;
    * demo:
    ```C
        static unsigned bufsiz = 4096;
        module_param(bufsiz, uint, S_IRUGO);
        MODULE_PARM_DESC(bufsiz, "data bytes in biggest supported SPI message");
    ```

4. unsigned long copy_from_user(void *to, const void __user *from, unsigned long n)
    该函数的目的是从用户空间拷贝数据到内核空间，失败返回没有被拷贝的字节数，成功返回0.
    * demo:
    ```
        copy_from_user(spidev->tx_buffer, buf, count);
    ```

5. unsigned long copy_to_user(void *to, const void *from, unsigned long n)
    to:目标地址（用户空间）
    from:源地址（内核空间）
    n:将要拷贝数据的字节数
    返回：成功返回0，失败返回没有拷贝成功的数据字节数
    * demo
    ```
        copy_from_user(spidev->tx_buffer, buf, count);
    ```

6.  static inline void *kcalloc(size_t n, size_t size, gfp_t flags)
        为一个 数组 分配memory并将memory中的内容清零

    static inline void *kzalloc(size_t size, gfp_t flags)
        kzalloc()是kmalloc()和memset()函数的整合

    #include <linux/slab.h> void *kmalloc(size_t size, int flags);
    给 kmalloc 的第一个参数是要分配的块的大小. 第 2 个参数, 分配标志
        GFP_ATOMIC  用来从中断处理和进程上下文之外的其他代码中分配内存. 从不睡眠.
        GFP_KERNEL  内核内存的正常分配. 可能睡眠.
        GFP_USER    用来为用户空间页来分配内存; 它可能睡眠.
        GFP_HIGHUSER    如同 GFP_USER, 但是从高端内存分配, 如果有. 高端内存在下一个子节描述.
        GFP_NOIO
        GFP_NOFS    这个标志功能如同 GFP_KERNEL, 但是它们增加限制到内核能做的来满足请求. 一个 GFP_NOFS 分配不允许进行任何文件系统调用, 而 GFP_NOIO 根本不允许任何 I/O 初始化. 它们主要地用在文件系统和虚拟内存代码, 那里允许一个分配睡眠, 但是递归的文件系统调用会是一个坏注意.

7. 完成量 completion，同步机制，即completion，它用于一个执行单元等待另一个执行单元执行完某事。
    1. API
        1. 定义completion
          struct completion my_completion;
        2. 初始化completion
          init_completion(&my_completion);
          
          DECLARE_COMPLETION(my_completion);   #对my_completion的定义和初始化可以通过如下快捷方式实现
          DECLARE_COMPLETION_ONSTACK(complete);  //创建一个completion结构体放在内核堆栈中，如果不加ONSTACK就是静态申请，会存放在全局变量区
        3. 等待completion
          void wait_for_completion(struct completion *c);
        4. 唤醒completion
          void complete(struct completion *c);
          void complete_all(struct completion *c);
         前者只唤醒一个等待的执行单元，后者唤醒所有等待同一completion的执行单元。

8. access_ok (type, addr, size);
    type:访问类型，其值可为 VERIFY_READ 或者 VERIFY_WRITE 。注意，VERIFY_WRITE 是 VERIFY_READ 的超集 -- 如果可以安全的写内存块，那么自然也总能读到内存块。
    addr:用户空间的指针变量，其指向一个要检查的内存块开始处。
    size:要检查内存块的大小。
    返回值：此函数检查用户空间中的内存块是否可用。如果可用，则返回真(非0值)，否则返回假 (0) 。
    * demo
    ```C
        err = !access_ok(VERIFY_READ, (void __user *)arg, _IOC_SIZE(cmd));
    ```

    扩展:
        #include<unistd.h>
        int access(const char* pathname, int mode);
            pathname 是文件的路径名+文件名
            mode：指定access的作用，取值如下
                F_OK 值为0，判断文件是否存在
                X_OK 值为1，判断对文件是可执行权限
                W_OK 值为2，判断对文件是否有写权限
                R_OK 值为4，判断对文件是否有读权限
            返回值：成功0，失败-1


9. _IOC_NR(), _IOC_TYPE(), _IOC_SIZE(), _IOC_DIR()  这几个宏用来取得 cmd 命令中的域
    ```
        _IOC_NR()  :    读取基数域值 (bit0~ bit7)
        _IOC_TYPE :     读取魔数域值 (bit8 ~ bit15)
        _IOC_SIZE  :    读取数据大小域值 (bit16 ~ bit29)
        _IOC_DIR    :   获取读写属性域值 (bit30 ~ bit31)
    ```

10. put_user(x, ptr);  get_user(x, ptr);
    1. put_user --    Write a simple value into user space.
        x   Value to copy to user space.
        ptr Destination address, in user space.
    2. get_user --    Get a simple variable from user space.
        x   Variable to store result.
        ptr Source address, in user space.

11. Linux内核中 struct file_operations 含有下面两个函数指针：
    ```
        struct file_operations {
                ... ...
            long (*unlocked_ioctl) (struct file *, unsigned int, unsigned long);
            long (*compat_ioctl) (struct file *, unsigned int, unsigned long);
                ... ...
        }；
    ```
    1、compat_ioctl：支持64bit的driver必须要实现的ioctl，当有32bit的userspace application call 64bit kernel的IOCTL的时候，这个callback会被调用到。如果没有实现compat_ioctl，那么32位的用户程序在64位的kernel上执行ioctl时会返回错误：Not a typewriter

    2、如果是64位的用户程序运行在64位的kernel上，调用的是unlocked_ioctl，如果是32位的APP运行在32位的kernel上，调用的也是unlocked_ioctl

12. DECLARE_BITMAP(name,bits)
    功能: 声明一个保存数据类型是 unsigned long 类型的结构体，整个的数组成员，我们看做是保存了 bits 个位的容器。
    参数: 
        name，保存位的 unsigned long 类型的数据保存在内存中的首地址，或者是bitmap的首地址
        bits，我们要用到多少位，或者是bitmap中元素个数

13. 错误码宏
    1. void * ERR_PTR(long error)：     将错误编码转为指针
    2. long PTR_ERR(const void *ptr)：  将指针转为错误编码
    3. bool IS_ERR(const void *ptr)：   检查返回指针是否为一个错误编码，如果返回真，表示确实发生了错误
    4. bool IS_ERR_OR_NULL(const void *ptr)：   检查返回的指针是否为NULL或是否为一个错误编码
    5. int PTR_ERR_OR_ZERO(const void *ptr)：   将返回的指针转为0或者错误编码
    6. void * ERR_CAST(const void *ptr)：       将const void *转为void *，防止编译报错

