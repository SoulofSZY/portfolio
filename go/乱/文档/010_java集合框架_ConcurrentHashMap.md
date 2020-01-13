## java集合框架-ConcurrentHashMap

>Java 提供了不同层面的线程安全支持,传统集合框架:[1.Hashtable 等同步容器 2.同步的包装容器（如 Collections.synchronizedMap)],这两种实现都是较粗粒度的同步方式，高并发情况下，性能较低下。
>J.U.C包提供的线程安全容器类，这些类远优于早期的简单同步实现


1. HashMap不是线程安全的，并发情况会导致类似 CPU 占用 100% ，size(...)不准确等一些问题
2. Hashtable，历史级别map,本身比较低效，因为它的实现基本就是将 put 、get 、size等各种方法加上 “synchronized” 。简单来说，这就导致了所有并发操作都要竞争同一把锁，一个线程在进行同步操作时，其他线程只能等待，大大降低了并发操作的效率
3. Collections 提供的同步包装器,同步包装器只是利用输入 Map 构造了另一个同步版本，所有操作虽然不再声明成为 synchronized 方法，但是还是利用了 “this” 作为互斥的 mutex ，没有真正意义上的改进！


ConcurrentHashMap

1. <1.7 分离锁  size() 可能不准确且开销大
2. 1.8  cas