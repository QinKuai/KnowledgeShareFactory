## Redis基础

### 1. NoSQL

- 非关系型数据库都可以叫做NoSQL，也就是**存储结构和逻辑不限于结构性数据**，比如文本、json、定位、图片等关系性数据库不好处理的数据，可以使用类似于图、Map等结构。

- **数据结构不确定，很难使用标准的表描述，同时可能数据量极大或者增长速度极快，导致动态增减数据字段成为不可能**。
- NoSQL大类
  - **键值对：Redis**，用作缓存、日志系统，通常使用hash表实现。
    - 查找速度快。
    - 但数据无结构化。
  - **列存储数据库：HBase**，用于分布式文件系统，将同一列数据存储在一起。
    - 查找速度快，容易进行分布式扩展。
    - 但功能相对局限。
  - **文档型数据库：MongoDb**，用于web应用，与键值对类似，但值是结构化的。
    - 数据结构要求不严格，表结构可变。
    - 但查询性能不高，缺乏同一查询语句。
  - **图形数据库：Neo4J、InfoGrid**，用于社交网络、推荐系统，专注于构建关系图。
    - 能利用图结构相关的特性和算法，例如最短路径寻址、N度关系查找
    - 但很多时候需要对这张图做计算才能得出结果，而且不便于做分布式部署。



### 2. Redis

#### 2.1 基本

- Redis(**Re**mote **Di**ctionary **S**erver 远程字典服务)，开源的使用C语言编写，支持网络、可基于内存亦可持久化的日志型、键值对数据库，并且提供多语言API。
- 核心部分为单线程，因为程序的瓶颈并不在CPU部分，而在网络带宽和机器内存，能使用单线程实现就使用单线程实现了。
- 特性
  - 多样的内部数据类型
  - 内存高速缓存
  - 持久化技术
  - 集群
  - 事务
- 能支持的功能点
  - 内存存储、断电即失，因此有持久化（rdb、aof）支持
  - 效率高、高速缓存
  - 订阅系统
  - 地图信息分析
  - 计时器、计数器（浏览量...）



#### 2.2 Linux安装与启动

- Linux本地安装

  - 先获取官网的最新程序包到Linux本地，例如`wget http://download.redis.io/releases/redis-6.0.6.tar.gz`或者通过下载后scp传输也可以，方法很多。

  - 解压后即获取程序`tar -zvxf redis-6.0.6.tar.gz`

  - 保证拥有本地有gcc组件，没有就安装`yum install gcc-c++`，然后通过`make`编译redis。

    - 可能出现的make失败的情况，可能是gcc版本过低，6.0+的redis需要5.3+的gcc，在centos中需要通过以下命令

      ```b
      yum -y install centos-release-scl
      yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
      
      # 临时改变
      scl enable devtoolset-9 bash
      # 永久改变
      echo "source /opt/rh/devtoolset-9/enable" >> /etc/profile
      ```

  - 默认安装路径`/usr/local/bin`，目录下有`redis-cli`和`redis-server`等redis组件。

- 启动redis
  - 将解压目录中的`redis.conf`移动到安装目录`/usr/local/bin`中。
  - 将`redis.conf`中的`daemonize`改为`yes`，保证redis为后台启动。
  - 通过指定的配置文件启动redis服务，`./redis-server redis.conf`。
- 连接redis
  
  - 连接服务，`redis-cli -p 6379`（默认端口6379）。
- 关闭redis
  - 连接redis，`redis-cli -p 6379`；
  - `shutdown`，关闭redis-server服务；
  - `exit`退出redis-cli客户端
- 查看redis进程
  
  - `ps -ef | grep redis`查看与redis相关的进程。



#### 2.3 基础知识

- **Redis默认有16个数据库，在配置文件中由databases字段配置**。默认使用的是第0个数据库。
  - 通过`select index`，index从0到15，例如切换到数据库3，`select 3`。
  - 通过`dbsize`可以获取到数据库的大小
- **设置、获取值**
  - `set keyName value`，设置键值对。
  - `setex keyName times value`，设置键值对并给定过期时间。单位为秒。
  - `setnx keyName value`，只有keyName不存在时，设置该键的值。返回1设置成功，返回0设置失败。
  - `get keyName`，获取键对应的值。
  - `getset keyName value`，设置该键的值，并返回原来的值。键不存在时就是设置并返回null。
- **批量设置、获取**
  - `mset keyName1 value1 [keyName2 value2...]`，批量设置键值对。如果有相同的则会设置靠后的值。
  - `msetnx keyName1 value1 [keyName2 value2...]`，如果键不存在则批量设置。原子性操作，如果一个键值对设置失败则都失败，返回0。
  - `mget keyName1 [keyName2...] `，批量获取键对应的值。
- **清除键**
  - `flushdb`，清除当前库的所有键；
  - `flushall`，清除redis内所有键。
- **键存在性**
  - `exists keyName`，查看该键是否存在
- **删除键**
  - `del keyName`，删除某个键值对
- **库间移动键**
  - `move keyName dbIndex`，移动到某一键值对
- **设置过期时间**
  - `expire keyName times(s)`，设置过期时间，单位为秒
  - `ttl keyName`，查看剩余时间，-2表示该键不存在，-1表示该键无过期时间，其余非负数为剩余时间 
- **查看key的值类型** 
  - `type keyName`，查看key对应的值类型



#### 2.4 数据类型

##### 2.4.1 String

- **数据类型**在Redis中也是使用的字符串类型表示的。

- **常用于计数器、统计多单位的数量等功能点。**

- 支持命令

  - **字符串操作**
  - `append keyName "newStr"`，在该键的值后续增加新字符串，返回值为总的字符串长度。如果key不存在则相当于设置键值对。
  - `strlen keyName`，查看该键的值的字符串长度。
  - `getrange keyName 0 3`，截取字符串，范围为[0,3]。
  - `setrange keyName 1 abc`，替换字符串，从索引位置1开始，按照给定的值替换原有的字符串。

  

  - **数值操作**

  - `incr keyName`，数值++。
  - `incrby keyName 10`，数值上+10。
  - `decr keyName`，数值--。
  - `decrby keyName 10`，数值上-10。



##### 2.4.2 List

- 列表结构，可以变为栈、队列、阻塞队列等等，允许重复值存在。

- **根据不同的左右插入、或者左右输出就能变化具体的数据结构**。

- 支持命令

  - **设置、移除**

  - `lpush keyName value `，设置某一list的第一个值

  - `rpush keyName value`，设置某一list的最后一个值 

  - `lpop keyName`，移除某一list的第一个值

  - `rpop keyName`，移除某一list的最后一个值

  - `lrem keyName count value`，移除指定数量的指定的值。

  - `rpoplpush srcKey desKey`，移除源列表中的最后一个元素，并将该元素移动到新列表中的第一个元素（只支持该种方式的组合命令，右出左入）。

  - `linsert keyName before|after valueInList value`，在某个给定值前或者后插入某个值，给定值的位置为从前到后（从左到右）的第一个相匹配的值。

    

  - **查看list信息**

  - `lrange keyName 0 3`，从左开始获取索引从0到3的list元素，若后者为-1则表示获取整个list。

  - `llen keyName`，返回list的长度。

  

  - **通过index更新、获取值**
  - `lset keyName index value`，通过索引更新值，若key不存在则会报错no such key。对于超出现有长度的index，则会报错超过范围。
  - `lindex keyName index`，通过list索引获取对应的值，若index超过list范围则返回null。

  

  - **截取**
  - `ltrim keyName 0 1`，截取从索引从0到1的列表。



##### 2.4.3 Set

- 内部元素不重复的一种集合。

- 支持命令

  - **添加、移除值**
  - `sadd setName value1 [value2...]`，在集合中添加元素（集合不存在时会创建集合）。
    - **添加的元素在集合中不重复，则计数添加了1并添加入集合，重复则返回0并不添加，最终返回添加了的元素数目**。（也就是说，如果一个sadd语句中，存在集合中没有的元素和已有的元素，那么sadd会添加哪些没有的元素，返回这些集合中没有的元素数量）
  - `srem setName value1 [value2...]`，在集合中移除元素 。
    - 移除中值若在集合中不存在，则不影响其他移除值的影响

  

  - **查看集合信息、判断是否有某元素**
  - `smembers setName`，获取集合中的所有元素。
  - `sismemebr setName value`，判断集合中是否存在某个值，存在返回1，不存在返回0。
  - `scard setName`，查看集合元素数量。

  

  - **随机抽选**
  - `srandmember setName [count]`，随机获取集合中的一个元素或是给定元素数量。
  - `spop setName [count]`，随机移除集合中的一个元素或是给定元素数量。

  

  - **移动集合元素**
  - `smove srcSetName desSrcName value`，将一个集合中的某个元素移动到目标集合。

  

  - **集合运算**
  - `sdiff setName1 setName2 [setName3...]`，set1与set2的差集，即set1减去set1和set2的交集，剩下的元素。
  - `sunion setName1 setName2 [setName3...]`，并集。
  - `sinter setName1 setName2 [setName3...]`，交集。



##### 2.4.4 Hash

- Map集合，这时值类型相当于是一个Map

- 支持命令

  - **添加、移除元素**
  - `hset hashName innerKeyName value [innerKeyName2 value...]`，往Map添加一个或多个键值对，在hashName不存在时创建新的Hash实例，内部就是一张Map结构。
  - `hmset hashName innerKeyName1 value1 [innerKeyName2 value...]`，也能实现hset命令的功能。
  - `hsetnx hashName innerKeyName value`，如果内部key不存在则添加该值，不存在则不添加。
  - `hdel hashName innerKeyName1 [innerKeyName2]`，删除一个或多个Map实例内部key。

  

  - **查看Map信息**
  - `hgetall hashName`，获取Map中的所有内部key和值，以一键一值的顺序输出。
  - `hget hashName innerKeyName`，获取某个Map的内部key对应的值。
  - `hkeys hashName`，获取某个Map的所有内部Key。
  - `hvals hashName`，获取某个Map的所有内部值。
  - `hlen hashName`，获取某个Map的内部key数量。
  - `hexists hashName innerKeyName`，判断某个Map中的内部key是否存在，存在返回1，不存在返回0。

  

  - **数值增减**
  - `hincrby hashName innerKeyName count`，对某个Map中的某个内部key对应的值增减给定数值，count可以为负数。并且redis没有hdecrby命令。



##### 2.4.5 Zset

- 有序集合类型，在不重复集合的基础上增加了有序性。

- **集合内部默认保持有序，先按照score升序排列，然后按照value的字典序排列**。

- 支持命令

  - **添加、移除**

  - `zadd setName score1 value1 [score2 value2...]`，向有序集合中添加元素，并给定每个元素的score，对于已经存在于集合中的元素，该命令会更新其score。

  - `zrem setName value1 [value2...]`，从有序集合中移除元素。

    

  - **获取有序集合信息**

  - `zrange setName 0 2 [withscores]`，通过给定范围获取有序集合元素，0为起始索引，2为终止索引,若为-1则表示查看所有该集合中的元素。带上withscores会在结果集中返回每个元素的score。

  - `zrangebyscore setName minVal maxVal [withscores]`，通过指定score最大最小值，获取该闭区间内的集合元素，升序排列。*（最小值可以为 -inf，最大值可以为 +inf）*

  - `zrevrange setName 0 2 [withscores]`，功能与zrange相似，但输出结果为降序排列。

  - `zrevrangebyscore setName minVal maxVal [withscores]`，与zrangebyscore相似，但输出结果为降序排列。

  - `zcount setName minVal maxVal`，获取在score最大最小值闭区间内的元素数量
  
  - `zcard setName`，获取该有序集合元素数量。

#### 2.5 特殊数据类型

##### 2.5.1 geospatial地理位置

- 该数据类型底层结构为一个Zset，有序集合内部由memberName做元素值，score通过geohash算法计算经纬度值得出。*（因此对于geo元素的操作可以也能通过zset命令操作，但只推荐删除指令）*
- **该数据类型不能添加两级圈坐标（-85.05到85.05）**。
- 支持命令:
  - `geoadd keyName lng lat memberName`，添加地理位置，需要指定memberName。
  - `geopos keyName memberName`，通过memberName获取到地理位置。
  - `geodist keyName memberName1 memberName2 [m|km|ft|mi]`，获取两地之间的直线距离，并可选指定单位。
  - `georadius keyName lng lat radiusValue [m|km|ft|mi] `，获取在给定坐标、给定半径范围内的，该集合中的结果。
    - [withcoord]，结果中附带经纬度信息
    - [withdist]，结果中附带直线距离信息，单位与前面指定的相同
    - [withhash]，附带通过geohash编码后的52位有符号整数，实际作用不大
    - [Count count]，获取指定数量的元素，并不能加速查找周边元素的速度，但能减少传输的带宽消耗
    - [asc | desc]，有近到远|由远到近
  - `georadiusbymember keyName memberName radiusValue [m|km|ft|mi]`，获取给定memberName、给定半径范围内的，该集合中的结果。
    - 参数设置与georadius相同。
  
  
  
  - **通过Zset指令操作（因为geo底层为Zset）**
  - `zrange keyName 0 -1`，获取全部地理位置memberName。
  - `zrem keyName memberName`，移除某个地区的地理位置。

##### 2.5.2 hyperloglog

- hyperloglog是一个算法，可以使用固定大小的字节计算任意大小的DV(Distinct Value)。

- **基数，表示一个数据集中，去重之后的数据量**。
- 对于统计网页的UV(User Views)，一个用户需要当作同一人的情况下。
  - 一般通过set保存用户id，但一般用户id都会是比较长的字符串，数据量大之后会导致内存消耗严重。
  - **使用hyperloglog算法做处理，在2^64个不同元素的情况下，只需要使用12KB的内存，但存在0.81%的错误率**。
  - **Redis直接支持了该算法的集合API**。
- 支持命令
  - `pfadd keyName value1 [value2...]`，添加元素。
  - `pfcount keyName`，获取集合的基数值，也就是去重后的数量。
  - `pfmerge keyName1 keyName2 keyName3`，将keyName2和keyName3中的元素合并后，赋值到keyName1中（不存在则新建）。



##### 2.5.3 bitmap

- 对于只有两种状态的情况，使用bitmap通过位来记录、获取数据，能相当节省存储空间。
- 底层由String实现。
- 支持命令：
  - `setbit keyName offset val`，设置给定offset的值，offset为非负整数，val只能是0或是1。
  - `setget keyName offset`，获取给定的offset对应的value，若offset之前没有被设置值，默认为0。
  - `bitcount keyName [start end]`，**获取所有或者给定范围内被设定为1的bit数量，start和end的单位是字节，也就是8个bit。例如0 0对应的就是前8个bit，0 1对应前16个bit，0 -1对应所有情况**。



#### 2.6 事务

- 事务就是所有操作同时成功或者同时失败，保证原子性。
- **Redis单条命令保证原子性，但事务不保证原子性，没有隔离级别的概念，也被称为伪事务**。
- Redis事务流程
  - 开启事务，`multi`。
  - 命令入队，例如`set keyName val`，返回`Queued`。
    - **在命令入队中，如果出现命令语法问题，会导致依旧能入队，但exec时事务被discard掉**。
  - 执行事务，`exec`，同一按照先进先出顺序执行，并返回命令执行结果。
    - **运行时出现了错误，例如对字符串执行incr，该命令会失败，但其他事务中的其他命令依旧执行**。
  - *放弃事务，`discard`，在`multi`命令后，所有命令队列中的命令不再执行*。

- Redis实现乐观锁

  - 乐观、悲观对比

    - 悲观锁，对所有操作都保证上锁，保证并发一致性。
    - 乐观锁，对于所有操作都不会选择上锁，更新数据时判断一下标志字段的值，例如version值，查看期间有无修改。

  - **通过`watch keyName`实现乐观锁，如果在exec前有其他线程更新了money的值（没有ABA问题，只要更新了，就会事务失败），则事务会整个执行失败，返回(nil)**。exec执行后会自动释放对应字段的锁。

    ```bash
    > watch money
    OK
    > multi
    OK
    > decrby money 20
    QUEUED
    > incrby out 20
    QUEUED
    > exec
    (nil)
    ```

