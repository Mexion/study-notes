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


 # Redis key

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



# Redis String

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

# Redis List

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

# Redis Set

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

# Redis Sorted Set

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

## ZRANGE 从低到高查看指定区间成员

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

## ZREVRANGE 从高到低查看指定区间成员

`ZREVRANGE key start stop [WITHSCORES]`返回有序集中，指定区间内的成员。

其中成员的位置按分数值递减(从大到小)来排序。

具有相同分数值的成员按字典序逆序来排列。

如果你需要成员按值递减(从大到小)来排列，请使用 `ZREVRANGE`命令。

下标参数 `start` 和 `stop` 都以 `0` 为底，也就是说，以 `0` 表示有序集第一个成员，以 `1` 表示有序集第二个成员，以此类推。

你也可以使用负数下标，以 `-1` 表示最后一个成员， `-2` 表示倒数第二个成员，以此类推。

`WITHSCORES`选项可以让分数也一并返回。

```bash
127.0.0.1:6379[1]> ZADD zset 1 one 2 two
(integer) 2
127.0.0.1:6379[1]> ZREVRANGE zset 0 -1
1) "two"
2) "one"
127.0.0.1:6379[1]> ZREVRANGE zset 0 -1 WITHSCORES
1) "two"
2) "2"
3) "one"
4) "1"
```

## ZRANGEBYSCORE 从小到大返回指定分数区间的成员

`ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]`命令返回指定分数区间的成员列表。

`min`和`max`指定分数区间，包括分数等于`min`或者`max`的成员（闭区间），如果要指定开区间，可以在`min`或者`max`前增加`(`符号。 `min`和`max`可以是`-inf`和`+inf`，让你不需要知道的有序集合最高或最低分数来自或达到一定的分数获得的所有元素。 这些元素按照分数**从低到高**排序。

`LIMIT offset count`的用法 和`SQL`中的基本一致，从`offset`开始，取`count`条，比如`LIMIT 2  2`，就是从第二条开始取两条。 注意当 `offset` 很大时，定位 `offset` 的操作可能需要遍历整个有序集，此过程最坏复杂度为 `O(N)` 时间。 

```bash
127.0.0.1:6379[1]> ZADD zset 1 one 2 two 3 three 4 four 5 five 6 six
(integer) 6
127.0.0.1:6379[1]> ZRANGEBYSCORE zset 2 5
1) "two"
2) "three"
3) "four"
4) "five"
127.0.0.1:6379[1]> ZRANGEBYSCORE zset 2 5 WITHSCORES
1) "two"
2) "2"
3) "three"
4) "3"
5) "four"
6) "4"
7) "five"
8) "5"
127.0.0.1:6379[1]> ZRANGEBYSCORE zset (2 5 WITHSCORES
1) "three"
2) "3"
3) "four"
4) "4"
5) "five"
6) "5"
127.0.0.1:6379[1]> ZRANGEBYSCORE zset -inf +inf
1) "one"
2) "two"
3) "three"
4) "four"
5) "five"
6) "six"
127.0.0.1:6379[1]> ZRANGEBYSCORE zset -inf +inf LIMIT 2 2
1) "three"
2) "four"
```

## ZRANGEBYLEX 通过字典区间返回成员

`ZRANGEBYLEX key min max [LIMIT offset count]`命令通过字典区间返回有序集合的成员。

当有序集合的所有成员都具有相同的分值时， 有序集合的元素会根据成员的字典序来进行排序， 而这个命令则可以返回给定的有序集合键 `key` 中， 值介于 `min` 和 `max` 之间的成员。

如果有序集合里面的成员带有不同的分值， 那么命令返回的结果是未指定的。

命令会使用 `C 语言`的 `memcmp()` 函数， 对集合中的每个成员进行逐个字节的对比， 并按照从低到高的顺序， 返回排序后的集合成员。 如果两个字符串有一部分内容是相同的话， 那么命令会认为较长的字符串比较短的字符串要大。

可选的 `LIMIT offset count` 参数用于获取指定范围内的匹配元素 。 需要注意的一点是， 如果 `offset` 参数的值非常大的话， 那么命令在返回结果之前， 需要先遍历至 `offset` 所指定的位置， 这个操作会为命令加上最多 `O(N) `复杂度。

合法的 `min` 和 `max` 参数必须包含 `(` 或者 `[` ， 其中 `(` 表示开区间（指定的值不会被包含在范围之内）， 而 `[` 则表示闭区间（指定的值会被包含在范围之内）。

特殊值 `+` 和 `-` 在 `min` 参数以及 `max` 参数中具有特殊的意义， 其中 `+` 表示正无限， 而 `-` 表示负无限。 因此， 向一个所有成员的分值都相同的有序集合发送命令 `ZRANGEBYLEX - +` ， 命令将返回有序集合中的所有元素。

```bash
127.0.0.1:6379> ZADD myzset 0 a 0 b 0 c 0 d 0 e 0 f 0 g
(integer) 7

127.0.0.1:6379> ZRANGEBYLEX myzset - [c
1) "a"
2) "b"
3) "c"

127.0.0.1:6379 ZRANGEBYLEX myzset - (c
1) "a"
2) "b"

127.0.0.1:6379> ZRANGEBYLEX myzset [aaa (g
1) "b"
2) "c"
3) "d"
4) "e"
5) "f"
```



## ZREVRANGEBYSCORE 从大到小返回指定分数区间的成员

`ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]`命令返回指定分数区间的成员列表。

`max`和`min`指定分数区间，包括分数等于`max`或者`min`的成员（闭区间），如果要指定开区间，可以在`max`或者`min`前增加`(`符号。 `max`和`min`可以是`+inf`和`-inf`，让你不需要知道的有序集合最高或最低分数来自或达到一定的分数获得的所有元素。 这些元素按照分数**从高到低**排序。

`LIMIT offset count`的用法 和`SQL`中的基本一致，从`offset`开始，取`count`条，比如`LIMIT 2  2`，就是从第二条开始取两条。 注意当 `offset` 很大时，定位 `offset` 的操作可能需要遍历整个有序集，此过程最坏复杂度为 `O(N)` 时间。 

```bash
127.0.0.1:6379[1]> ZADD zset 1 one 2 two 3 three 4 four 5 five 6 six
(integer) 6
127.0.0.1:6379[1]> ZREVRANGEBYSCORE zset +inf -inf
1) "six"
2) "five"
3) "four"
4) "three"
5) "two"
6) "one"
127.0.0.1:6379[1]> ZREVRANGEBYSCORE zset 4 1 WITHSCORES
1) "four"
2) "4"
3) "three"
4) "3"
5) "two"
6) "2"
7) "one"
8) "1"
127.0.0.1:6379[1]> ZREVRANGEBYSCORE zset 4 1 WITHSCORES LIMIT 1 2
1) "three"
2) "3"
3) "two"
4) "2"
```

## ZRANK 返回指定成员从小到大的排名

`ZRANK key member` 返回有序集 `key` 中成员 `member` 的排名。其中有序集成员按 `score` 值递增(从小到大)顺序排列。 

 排名以 `0` 为底，也就是说， `score` 值最小的成员排名为 `0` 。 

```bash
127.0.0.1:6379> ZADD zset 1 one 2 two 3 three 4 four 5 five
(integer) 5
127.0.0.1:6379> ZRANK zset one
(integer) 0
127.0.0.1:6379> ZRANK zset four
(integer) 3
```

## ZREVRANK 返回指定成员从大到小的排名

`ZREVRANK key member` 返回有序集 `key` 中成员 `member` 的排名。其中有序集成员按 `score` 值递减(从大到小)顺序排列。 

 排名以 `0` 为底，也就是说， `score` 值最大的成员排名为 `0` 。 

```bash
127.0.0.1:6379> ZADD zset 1 one 2 two 3 three 4 four 5 five
(integer) 5
127.0.0.1:6379> ZREVRANK zset one
(integer) 4
127.0.0.1:6379> ZREVRANK zset four
(integer) 1
```

## ZCARD 返回所有成员数量

`ZCARD key`可以返回有序集合中成员的数量。

```bash
127.0.0.1:6379> ZADD zset 1 one 2 two 3 three
(integer) 3
127.0.0.1:6379> ZCARD zset
(integer) 3
```

## ZCOUNT  返回指定区间的成员数量

`ZCOUNT key min max`返回有序集合中指定分数区间的成员数量。

```bash
127.0.0.1:6379> ZADD zset 1 one 2 two 3 three
(integer) 3
127.0.0.1:6379> ZCOUNT zset 2 3
(integer) 2
127.0.0.1:6379> ZCOUNT zset 2 2
(integer) 1
127.0.0.1:6379> ZCOUNT zset (2 3
(integer) 1
```

## ZLEXCOUNT 通过字典区间返回指定范围的成员

`ZLEXCOUNT key min max` 对于一个所有成员的分值都相同的有序集合键 `key` 来说， 这个命令会返回该集合中， 成员介于 `min` 和 `max` 范围内的元素数量。 

```bash
127.0.0.1:6379> ZADD myzset 0 a 0 b 0 c 0 d 0 e
(integer) 5

127.0.0.1:6379> ZADD myzset 0 f 0 g
(integer) 2

127.0.0.1:6379> ZLEXCOUNT myzset - +
(integer) 7

127.0.0.1:6379> ZLEXCOUNT myzset [b [f
(integer) 5
```



## ZSCORE 返回指定成员的分数

`ZSCORE key member`返回有序集 `key` 中，成员 `member` 的 `score` 值。

如果 `member` 元素不是有序集 `key` 的成员，或 `key` 不存在，返回 `nil` 。

```bash
127.0.0.1:6379> ZADD zset 2 a
(integer) 1
127.0.0.1:6379> ZSCORE zset a
"2"
```



## ZINCRBY 为成员指定增量

`ZINCRBY key increment menber`为有序集 `key` 的成员 `member` 的 `score` 值加上增量 `increment` 。

可以通过传递一个负数值 `increment` ，让 `score` 减去相应的值，比如 `ZINCRBY key -5 member` ，就是让 `member` 的 `score` 值减去 `5` 。

当 `key` 不存在，或 `member` 不是 `key` 的成员时， `ZINCRBY key increment member` 等同于 `ZADD key increment member` 。

当 `key` 不是有序集类型时，返回一个错误。

`score` 值可以是整数值或双精度浮点数。

```bash
127.0.0.1:6379> ZADD zset 1 a
(integer) 1
127.0.0.1:6379> ZINCRBY zset 4 a
"5"
127.0.0.1:6379> ZSCORE zset a
"5"
127.0.0.1:6379> ZINCRBY zset -2 a
"3"
127.0.0.1:6379> ZSCORE zset a
"3"
```

## ZREM 移除一个或多个成员

`ZREM key member [member2 ...]`命令用于移除有序集中的一个或多个成员，不存在的成员将被忽略。

当 `key` 存在但不是有序集类型时，返回一个错误。

```bash
127.0.0.1:6379> ZADD zset 1 one 2 two 3 three
(integer) 3
127.0.0.1:6379> ZREM zset one two
(integer) 2
127.0.0.1:6379> ZRANGE zset 0 -1
1) "three"
```

## ZREMRANGEBYRANK 移除指定排名区间内的所有成员

`ZREMRANGEBYRARNK key start stop`命令用于移除有序集中，指定排名区间内的所有成员。

```bash
127.0.0.1:6379> ZADD zset 1 one 2 two 3 three
(integer) 3
127.0.0.1:6379> ZREMRANGEBYRANK zset 0 1
(integer) 2
127.0.0.1:6379> ZRANGE zset 0 -1
1) "three"
```

## ZREMRANGEBYSCORE 移除指定分数区间内的所有成员

`ZREMRANGEBYSCORE key min max` 命令用于移除有序集中，指定分数区间内的所有成员。 

```bash
127.0.0.1:6379> ZADD zset 1 one 2 two 3 three
(integer) 3
127.0.0.1:6379> ZREMRANGEBYSCORE zset 2 3
(integer) 2
127.0.0.1:6379> ZRANGE zset 0 -1
1) "one"
```

## ZREMRANGEBYLEX 移除给定字典区间的所有成员

`ZREMRANGEBYLEX key min max`命令用于移除有序集合中，指定字典区间的所有成员。

```bash
127.0.0.1:6379> ZADD myzset 0 aaaa 0 b 0 c 0 d 0 e
(integer) 5

127.0.0.1:6379> ZADD myzset 0 foo 0 zap 0 zip 0 ALPHA 0 alpha
(integer) 5

127.0.0.1:6379> ZRANGE myzset 0 -1
1) "ALPHA"
2) "aaaa"
3) "alpha"
4) "b"
5) "c"
6) "d"
7) "e"
8) "foo"
9) "zap"
10) "zip"

127.0.0.1:6379> ZREMRANGEBYLEX myzset [alpha [omega
(integer) 6

127.0.0.1:6379> ZRANGE myzset 0 -1
1) "ALPHA"
2) "aaaa"
3) "zap"
4) "zip"
```

## ZINTERSTORE 将多个有序集合的交集存储到另一有序集合中

`ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]`命令 计算给定的一个或多个有序集的交集，其中给定 `key` 的数量必须以 `numkeys` 参数指定，并将该交集储存到 `destination` 。 

使用 `WEIGHTS` 选项，你可以为 *每个* 给定有序集 *分别* 指定一个乘法因子，每个给定有序集的所有成员的 `score` 值在传递给聚合函数之前都要先乘以该有序集的因子。

如果没有指定 `WEIGHTS` 选项，乘法因子默认设置为 `1` 。

使用 `AGGREGATE` 选项，你可以指定交集的结果集的聚合方式。

默认使用的参数 `SUM` ，可以将所有集合中某个成员的 `score` 值之 *和* 作为结果集中该成员的 `score` 值；使用参数 `MIN` ，可以将所有集合中某个成员的 *最小* `score` 值作为结果集中该成员的 `score` 值；而参数 `MAX` 则是将所有集合中某个成员的 *最大* `score` 值作为结果集中该成员的 `score` 值。

```bash
127.0.0.1:6379> ZADD mid_test 70 "Li Lei"
(integer) 1
redis 127.0.0.1:6379> ZADD mid_test 70 "Han Meimei"
(integer) 1
redis 127.0.0.1:6379> ZADD mid_test 99.5 "Tom"
(integer) 1

127.0.0.1:6379> ZADD fin_test 88 "Li Lei"
(integer) 1
redis 127.0.0.1:6379> ZADD fin_test 75 "Han Meimei"
(integer) 1
redis 127.0.0.1:6379> ZADD fin_test 99.5 "Tom"
(integer) 1

127.0.0.1:6379> ZINTERSTORE sum_point 2 mid_test fin_test
(integer) 3


127.0.0.1:6379> ZRANGE sum_point 0 -1 WITHSCORES     
1) "Han Meimei"
2) "145"
3) "Li Lei"
4) "158"
5) "Tom"
6) "199"

```

## ZUIONSTORE 将多个有序集合的并集存储到另一有序集合中

`ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]`将多个有序集合的并集存储到`destination`中。用法与`ZINTERSTORE`一致。

```bash
127.0.0.1:6379> ZRANGE programmer 0 -1 WITHSCORES
1) "peter"
2) "2000"
3) "jack"
4) "3500"
5) "tom"
6) "5000"

127.0.0.1:6379> ZRANGE manager 0 -1 WITHSCORES
1) "herry"
2) "2000"
3) "mary"
4) "3500"
5) "bob"
6) "4000"

127.0.0.1:6379> ZUNIONSTORE salary 2 programmer manager WEIGHTS 1 3  
(integer) 6

127.0.0.1:6379> ZRANGE salary 0 -1 WITHSCORES
1) "peter"
2) "2000"
3) "jack"
4) "3500"
5) "tom"
6) "5000"
7) "herry"
8) "6000"
9) "mary"
10) "10500"
11) "bob"
12) "12000"
```

## ZSAN 迭代有序集合中的成员

`ZSAN key cursor [MATCH pattern] [COUNT count]`命令用于迭代有序集合中的元素（包括元素成员和元素分值）。

- `cursor` - 游标。
- `pattern` - 匹配的模式。
- `count` - 指定从数据集里返回多少元素，默认值为 `10` 。

```bash
127.0.0.1:6379> ZADD site 1 "Google" 2 "Tencent" 3 "Taobao" 4 "Weibo"
(integer) 4
127.0.0.1:6379> ZSCAN site 0
1) "0"
2) 1) "Google"
   2) "1"
   3) "Tencent"
   4) "2"
   5) "Taobao"
   6) "3"
   7) "Weibo"
   8) "4"
```

## Redis Hash

 `Redis hash` 是一个 `string` 类型的 `field（字段）` 和 `value（值）` 的映射表，`hash` 特别适合用于存储对象。 

## HSET 设置hash字段

`HSET hash field value`命令用于将`hash`中的`field`的值设置成`value`。

如果给定的哈希表并不存在， 那么一个新的哈希表将被创建并执行 `HSET` 操作。

如果域 `field` 已经存在于哈希表中， 那么它的旧值将被新值 `value` 覆盖。

 当 `HSET` 命令在哈希表中新创建 `field` 域并成功为它设置值时， 命令返回 `1` ； 如果域 `field` 已经存在于哈希表， 并且 `HSET` 命令成功使用新值覆盖了它的旧值， 那么命令返回 `0` 。 

```bash
127.0.0.1:6379> HSET website google "www.google.com"
(integer) 1
```

## HGET 获取hash字段

`HGET hash field`命令在默认情况下返回给定字段的值。

如果给定字段不存在于哈希表中， 又或者给定的哈希表并不存在， 那么命令返回 `nil` 。

```bash
127.0.0.1:6379> HSET website google "www.google.com"
(integer) 1
127.0.0.1:6379> HGET website google
"www.google.com"
```

## HSETNX 仅当字段未存在于hash中时设置值

`HSETNX hash field value`用法与`HSET`一致，但仅当 `field` 尚未存在于哈希表的情况下， 将它的值设置为 `value` 。

如果给定字段已经存在于哈希表当中， 那么命令将放弃执行设置操作。

如果哈希表 `hash` 不存在， 那么一个新的哈希表将被创建并执行 `HSETNX` 命令。

```bash
127.0.0.1:6379> HSET website google "www.google.com"
(integer) 1
127.0.0.1:6379> HSETNX website tencent "www.tencent.com"
(integer) 1
127.0.0.1:6379> HSET website google "www.google.com"
(integer) 0
```

## HMSET 将多个键值对设置到hash中

`HMSET hash field value [field value ...]`同时将多个 `field-value` (字段-值)对设置到哈希表 `hash` 中。

此命令会覆盖哈希表中已存在的字段。

如果 `hash` 不存在，一个空哈希表被创建并执行`HMSET`操作。

```bash
127.0.0.1:6379> HMSET website google www.google.com yahoo www.yahoo.com
OK
127.0.0.1:6379> HGET website google
"www.google.com"
127.0.0.1:6379> HGET website yahoo
"www.yahoo.com"
```

## HMGET 返回一个或多个给定字段的值

`HMGET hash field [field ...]`返回哈希表中一个或多个字段的值。

如果给定的字段不存在于哈希表，那么返回一个 `nil` 值。

```bash
127.0.0.1:6379> HMSET pet dog "doudou" cat "nounou"    
OK

127.0.0.1:6379> HMGET pet dog cat fake_pet   
1) "doudou"
2) "nounou"
3) (nil) 
```



## HEXISTS 检查字段是否在hash中

`HEXISTS hash field`检查给定域 `field` 是否存在于哈希表 `hash` 当中。

存在时返回 `1` ， 不存在时返回 `0`。

```bash
127.0.0.1:6379> HSET website google "www.google.com"
(integer) 1
127.0.0.1:6379> HEXISTS website google
(integer) 1
```

## HLEN 返回hash中字段数量

`HLEN hash`返回该`hash`中字段的数量。

```bash
127.0.0.1:6379> HSET website google "www.google.com"
(integer) 1
127.0.0.1:6379> HSETNX website tencent "www.tencent.com"
(integer) 1
127.0.0.1:6379> HLEN website
(integer) 2
```

## HSTRLEN 返回给定字段相关联的值的字符串长度

`HSTRLEN hash field`返回哈希表 `hash` 中， 与给定字段 `field` 相关联的值的字符串长度。

如果给定的键或者字段不存在， 那么命令返回 `0` 。

```bash
127.0.0.1:6379> HMSET myhash f1 "HelloWorld" f2 "99" f3 "-256"
OK

127.0.0.1:6379> HSTRLEN myhash f1
(integer) 10

127.0.0.1:6379> HSTRLEN myhash f2
(integer) 2

127.0.0.1:6379> HSTRLEN myhash f3
(integer) 4
```



## HGETALL 返回hash中所有的字段和值

`HGETALL hash`返回`hash`中所有的字段和值。 在返回值里，紧跟每个字段之后是字段的值，所以返回值的长度是哈希表大小的两倍。 

```bash
127.0.0.1:6379> HSET website google "www.google.com"
(integer) 1
127.0.0.1:6379> HSETNX website tencent "www.tencent.com"
(integer) 1
127.0.0.1:6379> HGETALL website
1) "google"
2) "www.google.com"
3) "tencent"
4) "www.tencent.com"
```

## HKEYS 返回hash中所有的字段

`HKEYS hash`返回哈希表中所有字段。

```bash
127.0.0.1:6379> HSET website google "www.google.com"
(integer) 1
127.0.0.1:6379> HSETNX website tencent "www.tencent.com"
(integer) 1
127.0.0.1:6379> HKEYS website
1) "google"
2) "tencent"
```

## HVALS  返回hash中所有的值

`HVALS hash`返回哈希表汇总所有的值。

```bash
127.0.0.1:6379> HSET website google "www.google.com"
(integer) 1
127.0.0.1:6379> HSETNX website tencent "www.tencent.com"
(integer) 1
127.0.0.1:6379> HVALS website
1) "www.google.com"
2) "www.tencent.com"
```



## HDEL 删除hash中一个或多个字段

`HDEL hash field [field ...]`删除`hash`中一个或多个指定字段，不存在的字段将被忽略。

```bash
127.0.0.1:6379> HSET website google "www.google.com"
(integer) 1
127.0.0.1:6379> HSETNX website tencent "www.tencent.com"
(integer) 1
127.0.0.1:6379> HDEL website tencent
(integer) 1
127.0.0.1:6379> HVALS website
1) "www.google.com"
```



## HINCRBY 为指定字段的值加上增量

`HINCRBY hash field increment`为哈希表 `hash` 中的字段 `field` 的值加上增量 `increment` 。

增量也可以为负数，相当于对给定域进行减法操作。

如果 `hash` 不存在，一个新的哈希表被创建并执行 `HINCRBY`命令。

如果字段 `field` 不存在，那么在执行命令前，字段的值被初始化为 `0` 。

对一个储存字符串值的字段 `field` 执行 `HINCRBY`命令将造成一个错误。

```bash
127.0.0.1:6379> HSET counter views 10
(integer) 1
127.0.0.1:6379> HINCRBY counter views 5
(integer) 15
127.0.0.1:6379> HGET counter views
"15"
```

## HINCRBYFLOAT 为指定字段的值加上浮点数增量

`HINCRBYFLOAT hash field increment`为哈希表 `hash` 中的字段 `field` 的值加上浮点数增量 `increment` 。

```bash
127.0.0.1:6379> HSET counter views 10
(integer) 1
127.0.0.1:6379> HINCRBY counter views 5
(integer) 15
127.0.0.1:6379> HGET counter views
"15"
127.0.0.1:6379> HINCRBYFLOAT counter views 1.5
"16.5"
```

## HSCAN 迭代hash

`HSCAN hash cursor [MATCH pattern] [COUNT count]`用于迭代哈希表中的键值对。

- `cursor` - 游标。
- `pattern` - 匹配的模式。
- `count` - 指定从数据集里返回多少元素，默认值为 `10` 。

```bash
127.0.0.1:6379> HMSET website google www.google.com tencent www.tencent.com yahoo www.yahoo.com baidu www.baidu.com
OK
127.0.0.1:6379> HSCAN website 0
1) "0"
2) 1) "google"
   2) "www.google.com"
   3) "tencent"
   4) "www.tencent.com"
   5) "yahoo"
   6) "www.yahoo.com"
   7) "baidu"
   8) "www.baidu.com"
```

# Redis Geo

`Redis GEO` 主要用于存储地理位置信息，并对存储的信息进行操作（`version 3.2+`）。

`Redis GEO` 操作方法有：

- `GEOADD`：添加地理位置的坐标。
- `GEOPOS`：获取地理位置的坐标。
- `GEODIST`：计算两个位置之间的距离。
- `GEORADIUS`：根据用户给定的经纬度坐标来获取指定范围内的地理位置集合。
- `GEORADIUSBYMEMBER`：根据储存在位置集合里面的某个地点获取指定范围内的地理位置集合。
- `GEOHASH`：返回一个或多个位置对象的 `geohash` 值。

`Geo`的底层实现其实是`Sorted Set`，所以可以使用`Sorted Set`命令来操作`Geo`。

## GEOADD 添加地理位置坐标

`GEOADD key longitude latitude member [longtitude latitude member ...]` 用于存储指定的地理空间位置，可以将一个或多个经度(`longitude`)、纬度(`latitude`)、位置名称(`member`)添加到指定的 `key` 中。 

这些数据会以有序集合的形式被储存在键里面， 从而使得像 `GEORADIUS` 和 `GEORADIUSBYMEMBER` 这样的命令可以在之后通过位置查询取得这些元素。

`GEOADD` 命令以标准的 `x,y` 格式接受参数， 所以用户必须先输入经度， 然后再输入纬度。 `GEOADD` 能够记录的坐标是有限的： 非常接近两极的区域是无法被索引的。 精确的坐标限制由 `EPSG:900913 / EPSG:3785 / OSGEO:41001` 等坐标系统定义， 具体如下：

- 有效的经度介于 `-180` 度至 `180` 度之间。
- 有效的纬度介于 `-85.05112878` 度至 `85.05112878` 度之间。

当用户尝试输入一个超出范围的经度或者纬度时， `GEOADD` 命令将返回一个错误。

```bash
127.0.0.1:6379> GEOADD city 116.40 39.90 beijing
(integer) 1
127.0.0.1:6379> GEOADD city 106.50 29.53 chongqing 114.05 22.52 shenzhen
(integer) 2
127.0.0.1:6379> GEOADD city 121.47 31.23 shanghai
(integer) 1
127.0.0.1:6379> GEOADD city 120.16 30.24 hangzhou 108.96 34.26 xian
(integer) 2
```

## GEOPOS 返回指定名称的经纬度

`GEOPOS key member [member ...]`从`key`中返回所有给定`member`的经纬度。

 `GEOPOS` 命令返回一个数组， 数组中的每个项都由两个元素组成： 第一个元素为给定位置元素的经度， 而第二个元素则为给定位置元素的纬度。 当给定的位置元素不存在时， 对应的数组项为空值。 

 因为 `GEOPOS` 命令接受可变数量的位置元素作为输入， 所以即使用户只给定了一个位置元素， 命令也会返回数组回复。 

```bash
127.0.0.1:6379> GEOPOS city beijing
1) 1) "116.39999896287918"
   2) "39.900000091670925"
127.0.0.1:6379> GEOPOS city shanghai shenzhen
1) 1) "121.47000163793564"
   2) "31.229999039757836"
2) 1) "114.04999762773514"
   2) "22.520000087950386"
```

## GEODIST 返回两个位置之间的直线距离

`GEODIST key member1 member2 [unit]`返回两个给定位置之间的距离。

如果两个位置之间的其中一个不存在， 那么命令返回空值。

指定单位的参数 `unit` 必须是以下单位的其中一个：

- `m` 表示单位为米。
- `km` 表示单位为千米。
- `mi` 表示单位为英里。
- `ft` 表示单位为英尺。

如果用户没有显式地指定单位参数， 那么 `GEODIST` 默认使用米作为单位。

`GEODIST` 命令在计算距离时会假设地球为完美的球形， 在极限情况下， 这一假设最大会造成 `0.5%` 的误差。

```bash
127.0.0.1:6379> GEODIST city beijing shanghai
"1067378.7564"
127.0.0.1:6379> GEODIST city beijing shanghai km
"1067.3788"
```

## GEORADIUS 返回以指定位置为中心指定距离内的元素

`GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [ASC|DESC] [COUNT count]`

以给定的经纬度为中心， 返回键包含的位置元素当中， 与中心的距离不超过给定最大距离的所有位置元素。

范围可以使用以下其中一个单位：

- `m` 表示单位为米。
- `km` 表示单位为千米。
- `mi` 表示单位为英里。
- `ft` 表示单位为英尺。

在给定以下可选项时， 命令会返回额外的信息：

- `WITHDIST` ： 在返回位置元素的同时， 将位置元素与中心之间的距离也一并返回。 距离的单位和用户给定的范围单位保持一致。
- `WITHCOORD` ： 将位置元素的经度和维度也一并返回。
- `WITHHASH` ： 以 `52` 位有符号整数的形式， 返回位置元素经过原始 `geohash` 编码的有序集合分值。 这个选项主要用于底层应用或者调试， 实际中的作用并不大。

命令默认返回未排序的位置元素。 通过以下两个参数， 用户可以指定被返回位置元素的排序方式：

- `ASC` ： 根据中心的位置， 按照从近到远的方式返回位置元素。
- `DESC` ： 根据中心的位置， 按照从远到近的方式返回位置元素。

在默认情况下， `GEORADIUS` 命令会返回所有匹配的位置元素。 虽然用户可以使用 `COUNT ` 选项去获取前 N 个匹配元素， 但是因为命令在内部可能会需要对所有被匹配的元素进行处理， 所以在对一个非常大的区域进行搜索时， 即使只使用 `COUNT` 选项去获取少量元素， 命令的执行速度也可能会非常慢。 但是从另一方面来说， 使用 `COUNT` 选项去减少需要返回的元素数量， 对于减少带宽来说仍然是非常有用的。

`GEORADIUS` 命令返回一个数组， 具体来说：

- 在没有给定任何 `WITH` 选项的情况下， 命令只会返回一个像 `["beijing","shanghai","shenzhen"]` 这样的线性列表。
- 在指定了 `WITHCOORD` 、 `WITHDIST` 、 `WITHHASH` 等选项的情况下， 命令返回一个二层嵌套数组， 内层的每个子数组就表示一个元素。

在返回嵌套数组时， 子数组的第一个元素总是位置元素的名字。 至于额外的信息， 则会作为子数组的后续元素， 按照以下顺序被返回：

1. 以浮点数格式返回的中心与位置元素之间的距离， 单位与用户指定范围时的单位一致。
2. `geohash` 整数。
3. 由两个元素组成的坐标，分别为经度和纬度。

```bash
127.0.0.1:6379> GEORADIUS city 110 30 1000 km
1) "chongqing"
2) "xian"
3) "shenzhen"
4) "hangzhou"
127.0.0.1:6379> GEORADIUS city 110 30 1000 km WITHDIST
1) 1) "chongqing"
   2) "341.9374"
2) 1) "xian"
   2) "483.8340"
3) 1) "shenzhen"
   2) "924.6408"
4) 1) "hangzhou"
   2) "977.5143"
   127.0.0.1:6379>  GEORADIUS city 110 30 1000 km WITHDIST WITHCOORD WITHHASH ASC COUNT 2
1) 1) "chongqing"
   2) "341.9374"
   3) (integer) 4026042091628984
   4) 1) "106.49999767541885"
      2) "29.529999579006592"
2) 1) "xian"
   2) "483.8340"
   3) (integer) 4040115445396757
   4) 1) "108.96000176668167"
      2) "34.2599996441893"
```

## GEORADIUSBYMEMBER 返回以指定元素为中心指定距离内的元素

`GEORADIUSBYMEMBER key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [ASC|DESC] [COUNT count]` 这个命令和 `GEORADIUS` 命令一样， 都可以找出位于指定范围内的元素， 但是 `GEORADIUSBYMEMBER` 的中心点是由给定的位置元素决定的， 而不是像 `GEORADIUS` 那样， 使用输入的经度和纬度来决定中心点。 

详细信息，查看`GEORADIUS`命令。

```bash
127.0.0.1:6379> GEORADIUSBYMEMBER city xian 1000 km
1) "xian"
2) "chongqing"
3) "beijing"
```

## GEOHASH 返回一个或多个元素位置的Geohash表示

`GEOHASH key member [member ...]` 返回一个或多个位置元素的 [Geohash](https://en.wikipedia.org/wiki/Geohash) 表示。 

```bash
127.0.0.1:6379> GEOHASH city beijing
1) "wx4fbxxfke0"
127.0.0.1:6379> GEOHASH city shanghai shenzhen
1) "wtw3sj5zbj0"
2) "ws10578st80"
```

# Redis HyperLogLog

`Redis HyperLogLog` 是用来做基数统计的算法，`HyperLogLog` 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

在 `Redis` 里面，每个 `HyperLogLog` 键只需要花费 `12 KB` 内存，就可以计算接近 `2^64` 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

但是，因为 `HyperLogLog` 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 `HyperLogLog` 不能像集合那样，返回输入的各个元素。

> 基数： 集合去重后的元素个数。

## PFADD 添加元素到HyperLogLog

`PFADD key element [element ...]`命令将任意数量的元素添加到指定的`HyperLogLog`中。

```bash
127.0.0.1:6379> PFADD myset a b c d e f g h i j
(integer) 1
```

## PFCOUNT 返回给定HyperLogLog的基数估算值

`PFCOUNT key [key ...]`返回给定的`HyperLogLog`的基数估算值。

当命令作用于单个键时， 返回储存在给定键的 `HyperLogLog` 的近似基数， 如果键不存在， 那么返回 `0` 。

当 命令作用于多个键时， 返回所有给定 `HyperLogLog` 的并集的近似基数， 这个近似基数是通过将所有给定 `HyperLogLog` 合并至一个临时 `HyperLogLog` 来计算得出的。

通过 `HyperLogLog` 数据结构， 用户可以使用少量固定大小的内存， 来储存集合中的唯一元素 （每个 `HyperLogLog` 只需使用 `12k` 字节内存，以及几个字节的内存来储存键本身）。

命令返回的可见集合基数并不是精确值， 而是一个带有 `0.81%` 标准错误的近似值。

举个例子， 为了记录一天会执行多少次各不相同的搜索查询， 一个程序可以在每次执行搜索查询时调用一次 `PFADD`， 并通过调用 `PFCOUNT`命令来获取这个记录的近似结果。

```bash
127.0.0.1:6379> PFADD myset a b c d e f g h i j
(integer) 1
127.0.0.1:6379> PFADD myset2 a b c e f i j
(integer) 1
127.0.0.1:6379> PFCOUNT myset
(integer) 10
127.0.0.1:6379> PFCOUNT myset2
(integer) 7
127.0.0.1:6379> PFCOUNT myset myset2
(integer) 10
```

## PFMERGE 将多个HyperLogLog合并为一个

`PFMERGE destkey sourcekey [sourcekey ...]` 命令将多个 `HyperLogLog` 合并为一个 `HyperLogLog` ，合并后的 `HyperLogLog` 的基数估算值是通过对所有 给定 `HyperLogLog` 进行并集计算得出的。 

 合并得出的 `HyperLogLog` 会被储存在 `destkey` 键里面， 如果该键并不存在， 那么命令在执行之前， 会先为该键创建一个空的 `HyperLogLog` 。 

```bash
127.0.0.1:6379> PFADD myset a b c d e f g h i j
(integer) 1
127.0.0.1:6379> PFADD myset2 a b c e f i j
(integer) 1
127.0.0.1:6379> PFMERGE merge myset myset2
OK
127.0.0.1:6379> PFCOUNT merge
(integer) 10
```

## Redis Bitmap

`Redis Bitmap`并不是一种特殊的数据结构，它的本质是二进制字符串，也可以看成是由多个二进制位组成的数组。数组中的每个二进制位都有对应的索引（偏移量），既然是二进制那么每个索引就只能为`0`或者`1`。通过索引我们就可以对位图中指定的二进制位进行操作。

简单而言，可以想象成一个数组，数组的每个索引只能保存`0`或者`1`，我们通过索引来改变每个位置上的值。

位图我们可以使用位图操作`GETBIT/SETBIT`，也可以使用普通的`GET/SET`直接过去和设置整个位图的内容。

**位图的优势：**

1. 基于最小的单位`bit`进行存储，所以非常省空间
2. `SET/GET`时间复杂度都是`O(1)`，部分读取命令`O(N)`，速度快
3. 二进制数据的存储，进行相关计算的时候非常快
4. 方便扩容

**一般可以在如下场景使用：**

1. 用户签到
2. 用户在线状态
3. 统计活跃用户
4. 各种状态值

## SETBIT 设置二进制位的值

`SETBIT bitmap offset value`命令可以为位图指定偏移量（索引）上的二进制位设置值。
当`bitmap`不存在时，将自动生成一个新的字符串值。`offset`参数必须大于或等于 `0` ，小于 `2^32` (`bit` 映射被限制在 `512 MB` 之内)，`value`只能是`0`或者`1`，设置之后，将返回二进制位被设置之前的旧值作为结果。
字符串会进行伸展以确保它可以将 `value` 保存在指定的偏移量上。当字符串值进行伸展时，空白位置以 `0` 填充。

```bash
127.0.0.1:6379> SETBIT bitmap1 0 1
(integer) 0
127.0.0.1:6379> SETBIT bitmap1 1 0
(integer) 0
127.0.0.1:6379> SETBIT bitmap1 2 1
(integer) 0
```

## GETBIT 获取二进制位的值

`GETBIT bitmap offset`命令可以对`bitmap`所储存的字符串值，获取指定偏移量上的值。

 当 `offset` 比字符串值的长度大，或者 `bitmap` 不存在时，返回 `0` 。 

```bash
127.0.0.1:6379> SETBIT bitmap1 0 1
(integer) 0
127.0.0.1:6379> SETBIT bitmap1 1 0
(integer) 0
127.0.0.1:6379> SETBIT bitmap1 2 1
(integer) 0
127.0.0.1:6379> GETBIT bitmap1 0
(integer) 1
127.0.0.1:6379> GETBIT bitmap1 2
(integer) 1
```

## BITCOUNT 统计值为1的二进制位的数量

`BITCOUNT bitmap [start] [end]`计算给定字符串中，被设置为 `1` 的二进制位的数量。

一般情况下，给定的整个字符串都会被进行计数，通过指定额外的 `start` 或 `end` 参数，可以让计数只在特定的位上进行，需要注意的是：**这两个参数的单位是字节（`byte`），而不是`bit`，也就是说`0`统计的是`0 - 7`位上的值，以此类推**。

`start` 和 `end` 参数可以使用负数值： 比如 `-1` 表示最后一个字节， `-2` 表示倒数第二个字节，以此类推。

不存在的 `bitmap` 被当成是空字符串来处理，因此对一个不存在的 `bitmap` 进行 `BITCOUNT` 操作，结果为 `0` 。

```bash
127.0.0.1:6379> SETBIT bitmap 0 1
(integer) 0
127.0.0.1:6379> SETBIT bitmap 3 1
(integer) 0
127.0.0.1:6379> SETBIT bitmap 2 1
(integer) 0
127.0.0.1:6379> SETBIT bitmap 5 1
(integer) 0
127.0.0.1:6379> BITCOUNT bitmap
(integer) 4
127.0.0.1:6379> BITCOUNT bitmap 0 0
(integer) 4
```

## BITPOS 返回第一个指定的二进制位值

`BITPOS bitmap bit [start] [end]`返回位图中第一个值为 `bit` 的二进制位的位置。

在默认情况下， 命令将检测整个位图， 但用户也可以通过可选的 `start` 参数和 `end` 参数指定要检测的范围，**这两个参数的单位同样为字节，而不是位，也就是说`0`表示的是`0 - 7`位上的值，以此类推**。

同样还需要注意的是，**即使指定了`start`和`end`，返回的偏移量不是它在指定字节中的偏移量，而是对于整个位图的偏移量**。

```bash
127.0.0.1:6379> SETBIT bitmap 8 1
(integer) 0
127.0.0.1:6379> BITPOS bitmap 0
(integer) 0
127.0.0.1:6379> BITPOS bitmap 1
(integer) 8
127.0.0.1:6379> BITPOS bitmap 1 1 1
(integer) 8
```

## BITOP 执行二进制位运算

`BITOP operation destkey bitmap [bitmap ...]`对一个或多个保存二进制位的字符串 `bitmap` 进行位元操作，并将结果保存到 `destkey` 上。

`operation` 可以是 `AND` 、 `OR` 、 `NOT` 、 `XOR` 这四种操作中的任意一种：

- `BITOP AND destkey bitmap [bitmap ...]` ，对一个或多个 `bitmap` 求逻辑并（两者都为`1`，结果才为`1`,否则为`0`），并将结果保存到 `destkey` 。
- `BITOP OR destkey bitmap [bitmap ...]` ，对一个或多个 `bitmap` 求逻辑或（两者只要其一为`1`，结果为`1`，否则为`0`），并将结果保存到 `destkey` 。
- `BITOP XOR destkey bitmap [bitmap ...]` ，对一个或多个 `bitmap` 求逻辑异或（两者不相同时为`1`，否则为`0`），并将结果保存到 `destkey` 。
- `BITOP NOT destkey bitmap` ，对给定 `bitmap` 求逻辑非(取反，`1`变为`0`，`0`变为`1`)，并将结果保存到 `destkey` 。

除了 `NOT` 操作之外，其他操作都可以接受一个或多个 `bitmap` 作为输入。

处理不同长度的字符串时，较短的那个字符串所缺少的部分会被看作 `0` 。

```bash
127.0.0.1:6379> SETBIT bitmap1 1 1
(integer) 0
127.0.0.1:6379> SETBIT bitmap1 3 1
(integer) 0
127.0.0.1:6379> SETBIT bitmap2 0 1
(integer) 0
127.0.0.1:6379> SETBIT bitmap2 2 1
(integer) 0
127.0.0.1:6379> BITOP AND bitmap bitmap1 bitmap2
(integer) 1
```

## BITFIELD 在位图中存储整数值

`BITFIELD key [GET type offset] [SET type offset value] [INCRBY type offset increment] [OVERFLOW WRAP|SAT|FAIL]`命令允许在位图中的任意区域存储指定长度的整数值，并对这些整数值执行家法或减法操作。

- `offset`参数用于指定设置的起始偏移量。这个偏移量从`0`开始计算，偏移量为`0`表示设置从位图的第`1`个二进制位开始，偏移量为`1`则表示设置从位图的第`2`个二进制位开始，以此类推。如果被设置的值长度不止一位，那么设置将自动延伸至之后的二进制位。
- `type`参数用于指定被设置值的类型，这个参数的值需要以`i`或者`u`为前缀，后跟被设置值的位长度，其中`i`表示被设置的值为有符号整数，而`u`则表示被设置的值为无符号整数。比如`i8`表示被设置的值为有符号`8`位整数，而`u16`则表示被设置的值为无符号`16`位整数，诸如此类。`BITFIELD`的各个子命令目前最大能够对`64`位长的有符号整数（`i64`）和`63`位长的无符号整数（`u63`）进行操作。
- `value`参数用于指定被设置的整数值，这个值的类型应该和`type`参数指定的类型一致。如果给定值的长度超过了`type`参数指定的类型，那么`SET`命令将根据`type`参数指定的类型截断给定值。比如，如果用户尝试将整数`123`（二进制表示为`01111011`）存储到一个`u4`类型的区域中，那么命令会先将该值截断为`4`位长的二进制数字`1011`（即十进制数字`11`），然后再进行设置。 

 `INCRBY`可以对指定的二进制位范围执行加法操作，并返回它的旧值。用户可以通过向 `increment` 参数传入负值来实现相应的减法操作。 

`BITFIELD`还提供了三种溢出策略：

-  `WRAP（回绕）`。一个`i8`的整数，值为`127`，递增`1`会导致值变为`-128`，即向上溢出后将从类型最小值开始重新计算，向下溢出后将从类型最大值开始重新计算；
-  `SAT（饱和计算）`。一个`i8`的整数，值为`120`，递增`10`结果变为`127`（`i8` 类型所能储存的最大整数值），即向上溢出后将设置为类型最大值，向下溢出后将设置为类型最小值；
-  `FAIL`。  发生溢出时，操作失败。并返回空值表示计算未被执行。

 命令返回一个数组作为回复， 数组中的每个元素就是对应操作的执行结果（可以一次性执行多条语句）。 

```bash
# 从第1位开始至第4位，设值为5（有符号数）
127.0.0.1:6379> BITFIELD key SET i4 0 5
1) (integer) 0

# 从第1位开始至第4位，结果为有符号数
127.0.0.1:6379> BITFIELD key GET i4 0
1) (integer) 5

# 从第1位至第4位，结果为有符号数
# 从第5位开始数4位，即至第9位，设值为6，结果为无符号数
# 从第5位开始数4位，即至第9位，值增加1，结果为无符号数
127.0.0.1:6379> BITFIELD key GET i4 0 SET u4 4 6 INCRBY u4 4 1
1) (integer) 5
2) (integer) 0
3) (integer) 7
```

