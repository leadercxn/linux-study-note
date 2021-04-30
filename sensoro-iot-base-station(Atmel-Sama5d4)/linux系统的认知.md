# Linux系统的加深认知

1. /var/run
    /var/run 目录中存放的是自系统启动以来描述系统信息的文件。比较常见的用途是daemon进程将自己的pid保存到这个目录。标准要求这个文件夹中的文件必须是在系统启动的时候清空，以便建立新的文件。

2. flock
    Linux 下的文件锁。当多个进程可能会对同样的数据执行操作时，这些进程需要保证其它进程没有也在操作，以免损坏数据。通常，这样的进程会使用一个「锁文件」，也就是建立一个文件来告诉别的进程自己在运行，如果检测到那个文件存在则认为有操作同样数据的进程在工作。这样的问题是，进程不小心意外死亡了，没有清理掉那个锁文件，那么只能由用户手动来清理了。

3. gpio
    * /sys/class/gpio中的gpio的控制
    * /sys/class/gpio 的使用说明：
        1. gpio_operation 通过 /sys/文件接口操作IO端口 GPIO到文件系统的映射
        2. 控制GPIO的目录位于 /sys/class/gpio
        3. /sys/class/gpio/export 文件用于通知系统需要导出控制的GPIO引脚编号
        4. /sys/class/gpio/unexport 用于通知系统取消导出
        5. /sys/class/gpio/gpiochipX 目录保存系统中GPIO寄存器的信息，包括每个寄存器控制引脚的起始编号base，寄存器名称，引脚总数 导出一个引脚的操作步骤
        6. 首先计算此引脚编号，引脚编号 = 控制引脚的寄存器基数 + 控制引脚寄存器位数
        7. 向 /sys/class/gpio/export 写入此编号，比如12号引脚，在shell中可以通过以下命令实现
            ```C
                echo 12 > /sys/class/gpio/export
                命令成功后生成/sys/class/gpio/gpio12目录，如果没有出现相应的目录，说明此引脚不可导出
            ```
        8. direction 文件，定义输入输入方向，可以通过下面命令定义为输出
            ```
                echo out > /sys/class/gpio/gpio12/direction
            ```
            direction接受的参数：in, out, high, low。high/low同时设置方向为输出，并将value设置为相应的1/0。
            value文件是端口的数值，为1或0.
            ```
                echo 1 >/sys/class/gpio/gpio12/value
            ```

