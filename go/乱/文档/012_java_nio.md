## java_nio

场景： 文件拷贝

1. 传统io类库做法
2. nio类库提供的transferTo/transferFrom

对于Copy的效率，这个其实与操作系统和配置等情况相关，总体上来说，NIO transferTo/From的方式可能更快 ，因为它更能利用现代操作系统底层机制，避免不必要拷贝和上下文切换


**特别的Buffer**

1. Direct Buffer：如果看Buffer的方法定义，会发现它定义了isDirect()方法，返回当前Buffer是否是Direct类型。这是因为Java提供了堆内和堆外（Direct）Buffer，我们可以以它的 allocate 或者 allocateDirect 方法直接创建
2. MappedByteBuffer：它将文件按照指定大小直接映射为内存区域，当程序访问这个内存区域时将直接操作这块儿文件数据，省去了将数据从内核空间向用户空间传输的损耗。可以使用FileChannel.map创建MappedByteBuffer，它本质上也是种Direct Buffer

>Java 会尽量对 Direct Buffer 仅做本地 IO 操作，对于很多大数据量的 IO 密集操作，可能会带来非常大的性能优势

1. Direct Buffer生命周期内内存地址都不会再发生更改，进而内核可以安全地对其进行访问，很多IO操作会很高效
2. 减少了堆内对象存储的可能额外维护工作，所以访问效率可能有所提高