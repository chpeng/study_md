## IO

用户进程读取外部数据时，操作系统内核，首先会将数据缓存到内核缓冲中，然后用户进程通过调用系统函数，从内核缓冲区copy的用户进程空间

根据用户进程在读写数据时，是否等待，分为 同步IO 和异步IO

### 1. 同步IO

#### 阻塞IO

用户进程在读取数据时，直到数据从准备好后到内核缓存中在Copy到用户进程中，用户进程都处于阻塞状态

![clipboard.png](IO.assets/1593755892-55c466c2b5fc5_articlex.png)

#### 非阻塞IO

用户进程在调用系统函数读取数据时，当数据没有准备好（数据还没有全部到内核缓冲中），内核会直接返回一个错误（error）,用户进程通过判断这个错误，来确定数据是否准备好，如果如果没有准备好，就一直check,直到数据准备好后，用户进程调用系统函数，数据从内核缓存中Copy到用户进程中，这时候会阻塞

![clipboard.png](IO.assets/1505222224-55c466dda9803_articlex.png)

#### 多路复用IO（select  poll epoll）

这模型和阻塞IO模型差不多，用户进程会阻塞在 select() poll() epoll()系统函数上，不同的是
单个进程会同时处理多个用户连接，基本原理是就是，这三个是函数会不断的轮询所负责的文件描述符，知道有某个文件描述符上有数据到达事件，并通知用户进程处理。

![clipboard.png](IO.assets/1903235121-55c466eb17665_articlex.png)

#### 多路复用函数
##### select

```c
int select (int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

1. 每次轮询都需要将文件描述符集合（fd_set）从用户进程空间copy到内核缓存中
2. 每次轮询都需要遍历全部的文件描述符集合（fd_set）
3. 文件描述符有数量限制

##### poll

```c
int poll (struct pollfd *fds, unsigned int nfds, int timeout);

struct pollfd {
    int fd; /* file descriptor */
    short events; /* requested events to watch */
    short revents; /* returned events witnessed */
};
```

1. 每次轮询都需要将文件描述符集合（pollfd）从用户进程空间copy到内核缓存中,如果数量多的换，效率低
2. 每次轮询都需要遍历全部的文件描述符集合（pollfd）如果数量多的话，效率低
3. 文件描述符没有限制

##### epoll

```c#
int epoll_create(int size)；//创建一个epoll的句柄，size用来告诉内核这个监听的数目一共有多大
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
```

epoll 是 select/poll 的升级版，解决了这两个的缺点

1. 通过内存映射，可以是用户进程和内核使用同一块内存，将文件描述符集合放到这块内存中后，只需要从用户控件到内核缓存copy一次,解决了每次轮询都需要copy的缺点
2. epoll是基于事件的，当有事件发生时，通过回调函数，将文件描述符放到**就绪列表**中，轮询的时候，只需要轮询**就绪列表**中的fd，所以轮询的fd都是有事件发生的，也就是说，每次轮询的都是有效的io,解决了不用每次都需要从都从头遍历的缺点

### 2. 异步IO

用户进程在读取数据时，调用 内核系统函数，内核会直接返回，**系统会在数据准备好后，直接copy到用户进程空间中中，不用用户进程主动的调用系统函数，来从内核缓存区Copy到用户进程空间**，这样用户进程就不需要在这个系统函数上等待，所以是异步IO

![clipboard.png](IO.assets/1311869885-55c466fac00ba_articlex.png)

