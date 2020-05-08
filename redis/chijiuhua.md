[TOC]
# 持久化方案
> 在所有涉及存储的技术里面,持久化必不可少的两步是 快照/副本 和 日志,与用的什么平台无关,不管是redis,mysql,还是其他数据库,这是两个数据持久化方案.

# redis 的持久化方案
> redis 中的持久化方案有两种,一是 RDB(redis database) 对应快照/副本,二是 AOF(append only file) 对应日志.

## RDB 
### save
阻塞当前线程进行RDB  
### bgsave 
另起一个进程进行RDB

### fork 
在需要写快照的时候,父进程创建一个子进程,此时数据会在子进程里形成副本,父进程继续业务,子进程进行写快照.

运行的命令是 fork() 
redis 具有自己的虚拟地址,虚拟地址通过指针指向物理内存中数据的实际地址.fork()命令的作用是将指向数据的指针复制到子进程一份.  

如果此时主进程发生写操作,则会使用copy on write 技术.  

### copy on write 写时拷贝(fork 的机制)
> 写时拷贝的主要作用就是将拷贝推迟到写操作真正发生时，这也就避免了大量无意义的拷贝操作

> 在 fork 函数调用时，父进程和子进程会被 Kernel 分配到不同的虚拟内存空间中，所以在两个进程看来它们访问的是不同的内存

- 在真正访问虚拟内存空间时，Kernel 会将虚拟内存映射到物理内存上，所以父子进程共享了物理上的内存空间；
- 当父进程或者子进程对共享的内存进行修改时，共享的内存才会 __以页为单位__ 进行拷贝，父进程会保留原有的物理空间，而子进程会使用拷贝后的新物理空间；

在 Redis 服务中，子进程只会读取共享内存中的数据，它并不会执行任何写操作，只有父进程会在写入时才会触发这一机制，而对于大多数的 Redis 服务或者数据库，写请求往往都是远小于读请求的，所以使用 fork 加上写时拷贝这一机制能够带来非常好的性能，也让 BGSAVE 这一操作的实现变得非常简单。

### 缺陷
- 如果你希望在redis意外停止工作（例如电源中断）的情况下丢失的数据最少的话，那么RDB不适合你.虽然你可以配置不同的save时间点(例如每隔5分钟并且对数据集有100个写的操作),是Redis要完整的保存整个数据集是一个比较繁重的工作,你通常会每隔5分钟或者更久做一次完整的保存,万一在Redis意外宕机,你可能会丢失几分钟的数据.
- RDB 需要经常fork子进程来保存数据集到硬盘上,当数据集比较大的时候,fork的过程是非常耗时的,可能会导致Redis在一些毫秒级内不能响应客户端的请求.如果数据集巨大并且CPU性能不是很好的情况下,这种情况会持续1秒,AOF也需要fork,但是你可以调节重写日志文件的频率来提高数据集的耐久度.

## AOF
- 使用AOF 会让你的Redis更加耐久: 你可以使用不同的fsync策略：无fsync,每秒fsync,每次写的时候fsync.使用默认的每秒fsync策略,Redis的性能依然很好(fsync是由后台线程进行处理的,主线程会尽力处理客户端请求),一旦出现故障，你最多丢失1秒的数据.
- AOF文件是一个只进行追加的日志文件,所以不需要写入seek,即使由于某些原因(磁盘空间已满，写的过程中宕机等等)未执行完整的写入命令,你也也可使用redis-check-aof工具修复这些问题.
- Redis 可以在 AOF 文件体积变得过大时，自动地在后台对 AOF 进行重写：(4.0版本 之前) 重写后的新 AOF 文件包含了恢复当前数据集所需的最小命令集合。 整个重写操作是绝对安全的，因为 Redis 在创建新 AOF 文件的过程中，会继续将命令追加到现有的 AOF 文件里面，即使重写过程中发生停机，现有的 AOF 文件也不会丢失。 而一旦新 AOF 文件创建完毕，Redis 就会从旧 AOF 文件切换到新 AOF 文件，并开始对新 AOF 文件进行追加操作。具体操作是删除抵消的命令,合并重复的命令. 4.0之后,先将老的数据RDB到aof文件中,将增量数据追加到aof中
- AOF 文件有序地保存了对数据库执行的所有写入操作， 这些写入操作以 Redis 协议的格式保存， 因此 AOF 文件的内容非常容易被人读懂， 对文件进行分析（parse）也很轻松。 导出（export） AOF 文件也非常简单： 举个例子， 如果你不小心执行了 FLUSHALL 命令， 但只要 AOF 文件未被重写， 那么只要停止服务器， 移除 AOF 文件末尾的 FLUSHALL 命令， 并重启 Redis ， 就可以将数据集恢复到 FLUSHALL 执行之前的状态。

### 缺陷
- 对于相同的数据集来说，AOF 文件的体积通常要大于 RDB 文件的体积。
- 根据所使用的 fsync 策略，AOF 的速度可能会慢于 RDB 。 在一般情况下， 每秒 fsync 的性能依然非常高， 而关闭 fsync 可以让 AOF 的速度和 RDB 一样快， 即使在高负荷之下也是如此。 不过在处理巨大的写入载入时，RDB 可以提供更有保证的最大延迟时间（latency）。

## 4.0之后

4.0 之后,AOF中包含全量RDB ,将新的写操作进行追加记录