# Platform 平台驱动（重点）

## 模型
* 驱动 <== 总线 ==> 设备

| 驱动1 |                       | 设备1 |          
| 驱动2 |                       | 设备2 |
| 驱动3 | <====== |总线| ======> | 设备3 |
| 驱动4 |                       | 设备4 |
| 驱动5 |                       | 设备5 |

## 总线数据结构 
* bus_type 表示(没说是platform)总线  路径:include/linux/device.h
```C
    struct bus_type 
    {
        const char *name;                                               /* 总线名字 */
        const char *dev_name;
        struct device *dev_root;
        struct device_attribute *dev_attrs;
        const struct attribute_group **bus_groups;                      /* 总线属性 */
        const struct attribute_group **dev_groups;                      /* 设备属性 */
        const struct attribute_group **drv_groups;                      /* 驱动属性 */
        int (*match)(struct device *dev, struct device_driver *drv);    /* 匹配，相配; 完成设备和驱动之间匹配的函数 */
        int (*uevent)(struct device *dev, struct kobj_uevent_env *env);
        int (*probe)(struct device *dev);
        int (*remove)(struct device *dev);
        void (*shutdown)(struct device *dev);
        int (*online)(struct device *dev);
        int (*offline)(struct device *dev);
        int (*suspend)(struct device *dev, pm_message_t state);
        int (*resume)(struct device *dev);
        const struct dev_pm_ops *pm;
        const struct iommu_ops *iommu_ops;
        struct subsys_private *p;
        struct lock_class_key lock_key;
    };
```

## platform总线 (是总线的一个实例)
* 定义  路径:  drivers/base/platform.c
    ```C
        struct bus_type platform_bus_type = 
        {
            .name = "platform",
            .dev_groups = platform_dev_groups,
            .match = platform_match,
            .uevent = platform_uevent,
            .pm = &platform_dev_pm_ops,
        };
    ```

    * 对应的匹配函数 .match 接口
    ```C
        static int platform_match(struct device *dev,struct device_driver *drv)
        {
            struct platform_device *pdev = to_platform_device(dev);
            struct platform_driver *pdrv = to_platform_driver(drv);

            /*When driver_override is set,only bind to the matching driver*/
            if (pdev->driver_override)
                return !strcmp(pdev->driver_override, drv->name);

            /* Attempt an OF style match first */
            if (of_driver_match_device(dev, drv))   // of类型匹配， 设备树采用匹配方式
                return 1;

            /* Then try ACPI style match */
            if (acpi_driver_match_device(dev, drv)) // ACPI匹配
                return 1;

            /* Then try to match against the id table */
            if (pdrv->id_table)
                return platform_match_id(pdrv->id_table, pdev) != NULL; //id_table 匹配，每个 platform_driver 结构体有一个 id_table成员变量，顾名思义，保存了很多 id 信息。这些 id 信息存放着这个 platformd 驱动所支持的驱动类型
                
            /* fall-back to driver name match */
            return (strcmp(pdev->name, drv->name) == 0);    //第四种匹配方式，如果第三种匹配方式的 id_table 不存在的话就直接比较驱动和设备的 name 字段，看看是不是相等，如果相等的话就匹配成功 。 常用
        }
    ```

    * 着重解释:   设备树匹配的方式
    `of_driver_match_device()` device_driver 结构体(表示设备驱动)中有个名为 of_match_table的成员变量，此成员变量保存着驱动的 compatible匹配表，设备树中的每个设备节点的 compatible 属性会和 of_match_table 表中的所有成员比较，查看是否有相同的条目，如果有的话就表示设备和此驱动匹配，设备和驱动匹配成功以后 probe 函数就会执行


## platform 驱动
* platform驱动数据结构 include/linux/platform_device.h
    ```C
        struct platform_driver {
                                    int (*probe)(struct platform_device *);     //驱动与设备匹配成功以后 probe 函数就会执行，import
                                    int (*remove)(struct platform_device *);
                                    void (*shutdown)(struct platform_device *);
                                    int (*suspend)(struct platform_device *, pm_message_t state);
                                    int (*resume)(struct platform_device *);
                                    struct device_driver driver;                //device_driver 结构体变量
                                    const struct platform_device_id *id_table;  //platform总线 匹配驱动和设备 采用的方法
                                    bool prevent_deferred_probe;
                                };
    ```
    * 层层递进的结构体
        1. platform平台设备id数据结构体  struct platform_device_id 
        ```C
            struct platform_device_id {
                                        char name[PLATFORM_NAME_SIZE];
                                        kernel_ulong_t driver_data;
                                    };
        ```
        2. 设备驱动结构体(有点混淆) , 路径： include/linux/device.h
        ```C
            struct device_driver {
                                    const char *name;
                                    struct bus_type *bus;
                                    struct module *owner;
                                    const char *mod_name; /* used for built-in modules */
                                    bool suppress_bind_attrs; /* disables bind/unbind via sysfs */
                                    const struct of_device_id *of_match_table;      //采用设备树配对表
                                    const struct acpi_device_id *acpi_match_table;
                                    int (*probe) (struct device *dev);
                                    int (*remove) (struct device *dev);
                                    void (*shutdown) (struct device *dev);
                                    int (*suspend) (struct device *dev, pm_message_t state);
                                    int (*resume) (struct device *dev);
                                    const struct attribute_group **groups;
                                    const struct dev_pm_ops *pm;
                                    struct driver_private *p;
                                };
        ```
        * 其中 of_device_id 结构体类型
            ```C
                struct of_device_id {
                char name[32];
                char type[32];
                char compatible[128];   //import,设备树就是通过设备节点的 compatible 属性值和 of_match_table 中每个项目的 compatible 成员变量进行比较，如果有相等的就表示设备和此驱动匹配成功
                const void *data;
                };
            ```
* platform底层驱动流程
    1. 定义一个 platform_driver 结构体变量 ，实现结构体中的各个成员变量，重点是实现 匹配方法
    2. 编写 probe 函数 ，当 驱动 和 设备 匹配成功以后，probe函数就会执行
    3. 定义，初始化好 platform_driver 结构体变量后， 需在驱动入口函数里调用 platform_driver_register() 函数向linux内核注册一个platform驱动

* API接口
    注册函数
    ```C
        int platform_driver_register (struct platform_driver *driver)

        driver: 注册的 platform 驱动
        返回值： 负数 失败 ; 0 ，成功
    ```

    注销函数
    ```C
        void platform_driver_unregister(struct platform_driver *drv)

        drv：要卸载的 platform 驱动。
        返回值： 无。
    ```

* platform 驱动框架
    ```C
        /* 设备结构体 */
        struct xxx_dev{
            struct cdev cdev;
            /* 设备结构体其他具体内容 */
        };

        struct xxx_dev xxxdev; /* 定义个设备结构体变量 */

        static int xxx_open(struct inode *inode, struct file *filp)
        {
            /* 函数具体内容 */
            return 0;
        }

        static ssize_t xxx_write(struct file *filp, const char __user *buf, size_t cnt, loff_t *offt)
        {
            /* 函数具体内容 */
            return 0;
        }

        /**
        * 字符设备驱动操作集
        */
        static struct file_operations xxx_fops = {
            .owner = THIS_MODULE,
            .open = xxx_open,
            .write = xxx_write,
        };

        /*
        * platform 驱动的 probe 函数
        * 驱动与设备匹配成功以后此函数就会执行
        */
        static int xxx_probe(struct platform_device *dev)
        {
            ......
            cdev_init(&xxxdev.cdev, &xxx_fops); /* 注册字符设备驱动 */

            /* 函数具体内容 */

            /* 创建类 */

            return 0;
        }

        static int xxx_remove(struct platform_device *dev)
        {
            ......
            cdev_del(&xxxdev.cdev);/* 删除 cdev */
            /* 函数具体内容 */
            return 0;
        }

        /* 匹配列表 important*/
        static const struct of_device_id xxx_of_match[] = {
            { .compatible = "xxx-gpio" },   //该驱动的 compatible 特性 ，用来 匹配设备树中 具有相同 compatable 属性的设备
            { /* Sentinel */ }              //必须为空
        };

        /*
        * platform 平台驱动结构体
        */

        static struct platform_driver xxx_driver = 
        {
            .driver = {
                .name = "caonima",                  //用于传统的驱动与设备匹配（无设备树）
                .of_match_table = xxx_of_match, //用于设备树下的驱动与设备检查（有设备树）
            },
            .probe = xxx_probe,
            .remove = xxx_remove,
        };

        /* 驱动模块加载 */
        static int __init xxxdriver_init(void)
        {
            return platform_driver_register(&xxx_driver);
        }

        /* 驱动模块卸载 */
        static void __exit xxxdriver_exit(void)
        {
            platform_driver_unregister(&xxx_driver);
        }

        module_init(xxxdriver_init);
        module_exit(xxxdriver_exit);
    ```

    框架大概说明：
    xxx_probe 函数，当驱动和设备匹配成功以后此函数就会执行，以前在驱动入口 init 函数里面编写的字符设备驱动程序就全部放到此 probe 函数里面。比如注册字符设备驱动、添加 cdev、创建类等等。

    xxx_remove 函数， platform_driver 结构体中的 remove 成员变量，当关闭 platfor备驱动的时候此函数就会执行，以前在驱动卸载 exit 函数里面要做的事情就放到此函数中来。比如，使用 iounmap 释放内存、删除 cdev，注销设备号等等。


## platform 设备
1. 支持设备树的话，直接在设备树上添加
2. 折腾式操作： 用 platform_device 这个结构体表示 platform 设备

* 数据结构，路径 include/linux/platform_device.h
    ```C
        struct platform_device {
            const char *name;       //设备的名字（important , 要和 platform 驱动匹配的name 字段相同,如上述的name = "caonima"）
            int id;
            bool id_auto;
            struct device dev;
            u32 num_resources;
            struct resource *resource;      //资源数量
            const struct platform_device_id *id_entry;
            char *driver_override;          /* Driver name to force a match */

            /* MFD cell pointer */
            struct mfd_cell *mfd_cell;

            /* arch specific additions */
            struct pdev_archdata archdata;
        };
    ```
    * 层层递进的结构体数据
        1. 资源结构体
        ```C
            struct resource {
                    resource_size_t start;  //起始信息
                    resource_size_t end;    //终止信息
                    const char *name;       //资源名字
                    unsigned long flags;    //资源类型
                    struct resource *parent, *sibling, *child;
                };
        ```
            资源类型 (路径:  include/linux/ioport.h)
            ```C
                #define IORESOURCE_BITS 0x000000ff      /* Bus-specific bits */
                #define IORESOURCE_TYPE_BITS 0x00001f00 /* Resource type */
                #define IORESOURCE_IO 0x00000100        /* PCI/ISA I/O ports */
                #define IORESOURCE_MEM 0x00000200
                #define IORESOURCE_REG 0x00000300       /* Register offsets */
                #define IORESOURCE_IRQ 0x00000400
                #define IORESOURCE_DMA 0x00000800
                #define IORESOURCE_BUS 0x00001000
            ```


* API接口函数
    1. 注册函数 
    ```C
        int platform_device_register(struct platform_device *pdev)

        pdev：      要注册的 platform 设备。
        返回值：     负数，失败； 0，成功。
    ```

    2. 注销函数
    ```C
        void platform_device_unregister(struct platform_device *pdev)
    ```

* platform设备框架
    ```C
        /* 寄存器地址定义*/
        #define PERIPH1_REGISTER_BASE (0X20000000) /* 外设 1 寄存器首地址 */
        #define PERIPH2_REGISTER_BASE (0X020E0068) /* 外设 2 寄存器首地址 */
        #define REGISTER_LENGTH 4

        /* 资源 */
        static struct resource xxx_resources[] = {
            [0] = {
                .start = PERIPH1_REGISTER_BASE,
                .end = (PERIPH1_REGISTER_BASE + REGISTER_LENGTH - 1),
                .flags = IORESOURCE_MEM,    //内存类型
            },
            [1] = {
                .start = PERIPH2_REGISTER_BASE,
                .end = (PERIPH2_REGISTER_BASE + REGISTER_LENGTH - 1),
                .flags = IORESOURCE_MEM,
            },
        };

        /* platform 设备结构体 */
        static struct platform_device xxxdevice = {
            .name = "xxx-gpio",             //改名字要和驱动的名字一致
            .id = -1,
            .num_resources = ARRAY_SIZE(xxx_resources),
            .resource = xxx_resources,
        };

        /* 设备模块加载 */
        static int __init xxxdevice_init(void)
        {
            return platform_device_register(&xxxdevice);
        }

        /* 设备模块注销 */
        static void __exit xxx_resourcesdevice_exit(void)
        {
            platform_device_unregister(&xxxdevice);
        }

        module_init(xxxdevice_init);
        module_exit(xxxdevice_exit);
    ```

## 总结以上，以折腾式写 platform驱动 && platform设备，需要把驱动 和 设备分开两个文件编写，最后在应用层运行驱动文件，总线会根据驱动的名字 和 设备的名字进行匹配 。在驱动文件的 .probe 接口函数里，获取设备的资源，创建类什么的操作。



# 设备树下的 platform 驱动编写
* 操作的流程大概是
    1. 在设备树中创建设备节点
    2. 编写驱动代码
