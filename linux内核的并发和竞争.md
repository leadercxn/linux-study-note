# 自旋锁 
* 自旋锁的“自旋”也就是“原地打转”的意思，“原地打转”的目的是为了等待自旋锁可以用，可以访问共享资源。
    * 个人理解：类似于实时系统中的互斥量，只有 0 和 1 ，且阻塞 。 
    * "自旋"状态的理解：线程自旋是需要消耗cpu的，说白了就是让cpu在做无用功，如果一直获取不到锁，那线程也不能一直占用cpu自旋做无用功，所以需要设定一个自旋等待的最大时间。

* 数据结构
    * 路径：kernel_code/include/linux/spinlock_types.h
    * 
        ```
            typedef struct spinlock {
                                        union {
                                            struct raw_spinlock rlock;

                                        #ifdef CONFIG_DEBUG_LOCK_ALLOC
                                        # define LOCK_PADSIZE (offsetof(struct raw_spinlock, dep_map))
                                                struct {
                                                    u8 __padding[LOCK_PADSIZE];
                                                    struct lockdep_map dep_map;
                                                };
                                        #endif
                                            };
                                    } spinlock_t;
        ```

* 自旋锁API函数
    1. `DEFINE_SPINLOCK(spinlock_t lock)`         定义并初始化一个自旋变量
    2. `int spin_lock_init(spinlock_t *lock) `    初始化自旋锁
    3. `void spin_lock(spinlock_t *lock) `        获取指定的自旋锁，也叫加锁（获取信号量）
    4. `void spin_unlock(spinlock_t *lock) `      释放自旋锁 （释放信号量）
    5. `int spin_trylock(spinlock_t *lock) `      尝试获取某个自旋锁 ，如果获取不了，就返回0
    6. `int spin_is_locked(spinlock_t *lock) `    检查指定的自旋锁是否被获取，如果没有被获取就返回非 0，否则返回 

    7. `void spin_lock_irq(spinlock_t *lock)`       禁止本地中断，并获取自旋锁。    //不建议使用
    8. `void spin_unlock_irq(spinlock_t *lock)`     激活本地中断，并释放自旋锁      //不建议使用
    9.  `void spin_lock_irqsave(spinlock_t *lock,unsigned long flags)`        保存中断状态，禁止本地中断，并获取自旋锁
    10. `void spin_unlock_irqrestore(spinlock_t*lock, unsigned long flags)`   将中断状态恢复到以前的状态，并且激活本地中断，释放自旋锁。

* 使用注意
    1. 不能递归申请自旋锁，因为一旦通过递归的方式申请一个你正在持有的锁，那么你就必须“自旋”，等待锁被释放，然而你正处于“自旋”状态，根本没法释放锁。结果就是自己把自己锁死了！
    2. 自旋锁保护的临界区内不能调用任何可能导致线程休眠的 API 函数，否则的话可能导致死锁。

# 信号量
* 与一般的RTOS一样，用于同步。
* 数据结构
    * 路径：kernel_code/include/linux/semaphore.h
    * 
        ```
            struct semaphore {
                                raw_spinlock_t		lock;
                                unsigned int		count;
                                struct list_head	wait_list;
                            };
        ```
* API函数
    1. `DEFINE_SEAMPHORE(name)`                             定义一个信号量，并且设置信号量的值为 1。
    2. `void sema_init(struct semaphore *sem, int val)`     初始化信号量 sem，设置信号量值为 val。
    3. `void down(struct semaphore *sem)`                   获取信号量，因为会导致休眠，因此不能在中断中使用。
    4. `int down_trylock(struct semaphore *sem);`           尝试获取信号量，如果能获取到信号量就获取，并且返回 0。如果不能就返回非 0，并且不会进入休眠。
    5. `int down_interruptible(struct semaphore *sem)`      获取信号量，和 down 类似，只是使用 down 进入休眠状态的线程不能被信号打断。而使用此函数进入休眠以后是可以被信号打断的。
    6. `void up(struct semaphore *sem)`                     释放信号量


# 互斥体
* 与一般的RTOS一样，只有0和1，资源区只能被一个线程访问
    * 路径：kernel_code/include/linux/mutex.h
    * 
        ```
            struct mutex {
                            /* 1: unlocked, 0: locked, negative: locked, possible waiters */
                                atomic_t		count;
                                spinlock_t		wait_lock;
                                struct list_head	wait_list;
                            #if defined(CONFIG_DEBUG_MUTEXES) || defined(CONFIG_MUTEX_SPIN_ON_OWNER)
                                struct task_struct	*owner;
                            #endif
                            #ifdef CONFIG_MUTEX_SPIN_ON_OWNER
                                struct optimistic_spin_queue osq; /* Spinner MCS lock */
                            #endif
                            #ifdef CONFIG_DEBUG_MUTEXES
                                void			*magic;
                            #endif
                            #ifdef CONFIG_DEBUG_LOCK_ALLOC
                                struct lockdep_map	dep_map;
                            #endif
                        };
        ```
* API函数
    1. `DEFINE_MUTEX(name)`             定义并初始化一个 mutex 变量。
    2. `void mutex_init(mutex *lock)`   初始化 mutex。
    3. `void mutex_lock(struct mutex *lock)`    获取 mutex，也就是给 mutex 上锁。如果获取不到就进休眠。
    4. `void mutex_unlock(struct mutex *lock)`  释放 mutex，也就给 mutex 解锁。
    5. `int mutex_trylock(struct mutex *lock)`  尝试获取 mutex，如果成功就返回 1，如果失败就返回 0。
    6. `int mutex_is_locked(struct mutex *lock)`判断 mutex 是否被获取，如果是的话就返回1，否则返回 0。
    7. `int mutex_lock_interruptible(struct mutex *lock)` 使用此函数获取信号量失败进入休眠以后可以被信号打断。

