reids学习

# 1.reids入门

## 1.1.redis文档地址

http://www.redis.cn/   redis中文网

http://redis.io   redis官网

http://redisdoc.com  redis命令参考大全



## 1.2安装

1.首先linux里面要有gcc编译工具

gcc -v看一下有没



2.解压压缩包

3.进入redis执行make命令

## 1.3 HelloWorld

etc：所有的系统管理所需要的配置文件和子目录

usr：这是一个非常重要的目录，用户的很多应用程序和文件都放在这个目录下，类似与windows下的programfiles目录

所以安装之后的redis去usr/local/bin下做各种命令的执行（此处因为我记错了，以为是etc放安装东西的文件，所以讲两者的区别放在这）



查看后台有没有启动redis ps -ef |grep redis

下面开始helloworld

1.为了保证配置文件不会被自己瞎鸡儿修改不能用，先复制一份配置文件到etc下面

cd /etc

mkdir redis

cd /usr/redis-.....

cp redis.conf /etc/redis

2.修改复制之后的那个redis.conf

vim redis.conf

![1543315228059](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1543315228059.png)

默认的reids不会以后台daemon运行，如果把这里改成了yes，它就会以后台什么玩意运行

此处学习一个快捷操作（shift+$），移动到行的最后

3.启动redis

cd  /usr/local/bin

redis-server /etc/redis/redis.conf

redis-cli -p 6379

测试是否启动成功   ping，他会显示pong

会显示进入下面这种状态

![1543315877695](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1543315877695.png)



4.可以简单是试一试

set k1 hello

然后发现可以用get k1拿到上面那个hello





换一个终端可以用ps -ef|grep redis看到后台有redis启动

![1543315836427](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1543315836427.png)



5.关闭redis

SHUTDOWN

exit

退回到原来的命令行

## 1.4扒来的杂项知识

1.单进程

单进程模型来处理客户端的请求。对读写等事件的响应

是通过对epoll函数的包装来做到的。Redis的实际处理速度完全依靠主进程的执行效率

Epoll是Linux内核为处理大批量文件描述符而作了改进的epoll，是Linux下多路复用IO接口select/poll的增强版本。

它能显著提高提高程序在大量并发连接中只有少量活跃的情况下的系统CPU利用率

2.库

![1543316944443](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1543316944443.png)

默认16个库，从0号库开始

使用select 0这种方式可以切换各个库

3.一些命令



dbsize命令查看当前数据库的key的数量

keys *命令查看有哪些key(试了一下，也是看当前库)

keys命令还支持ant风格，也就是可以用？做占位符

例子：keys k?

![1543317104814](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\1543317104814.png)



flushdb清空当前库 

flushall清空所有库

4.统一密码管理

16个库都是同样密码，要么都OK要么一个也连接不上

5.reids索引都是从零开始

6.默认端口是6379（用数字键盘能打出merz，意大利一个女歌手，redis的作者可能跟她有关系）

## 2.redis的数据类型

### 2.1redis有五大数据类型

#### ①String （字符串）

String是redis最基本的类型，你可以理解成与Memcached一模一样的类型，一个key对应一个value

String类型是二进制安全的，意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象

String类型是Redis最基本的数据类型，一个redis中字符串value最多是512M

#### ②Hash（哈希，类似java中的Map）

Redis Hash是一个键值集合

Redis Hash是一个string类型的field和value的映射表，hash特别适合用于存储对象

类型java里面的Map<String,Object>



#### ③List（列表）

Redis列表是简答的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）

它的底层实际是个链表

#### ④Set（集合）

Redis的Set是string类型的无序无重复集合。它是通过HashTable实现的

#### ⑤Zset（又称sorted set有序集合）

Redis zset和set一样也是string类型元素的结合，且不允许重复的成员

不同的是每个元素关联一个double类型分数

redis正是通过分数来为集合中的成员进行从小到大的排序，zset的成员是唯一的，但分数（score）却可以重复

### 2.2对各种数据类型的操作

#### ①key关键字

keys *查看当前库有哪几个key

exists key的名字   判断某个key是否存在，存在返回1，不存在返回0

move key db-->  把这个key移动到别的库。当前库就没有了，被移除了

expire key 秒钟：为给定的key设置过期时间（设置成-2也就是删除了）

ttl key（time to leave） 查看还有多少秒过期，-1表示永不过期，-2表示已过期

type key查看你的key是什么类型

#### ②Redis字符串（string）

单值单value

##### 1)set/get/del/append/strlen

set key value设置键值，如果对应的键已经有值，那么将覆盖那个值

get key  通过键获取值

del key1 key2 删除，能一次性删除多个

append key value，在当前值的末尾添上value

strlen key 看看某个key对应的值有多长

##### 2)Incr/decr/incrby/decrby，一定要是数字才能进行加减

incr key ，将key对应的值加

decr key，将key对应的值减1

incrby key number ,将key对应的值加number

decrby key number，将key对应的值加number



##### 3)getrange/setrange

getrange key num1 num2 ，从num1到num2截取key对应的值（getrange key 0 -1表示截取全部）

setrange key num1 value，从num1开始往key对应的值设置值value（对应的0-value的长度   的原来的值的部分被覆盖）

##### 4)setex(set with expire)键秒值/setnx(set if not exist)

setex key 秒 value，设置key-value并且设置存活时间

setnx key value，如果不存在，才设置对应的key-value

##### 5)mset/mget/msetnx

mset k1 v1 k2 v2 k3 v3，一次性设置多个key-value

mget，一次性获取多个

msetnx k1 v1 k2 v2 k3 v3，如果不存在，一次性设置多个key-value（如果其中有一个不存在，那么都设置不进去）

##### 6)getset(先get再set)

getset key value ，获取key对应的值，并且设置key-value





#### ③Redis列表（List）

单值多value

##### 1)lpush/rpush/lrange

lpush key v1 v2 v3 v4，从左向右push

rpush key v1 v2  v3 v4，从右向左push

lrange list01 num1 num2，从左边一个个的出，从num1 出到num2

##### 2)lpop/rpop

lpop key ，从左pop一个数

rpop key ，从右pop一个数

##### 3）lindex，按照索引下标获得元素（从上到下）

lindex list01 number ，获得下标为number的数

##### 4）llen

llen list01 ，查看列表里面有多少个value

##### 5）lrem key 删N个value，

lrem key count value，从左到又依次删除count个value

##### 6）ltrim key 开始index 结束index   ，截取指定范围的值后再赋值给key

ltrim key num1 num2，把key里面的num1到num2的内容截取出来并且赋给key

##### 7）rpoplpush源列表  目的列表

从源列表的右边pop出一个元素，从目的列表的左边push

##### 8）lset key index value

设置key中的第index个元素为value

##### 9）linsert key before/after 值1  值2

在值1之前/之后插入值2

10)性能总结

它是一个字符串链表，left，right都可以插入添加

如果键不存在，创建新的链表

如果键已存在，新增内容

如果值全移除，对应的键也就消失了

链表的操作无论是头和尾效率都极高，但假如是对中间元素进行操作，效率就很惨淡了

### ④Redis集合（Set）

单值多value

##### 1)sadd/smembers/sismember

sadd key mem1 mem2 mem3 ，往key添加mems（如果有重复，会自动去掉）

smembers key，查看key里面的members

sismember key member ，查看member是不是key里面的元素（是返回1，不是返回0）

##### 2)scard 获取集合里面的元素的个数

scard key 

##### 3)srem key value删除集合中元素

srem key mem1 mem2 mem3 mem4（）

##### 4)srandmember key count（随机出几个数）

##### 5)spop key number 随机出栈

这个跟上面的区别是出栈之后原来的就没了

##### 6）smove key1 key2 在key1里的某个值

将key1里的某个值赋给key2

##### 7）数学集合类

差集sdiff set01 set02 ，得到set01里面有，但set02里面没有的元素

交集sinter set01 set02，去set01和set02的交集

并集sunion set01 set02，取set01和set02的并集



#### ⑤Redis哈希（Hash）

kv模式不变，但v是一个键值对

##### 1）hset/hget/hmset/hmget/hgetall/hdel

hset key field value，给key里面的field设置值value(field不重复就是往里面添加，重复将会覆盖)

hget key field，获取key里面field对应的值

hmset key f1 v1 f2 v2 f3 v3，一次性往key里面是设置多个f-v

hmget key f1 f2 f3 f4，一次性获取key中多个f对应的值

hgetall key，一次性获取key中的东西，一个field一个value

hdel key field，删除key中的field-value对

##### 2）hlen

hlen key，看key中有多个对field-value

##### 3）hexists

hexists key field，看key里面的field是否存在



##### 4）hkeys/hvals

hkeys key，查看所有key中的field

hvals key，查看所有key中的value

##### 5）hincrby/hincrbyfloat

hincrby key field count，将key中的field对应的value增加count

hincrby key field count

##### 6)hsetnx key field count



#### ⑥Redis有序集合Zset（sorted set）

在set基础上，加一个score值。之前set是k1 v1 v2 v3

现在zset是k1 score v1 score2 v2

1）zadd/zrange

zadd key s1 v1 s2 v2 s3 v3，给key里面添加多个sorce-value对 

zrange key num1 num2 [withscores]，取key里面num1到num2的value（如果写了withscores，那么也把score显示出来）

2）zrangebyscore key开始score结束score



3）

4）zcard/zcount/zrank/zscore

zcard key，查看一共有多少个值

zcount key s1 s2，统计分数在s1-s2之间的个数

zrank key value，拿到key中的value对应的下标

zscore key value，拿到key中value对应的score

5)zrevrank/zrevrange/zrevrangebyscore

zrevrank key value，逆序拿到num的下标

zrevrange key num1 num2 [withscores]，逆序拿到key里面num1到num2的元素

zrevrangebyscore key max min [withscores]，和上面那个zscore正好反过来，获取key中从结束分数到开始分数的值





其他杂项命令

config get requirepass

config set requirepass "123456"

auth "123456"

config get dir

# 





# redis持久化

## 1.rdb

Redis DataBase

### 是什么

在指定的时间间隔内将内存中的数据集快照写入磁盘

也就是行话讲Snapshot快照，它恢复时是将快照文件直接读到内存里

Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，朱金城是不进行任何IO操作的，这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的搞笑。RDB的缺点是最后一次持久化后的数据可能丢失

### fork

fork的作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等），数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程

rdb保存的是dump.rdb文件

在redis.conf配置文件的snapshot配置区域，有save，可以配置多长时间如果做了多少次修改，那么久备份一个dump.rdb文件，手动打save也可以立即备份一个dump.rdb

执行flushall命令，也会产生dump.rdb文件，但里面是空的，没有意义

### 如何恢复

开启redis自动读取dump.rdb文件中数据（这个默认读取dump.rdb可以在redis.conf里面配置）

## 2.aof

Append Only File

### 是什么

以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来（读操作不记录），只许通知文件但不可以改写文件，redis启动之初会读取该文件重构新建数据。换言之，redis重启的话就根据日志文件的内容将写指令从前到后换行一次以完成数据的恢复工作

aof保存的是appendonly.aof文件，默认这个是不开的，可以在conf文件里面开启

如果aof和rdb都存在，加载aof，如果aof有错误，可以用redis-check-aof --fix

### rewrite

#### 是什么

aof采用文件追加的方式，文件会越来越大为避免出现此种情况，新增了重写机制，当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集，可以使用命令bgrewriteaof

#### 重写原理

AOF文件持续增长而过大时，会fork出一条新进程来讲文件重写（也是先写临时文件字后在rename），遍历新进程的内存中数据，每条记录有一条Set语句，重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件，这点和快照有点类似

#### 触发机制

Redis会记录上次重写时的AOF大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发



总结：

RDB持久化方式能够在指定的事件间隔能对你的数据进行快照存储

AOF持久化方式记录每次对服务器的写操作，当服务器重启的时候回重新执行这些命令来恢复原始的数据，AOF命令以redis协议追加保存每次写的操作到文件末尾

Redis还能对AOF文件进行后台重写，使得AOF文件的体积不至于过大

只做缓存：如果你只希望你的数据在服务器运行的时候存在，你也可以不使用任何持久化方式。同时开启两种持久化方式

同时开启两种持久化方式：在这种情况下，当redis重启的时候会优先载入AOF文件来恢复原始的数据，因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整。RDB的数据不实时，同时使用两者时服务器重启也只会找AOF文件，那要不要只使用AOF呢？

作者建议不要，因为RDB更适合用于备份数据库（AOF在不断变化的不好备份）。快速重启，而且不会有AOF可能潜在的bug，留着作为一个万一的手段





## redis的事务

可以一次性执行多个命令，本质是一组命令的集合，一个事务中的所有命令都会序列化，按顺序地串行化执行而不会被其他命令插入，不许加塞

### redis事务命令

discard：取消事务，放弃执行事务块内的所有命令

exec：执行所有事务块内的命令

multi：标记一个事务块的开始

unwatch：取消unwatch命令对所有key的监视

watch key：监视一个（或多个）key，如果在事务执行之前这个（或这些）key被其他命令所改动，那么事务将被打断



### 怎么用

###### Case1：正常执行

#### Case2：放弃事务

#### Case3：全体连坐

Case4：冤头债主

Case5：watch监控





Redis的发布订阅

Redis的复制

是什么

行话：也就是我们所说的主从复制，主机数据更新后根据配置和策略，自动同步到备机的master/slaver机制，Master以写为主，Slave以读为主



















