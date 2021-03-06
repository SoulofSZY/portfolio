## java_IO
> 区分同步或异步（synchronous/asynchronous）。简单来说，同步是一种可靠的有序运行机制，当进行同步操作时，后续的任务是等待当前调用返回，才会进行下一步；而异步则相反，其他任务不需要等待当前调用返回，通常依靠事件、回调等机制来实现任务间次序关系

> 区分阻塞与非阻塞（blocking/non-blocking）。在进行阻塞操作时，当前线程会处于阻塞状态，无法从事其他任务，只有当条件就绪才能继续，比如ServerSocket新连接建立完毕，或数据读取、写入操作完成；而非阻塞则是不管 IO 操作是否结束，直接返回，相应操作在后台继续处理

**IO**

1. IO不仅仅是对文件的操作，网络编程中，比如Socket通信，都是典型的IO操作目标
2. 输入流、输出流（InputStream/OutputStream）是用于读取或写入字节的，例如操作图片文件
3. Reader/Writer则是用于操作字符，增加了字符编解码等功能，适用于类似从文件中读取或者写入文本信息。本质上计算机操作的都是字节，不管是网络通信还是文件读取， Reader/Writer 相当于构建了应用逻辑和原始数据之间的桥梁
4. BufferedOutputStream等带缓冲区的实现，可以避免频繁的磁盘读写，进而提高IO处理效率。这种设计利用了缓冲区，将批量数据进行一次操作，但**在使用中千万别忘了 flush**
5. 很多IO工具类都实现了Closeable接口，因为需要进行资源的释放。比如，打开FileInputStream，它就会获取相应的文件描述（FileDescriptor），需要利用 try-with-resources 、 try-fnally 等机制保证FileInputStream 被明确关闭，进而相应文件描述符也会失效，否则将导致资源无法被释放

**NIO**

主要组成

1. Buffer:高效的数据容器，除了布尔类型，所有原始数据类型都有相应的Buffer实现
2. Channel:类似在Linux之类操作系统上看到的文件描述符，是NIO中被用来支持批量式IO操作的一种抽象。File 或者 Socket ，通常被认为是比较高层次的抽象，而 Channel 则是更加操作系统底层的一种抽象，这也使得 NIO 得以充分利用现代操作系统底层机制，获得特定场景的性能优化，例如， DMA （ Direct Memory Access ）等。不同层次的抽象是相互关联的，可以通过 Socket 获取 Channel ，反之亦然
3. Selector，是NIO实现多路复用的基础，它提供了一种高效的机制，可以检测到注册在Selector上的多个Channel中，是否有Channel处于就绪状态，进而实现了单线程对多 Channel 的高效管理。Selector 同样是基于底层操作系统机制，不同模式、不同版本都存在区别
4. Chartset，提供Unicode字符串定义，NIO也提供了相应的编解码器等

**总结**

IO 都是同步阻塞模式，所以需要多线程以实现多任务处理。而 NIO 则是利用了单线程轮询事件的机制，通过高效地定位就绪的 Channel ，来决定做什
么，仅仅 select 阶段是阻塞的，可以有效避免大量客户端连接时，频繁线程切换带来的问题，应用的扩展能力有了非常大的提高