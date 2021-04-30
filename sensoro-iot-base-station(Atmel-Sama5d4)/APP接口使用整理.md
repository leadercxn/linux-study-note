# APP接口使用整理

1. fcntl 使用
    针对(文件)描述符提供控制.参数fd是被参数cmd操作(如下面的描述)的描述符
    * 头文件
        #include <unistd.h>
        #include <fcntl.h>
    * 函数原型：          
        int fcntl(int fd, int cmd);
        int fcntl(int fd, int cmd, long arg);         
        int fcntl(int fd, int cmd, struct flock *lock);
    * 功能：
        1. 复制一个现有的描述符（cmd=F_DUPFD）.
        2. 获得／设置文件描述符标记(cmd=F_GETFD或F_SETFD).
        3. 获得／设置文件状态标记(cmd=F_GETFL或F_SETFL).
        4. 获得／设置异步I/O所有权(cmd=F_GETOWN或F_SETOWN).
        5. 获得／设置记录锁(cmd=F_GETLK,F_SETLK 或 F_SETLKW).

2. popen 、 pclose 的使用
    1. 原型
        FILE * popen ( const char * command , const char * type );
        int pclose ( FILE * stream );

        type 参数只能是读或者写中的一种，得到的返回值（标准 I/O 流）也具有和 type 相应的只读或只写类型。如果 type 是 "r" 则文件指针连接到 command 的标准输出；如果 type 是 "w" 则文件指针连接到 command 的标准输入。
        command 参数是一个指向以 NULL 结束的 shell 命令字符串的指针。这行命令将被传到 bin/sh 并使用-c 标志，shell 将执行这个命令。
        popen 的返回值是个标准 I/O 流，必须由 pclose 来终止。前面提到这个流是单向的。所以向这个流写内容相当于写入该命令的标准输入；命令的标准输出和调用 popen 的进程相同。与之相反的，从流中读数据相当于读取命令的标准输出；命令的标准输入和调用 popen 的进程相同。
    2. 说明
        popen() 函数通过创建一个管道，调用 fork 产生一个子进程，执行一个 shell 以运行命令来开启一个进程。这个进程必须由 pclose() 函数关闭，而不是 fclose() 函数。pclose() 函数关闭标准 I/O 流，等待命令执行结束，然后返回 shell 的终止状态。如果 shell 不能被执行，则 pclose() 返回的终止状态与 shell 已执行 exit 一样。
    3. demo
        ```C
            FILE *fp = NULL;

            fp = popen("/root/bin/bst_system_config --lora.band ?", "r");
            pclose(fp);
        ```

3. fread 使用
    1. 函数原型: 
    ```C
        size_t fread( void *buffer, size_t size, size_t count, FILE *stream );                      //C99前
        size_t fread( void *restrict buffer, size_t size, size_t count, FILE *restrict stream );    //C99起

        buffer  指向要读取的数组中首个对象的指针 。要存储数据流的内存空间指针
        size    每个对象的大小（单位是字节）
        count   要读取的对象个数
        stream  输入流
    ```
    2. 函数说明
        从给定输入流stream读取最多count个对象到数组buffer中（相当于以对每个对象调用size次fgetc），把buffer当作unsigned char数组并顺序保存结果。流的文件位置指示器前进读取的字节数。
        若出现错误，则流的文件位置指示器的位置不确定。若没有完整地读入最后一个元素，则其值不确定。
    3. demo
        ```C
            fp = popen(cmd_buf, "r");
            n = fread(rlt_buf, sizeof(char), BUF_SIZE, fp);
        ```

4. tcgetattr 、 cfsetispeed 、 cfsetospeed 、 tcsetattr 、 tcflush 
    1. int tcgetattr(int fd, struct termios *termios_p);    //用于获取与终端相关的参数。
        参数fd为终端的文件描述符，
        返回的结果保存在termios 结构体中
    2. int tcsetattr(int fd, int optional_actions, const struct termios *termios_p);    //用于设置终端相关的参数。

    3. int cfsetospeed(struct termios*termptr, speed_t speed); //
        struct termios *termptr -指向termios结构的指针
        speed_t speed - 需要设置的输出波特率
        返回值: 如果成功返回0,否则返回-1
    4. int cfsetispeed(struct termios*termptr, speed_t speed);
        struct termios *termptr -指向termios结构的指针
        speed_t speed - 需要设置的输入波特率
        返回值:如果成功返回0,否则返回-1
    5. int tcflush(int fd, int queue_selector); //清空终端未完成的输入/输出请求及数据。
        fd                // 终端I/O打开的句柄
        queue_selector    // 控制tcflush的操作，取值为下面三个常数中的一个：
                    TCIFLUSH  // 清除正收到的数据，且不会读取出来。
                    TCOFLUSH  // 清除正写入的数据，且不会发送至终端。
                    TCIOFLUSH // 清除所有正在发生的I/O数据。
        0     // 成功
        -1    // 失败，并且为 errno 置值来指示错误

5. tzset
    1. void tzset(void);    //设置时间环境变量。

6. uint32_t htonl(uint32_t hostlong);
    功能：将主机数转换成无符号长整形的网络字节顺序。
    所谓网络字节顺序（大尾顺序）就是指一个数在内存中存储的时候“高对低，低对高”（即一个数的高位字节存放于低地址单元，低位字节存放在高地址单元中）。但是计算机的内存存储数据时有可能是大尾顺序或者小尾顺序。


7. getaddrinfo()函数详解，
    1. [链接](https://blog.csdn.net/fanx021/article/details/80549945)   
        IPv4中使用 gethostbyname()函数 `完成主机名到地址解析` ，这个函数仅仅支持IPv4，且不允许调用者指定所需地址类型的任何信息，返回的结构只包含了用于存储IPv4地址的空间。IPv6中引入了getaddrinfo()的新API，它是协议无关的，既可用于IPv4也可用于IPv6。
    2. int getaddrinfo( const char *hostname, const char *service, const struct addrinfo *hints, struct addrinfo **result );
        hostname: 一个主机名或者地址串(IPv4的点分十进制串或者IPv6的16进制串)
        service: 服务名可以是十进制的端口号，也可以是已定义的服务名称，如ftp、http等
        hints: 可以是一个空指针，也可以是一个指向某个 addrinfo结构体的指针，调用者在这个结构中填入关于期望返回的信息类型的暗示。举例来说：如果指定的服务既支持TCP也支持UDP，那么调用者可以把hints结构中的ai_socktype成员设置成SOCK_DGRAM使得返回的仅仅是适用于数据报套接口的信息。
        result: 本函数通过result指针参数返回一个指向addrinfo结构体链表的指针。
        返回值: 0——成功，非0——出错

8. clock_nanosleep()
    1. 函数原型:
        ```C
        INT clock_nanosleep（clockid_t clock_id ，INT flags，const struct timespec *request,struct timespec *remain）;

            lock_id参数指定针对其睡眠时钟间隔将被测量。该参数可以具有 以下值之一：
                CLOCK_REALTIME 可设置的系统范围的实时时钟。
                CLOCK_MONOTONIC 一个不可设置的，单调递增的时钟，用于测量从过去某个未指定的点开始的时间，该时间在系统启动后不会更改。
                CLOCK_PROCESS_CPUTIME_ID 可设置的每个进程时钟，用于测量进程中所有线程消耗的CPU时间。

            返回值：0 : 在请求的时间间隔内成功睡眠后
                   ERRORS中列出的正错误号之一 : 如果调用被信号处理程序中断或遇到错误
        ```
    2. 功能说明
        clock_nanosleep（）允许调用线程以纳秒精度指定的时间间隔进行睡眠。它的不同之处在于允许调用者选择要测量睡眠间隔的时钟，并且允许将睡眠间隔指定为绝对值或相对值。

9. 信号
    ```
        1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL
        5) SIGTRAP      6) SIGABRT      7) SIGBUS       8) SIGFPE
        9) SIGKILL     10) SIGUSR1     11) SIGSEGV     12) SIGUSR2
        13) SIGPIPE     14) SIGALRM     15) SIGTERM     16) SIGSTKFLT
        17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
        21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU
        25) SIGXFSZ     26) SIGVTALRM   27) SIGPROF     28) SIGWINCH
        29) SIGIO       30) SIGPWR      31) SIGSYS      34) SIGRTMIN
        35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3  38) SIGRTMIN+4
        39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
        43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12
        47) SIGRTMIN+13 48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14
        51) SIGRTMAX-13 52) SIGRTMAX-12 53) SIGRTMAX-11 54) SIGRTMAX-10
        55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7  58) SIGRTMAX-6
        59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
        63) SIGRTMAX-1  64) SIGRTMAX

        SIGRTMIN这里面的SIG就是signal，RT就是real time实时，MIN和MAX都知道了。
    ```

    * 应用层 signal 相关数据结构
        + struct sigaction 结构
            ```C
                struct sigaction 
                {
                    void (*sa_handler)(int);
                    void (*sa_sigaction)(int, siginfo_t *, void *);
                    sigset_t sa_mask;
                    int sa_flags;
                    void (*sa_restorer)(void);
                }

                    sa_handler 此参数和signal()的参数handler相同，此参数主要用来对信号旧的安装函数signal()处理形式的支持
                    sa_sigaction 信号处理函数指针2，新的信号安装机制，处理函数被调用的时候，不但可以得到信号编号，而且可以获悉被调用的原因以及产生问题的上下文的相关信息。
                    sa_mask 信号掩码 （信号处理函数执行中才屏蔽）
                    sa_flags 信号处理标志
                        SA_RESETHAND：当调用信号处理函数时，将信号的处理函数重置为缺省值SIG_DFL
                        SA_RESTART：如果信号中断了进程的某个系统调用，则系统自动启动该系统调用
                        SA_NODEFER ：一般情况下， 当信号处理函数运行时，内核将阻塞该给定信号。但是如果设置了 SA_NODEFER标记， 那么在该信号处理函数运行时，内核将不会阻塞该信号
                        SA_SIGINFO：信号处理函数是带有三个参数的sa_sigaction
                    sa_restorer 保留

            ```
        + 
            ```C
                    struct sigevent
                    {
                        int sigev_notify; //notification type
                        int sigev_signo; //signal number
                        union sigval   sigev_value; //signal value
                        void (*sigev_notify_function)(union sigval);
                        pthread_attr_t *sigev_notify_attributes;
                    }

                    union sigval
                    {
                        int sival_int;      //integer value
                        void *sival_ptr;    //pointer value
                    }
            ```

    * API
        ```C
            int sigaction(int signum, const struct sigaction *act,struct sigaction *oldact);

            signum  信号码
            act     信号处理方式
            oldact  原信号处理方式，可为NULL

            功能是检查或修改与指定信号相关联的处理动作
        ```

10. timer 定时器
    1. 创建定时器 timer_create
        ```C
            int timer_create(clockid_t clock_id, struct sigevent *evp,timer_t* timerid)
                创建的定时器是每个进程自己的，不是在fork时继承的。

                clock_id 说明定时器是基于哪个时钟的。可以为：
                    CLOCK_REALTIME:  Systemwide realtime clock.
                    CLOCK_MONOTONIC:  Representsmonotonic time. Cannot be set. 单调递增的计数器，不可设置
                    CLOCK_PROCESS_CPUTIME_ID:  High resolution per-process timer.
                    CLOCK_THREAD_CPUTIME_ID:T  hread-specific timer.
                    CLOCK_REALTIME_HR:  High resolution version of CLOCK_REALTIME.
                    CLOCK_MONOTONIC_HR:  High resolution version of CLOCK_MONOTONIC.
                evp 指定了定时器到期要产生的异步通知
                *timerid 装载的是被创建的定时器的ID。
        ```
    2. 启动一个定时器 timer_settime
        ```C
            int timer_settime(timer_t timerid, int flags, const struct itimerspec *value, struct itimerspect *ovalue);
 
            struct itimespec{
                struct timespec it_interval; 
                struct timespec it_value;   
            }; 

            it_value 用于指定当前的定时器到期 时间
            如果it_interval的值为0，则定时器不是一个时间间隔定时器，一旦it_value到期就会回到未启动状态
        ```
    3. 删除一个定时器 timer_delete
        ```C
        int timer_delete (timer_t timerid);

        一次成功的timer_delete()调用会销毁关联到timerid的定时器并且返回0。执行失败时，此调用会返回-1并将errno设定会 EINVAL，这个唯一的错误情况代表timerid不是一个有效的定时器。
        ```

11. 互斥锁
    * demo
        ```C
            static pthread_mutex_t mx_meas_dw = PTHREAD_MUTEX_INITIALIZER; 

            pthread_mutex_lock(&mx_meas_dw);    //互斥锁上锁

            ...
            pthread_mutex_unlock(&mx_meas_dw);  //互斥锁解锁
        ```

12. struct timespec 和 struct timeval
    1. timespec
        ```
            typedef long time_t;
            #ifndef _TIMESPEC
            #define _TIMESPEC
                struct timespec {
                    time_t tv_sec;  // seconds,秒
                    long tv_nsec;   // and nanoseconds，纳秒
                };
            #endif

            int clock_gettime(clockid_t, struct timespec *)获取特定时钟的时间

            CLOCK_REALTIME 统当前时间，从1970年1.1日算起
            CLOCK_MONOTONIC 系统的启动时间，不能被设置
            CLOCK_PROCESS_CPUTIME_ID 本进程运行时间
            CLOCK_THREAD_CPUTIME_ID 本线程运行时间
        ```
    2. timeval
        ```C
            struct timeval {
                    time_t tv_sec;      // seconds ,秒
                    long tv_usec;       // microseconds，微妙
                };

            struct timezone {
                int tz_minuteswest;     //miniutes west of Greenwich
                int tz_dsttime;         //type of DST correction
            };

            一般由函数int gettimeofday(struct timeval *tv, struct timezone *tz)获取系统的时间
        ```

