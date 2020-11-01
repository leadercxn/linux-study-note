# I2C驱动框架
* 分层思想
    1. I2C 总线驱动，也叫I2C适配器驱动
    2. I2C 设备驱动

# I2C适配器驱动
* 数据结构 include/linux/i2c.h
    ```C
        struct i2c_adapter {
                                struct module *owner;
                                unsigned int class;                                     /* classes to allow probing for */
                                const struct i2c_algorithm   *algo;                     /* 总线访问算法  (重要)*/
                                void *algo_data;
                                /* data fields that are valid for all devices */
                                struct rt_mutex bus_lock;
                                int timeout;                                            /* in jiffies */
                                int retries;
                                struct device dev;                                      /* the adapter device */
                                int nr;
                                char name[48];
                                struct completion dev_released;
                                struct mutex userspace_clients_lock;
                                struct list_head userspace_clients;
                                struct i2c_bus_recovery_info *bus_recovery_info;
                                const struct i2c_adapter_quirks *quirks;
                            };
    ```

    * i2c_algorithm 结构体定义在 include/linux/i2c.h  用来定义接口
    ```C
        struct i2c_algorithm {
                                ...
                                int (*master_xfer)(struct i2c_adapter *adap,
                                                   struct i2c_msg *msgs,
                                                   int num);
                                int (*smbus_xfer) (struct i2c_adapter *adap, u16 addr,
                                                   unsigned short flags, char read_write,
                                                   u8 command, int size, union i2c_smbus_data *data);
                                /* To determine what the adapter supports */
                                u32 (*functionality) (struct i2c_adapter *);
                                ...
                            };
    ```

* API接口
    1. 向系统注册设置好的  i2c_adapter
        + int i2c_add_adapter(struct i2c_adapter *adapter)         //使用动态的总线号
        + int i2c_add_numbered_adapter(struct i2c_adapter *adap)   //使用静态总线号
    2. 删除 I2C 适配器
        + void i2c_del_adapter(struct i2c_adapter * adap)

# I2C设备驱动
* 数据结构  include/linux/i2c.h
    ```C
        /**
        * 设备结构体
        */
        struct i2c_client {
                            unsigned short flags; /* 标志 */
                            unsigned short addr; /* 芯片地址， 7 位，存在低 7 位*/
                            ...
                            char name[I2C_NAME_SIZE]; /* 名字 */
                            struct i2c_adapter *adapter; /* 对应的 I2C 适配器 */
                            struct device dev; /* 设备结构体 */
                            int irq; /* 中断 */
                            struct list_head detected;
                            ...
                        };
    ```

    ```C
        /**
        * 接口回调结构体
        */
        struct i2c_driver {
                        unsigned int class;

                        /* Notifies the driver that a new bus has appeared. You should avoid using this, it will be removed in a near future.*/
                        int (*attach_adapter)(struct i2c_adapter *) __deprecated;

                        /* Standard driver model interfaces */
                        int (*probe)(struct i2c_client *, const struct i2c_device_id *); //当 I2C 设备和驱动匹配成功以后 probe 函数就会执行

                        int (*remove)(struct i2c_client *);

                        /* driver model interfaces that don't relate to enumeration */
                        void (*shutdown)(struct i2c_client *);

                        /* Alert callback, for example for the SMBus alert protocol.The format and meaning of the data value depends on the
                        * protocol.For the SMBus alert protocol, there is a single bit of data passed as the alert response's low bit("event flag"). */
                        void (*alert)(struct i2c_client *, unsigned int data);

                        /* a ioctl like command that can be used to perform specific functions with the device.*/
                        int (*command)(struct i2c_client *client, unsigned int cmd, void *arg);

                        struct device_driver driver;

                        const struct i2c_device_id *id_table;

                        /* Device detection callback for automatic device creation */
                        int (*detect)(struct i2c_client *, struct i2c_board_info *);

                        const unsigned short *address_list;

                        struct list_head clients;
                    }
    ```
* API
    1. 向linux内核注册 i2c_driver 
        + int i2c_register_driver(struct module *owner, struct i2c_driver *driver)
            ```C
                owner： 一般为 THIS_MODULE。
                driver：要注册的 i2c_driver。
                返回值： 0，成功；负值，失败
            ```
        + 简单的宏 封装
            ```C
                #define i2c_add_driver(driver) \
                        i2c_register_driver(THIS_MODULE, driver)
            ```
    2. 注销 i2c_driver
        + void i2c_del_driver(struct i2c_driver *driver)

# I2C设备数据收发的处理流程
* API 
    * 读写函数（基本的接口）
        int i2c_transfer(struct i2c_adapter *adap,struct i2c_msg *msgs,int num)
        ```C
            adap： 所使用的 I2C 适配器， i2c_client 会保存其对应的 i2c_adapter。
            msgs： I2C 要发送的一个或多个消息。
            num： 消息数量，也就是 msgs 的数量。
            返回值： 负值，失败，其他非负值，发送的 msgs 数量。
        ```
    * 发送,其实质还是调用了`i2c_transfer`函数
        ```C
            int i2c_master_send(const struct i2c_client *client, const char *buf, int count);

            client： I2C 设备对应的 i2c_client。
            buf：    要发送的数据。
            count：  要发送的数据字节数，要小于 64KB，以为 i2c_msg 的 len 成员变量是一个 u16(无符号 16 位)类型的数据。
            返回值： 负值，失败，其他非负值，发送的字节数。
        ```
    * 接收，其实质还是调用了`i2c_transfer`函数
        ```C
            int i2c_master_recv(const struct i2c_client *client,char *buf,int count)

            client： I2C 设备对应的 i2c_client。
            buf：要接收的数据。
            count： 要接收的数据字节数，要小于 64KB，以为 i2c_msg 的 len 成员变量是一个 u16(无符号 16 位)类型的数据。
            返回值： 负值，失败，其他非负值，发送的字节数。
        ```

* 数据结构 -- 消息结构体  路径：include/uapi/linux/i2c.h
    ```C
        struct i2c_msg {
                            __u16 addr;	/* slave address			*/
                            __u16 flags;
                        #define I2C_M_TEN		0x0010	/* this is a ten bit chip address */
                        #define I2C_M_RD		0x0001	/* read data, from slave to master */
                        #define I2C_M_STOP		0x8000	/* if I2C_FUNC_PROTOCOL_MANGLING */
                        #define I2C_M_NOSTART		0x4000	/* if I2C_FUNC_NOSTART */
                        #define I2C_M_REV_DIR_ADDR	0x2000	/* if I2C_FUNC_PROTOCOL_MANGLING */
                        #define I2C_M_IGNORE_NAK	0x1000	/* if I2C_FUNC_PROTOCOL_MANGLING */
                        #define I2C_M_NO_RD_ACK		0x0800	/* if I2C_FUNC_PROTOCOL_MANGLING */
                        #define I2C_M_RECV_LEN		0x0400	/* length will be first received byte */
                            __u16 len;		/* msg length				*/
                            __u8 *buf;		/* pointer to msg data			*/
                        };
    ```

# demo
    ```C
        /* i2c 驱动的 probe 函数 
        *  I2C设备 和 I2C驱动 匹配以后，probe函数就会执行
        */
        static int xxx_probe(struct i2c_client *client,
        {
            /* 函数具体程序 */
            return 0;
        }

        /* i2c 驱动的 remove 函数 */
        static int ap3216c_remove(struct i2c_client *client)
        {
            /* 函数具体程序 */
            return 0;
        }

        /* 传统匹配方式 ID 列表 */
        static const struct i2c_device_id xxx_id[] = {
            {"xxx", 0},
            {}
        };

        /* 设备树匹配列表 */
        static const struct of_device_id xxx_of_match[] = {
            { .compatible = "xxx" },
            { /* Sentinel */ }
        };

        /* i2c 驱动结构体 */
        static struct i2c_driver xxx_driver = {
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
            int ret = 0;
            ret = i2c_add_driver(&xxx_driver);
            return ret;
        }

        /* 驱动出口函数 */
        static void __exit xxx_exit(void)
        {
            i2c_del_driver(&xxx_driver);
        }
        module_init(xxx_init);
        module_exit(xxx_exit);
    ```


# 编写具体的设备驱动
* 针对使用设备树的时候
    1. 在设备树文件下的i2c节点下添加对应的 器件设备的相关信息
    ```demo
        &i2c1 {
                    clock-frequency = <100000>;
                    pinctrl-names = "default";
                    pinctrl-0 = <&pinctrl_i2c1>;
                    status = "okay";

                    mag3110@0e {
                        compatible = "fsl,mag3110";
                        reg = <0x0e>;
                        position = <2>;
                    };

                    fxls8471@1e {
                        compatible = "fsl,fxls8471";
                        reg = <0x1e>;
                        position = <0>;
                        interrupt-parent = <&gpio5>;
                        interrupts = <0 8>;
                    };
                };
    ```
    2. 构建好 i2c_msg
    3. 发送，或者读取，调用API接口