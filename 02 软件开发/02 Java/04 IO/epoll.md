## Epoll

### Epoll的操作

```script
int epoll_create(int size);  
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);  
int epoll_wait(int epfd, struct epoll_event *events,int maxevents, int timeout);  
```

1. epoll_create：建立一个epoll对象（在epoll文件系统中给这个句柄分配资源）
2. epoll_ctl：向epoll对象中添加socket，包含该socket的节点会被加入到一个**红黑树**上
3. epoll_wat：收集发生事件的连接，触发了事件的连接会被放入一个**双向链表**

### 工作原理

1. 红黑树

   epoll_ctl在向epoll对象中添加、修改、删除事件时，从RBT红黑树中查找事件也非常快

2. 双向链表

   所有添加到epoll中的事件都会与设备（网卡）驱动程序建立回调关系，也就是说相应事件的发生会调用这里的回调方法。这个回调方法在内核中叫做ep_poll_callback，它会把这样的事件放到上面的rdlist双向链表中

### 两种触发模式

#### EPOLLLT（水平触发）

​		默认模式

​		当被监控的文件描述符上有可读写事件发生时，epoll_wait()会通知处理程序去读写。如果这次没有把数据一次性全部读写完(如读写缓冲区太小)，那么下次调用 epoll_wait()时，它还会通知你在上次没读写完的文件描述符上继续读写

#### EPOLLET（边缘触发）

​		当被监控的文件描述符上有可读写事件发生时，epoll_wait()会通知处理程序去读写。如果这次没有把数据全部读写完(如读写缓冲区太小)，那么下次调用epoll_wait()时，它不会通知你，也就是它只会通知你一次，直到该文件描述符上出现第二次可读写事件才会通知你

### epoll与select poll的比较

1. select poll是采用轮询方式, epoll采用回调的方式
2. select poll每次等待socket事件，都需要**把所有socket从用户态拷贝至内核态**，epoll则只需将socket**添加一次到红黑树**上即可.
3. select用数组来存放socket, 因此受到数量限制. poll用链表存放, epoll用红黑树存放, 因此不受数量限制.
4. select、poll、epoll虽然都会返回就绪的文件描述符数量。**但是select和poll并不会明确指出是哪些文件描述符就绪，而epoll会**。造成的区别就是，系统调用返回后，调用select和poll的程序需要遍历监听的整个文件描述符找到是谁处于就绪，而epoll则直接处理即可。