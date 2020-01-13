## nio 入门
### Zero-Copy linux操作系统 文件拷贝的演进
>场景： 将静态内容（类似图片、文件）展示给用户

>分析：需要先将静态内容从磁盘中拷贝出来放到一个内存buf中，然后将这个buf通过socket传输给用户，进而用户或者静态内容的展示

早期文件拷贝流程：

	read(file, tmp_buf, len);
	write(socket, tmp_buf, len);

1. 调用read时，文件拷贝到了kernel模式
2. CPU控制将kernel模式数据copy到user模式下
3. 调用write时，先将user模式下的内容copy到kernel模式下的socket的buffer中
4. 最后将kernel模式下的socket buffer的数据copy到网卡设备中传送

此过程文件拷贝了4次，上下文切换了4次

> Zero-Copy的技术去掉这些无谓的copy。应用程序用Zero-Copy来请求kernel直接把disk的data传输给socket，而不是通过应用程序传输。Zero-Copy大大提高了应用程序的性能，并且减少了kernel和user模式上下文的切换

Java NIO中的FileChannal.transferTo()方法就是这样的实现，这个实现是依赖于操作系统底层的sendFile()实现的

#### Zero-Copy演进
版本1
Linux 2.1内核开始引入了sendfile函数,用于将文件通过socket传送

1. （通过DMA）将数据从磁盘读取到kernel buffer中
2. 将kernel buffer拷贝到socket buffer中
3. 将socket buffer中的数据copy到网卡设备（protocol engine）中发送

此过程文件拷贝了3次，上下文切换了2次

版本2
Linux2.4 内核对sendfile做了改进

1. 将文件拷贝到kernel buffer中
2. 向socket buffer中追加当前要发生的数据在kernel buffer中的位置和偏移量
3. 根据socket buffer中的位置和偏移量直接将kernel buffer的数据copy到网卡设备（protocol engine）中

此过程文件拷贝了2次，上下文切换了2次，
kernel空间内部未发生拷贝（Zero-Copy）

> Zero-Copy技术的使用场景有很多，比如Kafka, 又或者是Netty等，可以大大提升程序的性能


***

### NIO理论
#### 用户空间&内核空间
>我们知道现在操作系统都是采用虚拟存储器，那么对32位操作系统而言，它的寻址空间（虚拟存储空间）为4G（2的32次方）。操心系统的核心是内核，独立于普通的应用程序，可以访问受保护的内存空间，也有访问底层硬件设备的所有权限。为了保证用户进程不能直接操作内核，保证内核的安全，操心系统将虚拟空间划分为两部分，一部分为内核空间，一部分为用户空间。针对linux操作系统而言，将最高的1G字节（从虚拟地址0xC0000000到0xFFFFFFFF），供内核使用，称为内核空间，而将较低的3G字节（从虚拟地址0x00000000到0xBFFFFFFF），供各个进程使用，称为用户空间。每个进程可以通过系统调用进入内核，因此，Linux内核由系统内的所有进程共享。于是，从具体进程的角度来看，每个进程可以拥有4G字节的虚拟空间。

整个linux内部结构可以分为三部分，从最底层到最上层依次是：硬件-->内核空间-->用户空间

#### Linux 网络 I/O模型
> 为了OS的安全性等的考虑，进程是无法直接操作I/O设备的，其必须通过系统调用请求内核来协助完成I/O动作，而内核会为每个I/O设备维护一个buffer

一个io的请求过程：

1. 用户进程发起请求，
2. 内核接受到请求后，从I/O设备中获取数据到buffer中，再将buffer中的数据copy到用户进程的地址空间，
3. 该用户进程获取到数据后再响应客户端。

##### I/O模式
数据输入至buffer需要时间，而从buffer复制数据至进程也需要时间。
1. 等待数据准备
2. 将数据从内核拷贝到进程中	

根据这两段时间内等待方式的不同，I/O操作可以分为**五种模式**

1. 阻塞I/O (Blocking I/O)
2. 非阻塞I/O (Non-Blocking I/O)
3. I/O复用（I/O Multiplexing)
4. 信号驱动的I/O (Signal Driven I/O)
5. 异步I/O (Asynchrnous I/O) 

###### 阻塞I/O (Blocking I/O)
> 在linux中，默认情况下所有的socket都是blocking

>当用户进程调用了recvfrom这个系统调用，内核就开始了IO的第一个阶段：等待数据准备。对于network io来说，很多时候数据在一开始还没有到达（比如，还没有收到一个完整的UDP包），这个时候内核就要等待足够的数据到来。而在用户进程这边，整个进程会被阻塞。当内核一直等到数据准备好了，它就会将数据从内核中拷贝到用户内存，然后内核返回结果，用户进程才解除block的状态，重新运行起来

在IO执行的两个阶段都被block

###### 非阻塞I/O (Non-Blocking I/O)
>当用户进程调用recvfrom时，系统不会阻塞用户进程，而是立刻返回一个ewouldblock错误，从用户进程角度讲 ，并不需要等待，而是马上就得到了一个结果。用户进程判断标志是ewouldblock时，就知道数据还没准备好，于是它就可以去做其他的事了，于是它可以再次发送recvfrom，一旦内核中的数据准备好了。并且又再次收到了用户进程的system call，那么它马上就将数据拷贝到了用户内存，然后返回

该种模式，需要轮询内核检查数据是否准备好，通常是比较浪费CPU时间

在IO执行的第二个阶段阻塞

###### I/O复用（I/O Multiplexing)
也称作event driven IO

好处：select/epoll的好处就在于单个process就可以同时处理多个网络连接的IO

基本原理：select/epoll这个function会不断的轮询所负责的所有socket，当某个socket有数据到达了，就通知用户进程

>当用户进程调用了select，那么整个进程会被block，而同时，内核会“监视”所有select负责的socket，当任何一个socket中的数据准备好了，select就会返回。这个时候用户进程再调用read操作，将数据从内核拷贝到用户进程.

>在IO multiplexing Model中，实际中，对于每一个socket，一般都设置成为non-blocking，但是，如上图所示，整个用户的process其实是一直被block的。只不过process是被select这个函数block，而不是被socket IO给block

####### 文件描叙符fd
Linux的内核将所有外部设备都可以看做一个文件来操作。那么我们对与外部设备的操作都可以看做对文件进行操作。我们对一个文件的读写，都通过调用内核提供的系统调用；内核给我们返回一个filedescriptor（fd,文件描述符）。而对一个socket的读写也会有相应的描述符，称为socketfd(socket描述符）。描述符就是一个数字，指向内核中一个结构体（文件路径，数据区，等一些属性）。那么我们的应用程序对文件的读写就通过对描述符的读写完成。

####### select
>基本原理：select 函数监视的文件描述符分3类，分别是writefds、readfds、和exceptfds。调用后select函数会阻塞，直到有描述符就绪（有数据 可读、可写、或者有except），或者超时（timeout指定等待时间，如果立即返回设为null即可），函数返回。当select函数返回后，可以通过遍历fdset，来找到就绪的描述符。

缺点:

1. select最大的缺陷就是单个进程所打开的FD是有一定限制的，它由FDSETSIZE设置，32位机默认是1024个，64位机默认是2048。 一般来说这个数目和系统内存关系很大，具体数目可以cat /proc/sys/fs/file-max察看。
2. 对socket进行扫描时是线性扫描，即采用轮询的方法，效率较低。 当套接字比较多的时候，每次select()都要通过遍历FDSETSIZE个Socket来完成调度，不管哪个Socket是活跃的，都遍历一遍。这会浪费很多CPU时间。如果能给套接字注册某个回调函数，当他们活跃时，自动完成相关操作，那就避免了轮询，这正是epoll与kqueue做的
3. 需要维护一个用来存放大量fd的数据结构，这样会使得用户空间和内核空间在传递该结构时复制开销大

####### poll
> 基本原理：poll本质上和select没有区别，它将用户传入的数组拷贝到内核空间，然后查询每个fd对应的设备状态，如果设备就绪则在设备等待队列中加入一项并继续遍历，如果遍历完所有fd后没有发现就绪设备，则挂起当前进程，直到设备就绪或者主动超时，被唤醒后它又要再次遍历fd。这个过程经历了多次无谓的遍历。

它没有最大连接数的限制，原因是它是基于链表来存储的

缺点：

1. 大量的fd的数组被整体复制于用户态和内核地址空间之间，而不管这样的复制是不是有意义。
2. poll还有一个特点是“水平触发”，如果报告了fd后，没有被处理，那么下次poll时会再次报告该fd。

**随着监视的描述符数量的增长，select/poll效率也会线性下降**

####### epoll
epoll是在2.6内核中提出的，是之前的select和poll的增强版本。相对于select和poll来说，epoll更加灵活，没有描述符限制。epoll使用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件存放到内核的一个事件表中，这样在用户空间和内核空间的copy只需一次。

>基本原理：epoll支持水平触发和边缘触发，最大的特点在于边缘触发，它只告诉进程哪些fd刚刚变为就绪态，并且只会通知一次。还有一个特点是，epoll使用“事件”的就绪通知方式，通过epollctl注册fd，一旦该fd就绪，内核就会采用类似callback的回调机制来激活该fd，epollwait便可以收到通知

epoll的优点:

1. 没有最大并发连接的限制，能打开的FD的上限远大于1024（1G的内存上能监听约10万个端口）
2. 效率提升，不是轮询的方式，不会随着FD数目的增加效率下降。 只有活跃可用的FD才会调用callback函数；即Epoll最大的优点就在于它只管理“活跃”的连接，而跟连接总数无关，因此在实际的网络环境中，Epoll的效率就会远远高于select和poll。
3. 内存拷贝，利用mmap()文件映射内存加速与内核空间的消息传递；即epoll使用mmap减少复制开销。

###### 信号驱动的I/O (Signal Driven I/O)
> signal driven IO在实际中并不常用

>首先用户进程建立SIGIO信号处理程序，并通过系统调用sigaction执行一个信号处理函数，这时用户进程便可以做其他的事了，一旦数据准备好，系统便为该进程生成一个SIGIO信号，去通知它数据已经准备好了，于是用户进程便调用recvfrom把数据从内核拷贝出来，并返回结果。

###### 异步I/O
> 这个模型和前面的信号驱动I/O模型的主要区别是，在信号驱动的I/O中，内核告诉我们何时可以启动I/O操作，但是异步I/O时，内核告诉我们何时I/O操作完成


>当用户进程向内核发起某个操作后，会立刻得到返回，并把所有的任务都交给内核去完成（包括将数据从内核拷贝到用户自己的缓冲区），内核完成之后，只需返回一个信号告诉用户进程已经完成就可以了。

***

### java NIO 

NIO主要有三大核心部分：Channel(通道)，Buffer(缓冲区), Selector。传统IO基于字节流和字符流进行操作，而NIO基于Channel和Buffer(缓冲区)进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector(选择区)用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个线程可以监听多个数据通道

#### Channel
NIO中的Channel的主要实现有：

1. FileChannel          文件
2. DatagramChannel		UDP
3. SocketChannel		TCP
4. ServerSocketChannel

#### Buffer
NIO中的关键Buffer实现有：

1. ByteBuffer
2. CharBuffer
3. DoubleBuffer
4. FloatBuffer
5. IntBuffer
6. LongBuffer
7. ShortBuffer
8. MappedByteBuffer
9. HeapByteBuffer
10. DirectByteBuffer

Buffer简单地理解为一组基本数据类型的元素列表，它通过几个变量来保存这个数据的当前位置状态：capacity, position, limit, mark：

1. capacity: 缓冲区数组的总长度
2. position: 下一个要操作的数据元素的位置
3. limit:缓冲区数组中不可操作的下一个元素的位置：limit<=capacity
4. mark:用于记录当前position的前一个位置或者默认是-1

Buffer使用步骤：

1. 分配空间 [ ByteBuffer buf = ByteBuffer.allocate(1024)]
2. 写数据到Buffer [ int bytesRead = fileChannel.read(buf)]
3. 调用filp()方法 [ buf.flip()]
4. 从Buffer中读取数据 [buf.get()]
5. 调用clear()方法或者compact()方法

##### buffer部分函数说明
1. buf.put(...)  写入数据到buf
2. buf.get()  从buf中读取数据
3. buf.flip() position设回0，mark设回-1，并将limit设成之前的position的值，为读数据做准备
4. buf.clear() position将被设回0，mark设回-1,limit设置成capacity,相当于清空buf
5. buf.compact() 将所有未读的数据拷贝到Buffer起始处。然后将position设到最后一个未读元素正后面,limit属性依然像clear()方法一样，设置成capacity 这样未消费的数据，仍旧可以继续消费
6. buf.mark()/buf.reset() 断点
7. buf.rewine() 将position设回0


#### Selector
Selector运行单线程处理多个Channel