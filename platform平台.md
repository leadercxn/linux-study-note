# Platform 平台驱动（重点）

## 模型
* 驱动 <== 总线 ==> 设备

| 驱动1 |                       | 设备1 |          
| 驱动2 |                       | 设备2 |
| 驱动3 | <====== |总线| ======> | 设备3 |
| 驱动4 |                       | 设备4 |
| 驱动5 |                       | 设备5 |

## 数据结构
* 路径:  include/linux/device.h

* bus_type
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

## platform 总线
* 路径:  drivers/base/platform.c
* 定义
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
    对应的匹配函数 platform_dev_groups()
    ```C
        static int platform_match(struct device *dev,struct device_driver *drv)
        {
            struct platform_device *pdev = to_platform_device(dev);
            struct platform_driver *pdrv = to_platform_driver(drv);

            /*When driver_override is set,only bind to the matching driver*/
            if (pdev->driver_override)
                return !strcmp(pdev->driver_override, drv->name);

            /* Attempt an OF style match first */
            if (of_driver_match_device(dev, drv))
                return 1;

            /* Then try ACPI style match */
            if (acpi_driver_match_device(dev, drv))
                return 1;

            /* Then try to match against the id table */
            if (pdrv->id_table)
                return platform_match_id(pdrv->id_table, pdev) != NULL;
                
            /* fall-back to driver name match */
            return (strcmp(pdev->name, drv->name) == 0);
        }
    ```








