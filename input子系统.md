# input 子系统

* 模型

    硬件设备 ==> 设备类型 ==> input 驱动层 ==> input 核心层 ==> input 事件处理层 ==> 设备访问节点

* 核心层文件 ： 内核源码/ drivers/input/input.c
    该文件是 向Linux内核注册一个字符设备，即后面的输入子设备都是挂载到该input设备下,所以他们有着相同的主设备号 `#define INPUT_MAJOR 13`

* 数据结构（input设备驱动）,路径： include/linux/input.h
    ```C
        struct input_dev {
                const char *name;
                const char *phys;
                const char *uniq;
                struct input_id id;
                unsigned long propbit[BITS_TO_LONGS(INPUT_PROP_CNT)];
                unsigned long evbit[BITS_TO_LONGS(EV_CNT)];         /* 输入事件类型的位图 */
                unsigned long keybit[BITS_TO_LONGS(KEY_CNT)];       /* 按键值的位图 */
                unsigned long relbit[BITS_TO_LONGS(REL_CNT)];       /* 相对坐标的位图 */
                unsigned long absbit[BITS_TO_LONGS(ABS_CNT)];       /* 绝对坐标的位图 */
                unsigned long mscbit[BITS_TO_LONGS(MSC_CNT)];       /* 杂项事件的位图 */
                unsigned long ledbit[BITS_TO_LONGS(LED_CNT)];       /*LED 相关的位图 */
                unsigned long sndbit[BITS_TO_LONGS(SND_CNT)];       /* sound 有关的位图 */
                unsigned long ffbit[BITS_TO_LONGS(FF_CNT)];         /* 压力反馈的位图 */
                unsigned long swbit[BITS_TO_LONGS(SW_CNT)];         /*开关状态的位图 */
                ...
                bool devres_managed;
            };
    ```

    evbit 表示输入事件类型，可选的事件类型定义在 include/uapi/linux/input.h 
    ```C
        #define EV_SYN 0x00 /* 同步事件 */
        #define EV_KEY 0x01 /* 按键事件 */
        #define EV_REL 0x02 /* 相对坐标事件 */
        #define EV_ABS 0x03 /* 绝对坐标事件 */
        #define EV_MSC 0x04 /* 杂项(其他)事件 */
        #define EV_SW 0x05 /* 开关事件 */
        #define EV_LED 0x11 /* LED */
        #define EV_SND 0x12 /* sound(声音) */
        #define EV_REP 0x14 /* 重复事件 */
        #define EV_FF 0x15 /* 压力事件 */
        #define EV_PWR 0x16 /* 电源事件 */
        #define EV_FF_STATUS 0x17 /* 压力状态事件 */
    ```

    keybit 是按键事件使用的位图，内核定义了按键键值
    ```C
        #define KEY_RESERVED 0
        #define KEY_ESC 1
        #define KEY_1 2
        #define KEY_2 3
        #define KEY_3 4
        #define KEY_4 5
        #define KEY_5 6
        #define KEY_6 7
        #define KEY_7 8
        #define KEY_8 9
        #define KEY_9 10
        #define KEY_0 11
        ...
        #define BTN_TRIGGER_HAPPY39 0x2e6
        #define BTN_TRIGGER_HAPPY40 0x2e7
    ```

* API 创建注册函数
    1. 申请一个 input_dev 结构体变量
    ```C
        struct input_dev *input_allocate_device(void)

        返回值： 申请到的 input_dev
    ```

    2. 释放
    ```C
        void input_free_device(struct input_dev *dev)

        dev：需要释放的 input_dev。
        返回值： 无。
    ```

    3. 初始化
    ```C
        int input_register_device(struct input_dev *dev)

        dev：要注册的 input_dev 。
        返回值： 0， input_dev 注册成功；
                负值， input_dev 注册失败。
    ```

    4. 注销
    ```C
        void input_unregister_device(struct input_dev *dev)

        dev：要注销的 input_dev 。
        返回值： 无。
    ```

* 逻辑框架
    1. 使用 input_allocate_device 函数申请一个 input_dev
    2. 初始化 input_dev 的事件类型以及事件值
    3. 使用 input_register_device 函数向 Linux 系统注册前面初始化好的 input_dev
    4. 卸载 input 驱动，先使用 `void input_unregister_device(struct input_dev *dev)` 来注销原来注册的 input_dev，然后使用 `void input_free_device(struct input_dev *dev)`来释放申请的 input 结构体

    ```C demo 框架
        /**
        *  因为输入子设备系统本身就是内核的一个设备，所以在驱动文件里，无需再注册设备，无需 file_operations_t 设备操作结构体
        */

        struct input_dev *inputdev; /* input 结构体变量 */

        /* 驱动入口函数 */
        static int __init xxx_init(void)
        {
            ......
            inputdev = input_allocate_device(); /* 申请 input_dev */
            inputdev->name = "test_inputdev"; /* 设置 input_dev 名字 */

            /*********第一种设置事件和事件值的方法***********/
            __set_bit(EV_KEY, inputdev->evbit); /* 设置产生按键事件 */
            __set_bit(EV_REP, inputdev->evbit); /* 重复事件 */
            __set_bit(KEY_0, inputdev->keybit); /*设置产生哪些按键值 */

            /*********第二种设置事件和事件值的方法***********/
            keyinputdev.inputdev->evbit[0] = BIT_MASK(EV_KEY) | BIT_MASK(EV_REP);
            keyinputdev.inputdev->keybit[BIT_WORD(KEY_0)] |= BIT_MASK(KEY_0);

            /*********第三种设置事件和事件值的方法***********/
            keyinputdev.inputdev->evbit[0] = BIT_MASK(EV_KEY) | BIT_MASK(EV_REP);
            input_set_capability(keyinputdev.inputdev, EV_KEY, KEY_0);

            /************************************************/
            /* 注册 input_dev */
            input_register_device(inputdev);
            ......
            return 0;
        }

        /* 驱动出口函数 */
        static void __exit xxx_exit(void)
        {
            input_unregister_device(inputdev); /* 注销 input_dev */
            input_free_device(inputdev); /* 删除 input_dev */
        }
    ```

* API 上报函数
    1. 上报事件
    ```C
        void input_event(   struct input_dev *dev,
                            unsigned int type,
                            unsigned int code,
                            int value);

        dev：   需要上报的 input_dev。
        type:   上报的事件类型，比如 EV_KEY。
        code：  事件码，也就是我们注册的按键值，比如 KEY_0、 KEY_1 等等。
        value： 事件值，比如 1 表示按键按下， 0 表示按键松开。
        返回值： 无
    ```

    2. 上报同步事件
    ```C
        void input_sync(struct input_dev *dev)

        dev：需要上报同步事件的 input_dev。
        返回值： 无。
    ```

    * 模板 demo
    ```C
        /* 用于按键消抖的定时器服务函数 */
        void timer_function(unsigned long arg)
        {
            unsigned char value;
            value = gpio_get_value(keydesc->gpio);          /* 读取 IO 值 */
            if(value == 0)
            { 
                /* 按下按键 */
                /* 上报按键值 */
                input_event(inputdev, EV_KEY, KEY_0, 1);   /* 最后一个参数 1， 按下 */
                input_sync(inputdev); /* 同步事件 */
            } 
            else 
            { 
                /* 按键松开 */
                input_event(inputdev, EV_KEY, KEY_0, 0);   /* 最后一个参数 0， 松开 */
                input_sync(inputdev); /* 同步事件 */
            }
        }
    ```

* 驱动函数写完之后，通过insmod挂载到系统之后，显示的设备路径是
    /dev/input/event1


* 事件数据结构,路径 include/uapi/linux/input.h (应用层app)
```C
    struct input_event  time：时间，也就是此事件发生的时间，为 timeval 结构体类型
    {
        struct timeval time;
        __u16 type;     //事件类型，比如 EV_KEY，表示此次事件为按键事件，此成员变量为 16 位
        __u16 code;     //事件码，比如在 EV_KEY 事件中 code 就表示具体的按键码，如： KEY_0、 KEY_1等等这些按键。此成员变量为 16 位
        __s32 value;    //值，比如 EV_KEY 事件中 value 就是按键值
    };
```
    所有的输入设备最终都是按照 input_event 结构体呈现给用户的，用户应用程序可以通过 input_event 来获取到具体的输入事件或相关的值，比如按键值等。

