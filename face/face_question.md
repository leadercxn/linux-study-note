1. nordic 有几层

2. GAP和GATT的区别
    * GATT
        低功耗蓝牙 BLE 的连接是建立在 GATT (Generic Attribute Profile) 协议之上的；
        GATT 是一个在蓝牙连接之上的发送和接收很短的数据段的通用规范，这些很短的数据段被称为属性（Attribute）
        GATT 它定义两个 BLE 设备通过 Service 和 Characteristic 进行通信
    * GAP
        GAP（Generic Access Profile），它在用来控制设备连接和广播；
        GAP 使你的设备被其他设备可见，并决定了你的设备是否可以或者怎样与合同设备进行交互；
        GAP 给设备定义了若干角色，其中主要的两个是：外围设备（Peripheral - 从机 - 服务端）和中心设备（Central - 主机 - 客户端）
3. 绑定和配对的区别
    * 配对
        1. 起初未提供安全性的两个设备如果希望做一些需要安全性的工作，就必须先配对。
        2. 配对涉及两个设备的身份认证，链路加密以及随后的密钥分配，身份解析密钥等。如果配对时设置了绑定位，分配的秘钥用户可以存储在flash中这样两个设备再第二次重连时的安全启动会更快。而不需要像第一次一样需要再启动整个配对过程。
        3. 配对有三个阶段： 
            + 第一阶段：配对信息交换（主要就是两边设备的i/o能力，设置绑定标志，链路是否需要MITM保护，如果设置绑定分配哪些密钥等信息。） 
            + 第二阶段：链路认证（以前的静态密码，动态密码，这个输入密码的过程就是认证的一种方式。） 
            + 第三阶段：密钥分配
    * 绑定
        1. 将密钥及相关身份信息保存到数据库中。如果设备不保存这些值，他们虽然能匹配，但不能绑定。只要当中某一个设备不保存，重新连接后，只有一个设备拥有LTK，因此加密的启动将会失败。
        2. 为了避免这种情况，两个设备在最初配对时就会交换绑定信息，从而能够清楚地知道对方是否保留了该绑定信息。如果对方设备不保存信息，那么一旦启动加密的尝试失败，主机将试图再次配对。
    * 总的来说，绑定会保存密钥到flash区，或者数据库。配对不会，配对要保存，得设置绑定位


4. 怎么保证线程的安全
    1. 对公共资源访问的时候，要加锁，信号量，保证数据安全
    2. 多线程并发情况下，线程共享的变量改为方法级的局部变量。
    3. 防止线程挂掉，需要管理线程和喂狗处理
    4. 多线程并发的，采用优先级
    5. 堆申请完，退出时，要释放

5. freertos 内存管理的集中方法，有什么区别
    五种,
    heap_1.c 使用大数组，可申请，没释放，系统使用创建消息量、定时器等也基本不需要释放；
    heap_2.c使用大数组，可申请，可释放，但是不会合并相邻空闲块，会产生碎片；
    heap_4.c使用大数组，可申请，可释放，相邻区块可以合并；
    heap_3.c使用malloc和free封装；
    heap_5.c允许使用多个内存堆块，可内部RAM可外部RAM，每个内存堆的起始地址和大小需用户定义；

6. 创建线程，怎么合理分配堆栈
    1. [使用freertos如何确定分配堆栈空间大小](https://blog.csdn.net/g200407331/article/details/108724032)
    2. [FreeRTOS ------ 栈、堆、任务栈](https://blog.csdn.net/dee53994040/article/details/102178824?utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control)

7. poll,epoll,select的区别
    1. select
        1. select目前几乎在所有的平台上支持，其良好跨平台支持也是它的一个优点。select的一 个缺点在于单个进程能够监视的文件描述符的数量存在最大限制，在Linux上一般为1024，可以通过修改宏定义甚至重新编译内核的方式提升这一限制，但 是这样也会造成效率的降低。
        2. select 函数监视的文件描述符分3类，分别是writefds、readfds、和exceptfds。调用后select函数会阻塞，直到有描述副就绪（有数据 可读、可写、或者有except），或者超时（timeout指定等待时间，如果立即返回设为null即可），函数返回。当select函数返回后，可以 通过遍历fdset，来找到就绪的描述符。
    2. poll
        1. pollfd结构包含了要监视的event和发生的event，不再使用select“参数-值”传递的方式。同时，pollfd并没有最大数量限制（但是数量过大后性能也是会下降）。 和select函数一样，poll返回后，需要轮询pollfd来获取就绪的描述符。
        2. 从上面看，select和poll都需要在返回后，通过遍历文件描述符来获取已经就绪的socket。事实上，同时连接的大量客户端在一时刻可能只有很少的处于就绪状态，因此随着监视的描述符数量的增长，其效率也会线性下降。
    3. epoll
        1. epoll_create,epoll_ctl和epoll_wait三个函数
        2. 监视的描述符数量不受限制，它所支持的FD上限是最大可以打开文件的数目，这个数字一般远大于2048
        3. IO的效率不会随着监视fd的数量的增长而下降。epoll不同于select和poll轮询的方式，而是通过每个fd定义的回调函数来实现的。只有就绪的fd才会执行回调函数。
        4. epoll是通过内核于用户空间mmap同一块内存，避免了无畏的内存拷贝

8. http的get，post区别
    * get 是获取数据
        1. GET传输数据的时候是在`URL地址中`的、对所有人都是是可见的、是不安全的、是有浏览器缓存记录的。所以说GET是不安全的，发送密码等数据的时候不要用GET传输。
        2. GET只能传输ASCLL字符，不能进行编码。
        3. GET请求的过程：也是先进性3次握手，然后服务器返回成功响应。
    * post 是提交数据
        1. POST传输的时候是放在HTTP的请求体之中的，并且是经过urlencode编码的所以是相对安全的。
        2. POST是没有对数据类型的限制的，二进制数据也是可以的。
        3. POST请求的过程：先进行3次握手，然后服务器返回100continue响应，浏览器再次发送数据，服务器返回200成功响应。


9. ota怎么跳转
    1. 跑完正常的bootload的代码后，关闭外设，中断，把主堆栈指针指向 app的程序空间的地址 ，然后跳转

10. 怎么快速的找到未知链表中的中间节点
    1. 方法一(普通方法):
        首先遍历一遍单链表，确定单链表长度L，然后再次从头结点出发，循环L/2次找到单链表的中间结点。
        ```C++
            /**
            * 获取单链表的中间节点(普通方法)
            */
            status getMidNode(LinkList L){
                int j = 0,len = 0;
                LinkList p = L;
                //循环单链表，确定单链表长度L
                while(p){
                    if(p->next == NULL){
                        break;
                    }
                    p = p->next;
                    len++;
                }
                //printf("len=%d",len);
                //循环L/2次找到单链表的中间结点
                p = L;
                while(p){
                    p = p->next;
                    j++;
                    if(j > (int)len/2){
                        break;
                    }
                }
                printf("j=%d, p->data=%d\n",j,p->data);
                return OK;
            }
        ```
    2. 方法二(快慢指针的方法):
        设置两个指针p、mid都指向单链表的头结点。其中p的移动速度是mid的2倍，当p指向末尾结点时，mid刚好就在中间了，这也是标尺的思想。
        ```C++
            /**
            * 获取单链表的中间节点（利用快慢指针）
            */
            status getMidNode(LinkList L){
                LinkList p = L, mid = L;
                while(p){
                    if(p->next == NULL){
                        break;
                    }
                    p = p->next->next;
                    mid = mid->next;
                }
                printf("mid->data=%d\n",mid->data);
                return OK;
            }
        ```


