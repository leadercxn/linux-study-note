# SPI 驱动框架

* 数据结构 include/linux/spi/spi.h
    1. spi 主设备结构体
        ```C
            struct spi_master{
                ...
                int	(*transfer)(struct spi_device *spi,struct spi_message *mesg);   //传输函数
                ...
            };
        ```

    2. SPI 设备驱动,接口
        ```C
            struct spi_driver {
                                const struct spi_device_id *id_table;
                                int (*probe)(struct spi_device *spi);
                                int (*remove)(struct spi_device *spi);
                                void (*shutdown)(struct spi_device *spi);
                                struct device_driver driver;
                            };
        ```

* API
    1. 申请 spi_master 
    ```C
        struct spi_master *spi_alloc_master(struct device *dev,unsigned size)

        dev：设备，一般是 platform_device 中的 dev 成员变量。
        size： 私有数据大小，可以通过 spi_master_get_devdata 函数获取到这些私有数据。
        返回值： 申请到的 spi_master。
    ```

    2. 释放 spi_master
    ```C
        void spi_master_put(struct spi_master *master)

        master：要释放的 spi_master。
        返回值： 无
    ```

    3. 注册 spi_master
    ```C
        int spi_register_master(struct spi_master *master)

        master：要注册的 spi_master。
        返回值： 0，成功；负值，失败。
    ```

    4. 注销 spi_master
    ```C
        void spi_unregister_master(struct spi_master *master)

        master：要注销的 spi_master。
    ```


# demo框架
```C
    /* 上面写 设备操作数据结构体 */
        static file_operations_t dev_ap3216c_fops ={
            .owner = THIS_MODULE ,
            .open  = icm20608_open,
            .read  = icm20608_read,
        };

    /* probe 函数 */
    static int xxx_probe(struct spi_device *spi)
    {
        /* 具体函数内容 */
        return 0;
    }

    /* remove 函数 */
    static int xxx_remove(struct spi_device *spi)
    {
        /* 具体函数内容 */
        return 0;
    }

    /* 传统匹配方式 ID 列表 */
    static const struct spi_device_id xxx_id[] = {
        {"xxx", 0},
        {}
    };

    /* 设备树匹配列表 */
    static const struct of_device_id xxx_of_match[] = {
        { .compatible = "xxx" },
        { /* Sentinel */ }
    };

    /* SPI 驱动结构体 */
    static struct spi_driver xxx_driver = {
                                            .probe = xxx_probe,
                                            .remove = xxx_remove,
                                            .driver = {
                                            .owner = THIS_MODULE,
                                            .name = "xxx",
                                            .of_match_table = xxx_of_match,
                                            },
                                            .id_table = xxx_id,
                                        };

    /* 驱动入口函数 */
    static int __init xxx_init(void)
    {          
        return spi_register_driver(&xxx_driver);
    }

    /* 驱动出口函数 */
    static void __exit xxx_exit(void)
    {
        spi_unregister_driver(&xxx_driver);
    }

    module_init(xxx_init);
    module_exit(xxx_exit);                           
```

# 构建设备树节点
* 参考  imx6qdl-sabresd.dtsi
    ```C
        &ecspi1 {
                fsl,spi-num-chipselects = <1>;
                cs-gpios = <&gpio4 9 0>;
                pinctrl-names = "default";
                pinctrl-0 = <&pinctrl_ecspi1>;
                status = "okay";

                flash: m25p80@0 {
                    #address-cells = <1>;
                    #size-cells = <1>;
                    compatible = "st,m25p32";
                    spi-max-frequency = <20000000>;
                    reg = <0>;
                };
            };
    ```
* 过程
    1. 先在原理图上找到对应的IO，然后根据IO的名称在 设备树文件夹中的 `imx6ul-pinfunc.h`头文件中搜索对应的IO配置宏。假如有些IO复用，记得多根据IO类似名字，搜索整个头文件对应的宏
    ```C
        pinctrl_ecspi3: icm20608 {
                                fsl,pins = <
                                    MX6UL_PAD_UART2_TX_DATA__GPIO1_IO20 0x10b0 /* CS */
                                    MX6UL_PAD_UART2_RX_DATA__ECSPI3_SCLK 0x10b1 /* SCLK */
                                    MX6UL_PAD_UART2_RTS_B__ECSPI3_MISO 0x10b1 /* MISO */
                                    MX6UL_PAD_UART2_CTS_B__ECSPI3_MOSI 0x10b1 /* MOSI */
                                >;
                                    };
    ```


# SPI 设备数据收发流程
* 数据结构
    1. 传输接口 struct spi_transfer 
    ```C
        struct spi_transfer 
        {
            /* it's ok if tx_buf == rx_buf (right?)
            * for MicroWire, one buffer must be null
            * buffers must work with dma_*map_single() calls, unless
            * spi_message.is_dma_mapped reports a pre-existing mapping
            */
            const void	*tx_buf;
            void		*rx_buf;
            unsigned	len;    //SPI 是全双工通信，因此在一次通信中发送和接收的字节数都是一样的，所以 spi_transfer 中也就没有发送长度和接收长度之分

            dma_addr_t	tx_dma;
            dma_addr_t	rx_dma;
            struct sg_table tx_sg;
            struct sg_table rx_sg;

            unsigned	cs_change:1;
            unsigned	tx_nbits:3;
            unsigned	rx_nbits:3;
        #define	SPI_NBITS_SINGLE	0x01 /* 1bit transfer */
        #define	SPI_NBITS_DUAL		0x02 /* 2bits transfer */
        #define	SPI_NBITS_QUAD		0x04 /* 4bits transfer */
            u8		bits_per_word;
            u16	s;
            u32		speed_hz;	delay_usec

            struct list_head transfer_list;
        };

    ```

    2. 传输信息数据结构 struct spi_message
    ```C
        struct spi_message
        {
            struct list_head	transfers;

            struct spi_device	*spi;

            unsigned		is_dma_mapped:1;

            /* REVISIT:  we might want a flag affecting the behavior of the
            * last transfer ... allowing things like "read 16 bit length L"
            * immediately followed by "read L bytes".  Basically imposing
            * a specific message scheduling algorithm.
            *
            * Some controller drivers (message-at-a-time queue processing)
            * could provide that as their default scheduling algorithm.  But
            * others (with multi-message pipelines) could need a flag to
            * tell them about such special cases.
            */

            /* completion is reported through a callback */
            void			(*complete)(void *context);
            void			*context;
            unsigned		frame_length;
            unsigned		actual_length;
            int			status;

            /* for optional use by whatever driver currently owns the
            * spi_message ...  between calls to spi_async and then later
            * complete(), that's the spi_master controller driver.
            */
            struct list_head	queue;
            void			*state;
        };
    ```

* API
    1. 初始化 spi_message 结构体 （在使用 spi_message之前需要对其进行初始化,直接定义变量，再初始化。不用定义变量后，再赋值变量成员的值后再初始化）
    ```C
        void spi_message_init(struct spi_message *m)
        m： 要初始化的 spi_message。
    ```

    2. 将 spi_transfer 添加到 spi_message (spi_message 初始化完成以后需要将 spi_transfer 添加到 spi_message 队列中)
    ```C
        void spi_message_add_tail(struct spi_transfer *t, struct spi_message *m)

        t： 要添加到队列中的 spi_transfer。
        m： spi_transfer 要加入的 spi_message。
    ```

    3. 同步传输（会阻塞，知道等待SPI传输完成）
    ```C
        int spi_sync(struct spi_device *spi, struct spi_message *message)

        spi： 要进行数据传输的 spi_device。
        message：要传输的 spi_message。
        返回值： 无。
    ```

    4. 异步传输（不会等待SPI传输完成，异步传输需要设置 spi_message 中的 complete成员变量， complete 是一个回调函数，当 SPI 异步传输完成以后此函数就会被调用）
    ```C
        int spi_async(struct spi_device *spi, struct spi_message *message)

        spi： 要进行数据传输的 spi_device。
        message：要传输的 spi_message。
    ```

# 编写具体额设备驱动
* 读、写操作的流程
    1. 申请并初始化 spi_transfer，设置 spi_transfer 的 tx_buf 成员变量， tx_buf 为要发送的数据。然后设置 rx_buf 成员变量， rx_buf 保存着接收到的数据。最后设置 len 成员变量，也就是要进行数据通信的长度。
    2. 使用 spi_message_init 函数初始化 spi_message。
    3. 使用 spi_message_add_tail函数将前面设置好的 spi_transfer添加到 spi_message队列中。
    4. 使用 spi_sync 函数完成 SPI 数据同步传输。
* demo
```C
/* SPI 多字节发送 */
static int spi_send(struct spi_device *spi, u8 *buf, int len)
{
    int ret;
    struct spi_message m;
    struct spi_transfer t = {
    .tx_buf = buf,
    .len = len,
    };
    spi_message_init(&m);           /* 初始化 spi_message */
    spi_message_add_tail(t, &m);    /* 将 spi_transfer 添加到 spi_message 队列 */
    ret = spi_sync(spi, &m);        /* 同步传输 */
    return ret;
}

/* SPI 多字节接收 */
static int spi_receive(struct spi_device *spi, u8 *buf, int len)
{
    int ret;
    struct spi_message m;
    struct spi_transfer t = {
    .rx_buf = buf,
    .len = len,
    };
    spi_message_init(&m);           /* 初始化 spi_message */
    spi_message_add_tail(t, &m);    /* 将 spi_transfer 添加到 spi_message 队列 */
    ret = spi_sync(spi, &m);        /* 同步传输 */
    return ret;
}
```


# SPI(icm20608) 与 I2C(ap3216c器件) 底层驱动发送、接收流程的区别
* 共同点
    1. 都是先在设备书.dts文件中写好设备节点
    2. 在驱动文件中，写好一个 操作结构体(自定义)，在该层中，利用demo的格式，自定义写好对应的读写寄存器函数（读写一个 或者 多个的API接口）
    3. 假如器件设备要需要初始化，则利用自定义写好的API函数，写好器件设备的初始化函数
    4. 然后把读取设备有用的数据 作为该层驱动的 文件操作集合的 读（.read）函数接口
    5. 或者 把有用的数据写入器件设备 作为该驱动层的 文件操作集合的 写（.write）函数接口
    6. 写好对应的 probe 和 release函数

* 写驱动时的差异




# SPI ICM20608器件踩的坑
1. 设备树中有设备节点共用了同一引脚，导致配置不成功
    * 解决方法，找出对应的引脚，注意查重，有相同引脚的设备节点，查看硬件原理图，把多余的屏蔽掉
2. spi 器件的读函数中，发送使用传输结构体的 tx_buf，接收是用 rx_buf
    ```C
    tx_buff[0] = reg | 0x80;
    p_spi_trans->tx_buf = tx_buff;
    p_spi_trans->len = 1;

    spi_message_init(&message_queue);
    spi_message_add_tail(p_spi_trans,&message_queue);

    err_code = spi_sync(p_spi_dev,&message_queue);
    if(err_code < 0)
    {
        trace_infoln("spi_sync fail");
    }


    /**
     * 第2次，读取数据
     */
    tx_buff[0] = 0xff;
    p_spi_trans->rx_buf = buf;          //这里是RX_BUFF,曾马虎踩坑了，害
    p_spi_trans->len = len;

    spi_message_init(&message_queue);
    spi_message_add_tail(p_spi_trans,&message_queue);

    err_code = spi_sync(p_spi_dev,&message_queue);
    ```

