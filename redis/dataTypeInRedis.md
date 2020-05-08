[TOC]
# key-value
数据在redis中是以key-value键值对形式存在的  

## value的类型

在redis中基本的数据类型有5种,由基本类型可以衍生出更多的类型
- string
- list
- set
- zset
- map
- GIS
- hyperloglog

### string

String类型在redis中是用byte数组表示的,它可以用来表示三种数据类型
- 字符串
    - set 将字符串值 value 关联到 key 。如果 key 已经持有其他值， SET 就覆写旧值， 无视类型。
    - setnx 只在键 key 不存在的情况下， 将键 key 的值设置为 value 。
    - get 返回与键 key 相关联的字符串值。
    - getset 将键 key 的值设为 value ， 并返回键 key 在被设置之前的旧值。
    - append 如果键 key 已经存在并且它的值是一个字符串， APPEND 命令将把 value 追加到键 key 现有值的末尾。如果 key 不存在， APPEND 就简单地将键 key 的值设为 value ， 就像执行 SET key value 一样。
    - setrange 从偏移量 offset 开始， 用 value 参数覆写(overwrite)键 key 储存的字符串值。
    - getrange 返回键 key 储存的字符串值的指定部分， 字符串的截取范围由 start 和 end 两个偏移量决定 (包括 start 和 end 在内)。
    - strlen 返回键 key 储存的字符串值的长度。
    - mset 同时为多个键设置值。MSET 是一个原子性(atomic)操作， 所有给定键都会在同一时间内被设置， 不会出现某些键被设置了但是另一些键没有被设置的情况。
    - msetnx 当且仅当所有给定键都不存在时， 为所有给定键设置值。MSETNX 是一个原子性(atomic)操作， 所有给定键要么就全部都被设置， 要么就全部都不设置， 不可能出现第三种状态。
    - mget 返回给定的一个或多个字符串键的值。
- 数值
    - incr 为键 key 储存的数字值加上一。
    - incrby 为键 key 储存的数字值加上增量 increment 。
    - incrbyfloat 为键 key 储存的值加上浮点数增量 increment 。
    - decr 为键 key 储存的数字值减去一。
    - decrby 将键 key 储存的整数值减去减量 decrement 。
- bitmap
    - getbit 获取指定位的位值
    - setbit 设置指定位的位值
    - bitfield 对字符串任意位字段执行整数运算
    - bitcount 计算给定字符串中，被设置为 1 的比特位的数量。
    - bitpos 返回位图中第一个值为 bit 的二进制位的位置。
    - bitop 在字符串之间进行按位运算


### list

在 redis 中 list 是以链表形式存在的所以可以用来表示 链表,数组,栈(同向命令),队列(反向命令),阻塞队列,单播队列,  
redis 中索引是可以为负数的,头元素索引位0,左边为负索引,右边为正索引,所以尾部元素索引为-1
- lpush 将一个或多个值 value 插入到列表 key 的表头
- lpushx 将值 value 插入到列表 key 的表头，当且仅当 key 存在并且是一个列表。
- rpush 将一个或多个值 value 插入到列表 key 的表尾(最右边)。
- rpushx 将值 value 插入到列表 key 的表尾，当且仅当 key 存在并且是一个列表。
- lpop 移除并返回列表 key 的头元素。
- rpop 移除并返回列表 key 的尾元素。
- rpoplpush 命令 RPOPLPUSH 在一个原子时间内，执行以下两个动作：
    - 将列表 source 中的最后一个元素(尾元素)弹出，并返回给客户端。
    - 将 source 弹出的元素插入到列表 destination ，作为 destination 列表的的头元素。
- lrem 根据参数 count 的值，移除列表中与参数 value 相等的元素。
- llen 返回列表 key 的长度。
- lindex 返回列表 key 中，下标为 index 的元素。
- linsert 将值 value 插入到列表 key 当中，位于值 pivot 之前或之后。
- lset 将列表 key 下标为 index 的元素的值设置为 value 。
- lrange 返回列表 key 中指定区间内的元素，区间以偏移量 start 和 stop 指定。
- ltrim 对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。
- blpop BLPOP 是列表的阻塞式(blocking)弹出原语。它是 LPOP key 命令的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被 BLPOP 命令阻塞，直到等待超时或发现可弹出元素为止。
- brpop BRPOP 是列表的阻塞式(blocking)弹出原语。它是 RPOP key 命令的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被 BRPOP 命令阻塞，直到等待超时或发现可弹出元素为止。
- brpoplpush BRPOPLPUSH 是 RPOPLPUSH source destination 的阻塞版本，当给定列表 source 不为空时， BRPOPLPUSH 的表现和 RPOPLPUSH source destination 一样。当列表 source 为空时， BRPOPLPUSH 命令将阻塞连接，直到等待超时，或有另一个客户端对 source 执行 LPUSH key value [value …] 或 RPUSH key value [value …] 命令为止。

### set
底层数据结构是 __intset 整数集合__ 和 __字典__

- sadd 将一个或多个 member 元素加入到集合 key 当中，已经存在于集合的 member 元素将被忽略。
- sismember 判断 member 元素是否集合 key 的成员。
- spop 移除并返回集合中的一个随机元素。
- srandommember 如果命令执行时，只提供了 key 参数，那么返回集合中的一个随机元素。从 Redis 2.6 版本开始， SRANDMEMBER 命令接受可选的 count 参数：
    - 如果 count 为正数，且小于集合基数，那么命令返回一个包含 count 个元素的数组，数组中的元素各不相同。如果 count 大于等于集合基数，那么返回整个集合。
    - 如果 count 为负数，那么命令返回一个数组，数组中的元素可能会重复出现多次，而数组的长度为 count 的绝对值。
- srem 移除集合 key 中的一个或多个 member 元素，不存在的 member 元素会被忽略。
- smove 将 member 元素从 source 集合移动到 destination 集合。
- scard 返回集合 key 的基数(集合中元素的数量)。
- smembers 返回集合 key 中的所有成员。
- sscan 命令用于迭代集合键中的元素。从CURSOR开始返回COUNT个通过MATCH匹配到的元素的集合.
- sinter 返回一个集合的全部成员，该集合是所有给定集合的交集。
- sinterstore 这个命令类似于 SINTER key [key …] 命令，但它将结果保存到 destination 集合，而不是简单地返回结果集。
- sunion 返回一个集合的全部成员，该集合是所有给定集合的并集。
- sunionstore 这个命令类似于 SUNION key [key …] 命令，但它将结果保存到 destination 集合，而不是简单地返回结果集。
- sdiff 返回一个集合的全部成员，该集合是所有给定集合之间的差集。
- sdiffstore 这个命令的作用和 SDIFF key [key …] 类似，但它将结果保存到 destination 集合，而不是简单地返回结果集。

### zset

物理内存左小右大,底层数据结构在元素少的时候是 __ziplist__,在元素多的时候是 __skipList 跳表__ 加 __dict 字典__

- zadd 将一个或多个 member 元素及其 score 值加入到有序集 key 当中。
- zscore 返回有序集 key 中，成员 member 的 score 值。
- zincrby 为有序集 key 的成员 member 的 score 值加上增量 increment 。
- zcard 返回有序集 key 的基数。
- zcount 返回有序集 key 中， score 值在 min 和 max 之间(默认包括 score 值等于 min 或 max )的成员的数量。
- zrange 返回有序集 key 中，指定区间内的成员。具有相同 score 值的成员按字典序排列
- zrevrange 返回有序集 key 中，指定区间内的成员。其中成员的位置按 score 值递减(从大到小)来排列。 具有相同 score 值的成员按字典序的逆序排列
- zrangebyscore 返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。有序集成员按 score 值递增(从小到大)次序排列。具有相同 score 值的成员按字典序来排列(该属性是有序集提供的，不需要额外的计算)。
- zrevrangebyscore 返回有序集 key 中， score 值介于 max 和 min 之间(默认包括等于 max 或 min )的所有的成员。有序集成员按 score 值递减(从大到小)的次序排列。具有相同 score 值的成员按字典序的逆序排列。
- zrank 返回有序集 key 中成员 member 的排名。其中有序集成员按 score 值递增(从小到大)顺序排列。排名以 0 为底，也就是说， score 值最小的成员排名为 0 。
- zrevrank 返回有序集 key 中成员 member 的排名。其中有序集成员按 score 值递减(从大到小)排序。排名以 0 为底，也就是说， score 值最大的成员排名为 0 。
- zrem 移除有序集 key 中的一个或多个成员，不存在的成员将被忽略。
- zremrangebyrank 移除有序集 key 中，指定排名(rank)区间内的所有成员。
- zremrangebyscore 移除有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。
- zrangebylex 当有序集合的所有成员都具有相同的分值时， 有序集合的元素会根据成员的字典序（lexicographical ordering）来进行排序， 而这个命令则可以返回给定的有序集合键 key 中， 值介于 min 和 max 之间的成员。
- zlexcount 对于一个所有成员的分值都相同的有序集合键 key 来说， 这个命令会返回该集合中， 成员介于 min 和 max 范围内的元素数量。
- zremrangebylex 对于一个所有成员的分值都相同的有序集合键 key 来说， 这个命令会移除该集合中， 成员介于 min 和 max 范围内的所有元素。
- zscan 
- zunionstore 计算给定的一个或多个有序集的并集，其中给定 key 的数量必须以 numkeys 参数指定，并将该并集(结果集)储存到 destination 。

### map

- mset 将哈希表 hash 中域 field 的值设置为 value 。
- hsetnx 当且仅当域 field 尚未存在于哈希表的情况下， 将它的值设置为 value 。如果给定域已经存在于哈希表当中， 那么命令将放弃执行设置操作。
- hget 返回哈希表中给定域的值。
- hexists 检查给定域 field 是否存在于哈希表 hash 当中。
- hdel 删除哈希表 key 中的一个或多个指定域，不存在的域将被忽略。
- hlen 返回哈希表 key 中域的数量。
- hstrlen 返回哈希表 key 中， 与给定域 field 相关联的值的字符串长度（string length）。
- hincrby 为哈希表 key 中的域 field 的值加上增量 increment 。
- hincrbyfloat 为哈希表 key 中的域 field 加上浮点数增量 increment 。
- hmset 同时将多个 field-value (域-值)对设置到哈希表 key 中。
- hmget 返回哈希表 key 中，一个或多个给定域的值。
- hkeys 返回哈希表 key 中的所有域。
- hvals 返回哈希表 key 中所有域的值。
- hgetall 返回哈希表 key 中，所有的域和值。
- hscan

### hyperloglog
基数计数  
redis是基于HLL算法实现的HyperLogLog结构，用于统计一组数据集合中不重复的数据个数。 redis中统计数组大小设置为，hash函数生成64位bit数组，其中 位用来找到统计数组的位置，剩下50位用来记录第一个1出现的位置，最大位置为50，需要 位记录。  
> http://www.rainybowe.com/blog/2017/07/13/%E7%A5%9E%E5%A5%87%E7%9A%84HyperLogLog%E7%AE%97%E6%B3%95/index.html  
> https://en.wikipedia.org/wiki/HyperLogLog

# 插件
## RedisBloom

> 官方文档地址 https://oss.redislabs.com/redisbloom/  

### 安装和使用
- 访问 redis 官网: https://redis.io
- 进入模块 modules
- 选择 RedisBloom,进入github页面,复制源码地址
- 在系统中下载源码并解压编译
- 将 bloom.so 复制到 redis 目录里面
- 运行命令 ``` redis-server --loadmodule {the address of bloom.so}``` 安装插件
- 也可以写到配置文件里面
### 类型
- 布隆过滤器
- 布谷鸟过滤器