# Redis 通用命令

 ## SELECT 选择数据库

 `Redis`默认有`16`个数据库，默认使用第0个数据库，可以用`SELECT`命令选择数据库，比如: `SELECT 2` 选择`2`号数据库。

```bash
127.0.0.1:6379> SELECT 2
OK
127.0.0.1:6379[2]>
```

## DBSIZE 查看已存容量
使用`DBSIZE`可以查看该数据库已经存储的容量。

```bash
127.0.0.1:6379[1]> DBSIZE
(integer) 2
```
## FLUSHDB 清空当前数据库

清空当前数据库使用`FLUSHDB`。

```bash
127.0.0.1:6379[1]> FLUSHDB
OK
127.0.0.1:6379[1]> KEYS *
(empty list or set)
```

## FLUSHALL 清空所有数据库
清空所有数据库使用`FLUSHALL`。

```bash
127.0.0.1:6379[1]> FLUSHALL
OK
```


 # Redis key命令

## EXISTS 查看key是否存在

查看一个`key`是否存在可以使用`EXISTS key`，存在会返回`1`，否则返回`0`。

```bash
127.0.0.1:6379[2]> EXISTS name
(integer) 1
127.0.0.1:6379[2]> EXISTS age
(integer) 0
```

## KEYS * 查看所有的key

`KEYS * `可以查看当前数据库所有的`key`。

```bash
127.0.0.1:6379[2]> KEYS *
1) "name"
```

## MOVE 移动key

使用 `MOVE key databaseIndex` 可以将该值移动到目标数据库。

```bash
127.0.0.1:6379[2]> MOVE name 1
(integer) 1
```

## EXPIRE 设置过期时间

要为一个值设置过期时间可以使用 `EXPIRE key seconds`，比如 `EXPIRE name 10`，则这个值会在`10`秒后过期。

```bash
127.0.0.1:6379[1]> EXPIRE name 10
(integer) 1
```


## TTL 查看剩余时间

`TTL key` 可以查看这个值的剩余时间，如果返回的是负数，`-1`表示永不过期，`-2`表示已经过期。

```bash
127.0.0.1:6379[1]> TTL name
(integer) 7
127.0.0.1:6379[1]> TTL name
(integer) 3
127.0.0.1:6379[1]> TTL name
(integer) -2
```

## EXPIREAT UNIX时间戳设置过期时间

`EXPIREAT key timestamp`命令也可以设置过期时间，但是该命令接收的是`UNIX`时间戳。


## PEXPIRE 毫秒计设置过期时间

`PEXPIRE key milliseconds`也可以设置过期时间，但是接收的是毫秒。


## PEXPIREAT 毫秒计时间戳设置过期时间

`PEXPIREAT milliseconds-timestamp`，时间戳以毫秒计设置过期时间。


## PTTL 毫秒返回剩余时间

`PTTL key`会以毫秒返回剩余时间。

## PERSIST 移除过期时间
`PERSIST key`可以移除`key`的过期时间，使该`key`永不过期。当过期时间移除成功时，返回 `1` 。 如果 `key` 不存在或 `key` 没有设置过期时间，返回 `0` 。

```bash
127.0.0.1:6379[1]> PERSIST name
(integer) 1
```

## TYPE 查看key存储的值类型
使用 `TYPE key` 可以查看该`key`存储的值的类型。

```bash
127.0.0.1:6379[1]> TYPE name
string
```
## RANDOMKEY 随机返回key
使用`RANDOMKEY`命令可以从当前数据库中随机返回一个`key`，数据库为空时返回`nil`。

```bash
127.0.0.1:6379[1]> RANDOMKEY
"age"
127.0.0.1:6379[1]> RANDOMKEY
"name"
127.0.0.1:6379[1]> SELECT 0
OK
127.0.0.1:6379> RANDOMKEY
(nil)
```

## RENAME 重命名key

`RENAME old_key_name new_key_name`可以重命名`key`。成功时提示 `OK` ，失败时候返回一个错误。

当 `old_key_name` 和 `new_key_name` 相同，或者 `old_key_name` 不存在时，返回一个错误。 当 `new_key_name` 已经存在时， `RENAME` 命令将覆盖旧值。

```bash
127.0.0.1:6379[1]> RENAME name name1
OK
127.0.0.1:6379[1]> KEYS *
1) "name1"
2) "age"
127.0.0.1:6379[1]> RENAME abc efg
(error) ERR no such key
```

## RENAMENX 在新key不存在时重命名key

 `RENAMENX old_key_name new_key_name`命令用于在新的 `key` 不存在时修改 `key` 的名称 。  修改成功时，返回 `1` 。 如果 `new_key_name` 已经存在，返回 `0` 。 


## DEL 删除key

```bash
127.0.0.1:6379[1]> DEL name
(integer) 1
127.0.0.1:6379[1]> GET name
(nil)
```



## SCAN 迭代键

`SCAN cursor [MATCH pattern] [COUNT count]` 命令是一个基于游标的迭代器，每次被调用之后， 都会向用户返回一个新的游标， 用户在下次迭代时需要使用这个新游标作为 `SCAN` 命令的游标参数， 以此来延续之前的迭代过程。

`SCAN` 返回一个包含两个元素的数组， 第一个元素是用于进行下一次迭代的新游标， 而第二个元素则是一个数组， 这个数组中包含了所有被迭代的元素。如果新游标返回`0` 表示迭代已结束。

- `cursor` - 游标。
- `pattern` - 匹配的模式。
- `count` - 指定从数据集里返回多少元素，默认值为 `10` 。

```bash
127.0.0.1:6379[1]> SCAN 0
1) "0"
2) 1) "age"
   2) "name1"
```



# Redis String命令

 ##  SET 保存

保存可以使用 `SET key value`，如果这个`key`已经存在，则会覆盖`value`。

```bash
127.0.0.1:6379[2]> SET name siri
OK
```

## GETSET 设置值并返回旧值

`GETSET`为指定的`key`设置新值，并返回旧值。

当 `key` 没有旧值时，即 `key` 不存在时，返回 `nil` 。

当 `key` 存在但不是字符串类型时，返回一个错误。

```bash
127.0.0.1:6379[1]> SET name zhangsan
OK
127.0.0.1:6379[1]> GETSET name lisi
"zhangsan"
```



## SETNX 只当key不存在时保存

 `SETNX（SET if Not eXists）` 命令在指定的 `key` 不存在时，为 `key` 设置指定的值。 设置成功返回`1`，否则返回`0`。

```bash
127.0.0.1:6379[1]> SETNX age 18
(integer) 1
127.0.0.1:6379[1]> SETNX name daming
(integer) 0
```

## SETEX 设置值并设置过期时间

`SETEX`设置值的同时并为其设置过期时间。

```bash
127.0.0.1:6379[1]> SETEX name 60 daming
OK
127.0.0.1:6379[1]> TTL name
(integer) 56
```

## MSET 批量保存

 `MSET` 命令用于同时设置一个或多个 `key-value` 对。 

```bash
127.0.0.1:6379[1]> MSET name xiaoming age 18
OK
127.0.0.1:6379[1]> GET name
"xiaoming"
127.0.0.1:6379[1]> get age
"18"
```

## MSETNX 所有key都不存在时，批量保存

`MSETNX`命令用于当所有`key`都不存在时，批量保存。只要有一个`key`已经存在，都会失败（原子性），成功返回`1`，失败返回`0`。

```bash
127.0.0.1:6379[1]> MSETNX name1 xiaohong age1 16
(integer) 1
127.0.0.1:6379[1]> MGET name1 age1
1) "xiaohong"
2) "16"
127.0.0.1:6379[1]> MSETNX name1 zhangsan age2 20
(integer) 0
```



## GET 读取

读取可以用 `GET key`，比如 `GET name` 。

```bash
127.0.0.1:6379[2]> GET name
"siri"
```



## MGET 批量读取

 `MGET` 命令用于同时读取一个或多个 值。

```bash
127.0.0.1:6379[1]> MSET name xiaoming age 18
OK
127.0.0.1:6379[1]> MGET name age
1) "xiaoming"
2) "18"
```



## STRLEN 读取字符串长度

 返回 `key` 所储存的字符串值的长度。 

```bash
127.0.0.1:6379[1]> STRLEN name
(integer) 4
```



## APPEND 追加字符

使用 `APPEND key value` 可以往该`key`所对应的值后面追加字符，比如`key name`的`value`是`siri`，那么运行`APPEND name ii`，会变成`siriii`。如果`key`不存在就相当于`SET key`。

```bash
127.0.0.1:6379[1]> SET name siri
OK
127.0.0.1:6379[1]> APPEND name ii
(integer) 6
127.0.0.1:6379[1]> GET name
"siriii"
```

## INCR 数字自增

`INCR` 命令将 `key` 中储存的数字值增一。

如果 `key` 不存在，那么 `key` 的值会先被初始化为 `0` ，然后再执行 `INCR` 操作。

如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

```bash
127.0.0.1:6379[1]> SET view 0
OK
127.0.0.1:6379[1]> INCR view
(integer) 1
127.0.0.1:6379[1]> GET view
"1"
```

## INCRBY 数字指定增量

`INCRBY` 命令将 `key` 中储存的数字加上指定的增量值（可以是负数）。

如果 `key` 不存在，那么 `key` 的值会先被初始化为 `0` ，然后再执行 `INCRBY` 命令。

如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

```bash
127.0.0.1:6379[1]> GET view
"1"
127.0.0.1:6379[1]> INCRBY view 10
(integer) 11
127.0.0.1:6379[1]> GET view
"11"
```

## INCRBYFLOAT 数字指定浮点增量

`INCRBYFLOAT`命令为 `key` 中所储存的值加上指定的浮点数增量值（可以是负数）。

如果 `key` 不存在，那么 `INCRBYFLOAT` 会先将 `key` 的值设为 `0` ，再执行加法操作。

```bash
127.0.0.1:6379[1]> SET my_float 11.5
OK
127.0.0.1:6379[1]> INCRBYFLOAT my_float 0.1
"11.6"
127.0.0.1:6379[1]> INCRBYFLOAT my_float -0.1
"11.5"
```

## DECR 数字自减

`DECR`命令将`key`存储的数字减一。

如果 `key` 不存在，那么`key` 的值会先被初始化为 `0` ，然后再执行 `DECR` 操作。

如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

```bash
127.0.0.1:6379[1]> SET view_2 100
OK
127.0.0.1:6379[1]> DECR view_2
(integer) 99
```

## DECRBY 数字指定减量

`DECRBY` 命令将 `key` 所储存的值减去指定的减量值。

如果 `key` 不存在，那么 `key` 的值会先被初始化为 `0` ，然后再执行 `DECRBY` 操作。

如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

```bash
127.0.0.1:6379[1]> SET view 100
OK
127.0.0.1:6379[1]> DECRBY view 50
(integer) 50
127.0.0.1:6379[1]> GET view
"50"
```

## GETRANGE 返回字符串子串

`GETRANGE` 命令用于获取存储在指定 `key` 中字符串的子字符串。字符串的截取范围由 `start` 和 `end` 两个偏移量决定(包括 `start` 和 `end` 在内)。`end`设置成`-1`表示最后一个，`-2`表示倒数第二个，以此类推。

```bash
127.0.0.1:6379[1]> SET astring "this is a string"
OK
127.0.0.1:6379[1]> GETRANGE astring 0 3
"this"
127.0.0.1:6379[1]> GETRANGE astring 0 -1
"this is a string"
```

## SETRANGE 覆盖字符串子串

`SETRANGE key offset value` 命令用指定的字符串覆盖给定 `key` 所储存的字符串值，覆盖的位置从偏移量 `offset` 开始。 

```bash
127.0.0.1:6379[1]> SET name xiaohong
OK
127.0.0.1:6379[1]> SETRANGE name 4 ming
(integer) 8
127.0.0.1:6379[1]> GET name
"xiaoming"
```

# Redis List命令

`Redis`列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）

一个列表最多可以包含 `2^32 - 1` 个元素 (`4294967295`, 每个列表超过`40亿`个元素)。

## LPLUSH 插入头部

`LPUSH` 命令将一个或多个值插入到列表头部。 如果 `key` 不存在，一个空列表会被创建并执行 `LPUSH` 操作。 当 `key` 存在但不是列表类型时，返回一个错误。 

```bash
127.0.0.1:6379[1]> LPUSH list 1 2 3 4 5
(integer) 5
```

## LPUSHX 将一个值插入到已存在的列表头部

`LPLUSHX`将**一个值**插入到已存在的列表头部，列表不存在时操作无效。

```bash
127.0.0.1:6379[1]> LPUSHX list 6
(integer) 6
127.0.0.1:6379[1]> LRANGE list 0 -1
1) "6"
2) "5"
3) "4"
4) "3"
5) "2"
6) "1"
127.0.0.1:6379[1]> LPUSHX list3 1
(integer) 0
127.0.0.1:6379[1]> LRANGE list3 0 -1
(empty list or set)
```



## RPLUSH 插入尾部

`RPUSH` 命令将一个或多个值插入到列表尾部。 如果 `key` 不存在，一个空列表会被创建并执行 `RPUSH` 操作。 当 `key` 存在但不是列表类型时，返回一个错误。 

```bash
127.0.0.1:6379[1]> RPUSH list2 1 2 3 4 5
(integer) 5
```

## RPUSHX 将一个值插入到已存在的列表尾部

`RPLUSHX`将**一个值**插入到已存在的列表尾部，列表不存在时操作无效。

```bash
127.0.0.1:6379[1]> RPUSH list2  1 2 3
(integer) 3
127.0.0.1:6379[1]> RPUSHX list2 4
(integer) 4
```



## LINSERT 在列表元素前后插入元素

`LINSERT key BEFORE|AFTER pivot value`命令用于在列表的元素前或者后插入元素。当指定元素不存在于列表中时，不执行任何操作（`pivot`为插入基准目标）。

当列表不存在时，被视为空列表，不执行任何操作。

如果 `key` 不是列表类型，返回一个错误。

```bash
127.0.0.1:6379[1]> RPUSH list2 1 2 3 4 5
(integer) 5
127.0.0.1:6379[1]> LINSERT list2 AFTER 5 6
(integer) 6
127.0.0.1:6379[1]> LRANGE list2 0 -1
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"
127.0.0.1:6379[1]> LINSERT list2 BEFORE 1 0
(integer) 7
127.0.0.1:6379[1]> LRANGE list2 0 -1
1) "0"
2) "1"
3) "2"
4) "3"
5) "4"
6) "5"
7) "6"
```



## LRANGE 返回指定区间的元素

 `LRANGE key start end`返回列表中指定区间内的元素（从左到右，从前往后），区间以偏移量 `start` 和 `end` 指定。 其中 `0` 表示列表的第一个元素， `1` 表示列表的第二个元素，以此类推。 你也可以使用负数下标，以` -1` 表示列表的最后一个元素， `-2` 表示列表的倒数第二个元素，以此类推。 

```bash
127.0.0.1:6379[1]> LRANGE list 0 -1
1) "5"
2) "4"
3) "3"
4) "2"
5) "1"
127.0.0.1:6379[1]> LRANGE list 0 1
1) "5"
2) "4"
127.0.0.1:6379[1]> LRANGE list2 0 -1
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
```

## LINDEX 通过索引获取元素

`LINDEX`命令通过索引获取列表中的元素。 你也可以使用负数下标，以 `-1` 表示列表的最后一个元素， `-2` 表示列表的倒数第二个元素，以此类推。 

```bash
127.0.0.1:6379[1]> LINDEX list2 0
"1"
```

## LSET 通过索引设置元素

`LSET`通过索引来设置元素的值。

 当索引参数超出范围，或对一个空列表进行 `LSET` 时，返回一个错误。 

```bash
127.0.0.1:6379[1]> RPUSH list a b c
(integer) 3
127.0.0.1:6379[1]> LSET list 0 b
OK
127.0.0.1:6379[1]> LRANGE list 0 -1
1) "b"
2) "b"
3) "c"
```



## LLEN 获取列表长度

`LLEN`命令用于获取列表的长度，如果列表不存在，则返回`0`，如果不是一个列表类型返回错误。

```bash
127.0.0.1:6379[1]> LLEN list
(integer) 5
127.0.0.1:6379[1]> LLEN list3
(integer) 0
```

## LPOP 移除并返回列表第一个元素

 `LPOP` 命令用于移除并返回列表的第一个元素。 

```bash
127.0.0.1:6379[1]> LPOP list
"5"
127.0.0.1:6379[1]> LRANGE list 0 -1
1) "4"
2) "3"
3) "2"
4) "1"
```

## BLPOP 阻塞移除并返回列表第一个元素

 `BLPOP list1 list2 ... timeout` 命令移出并获取给定列表（可以给定多个列表，会依次从左往右寻找，直到找到第一个元素有值的列表）的第一个元素， **如果列表没有元素会阻塞列表直到等待超时（时间以秒计）或发现可弹出元素为止**。 

 如果列表为空，返回一个 `nil` 。 否则，返回一个含有两个元素的列表，第一个元素是被弹出元素所属的 `key` ，第二个元素是被弹出元素的值。 

```bash
127.0.0.1:6379[1]> RPUSH list a
(integer) 1
127.0.0.1:6379[1]> BLPOP list 1
1) "list"
2) "a"
```

## RPOP 移除并返回列表最后一个元素

 `RPOP` 命令用于移除并返回列表的最后一个元素。 

```bash
127.0.0.1:6379[1]> RPUSH list 1 2 3 4 5
(integer) 5
127.0.0.1:6379[1]> RPOP list
"5"
127.0.0.1:6379[1]> LRANGE list 0 -1
1) "1"
2) "2"
3) "3"
4) "4"
```

## BRPOP 阻塞移除并返回列表最后一个元素

 `BRPOP list1 list2 ... timeout` 命令移出并获取给定列表（可以给定多个列表，会依次从左往右寻找，直到找到最后一个元素有值的列表）的最后一个元素， **如果列表没有元素会阻塞列表直到等待超时（时间以秒计）或发现可弹出元素为止**。 

 如果列表为空，返回一个 `nil` 。 否则，返回一个含有两个元素的列表，第一个元素是被弹出元素所属的 `key` ，第二个元素是被弹出元素的值。 

```bash
127.0.0.1:6379[1]> RPUSH list a
(integer) 1
127.0.0.1:6379[1]> BRPOP list 1
1) "list"
2) "a"
```



## RPOPLPUSH 移除列表最后一位，并追加到另一列表头部并返回

`RPOPLPUSH`命令用于移除列表最后一个元素，并将该元素添加到另一个列表头部并返回。

```bash
127.0.0.1:6379[1]> RPUSH list 1 2 3
(integer) 3
127.0.0.1:6379[1]> RPUSH list2 4 5 6
(integer) 3
127.0.0.1:6379[1]> RPOPLPUSH list list2
"3"
127.0.0.1:6379[1]> LRANGE list 0 -1
1) "1"
2) "2"
127.0.0.1:6379[1]> LRANGE list2 0 -1
1) "3"
2) "4"
3) "5"
4) "6"
```

## BRPOPLPUSH 阻塞移除列表最后一位，并追加到另一列表头部并返回

`BRPOPLPUSH list another_list timeout`命令用于移除列表最后一个元素，并将该元素添加到另一个列表头部并返回。 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。

## LREM 移除指定的值

`LREM key count value`根据参数 `count` 的值，移除列表中与参数 `value` 相等的元素。

`count` 的值可以是以下几种：

- `count > 0` : 从表头开始向表尾搜索，移除与 `value` 相等的元素，数量为 `count` 。
- `count < 0` : 从表尾开始向表头搜索，移除与 `value` 相等的元素，数量为 `count` 的绝对值。
- `count = 0` : 移除表中所有与 `value` 相等的值。

```bash
127.0.0.1:6379[1]> RPUSH list 1 2 1 2 1 2 1 3 1
(integer) 9
127.0.0.1:6379[1]> LREM list 1 1
(integer) 1
127.0.0.1:6379[1]> LRANGE list 0 -1
1) "2"
2) "1"
3) "2"
4) "1"
5) "2"
6) "1"
7) "3"
8) "1"
127.0.0.1:6379[1]> LREM list -1 1
(integer) 1
127.0.0.1:6379[1]> LRANGE list 0 -1
1) "2"
2) "1"
3) "2"
4) "1"
5) "2"
6) "1"
7) "3"
127.0.0.1:6379[1]> LREM list 0 1
(integer) 3
127.0.0.1:6379[1]> LRANGE list 0 -1
1) "2"
2) "2"
3) "2"
4) "3"
```

## LTRIM 修剪列表

`LTRIM`对一个列表进行修剪， 让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。 

```bash
127.0.0.1:6379[1]> RPUSH list 1 2 3 4 5
(integer) 5
127.0.0.1:6379[1]> LTRIM list 1 3
OK
127.0.0.1:6379[1]> LRANGE list 0 -1
1) "2"
2) "3"
3) "4"
```

# Redis Set命令

`Redis` 的 `Set` 是 `String` 类型的无序集合。集合成员是**唯一**的，这就意味着集合中不能出现重复的数据。

`Redis` 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 `O(1)`。

集合中最大的成员数为 `2^32 - 1` (`4294967295`, 每个集合可存储`40多亿`个成员)。

## SADD 添加

`SADD key value1 value2 ...`命令可以将一个或多个成员元素加入到集合中。已经存在于集合的成员元素将被忽略。

假如集合 `key` 不存在，则创建一个只包含添加的元素作成员的集合。

当集合 `key` 不是集合类型时，返回一个错误。

```bash
127.0.0.1:6379[1]> SADD myset a b c
(integer) 3
```

## SMEMBERS 查看所有成员

`SMEMBERS key`命令返回集合中的所有成员，不存在的集合`key`被视为空集合。

```bash
127.0.0.1:6379[1]> SMEMBERS myset
1) "a"
2) "c"
3) "b"
```

## SISMEMBER 判断元素是否是集合成员

`SISMEMBER key value`命令判断元素是否是给定集合的成员。是返回`1`，否返回`0`。

```bash
127.0.0.1:6379[1]> SISMEMBER myset a
(integer) 1
127.0.0.1:6379[1]> SISMEMBER myset d
(integer) 0
```

## SCARD 查看集合成员数量

`SCARD key`命令返回集合中成员的数量。如果`key`不存在返回`0`。

```bash
127.0.0.1:6379[1]> SCARD myset
(integer) 3
```

## SRENDMEMBER 返回集合中一个或多个随机元素

`SRENDMEMBER key [count]` 命令用于返回集合中的随机元素。

从 `Redis 2.6` 版本开始， 该 命令接受可选的 `count` 参数：

- 如果 `count` 为正数，且小于集合基数，那么命令返回一个包含 `count` 个元素的数组，数组中的元素各不相同。如果 `count` 大于等于集合基数，那么返回整个集合。
- 如果 `count` 为负数，那么命令返回一个数组，数组中的元素可能会重复出现多次，而数组的长度为 `count` 的绝对值。

该操作和 `SPOP` 相似，但 `SPOP` 将随机元素从集合中移除并返回，而 `SREANDMEMBER` 则仅仅返回随机元素，而不对集合进行任何改动。

```bash
127.0.0.1:6379[1]> SRANDMEMBER myset 2
1) "b"
2) "c"
127.0.0.1:6379[1]> SRANDMEMBER myset 2
1) "a"
2) "b"
```

## SPOP 移除集合中一个或多个随机元素并返回

`SPOP key [count]` 命令用于移除集合中一个或多个随机元素并返回。 `count` 参数在 `3.2+` 版本可用。 

 该命令类似 `SRANDMEMBER`命令，但 `SPOP` 将随机元素从集合中移除并返回，而 `SRANDMEMBER` 则仅仅返回随机元素，而不对集合进行任何改动。 

```bash
127.0.0.1:6379[1]> SPOP myset
"b"
127.0.0.1:6379[1]> SMEMBERS myset
1) "a"
2) "c"
```

## SREM 移除集合中指定成员

`SREM key member1 member2 ...`命令用于移除集合中指定的一个或多个成员，不存在的成员会被忽略。

```bash
127.0.0.1:6379[1]> SADD myset a b c d
(integer) 4
127.0.0.1:6379[1]> SREM myset b c e
(integer) 2
127.0.0.1:6379[1]> SMEMBERS myset
1) "d"
2) "a"
```

## SDIFF 返回第一个集合中独有的元素

`SDIFF key other_key other_key2 ...` 返回第一个集合中独有的元素。

```bash
127.0.0.1:6379[1]> SADD set1 a b c d e f g
(integer) 7
127.0.0.1:6379[1]> SADD set2 a b c e h
(integer) 5
127.0.0.1:6379[1]> SADD set3 d g i
(integer) 3
127.0.0.1:6379[1]> SDIFF set1 set2 set3
1) "f"
```

## SDIFFSTORE 将第一个集合中独有的元素存入到指定集合中

`SDIFFSTORE destination key [key ...]`命令等同于`SDIFF`，但是不返回结果集，而是将结果存储在`destination`这个集合中。如果`destination`已经存在，则会覆盖。

```bash
127.0.0.1:6379[1]> SADD set1 a b c d e f g
(integer) 7
127.0.0.1:6379[1]> SADD set2 a b c e h
(integer) 5
127.0.0.1:6379[1]> SADD set3 d g i
(integer) 3
127.0.0.1:6379[1]> SDIFFSTORE set set1 set2 set3
(integer) 1
127.0.0.1:6379[1]> SMEMBERS set
1) "f"
```

## SINTER 返回给定集合的交集

`SINTER key [key2 ...]`命令返回给定集合的交集。 不存在的键被认为是空集。其中一个键是一个空集，结果集也是空的（因为与空集的交集总是导致一个空集）。 

```bash
127.0.0.1:6379[1]> SADD set1 a b c d e
(integer) 5
127.0.0.1:6379[1]> SADD set2 b c d
(integer) 3
127.0.0.1:6379[1]> SADD set3 d e f
(integer) 3
127.0.0.1:6379[1]> SINTER set1 set2 set3
1) "d"
```

## SINTERSTORE 将给定集合的交集存储到指定集合中

`SINTERSTORE destination key [key2 ...]`命令等同于`SINTER`，但是不返回结果集，而是将结果存储在`destination`这个集合中。如果`destination`已经存在，则会覆盖。

```bash
127.0.0.1:6379[1]> SADD set1 a b c d e
(integer) 5
127.0.0.1:6379[1]> SADD set2 b c d
(integer) 3
127.0.0.1:6379[1]> SADD set3 d e f
(integer) 3
127.0.0.1:6379[1]> SINTERSTORE set set1 set2 set3
(integer) 1
127.0.0.1:6379[1]> SMEMBERS set
1) "d"
```

## SUNION 返回给定集合的并集

`SUNION key [key2 ...]`命令返回给定集合的并集。 不存在的键被认为是空集。

```bash
127.0.0.1:6379[1]> SADD set1 a b c
(integer) 3
127.0.0.1:6379[1]> SADD set2 d e f
(integer) 3
127.0.0.1:6379[1]> SUNION set1 set2
1) "a"
2) "b"
3) "c"
4) "f"
5) "d"
6) "e"
```

## SUNIONSTORE 将给定集合的并集存储到指定集合

`SUNION destination key [key2 ...]`命令同`SUNION`命令，但是不返回结果集，而是将结果存储在`destination`这个集合中。如果`destination`已经存在，则会覆盖。

```bash
127.0.0.1:6379[1]> SADD set1 a b c
(integer) 3
127.0.0.1:6379[1]> SADD set2 d e f
(integer) 3
127.0.0.1:6379[1]> SUNIONSTORE set set1 set2
(integer) 6
127.0.0.1:6379[1]> SMEMBERS set
1) "a"
2) "b"
3) "c"
4) "f"
5) "d"
6) "e"
```



## SMOVE 移动一个成员到另一个集合

`SMOVE source destionation member`命令将指定成员 `member` 元素从 `source` 集合移动到 `destination` 集合。

`SMOVE` 是原子性操作。

如果 `source` 集合不存在或不包含指定的 `member` 元素，则 `SMOVE` 命令不执行任何操作，仅返回 `0` 。否则， `member` 元素从 `source` 集合中被移除，并添加到 `destination` 集合中去。

当 `destination` 集合已经包含 `member` 元素时， `SMOVE` 命令只是简单地将 `source` 集合中的 `member` 元素删除。

当 `source` 或 `destination` 不是集合类型时，返回一个错误。

```bash
127.0.0.1:6379[1]> SADD set a b c
(integer) 3
127.0.0.1:6379[1]> SMOVE set set1 a
(integer) 1
127.0.0.1:6379[1]> SMEMBERS set1
1) "a"
127.0.0.1:6379[1]> SMEMBERS set
1) "c"
2) "b"
```

## SSCAN 迭代集合中的成员

`SSAN key cursor [MATCH pattern] [COUNT count]`命令用于迭代集合中键的成员。

- `cursor` - 游标。
- `pattern` - 匹配的模式。
- `count` - 指定从数据集里返回多少元素，默认值为 `10` 。

```bash
127.0.0.1:6379[1]> SSCAN set 0
1) "0"
2) 1) "f"
   2) "a"
   3) "d"
   4) "e"
   5) "b"
   6) "c"
```

# Redis Sorted Set命令

`Redis` 有序集合和集合一样也是`string`类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个`double`类型的分数。`redis`正是通过分数来为集合中的成员进行从小到大的排序。

有序集合的成员是唯一的,但分数(`score`)却可以重复。

## ZADD 插入

`ZADD key score1 value1 score2 value2 ...` 命令用于将一个或多个成员元素及其分数值加入到有序集当中。

如果某个成员已经是有序集的成员，那么更新这个成员的分数值，并通过重新插入这个成员元素，来保证该成员在正确的位置上。

分数值可以是整数值或双精度浮点数。

如果有序集合 `key` 不存在，则创建一个空的有序集并执行 `ZADD` 操作。

当 `key` 存在但不是有序集类型时，返回一个错误。

```bash
127.0.0.1:6379[1]> ZADD zset 1 one 2 two
(integer) 2
```

## ZRANGE 查看指定区间成员

`ZRANGE key start stop [WITHSCORES]`返回有序集中，指定区间内的成员。

其中成员的位置按分数值递增(从小到大)来排序。

具有相同分数值的成员按字典序来排列。

如果你需要成员按值递减(从大到小)来排列，请使用 `ZREVRANGE`命令。

下标参数 `start` 和 `stop` 都以 `0` 为底，也就是说，以 `0` 表示有序集第一个成员，以 `1` 表示有序集第二个成员，以此类推。

你也可以使用负数下标，以 `-1` 表示最后一个成员， `-2` 表示倒数第二个成员，以此类推。

`WITHSCORES`选项可以让分数也一并返回。

```bash
127.0.0.1:6379[1]> ZADD zset 1 one 2 two
(integer) 2
127.0.0.1:6379[1]> ZRANGE zset 0 -1
1) "one"
2) "two"
127.0.0.1:6379[1]> ZRANGE zset 0 -1 WITHSCORES
1) "one"
2) "1"
3) "two"
4) "2"
```

