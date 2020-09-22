# MISC驱动（字符设备）

* 设备号
    所有misc驱动的主设备号都是10,子设备号（linux内核枚举） 路径：include/linux/miscdevice.h
    ```C
        #define PSMOUSE_MINOR 1
        #define MS_BUSMOUSE_MINOR 2 /* unused */
        #define ATIXL_BUSMOUSE_MINOR 3 /* unused */
        /*#define AMIGAMOUSE_MINOR 4 FIXME OBSOLETE */
        #define ATARIMOUSE_MINOR 5 /* unused */
        #define SUN_MOUSE_MINOR 6 /* unused */
        ....
        #define MISC_DYNAMIC_MINOR 255
    ```

* 数据结构     路径：include/linux/miscdevice.h
    ```C
        struct miscdevice {
                int minor;                          /* 子设备号 */
                const char *name;                   /* 设备名字 ,挂载到系统 /dev 目录下的名字 */
                const struct file_operations *fops; /* 设备操作集 */
                struct list_head list;
                struct device *parent;
                struct device *this_device;
                const struct attribute_group **groups;
                const char *nodename;
                umode_t mode;
            };
    ```

* API函数
    1. 注册函数
    ```C
        int misc_register(struct miscdevice * misc)

        misc：要注册的 MISC 设备。
        返回值： 负数，失败； 0，成功。 
    ```
    该函数相当于过去传统一套创建设备驱动的流程
        ```C
            alloc_chrdev_region();  /* 申请设备号 */
            cdev_init();            /* 初始化 cdev */
            cdev_add();             /* 添加 cdev */
            class_create();         /* 创建类 */
            device_create();        /* 创建设备 */
        ```


    2. 注销函数
    ```C
        int misc_deregister(struct miscdevice *misc)

        misc：要注销的 MISC 设备。
        返回值： 负数，失败； 0，成功。
    ```
    该函数相当于过去传统一套注销设备驱动的流程
        ```C
            cdev_del();                 /* 删除 cdev */
            unregister_chrdev_region(); /* 注销设备号 */
            device_destroy();           /* 删除设备 */
            class_destroy();            /* 删除类 */
        ```












