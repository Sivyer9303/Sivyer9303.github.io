### 简介

IO多路复用：

IO多路复用是指内核一旦发现进程指定的一个或者多个IO条件准备读取，它就通知该进程。

Select、Poll、Epoll都是属于IO多路复用的实现方式。三者各有优劣。

#### 相关概念说明

内核空间和用户空间 - 操作系统的核心为内核，为了保证用户不能直接操作内核，操作系统将虚拟空间分为两个部分，内核空间和用户空间,内核空间又被称为   				Kenel。

文件描述符fd - 是一个用于表述指向文件的引用的抽象化概念。是一个整数，根本上来说是一个索引，用于快速找到文件。只在Unix、Linux里才有。

文件描述符集合fd_set - fd_set其实这是一个数组的宏定义，实际上是一long类型的数组，每一个数组元素都能与一个打开的文件句柄(socket、文件、管道、设备				等)建立联系，建立联系的工作由程序员完成，当调用select()时，由内核根据IO状态修改fd_set的内容，由此来通知执行了select()的进程哪个句柄可读。

缓存I/O - 缓存 I/O 又被称作标准 I/O，大多数文件系统的默认 I/O 操作都是缓存 I/O。在 Linux 的缓存 I/O 机制中，操作系统会将 I/O 的数据缓存在文件系统的页				缓存（ page cache ）中，也就是说，数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间。因为				数据传输过程中往往需要进行多次在内核空间和用户空间之间进行拷贝，因此性能占用会比较高。

#### Select

![Select图解](./select示意.png)

select 函数监视的文件描述符分3类，分别是writefds、readfds、和exceptfds。调用后select函数会阻塞，直到有描述符就绪（有数据 可读、可写、或者有except），或者超时（timeout指定等待时间，如果立即返回设为null即可），函数返回。当select函数返回后，可以通过遍历fdset，来找到就绪的描述符。

select的缺点很明显，由于单个进程允许监视的fd有限，通常是1024，导致select函数中的fdset大小只能是小于1024。

select的优点是跨平台，几乎所有平台都支持。

#### Poll

过程和select差不多，但是所监视的方式不一样，select中监视fdset，而Poll监视的是pollfd，pollfd中会包含需要监视的event（读写之类的）、发生的event。

相较于select，Poll可以监视的fd没有限制，但是效率会随着fd的增多而线性减低。

#### Epoll

是select和poll的增强版本，相比于select和poll，Epoll不再通过监视fdset，而是通过创建epollfd来管理多个fd。

Epoll分为三个接口，分别是

```
int epoll_create(int size)；//创建一个epoll的句柄，size用来告诉内核这个监听的数目一共有多大
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
```

**1. int epoll_create(int size);**
创建一个epoll的句柄，size用来告诉内核这个监听的数目一共有多大，这个参数不同于select()中的第一个参数，给出最大监听的fd+1的值，`参数size并不是限制了epoll所能监听的描述符最大个数，只是对内核初始分配内部数据结构的一个建议`。
当创建好epoll句柄后，它就会占用一个fd值，在linux下如果查看/proc/进程id/fd/，是能够看到这个fd的，所以在使用完epoll后，必须调用close()关闭，否则可能导致fd被耗尽。

**2. int epoll_ctl(int epfd, int op, int fd, struct epoll_event \*event)；**
函数是对指定描述符fd执行op操作。
\- epfd：是epoll_create()的返回值。
\- op：表示op操作，用三个宏来表示：添加EPOLL_CTL_ADD，删除EPOLL_CTL_DEL，修改EPOLL_CTL_MOD。分别添加、删除和修改对fd的监听事件。
\- fd：是需要监听的fd（文件描述符）
\- epoll_event：是告诉内核需要监听什么事，struct epoll_event结构如下：

```
struct epoll_event {
  __uint32_t events;  /* Epoll events */
  epoll_data_t data;  /* User data variable */
};

//events可以是以下几个宏的集合：
EPOLLIN ：表示对应的文件描述符可以读（包括对端SOCKET正常关闭）；
EPOLLOUT：表示对应的文件描述符可以写；
EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）；
EPOLLERR：表示对应的文件描述符发生错误；
EPOLLHUP：表示对应的文件描述符被挂断；
EPOLLET： 将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的。
EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里
```

**3. int epoll_wait(int epfd, struct epoll_event \* events, int maxevents, int timeout);**
等待epfd上的io事件，最多返回maxevents个事件。
参数events用来从内核得到事件的集合，maxevents告之内核这个events有多大，这个maxevents的值不能大于创建epoll_create()时的size，参数timeout是超时时间（毫秒，0会立即返回，-1将不确定，也有说法说是永久阻塞）。该函数返回需要处理的事件数目，如返回0表示已超时。



#### Epoll工作模式

Epoll分为两种工作模式，ET和LT。

**LT模式**：当epoll_wait检测到描述符事件发生并将此事件通知应用程序，`应用程序可以不立即处理该事件`。下次调用epoll_wait时，会再次响应应用程序并通知此事件。
**ET模式**：当epoll_wait检测到描述符事件发生并将此事件通知应用程序，`应用程序必须立即处理该事件`。如果不处理，下次调用epoll_wait时，不会再次响应应用程序并通知此事件。



#### 三者对比

1.select poll监听的fd有数量上限，Epoll没有

2.select poll性能会随着监听的fd数量上升而线性下降，Epoll不会。

3.实现的机制有区别，select poll是通过遍历fdset来找到对应的fd，而Epoll是通过回调来触发。

### 参考资料

[Linux IO模式及 select、poll、epoll详解](https://segmentfault.com/a/1190000003063859)