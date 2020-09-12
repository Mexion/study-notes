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

# Redis 事务

`Redis`支持事务，但是`Redis`的事物和`MySQL`的事物有本质的区别。

我们知道`MySQL`的事务是具备`ACID`的特性，即 原子性（**A**tomicity）、一致性（**C**onsistency）、隔离性（**I**solation）、持久性（**D**urability）。 

那么`Redis`是否具备`ACID`？

- 原子性：将多个操作当成一个整体来执行，要么全部执行，要么一个也不执行。在`Redis`中，事务的`MULTI`命令后的操作，都将顺序生成在一个队列中，命令不会立即执行，队列中所有的命令会等待`EXEC`命令的提交后，才会被一次性执行。但是如果队列中某一条命令执行失败，并不会影响之前或之后的命令，之前或之后的命令依然会正常执行，同时`Redis`也没有回滚机制。也就是说：**`Redis`不能保证原子性**。
- 一致性： 事务的执行不能破坏数据库数据的完整性和一致性，一个事务在执行之前和执行之后，数据库都必须处于一致性状态。 `Redis`不能保证原子性，也不支持事务回滚，所以并**不能保证事务执行前后数据的一致性**。
- 隔离性： 在并发环境中，并发的事务时相互隔离的，一个事务的执行不能不被其他事务干扰。  `Redis` 是单线程的，也就意味着它总是以串行方式执行，同一时刻内不会有其他事务打断当前事务的执行。**`Redis`的事务具有隔离性 。**
- 持久性： 一旦事务提交，那么它对数据库中的对应数据的状态的变更就会永久保存到数据库中。 `Redis`有三种存储模式， 当服务器在无持久化的内存模式下运行时,事务不具有持久性,一旦服务器停机,包括事务数据在内的所有数据都将丢失。当服务器在`RDB`持久化模式下运作的时候,服务器只会在特定的保存条件满足的时候才会执行`BGSAVE`命令,对数据库进行保存操作,并且异步执行的`BGSAVE`不能保证事务数据被第一时间保存到硬盘里面,因此`RDB`持久化模式下的事务也不具有持久性。当服务器运行在`AOF`持久化模式下,并且`appedfsync`的选项的值为`always`时,程序总会在执行命令之后调用同步函数,将命令数据真正的保存到硬盘里面,因此 这种配置下的事务是具有持久性的。所以**`Redis`的持久性由它使用的模式决定**。

## MULTI 标记一个事务的开始

 `MULTI`命令可以标记一个事务的开始。事务内的多条命令会按照先后顺序被放进一个队列当中，最后由`EXEC`命令执行。 

```bash
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> SET name aabb
QUEUED
127.0.0.1:6379> GET name
QUEUED
```

## EXEC 执行一个事务

`EXEC`命令执行一个事务。 假如某个(或某些) `key` 正处于 `WATCH`命令的监视之下，且事务块中有和这个(或这些) `key` 相关的命令，那么 `EXEC`命令只在这个(或这些) `key` 没有被其他命令所改动的情况下执行并生效，否则该事务被打断。 

```bash
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> SET name aabb
QUEUED
127.0.0.1:6379> GET name
QUEUED
127.0.0.1:6379> EXEC
1) OK
2) "aabb"
```

## DISCARD 取消事务

`DISCARD`命令取消事务，放弃执行事务内的所有命令。

如果正在使用 `WATCH` 命令监视某个(或某些) `key`，那么取消所有监视，等同于执行命令 `UNWATCH` 。

```bash
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> SET age 18
QUEUED
127.0.0.1:6379> GET age
QUEUED
127.0.0.1:6379> DISCARD
OK
```

## WATCH 监视一个或多个key

`WATCH key [key ...]`命令监视一个(或多个) `key` ，如果在事务执行之前这个(或这些) `key` 被其他命令所改动，那么事务将被打断（乐观锁）。

```bash
127.0.0.1:6379> WATCH name age
OK
```

## UNWATCH 取消所有监视

取消 `WATCH`命令对所有 `key` 的监视。

如果在执行 `WATCH`命令之后， `EXEC`命令或 `DISCARD`命令先被执行了的话，那么就不需要再执行 `UNWATCH`了。因为 `EXEC`命令会执行事务，因此 `WATCH`命令的效果已经产生了；而 `DISCARD`命令在取消事务的同时也会取消所有对 `key` 的监视，因此这两个命令执行之后，就没有必要执行 `UNWATCH`了。

```bash
127.0.0.1:6379> WATCH name age
OK
127.0.0.1:6379> UNWATCH
OK
```



 # Redis 配置文件

| 序号 | 配置项                                                       | 说明                                                         |
| :--- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 1    | `daemonize no`                                               | `Redis` 默认不是以守护进程的方式运行，可以通过该配置项修改，使用 yes 启用守护进程（`Windows` 不支持守护线程的配置为 `no` ） |
| 2    | `pidfile /var/run/redis.pid`                                 | 当 `Redis` 以守护进程方式运行时，`Redis` 默认会把 `pid` 写入 `/var/run/redis.pid` 文件，可以通过 `pidfile` 指定 |
| 3    | `port 6379`                                                  | 指定 `Redis` 监听端口，默认端口为 `6379`，作者在自己的一篇博文中解释了为什么选用 `6379` 作为默认端口，因为 `6379` 在手机按键上 `MERZ` 对应的号码，而 `MERZ` 取自意大利歌女 `Alessia Merz` 的名字 |
| 4    | `bind 127.0.0.1`                                             | 绑定的主机地址                                               |
| 5    | `timeout 300`                                                | 当客户端闲置多长秒后关闭连接，如果指定为 `0` ，表示关闭该功能 |
| 6    | `loglevel notice`                                            | 指定日志记录级别，`Redis` 总共支持四个级别：`debug`、`verbose`、`notice`、`warning`，默认为 `notice` |
| 7    | `logfile stdout`                                             | 日志记录方式，默认为标准输出，如果配置 `Redis` 为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给 `/dev/null` |
| 8    | `databases 16`                                               | 设置数据库的数量，默认数据库为`0`，可以使用`SELECT` 命令在连接上指定数据库`id` |
| 9    | `save <seconds> <changes>`。`Redis` 默认配置文件中提供了三个条件：**save 900 1** **，save 300 10**， **save 60 10000**。分别表示 `900` 秒（`15` 分钟）内有 `1` 个更改，`300` 秒（`5` 分钟）内有 `10` 个更改以及 `60` 秒内有 `10000` 个更改。 | 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合 |
| 10   | `rdbcompression yes`                                         | 指定存储至本地数据库时是否压缩数据，默认为 `yes`，`Redis` 采用 `LZF` 压缩，如果为了节省 `CPU` 时间，可以关闭该选项，但会导致数据库文件变的巨大 |
| 11   | `dbfilename dump.rdb`                                        | 指定本地数据库文件名，默认值为 `dump.rdb`                    |
| 12   | `dir ./`                                                     | 指定本地数据库存放目录                                       |
| 13   | `slaveof/replicaof  <masterip> <masterport>`                 | 设置当本机为 `slave` 服务时，设置 `master` 服务的 `IP` 地址及端口，在 `Redis` 启动时，它会自动从 `master` 进行数据同步。`5.0.0`以后的版本，请使用`replicaof`。 |
| 14   | `masterauth <master-password>`                               | 当 `master` 服务设置了密码保护时，`slave` 服务连接 `master` 的密码 |
| 15   | `requirepass foobared`                                       | 设置 `Redis` 连接密码，如果配置了连接密码，客户端在连接 `Redis` 时需要通过 `AUTH <password>` 命令提供密码，默认关闭 |
| 16   | ` maxclients 128`                                            | 设置同一时间最大客户端连接数，默认无限制，`Redis` 可以同时打开的客户端连接数为 `Redis` 进程可以打开的最大文件描述符数，如果设置 `maxclients 0`，表示不作限制。当客户端连接数到达限制时，`Redis` 会关闭新的连接并向客户端返回 `max number of clients reached` 错误信息 |
| 17   | `maxmemory <bytes>`                                          | 指定 `Redis` 最大内存限制，`Redis` 在启动时会把数据加载到内存中，达到最大内存后，`Redis` 会先尝试清除已到期或即将到期的 `Key`，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。`Redis` 新的 `vm` 机制，会把 `Key` 存放内存，`Value` 会存放在 `swap` 区 |
| 18   | `appendonly no`                                              | 指定是否在每次更新操作后进行日志记录，`Redis` 在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 `redis` 本身同步数据文件是按上面 `save` 条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为 `no` |
| 19   | `appendfilename appendonly.aof`                              | 指定更新日志文件名，默认为 `appendonly.aof`                  |
| 20   | `appendfsync everysec`                                       | 指定更新日志条件，共有 `3` 个可选值：**no**：不调用`fsync()`。而是让操作系统自行决定sync的时间。（快）**always**：表示每次更新操作后都调用 `fsync()` 将数据写到磁盘（慢，安全）**everysec**：表示每秒调用一次`fsync()`（折中，默认值） |
| 21   | `vm-enabled no`                                              | 指定是否启用虚拟内存机制，默认值为 `no`，简单的介绍一下，`VM` 机制将数据分页存放，由 `Redis` 将访问量较少的页即冷数据 `swap` 到磁盘上，访问多的页面由磁盘自动换出到内存中 |
| 22   | `vm-swap-file /tmp/redis.swap`                               | 虚拟内存文件路径，默认值为 `/tmp/redis.swap`，不可多个 `Redis` 实例共享 |
| 23   | `vm-max-memory 0`                                            | 将所有大于 `vm-max-memory` 的数据存入虚拟内存，无论 `vm-max-memory` 设置多小，所有索引数据都是内存存储的(`Redis` 的索引数据 就是 `keys`)，也就是说，当 `vm-max-memory` 设置为 `0` 的时候，其实是所有 `value` 都存在于磁盘。默认值为 `0` |
| 24   | `vm-page-size 32`                                            | `Redis swap` 文件分成了很多的 `page`，一个对象可以保存在多个 `page` 上面，但一个 `page` 上不能被多个对象共享，`vm-page-size` 是要根据存储的 数据大小来设定的，建议如果存储很多小对象，`page` 大小最好设置为 `32` 或者 `64bytes`；如果存储很多大对象，则可以使用更大的 `page`，如果不确定，就使用默认值 |
| 25   | `vm-pages 134217728`                                         | 设置 `swap` 文件中的 `page` 数量，由于页表（一种表示页面空闲或使用的 `bitmap`）是放在内存中的，在磁盘上每 `8` 个 `pages` 将消耗 `1byte` 的内存。 |
| 26   | `vm-max-threads 4`                                           | 设置访问`swap`文件的线程数,最好不要超过机器的核数,如果设置为`0`,那么所有对`swap`文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为`4` |
| 27   | `glueoutputbuf yes`                                          | 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启 |
| 28   | `hash-max-zipmap-entries 64     hash-max-zipmap-value 512`   | 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法 |
| 29   | `activerehashing yes`                                        | 指定是否激活重置哈希，默认为开启                             |
| 30   | `include /path/to/local.conf`                                | 指定包含其它的配置文件，可以在同一主机上多个`Redis`实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件 |



# Redis 持久化

`Redis`与传统数据库的一个主要区别在于，它把所有的数据都存储在内存中，这使得用户可以以极快的速度读写数据，但是内存属于易失存储器，如果不将内存中的数据状态保存到磁盘，那么一旦系统断电，所有内存中保存的数据都会丢失，所以`Redis`提供了持久化功能，可以把内存中的数据以文件形式存储到硬盘上，而服务器也可以根据这些文件在系统停机之后实施数据恢复，让服务器的数据库重新回到停机之前的状态。

为了满足不同的持久化需求，`Redis`提供了`RDB持久化`、`AOF持久化`、`RDB-AOF混合持久化`等多种持久化方式，当然，也可以关闭持久化功能，让服务器处于无持久化状态。

## RDB 持久化

`RDB持久化`是`Redis`默认使用的持久化功能，该功能可以创建出一个经过压缩的二进制文件，其中包含了服务器再各个数据库中存储的键值对数据等信息。`RDB`持久化产生的文件以`.rdb`结尾。

`Redis`提供了多种创建`RDB`文件的方式，用户既可以使用`SAVE`命令或者`BGSAVE`命令手动创建`RDB`文件，也可以通过设置`save`配置选项让服务器再满足指定条件时自动执行`BGSAVE`命令。

### SAVE 阻塞服务器并创建RDB文件

`SAVE`命令执行一个同步保存操作，将当前 `Redis` 实例的所有数据快照以 `RDB` 文件的形式保存到硬盘。如果在执行`SAVE`命令时已经拥有了相应的`RDB`文件，那么会创建新的`RDB`文件替换已有的`RDB`文件。

一般来说，在生产环境很少执行 `SAVE`操作，因为它会阻塞所有客户端，保存数据库的任务通常由 `BGSAVE` 命令异步地执行。然而，如果负责保存数据的后台子进程不幸出现问题时， `SAVE` 可以作为保存数据的最后手段来使用。

```bash
127.0.0.1:6379> SAVE
OK
```

### BGSAVE 非阻塞创建RDB文件

`BGSAVE`命令可以以非阻塞的方式创建`RDB`文件。

因为`SAVE`命令会阻塞整个服务器，所以用户在使用该命令创建`RDB`文件期间服务器将无法为其他客户端提供服务。而`BGSAVE`可以解决这个问题，它与`SAVE`的不同之处在于，它不会直接使用父进程（`Redis`进程）来创建`RDB`文件，而是使用子进程创建`RDB`文件。在收到一个`BGSAVE`命令后，`Redis`会进行如下流程：

1. 创建（`fork`）一个子进程。
2. 子进程进行`SAVE`命令，创建新的`RDB`文件。
3. `RDB`文件创建完毕，子进程退出并通知父进程新文件已经完成。
4. 父进程使用新的`RDB`文件替换已有的`RDB`文件。

因为`BGSAVE`命令创建`RDB`文件的操作是由子进程以异步方式执行的，所以当执行这个命令时，服务器会立即向客户端返回`OK`，然后才会在后台开始进行具体的`RDB`文件创建工作。

不过需要注意的是，虽然`BGSAVE`不会像`SAVE`一样一直阻塞`Redis`服务器，但是由于需要创建子进程，所以父进程占用的内存越大，创建子进程这一操作耗费的时间就越长，因此`Redis`服务器仍然可能会由于创建子进程而被短暂地阻塞。

```bash
127.0.0.1:6379> BGSAVE
Background saving started
```



### 通过配置自动创建RDB文件

除了通过手动创建`RDB`文件，我们还可以修改配置`save`选项，让`Redis`在满足指定的条件时自动执行`BGSAVE`命令。

在配置中修改`save <seconds> <changes>`配置项，`seconds`用于指定触发持久化操作所需的时长，而`changes`用于指定触发持久化操作所需的次数。简单而言，如果`Redis`在`seconds`秒内，对其包含的各个数据库总共执行了至少`changes`次修改，那么会自动执行一次`BGSAVE`命令。

### RDB的缺陷

`RDB`的创建是间隔性保存全量数据快照，所以无论`SAVE`命令还是`BGSAVE`命令，停机时都可能造成数据的丢失，丢失的数据量取决于创建`RDB`文件的时间间隔，间隔越长，丢失的数据就越多。

虽然从技术上来说，用户可以在每次执行写命令之后都执行一次`SAVE`命令，以此来保证数据的安全，但这样会导致`Redis`服务器的性能将下降到无法正常使用的水平。

从`RDB`持久化的特征来看，它更像是一种数据备份手段而非一种普通的数据持久化手段。

## AOF持久化

与全量式的`RDB持久化`功能不同，`AOF`提供的是增量式的持久化功能，这种持久化的核心原理在于：服务器每次执行完写命令之后，都会以协议文本的方式将被执行的命令追加到`AOF`文件的末尾（保存的是命令）。这样一来，服务器在停机之后，只要重新执行`AOF`文件中保存的`Redis`命令，就可以将数据库恢复至停机前的状态。

### 开启AOF

可以通过配置`appendonly <value>`选项来决定是否打开`AOF`功能，`yes`为开启，`no`不开启。开启时，默认情况下会创建一个名为`appendonly.aof`的文件来作为`AOF`文件。

### 设置AOF文件的冲洗频率

`Redis`向用户提供了`appendfsync always|everysec|no`选项来控制系统冲洗`AOF`文件的频率。配置有三个值可选：

- `always`：每执行一次写命令，就对`AOF`文件执行一次冲洗操作。
- `everysec`：每秒对`AOF`文件执行一次冲洗操作。
- `no`： 不主动对`AOF`文件执行冲洗操作，由操作系统决定何时对`AOF`文件执行冲洗操作。

这三种冲洗策略不仅会影响服务器在停机时丢失的数据量，还会影响服务器在运行时的性能：

- 在使用`always`时，服务器在停机时最多只会丢失一个命令的数据，但使用这种方式将使`Redis`性能降低至传统关系型数据库的水平。
- 在使用`everysec`的情况下，服务器在停机时最多只会丢失一秒之内产生的命令数据，这是一种兼顾性能和安全性的折中方案。
- 在使用`no`时，服务器在停机时将丢失系统最后一次冲洗`AOF`文件之后产生的所有命令数据，丢失数据量的大小取决于系统冲洗`AOF`文件的频率。

因为`no`的不确定性，以及`always`对于性能的牺牲，所以`Redis`默认使用`everysec`策略。**除非有明确需求，否则不建议改动这个策略**。

### AOF重写

随着服务器的不断运行，被执行的命令将变得越来越多，而负责记录这些命令的`AOF`文件也会越来越大。与此同时，如果曾经对相同的键执行过多次修改操作，那么`AOF`文件还会出现多个冗余命令，比如修改同一个`key`的字符串值只需要执行最后一个命令即可，而针对同一个`key`的多个`SADD`、`RPUSH`等操作可以简化为一条命令。冗余命令的存在不仅增加了`AOF`文件的体积，也让`Redis`在停机重启时恢复数据耗费的时间增多。

为了减少`AOF`文件的体积，提高数据恢复效率，`Redis`提供了`AOF`重写功能。

#### BGREWRITEAOF 显式触发AOF重写

`BGREWRITEAOF`命令可以显式地触发`AOF`重写操作。

该命令是一个异步命令，在接收到这个命令之后，`Redis`会创建出一个子进程，生成新的`AOF`文件，生成完毕后子进程就会退出并通知父进程，父进程会使用新的`AOF`文件替换已有的`AOF`文件，完成整个重写工作。

如果在发送`BGREWRITEAOF`命令请求时，`Redis`正在创建`RDB`文件，那么重写操作会延后到`RDB`文件创建完毕再执行，避免两个写硬盘操作同时执行导致机器性能下降。

如果在执行重写时，又收到了重写请求，服务器会报错。

#### AOF重写配置选项

除了手动执行创建`AOF`文件之外，还可以设置以下两个配置选项让`Redis`自动触发`BGREWRITEAOF`：

- `auto-aof-rewrite-min-size <value>`
- `auto-aof-rewrite-percentage <value>`

`auto-aof-rewrite-min-size <value>`用于设置触发自动`AOF`文件重写所需的最小`AOF`文件体积，在小于此值时，不自动重写，默认值为`64mb`。

`auto-aof-rewrite-percentage <value>`用于设置触发自动`AOF`文件重写所需的文件体积增大比例。比如，`auto-aof-rewrite-percentage 100`表示如果当前`AOF`文件的体积比最后一次`AOF`文件重写之后的体积增大了一倍，那么将自动重写一次。如果`Redis`服务器刚刚启动，还没有执行过`AOF`文件重写，那么启动服务时使用的`AOF`文件的体积将被用作最后一次`AOF`文件重写的体积。

### AOF的优缺点

与`RDB`可能会丢失大量数据相比，`AOF`的安全性要高得多。通过`everysec`选项，数据丢失的时间窗口可以控制在一秒之内。

但`AOF`也有相应的缺点：

- 因为`AOF`文件存储的是协议文本，所以它的体积会比包含相同数据，二进制格式的`RDB`文件要大得多，并且生成`AOF`文件所需的时间也要比`RDB`文件更长。
- `RDB`可以直接通过文件恢复数据，而`AOF`需要通过执行文件中的命令来恢复数据，所以`RDB`数据恢复速度比`AOF`快得多，数据库体积越大，差距就越明显。
- `AOF`重写`BGREWRITEAOF`命令与`BGSAVE`一样都需要创建子进程，所以在数据库体积较大时，进行`AOF`文件重写将占用大量资源，并导致服务器被短暂阻塞。

## RDB-AOF混合持久化

基于`RDB`和`AOF`两种模式的优缺点，`Redis`在`4.0`版本开始引入`RDB-AOF混合持久化模式`，这种模式是基于`AOF`构建而来的，如果打开了`AOF`并且将`aof-use-rdb-preamble <value>`配置项设置为`yes`, 那么`Redis`服务器在执行`AOF`重写操作时，就会像执行`BGSAVE`命令那样，根据数据库当前的状态生成出相应的`RDB`数据，并将这些数据写入新建的`AOF`文件中，至于那些在`AOF`重写开始之后执行的`Redis`命令，则会继续以协议文本的方式追加到新`AOF`文件的末尾，即已有的`RDB`数据的后面。
换句话说，**在开启了`RDB-AOF`混合持久化功能之后，服务器生成的`AOF`文件将由两个部分组成，其中位于`AOF`文件开头的是`RDB`格式的数据，而跟在`RDB`数据后面的则是`AOF`格式的数据**。 

 通过使用`RDB-AOF`混合持久化功能，用户可以同时获得`RDB`持久化和`AOF`持久化的优点：服务器既可以通过`AOF`文件包含的`RDB`数据来实现快速的数据恢复操作，又可以通过`AOF`文件包含的`AOF`数据来将丢失数据的时间窗口限制在`1s`之内。 

## 无持久化

即使用户没有显式地开启`RDB`持久化功能和`AOF`持久化功能，`Redis`服务器也会默认进行`RDB`持久化。

如果用户想要彻底关闭这一默认的`RDB`持久化行为，让`Redis`服务器处于完全的无持久化状态，那么可以向它提供配置选项`save ""`，这样一来，服务器将不会再进行默认的`RDB`持久化，从而使得服务器处于完全的无持久化状态中。处于这一状态的服务器在关机之后将丢失关机之前存储的所有数据，这种服务器可以用作单纯的内存缓存服务器。 

# Redis 发布订阅

`Redis` 发布订阅是一种消息通信模式：发送者发送消息，订阅者接收消息。

 在`Redis`中，客户端可以通过订阅特定的频道来接收发送至该频道的消息，我们把这些订阅频道的客户端称为订阅者。一个频道可以有任意多个订阅者，而一个订阅者也可以同时订阅任意多个频道。除此之外，客户端还可以通过向频道发送消息的方式，将消息发送给频道的所有订阅者，我们把这些发送消息的客户端称为发送者。 

## PUBLISH 向频道发送信息

`PUBLISH channel message`命令可以将一条信息发送到指定频道，然后将接收到消息的客户端数量作为返回值。

```bash
127.0.0.1:6379> PUBLISH news.it hello world
(integer) 1
```

## SUBSCRIBE 订阅频道

`SUBSCRIBE channel [channel channel ...]`命令可以让客户端订阅给定的一个或多个频道。在每次成功订阅一个频道后，都会返回一条订阅消息，消息包含了被成功订阅的频道以及客户端目前已订阅的频道数量。

```bash
127.0.0.1:6379> SUBSCRIBE news.it
Reading messages... (press Ctrl-C to quit)
1) "subscribe"	# 返回值类型 ，显示订阅成功
2) "news.it"	# 订阅的频道名称
3) (integer) 1	# 目前已订阅的频道数量
```

订阅几个频道就会返回几条如上的订阅消息。

当客户端成为频道的订阅者之后，就会接收到来自被订阅频道的消息，我们把这些消息称为频道消息。与订阅消息一样，频道消息也是由`3`个元素组成。

- 消息的第一个元素为`"message"`，用于表明该消息是一条频道消息而非订阅消息。
- 消息的第二个元素为消息的来源频道。
- 消息的第三个元素为消息的正文。

```bash
127.0.0.1:6379> SUBSCRIBE news.it
Reading messages... (press Ctrl-C to quit)
... 			#省略订阅频道时返回的订阅消息
1) "message"	# 返回值类型 ，表明这是一条频道消息
2) "news.it"	# 消息来源频道
3) "hello world"	# 频道消息正文
```

## UNSUBSCRIBE 退订频道

`UNSUBSCRIBE [channel channel ...]`命令可以让客户端退订一个或多个频道。如果不提供频道名称，那么将会退订该客户端已经订阅的所有频道。

每次退订频道后，都会收到服务器返回的退订消息，消息由`3`个元素组成：

- 第一个元素为`unsubscribe`，表明该消息是一条由退订操作产生的消息。
- 第二个元素是被退订频道的名字。
- 第三个元素是客户端在执行退订操作之后，该客户端仍在订阅的频道数量。

 另外，虽然`Redis`提供了`UNSUBSCRIBE`命令，但由于各个客户端对于发布与订阅功能的支持不尽相同，所以并非所有客户端都可以使用`UNSUBSCRIBE`命令执行退订操作。

比如`Redis`自带的命令行客户端`redis-cli`在执行`SUBSCRIBE`之后就会进入阻塞状态，无法再执行任何其他命令，用户只能通过同时按下`Ctrl`和`C`键强制退出`redis-cli`程序，所以这个客户端实际上并不会用到`UNSUBSCRIBE`命令。也就是说这个命令只可以用在非阻塞方式订阅时。

## PSUBSCRIBE 订阅模式

`PSUBSCRIBE pattern [pattern pattern ...]`命令可以让客户端订阅给定的一个或多个模式。

简单来说，`pattern`可以使一个全局风格的匹配符，比如`news.*`模式可以匹配所有以`news.`为前缀的频道，而`news.[ie]t`则可以匹配`news.it`和`news.et`频道，诸如此类。

 与`SUBSCRIBE`命令一样，`PSUBSCRIBE`命令在成功订阅一个模式之后也会返回相应的订阅消息，这条消息由`3`个元素组成：

- 第一个元素是`psubscribe`，它表明这条消息是由`PSUBSCRIBE`命令引发的订阅消息。
- 第二个元素是被订阅的模式。
- 第三个元素是客户端目前订阅的模式数量 。

```bash
127.0.0.1:6379> PSUBSCRIBE "news.*"
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "news.*"
3) (integer) 1
```

客户端在订阅模式之后，就会收到所有与模式相匹配的频道的消息，我们把这些消息称为模式消息。模式消息与之前展示的订阅消息以及频道消息稍微有些不同，它由`4`个元素组成：

- 第一个元素为`pmessage`，它表示这是一条模式消息而不是订阅消息或者频道消息。
- 第二个元素为被匹配的模式。
- 第三个元素则是与模式相匹配的频道。
- 第四个元素为消息的正文，也就是消息的真正内容。 

```bash
127.0.0.1> PSUBSCRIBE "news.*" "notification.*" "chat.*"
Reading messages... (press Ctrl-C to quit)
...                  #省略订阅消息
1) "pmessage"        #这是一条模式消息
2) "news.*"          #匹配的模式
3) "news.it"         #被匹配的频道
4) "hello it"        #消息正文
```

## PUNSUBSCRIBE 退订模式

`PUNSUBSCRIBE [pattern pattern ...]`命令与`UNSUBSCRIBE`命令一样，但不同的是退订的是模式而不是单独的频道。当不指定`pattern`时，退订所有订阅的模式。

 `PUNSUBSCRIBE`命令每次退订一个模式之后，都会向客户端返回一条退订消息，该消息由`3`个元素组成：

- 第一个元素是`punsubscribe`，用于表明该消息是一条由`PUNSUBSCRIBE`命令引起的退订消息。
- 第二个元素是被退订的模式。
- 第三个元素是客户端在执行当前这个退订操作之后，仍在订阅的模式数量。 

`PUNSUBSCRIBE`命令同`UNSUBSCRIBE`一样，只在非阻塞状态订阅的客户端上可行，`Redis`自带的客户端不支持。

## PUBSUB 查看发布与订阅的相关信息

`PUBSUB <subcommand> [argument [argument …]]`命令可以查看发布与订阅的相关信息。它目前由三个子命令组成。

### 查看活跃频道

`PUBSUB CHANNELS [pattern]`可以列出当前的活跃频道。活跃频道指至少有一个订阅者的频道，订阅模式的客户端不计算在内。

`pattern` 参数是可选的：

- 如果不给出 `pattern` 参数，那么列出订阅与发布系统中的所有活跃频道。
- 如果给出 `pattern` 参数，那么只列出和给定模式 `pattern` 相匹配的那些活跃频道。

```bash
127.0.0.1:6379> PUBSUB CHANNELS
1) "news.sport"
2) "news.internet"
3) "news.it"

127.0.0.1:6379> PUBSUB CHANNELS news.i*
1) "news.internet"
2) "news.it"
```

### 查看给定频道的订阅者数量

`PUBSUB NUMSUB [channel channel ...]`可以返回给定频道的订阅者数量，订阅模式的客户端不计算在内。

 会返回一个多条批量回复，回复中包含给定的频道，以及频道的订阅者数量。 格式为：频道 `channel-1` ， `channel-1` 的订阅者数量，频道 `channel-2` ， `channel-2` 的订阅者数量，诸如此类。 回复中频道的排列顺序和执行命令时给定频道的排列顺序一致。 不给定任何频道而直接调用这个命令也是可以的， 在这种情况下， 命令只返回一个空列表。 

```bash
127.0.0.1:6379> PUBSUB NUMSUB news.it news.internet news.sport news.music
1) "news.it"    # 频道
2) "2"          # 订阅该频道的客户端数量
3) "news.internet"
4) "1"
5) "news.sport"
6) "1"
7) "news.music" # 没有任何订阅者
8) "0"
```

### 查看被订阅模式的总数量

`PUBSUB NUMPAT`命令可以返回订阅模式的数量。注意，这个命令返回的不是订阅模式的客户端数量，而是客户端订阅的所有模式的数量的总和。

```bash
127.0.0.1:6379> PUBSUB NUMPAT
(integer) 4
```

# Redis 集群

## 主从复制

主从复制，是指将一台`Redis`服务器的数据，复制到其他的`Redis`服务器，前者称为主节点，后者称为从节点。数据的复制是单向的，只能从主节点到从节点，主节点以写为主，从节点以写为主。对于开启了复制功能的主从服务器，主服务器在每次执行写操作后，都会与所有从服务器进行数据同步，以此将写操作产生的改动反映到各个从服务器之上。

默认情况下，每台`Redis`服务器都是主节点，且一个主节点可以有多个从节点或没有从节点，但一个从节点只能有一个主节点。

主从复制的作用主要包括：

1. 数据冗余： 主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
2. 故障恢复： 当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复。
3. 负载均衡： 在主从复制的基础上，配合读写分离，可以由主节点提供写服务，从节点提供读服务（即写数据时应用连接主节点，读数据时连接从节点），分担服务器负载，尤其在写少读多的场景下，通过多个从节点分担读负载，可以大大提高`Redis`服务器的并发量。
4. 高可用基石： 除了上述作用，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是`Redis`高可用的基础。

一般来说，要将`Redis`运用于工程项目中，只使用一台`Redis`是万万不能的，原因如下：

1. 从结构上，单个服务器会发生单点故障，并且一台服务器需要处理所有的请求负载，压力较大。
2. 从容量上，单个服务器内存容量有限，就算一台服务器内存容量为`256G`，也不能将所有内存用作`Redis`存储内存，一般来说，单台`Redis`最大使用内存不应该超过`20G`。

### SLAVEOF/REPLICAOF 将服务器设置为从服务器

 在很长的一段时间里，`Redis`一直使用`SLAVEOF`作为复制命令，但是从`5.0.0`版本开始，`Redis`正式将`SLAVEOF`命令改名为`REPLICAOF`命令并逐渐废弃原来的`SLAVEOF`命令。因此，如果你使用的是`Redis 5.0.0`之前的版本，那么请使用`SLAVEOF`命令代替`REPLICAOF`命令，并使用`slaveof`配置选项代替`replicaof`配置选项。与此相反，如果你使用的是`Redis 5.0.0`或之后的版本，那么就应该使用`REPLICAOF`命令而不是`SLAVEOF`命令，因为后者可能会在未来的某个时候被正式废弃。 

将服务器设置为从服务器可以使用命令（重启会变回主机）：

```bash
REPLICAOF host port   # host 为主服务器地址 port 为主服务器端口
```

还可以修改配置文件（永久）：

```bash
replicaof <host> <port>  # host 为主服务器地址 port 为主服务器端口
```

**主机可以读写，但是推荐只写，从机是只读的，不能写。**

### 取消复制

取消复制可以使用命令：

```bash
REPLICAOF no one
```

这会让从服务器停止复制，重新变回主服务器。

停止复制后不会清空数据库，而是会继续保留复制产生的数据。

### ROLE 查看服务器角色

`ROLE`命令可以查看服务器角色。

在主服务器上执行时，命令返回一个由三个元素组成的数组：

- 第一个元素是字符串`master`，表示角色为主服务器。
- 第二个元素是这个主服务器的复制偏移量，它是一个整数，记录了主服务器目前向复制数据流发送的数据数量。
- 第三个元素是一个数组，记录了这个主服务器下所以的从服务器。这个数组每项由三个元素组成，第一个元素是从服务器的`IP`，第二个是从服务器的端口，第三个元素为从服务器的复制偏移量，记录了服务器通过复制数据流接收到的复制数据数量。当主从偏移量一致时，它们的数据就是一致的。

在从服务器上执行，命令返回一个由五个元素组成的数组：

- 第一个元素是字符串`slave`，它表示这个服务器的角色是从服务器。
- 第二个元素记录了主服务器的`IP`。
- 第三个元素记录了主服务器的端口。
- 第四个元素是主服务器与从服务器的连接状态，`none`表示未连接，`connect`表示主从服务器正在握手，`connecting`表示主从服务器成功建立了连接，`sync`表示主从服务器正在数据同步，`connected`表示主从服务器已经进入在线更新状态，`unknown`表示连接状态未知。
- 第五个元素是从服务器当前的复制偏移量。

## 集群时需要修改的信息

在创建集群时，我们需要复制多个配置文件，并在启动每个服务时指定对应的配置文件，从服务器配置文件需要修改的信息：

- 修改`replicaof`选项。

- 修改端口避免冲突。
- 修改`pid`名字避免冲突。
- 修改`log`文件名避免冲突。
- 修改持久化文件名`dump.rdb`避免冲突。

## Sentinel 哨兵模式

 因为主从服务器拥有相同的数据，所以它们在理论上是可以互相替换的：如果我们把主服务器的某个从服务器转换为主服务器，让它代替原来的主服务器处理命令请求，那么它得出的结果应该与原来的主服务器处理相同请求时得出的结果一致。基于这个原理，我们可以在主服务器因故下线时，将它的其中一个从服务器转换为主服务器，并使用新的主服务器继续处理命令请求，这样整个系统就可以继续运转，不必仅因为主服务器的下线而停机。这种使用正常服务器替换下线服务器以维持系统正常运转的操作，一般被称为故障转移。
因为`Redis`支持主从复制特性，所以我们同样可以对下线的`Redis`主服务器实施故障转移。举个例子，假设现在有一个主服务器`127.0.0.1:6379`（简称`6379`），它有两个从服务器`127.0.0.1:6380`（简称`6380`）和`127.0.0.1:6381`（简称`6381`），这`3`个服务器分别处理一些客户端的命令请求。如果在某一时刻，主服务器`6379`因为故障而下线，那么我们可以向`6380`发送命令`REPLICAOF no one`，将它转换为主服务器，然后向另一个从服务器`6381`发送命令`REPLICAOF 127.0.0.16380`，让它去复制新的主服务器`6380`，这样就可以重新建立起一个能够正常运作的主从服务器连接，并继续处理客户端发送的命令请求。 

 监视多台`Redis`服务器，并在有需要的时候对下线的主服务器实施故障转移，这并不是一件容易的事情，在主从服务器数量众多的情况下更是如此。为了给`Redis`服务器提供自动化的故障转移功能，提高主从服务器的可用性, `Redis`为用户提供了`Sentinel`工具。`Redis Sentinel`可以通过心跳检测的方式监视多个主服务器以及它们属下的所有从服务器，并在某个主服务器下线时自动对其实施故障转移。
简单来说，如果我们在前面的例子中，使用了`Redis Sentinel`来监视主服务器`6379`以及它的两个从服务器`6380`、`6381`，那么在`6379`因为故障而下线时，`Redis Sentinel`就会把`6380`和`6381`的其中一个转换为主服务器，并让另一个从服务器去复制新的主服务器，这个过程完全不需要人工介入。 

### 开启哨兵

`Redis Sentinel`的程序文件名为`redis-sentinel`，它通常和普通`Redis`服务器`redis-server`位于同一个文件夹。因为用户需要在配置文件中指定想要被`Sentinel`监视的主服务器，并且`Sentinel`也需要在配置文件中写入信息以记录主从服务器的状态，所以用户在启动`Sentinel`的时候必须传入一个可写的配置文件作为参数，就像这样：

```bash
$ redis-sentinel /etc/sentinel.conf
```

如果用户给定的配置文件是不可写的，那么`Sentinel`将放弃启动并报告一个错误。
一个`Sentinel`配置文件至少需要包含以下选项，用于指定`Sentinel`要监视的主服务器：

```bash
sentinel monitor <master-name> <ip> <port> <quorum>
```

选项中的`master-name`参数用于指定主服务器的名字，这个名字在执行各种`Sentinel`操作的时候会经常用到；`ip`参数和`port`参数用于指定主服务器的`IP`地址和端口号；而`quorum`参数则用于指定判断这个主服务器下线所需的`Sentinel`数量。
举个例子，如果我们要监视主服务器`127.0.0.1:6379`，并在`Sentinel`中将它命名为`website_db`，那么可以在配置文件`sentinel.conf`里面包含以下选项：

```bash
sentinel monitor website_db 127.0.0.16379 1
```


因为我们目前只启动了一个`Sentinel`，所以`quorum`参数的值被设置成了`1`。也就是说，只要有一个`Sentinel`认为`website_db`主服务器下线了，`Sentinel`就可以对`website_db`实施故障转移了。 

### 设置从服务器优先级

 用户可以通过`replica-priority`配置选项来设置各个从服务器的优先级，优先级较高的从服务器在`Sentinel`选择新主服务器的时候会优先被选择。
`replica-priority`的默认值为`100`，这个值越小，从服务器的优先级就越高。举个例子，如果现在有`3`个从服务器，它们的优先级分别为`100`、`50`和`10`，那么`Sentinel`将优先选用优先级为10的从服务器作为新的主服务器。
通过这个配置选项，用户可以给性能较高的从服务器设置较高的优先级来尽可能地保证新主服务器的性能。因为主服务器在下线并重新上线之后也会变成从服务器，所以用户也应该为主服务器设置相应的优先级。
`replica-priority`值为`0`的从服务器永远不会被选为主服务器，用户可以通过这一设置将不适合用作主服务器的从服务器排除在新主服务器的候选名单之外。 

### 新主服务器挑选规则

 当`Sentinel`需要在多个从服务器中选择一个作为新的主服务器时，首先会根据以下规则从候选名单中剔除不符合条件的从服务器：
1）否决所有已经下线以及长时间没有回复心跳检测的疑似已下线从服务器。
2）否决所有长时间没有与主服务器通信，数据状态过时的从服务器。
3）否决所有优先级为`0`的从服务器。
然后根据以下规则，在剩余的候选从服务器中选出新的主服务器：
1）优先级最高的从服务器获胜。
2）如果优先级最高的从服务器有两个或以上，那么复制偏移量最大的那个从服务器获胜。
3）如果符合上述两个条件的从服务器有两个或以上，那么选出它们当中运行`ID`（运行`ID`是服务器启动时自动生成的随机`ID`，这条规则可以确保条件完全相同的多个从服务器最终得到一个有序的比较结果）最小的那一个。 

### Sentinel网络

 在实际应用中，只使用单个`Sentinel`监视主从服务器并不合适，因为：

- 单个`Sentinel`可能会形成单点故障，当唯一的`Sentinel`出现故障时，针对主从服务器的自动故障转移将无法实施。如果同时有多个`Sentinel`对主服务器进行监视，那么即使有一部分`Sentinel`下线了，其他`Sentinel`仍然可以继续进行故障转移工作。
- 单个`Sentinel`可能会因为网络故障而无法获得主服务器的相关信息，并因此错误地将主服务器判断为下线，继而执行实际上并无必要的故障转移操作。如果同时有多个`Sentinel`对主服务器进行监视，那么即使有一部分`Sentinel`与主服务器的连接中断了，其他`Sentinel`仍然可以根据自己对主服务器的检测结果做出正确的判断，以免执行不必要的故障转移操作。

为了避免单点故障，并让`Sentinel`能够给出真实有效的判断结果，我们可以使用多个`Sentinel`组建一个分布式`Sentinel`网络，网络中的各个`Sentinel`可以通过互通消息来更加准确地判断服务器的状态。在一般情况下，只要`Sentinel`网络中有半数以上的`Sentinel`在线，故障转移操作就可以继续进行。
当`Sentinel`网络中的其中一个`Sentinel`认为某个主服务器已经下线时，它会将这个主服务器标记为主观下线（`Subjectively Down, SDOWN`），然后询问网络中的其他`Sentinel`，是否也认为该服务器已下线（换句话说，也就是其他`Sentinel`是否也将这个主服务器标记成了主观下线）。当同意主服务器已下线的`Sentinel`数量达到`sentinel monitor`配置选项中`quorum`参数所指定的数量时，`Sentinel`就会将相应的主服务器标记为客观下线（`objectively down, ODOWN`），然后开始对其进行故障转移。
因为`Sentinel`网络使用客观下线机制来判断一个主服务器是否真的已经下线了，所以为了让这种机制能够有效地运作，用户需要将`quorum`参数的值设置为`Sentinel`数量的半数以上，从而形成一种少数服从多数的投票机制。举个例子，在一个拥有`3`个`Sentinel`的网络中，`quorum`参数的值至少需要设置成`2`；而在一个拥有`5`个`Sentinel`的网络中，`quorum`参数的值至少需要设置成`3`；诸如此类。因为构成少数服从多数机制至少需要`3`个成员进行投票，所以用 户至少需要使用`3`个`Sentinel`才能构建一个可信的`Sentinel`网络。

 组建`Sentinel`网络的方法非常简单，与启动单个`Sentinel`时的方法一样：用户只需要启动多个`Sentinel`，并使用`sentinel monitor`配置选项指定`Sentinel`要监视的主服务器，那些监视相同主服务器的`Sentinel`就会自动发现对方，并组成相应的`Sentinel`网络。 

### Sentinel管理命令

| **命令**                                              | **用法**                         |
| ----------------------------------------------------- | -------------------------------- |
| `SENTINEL master <master-name>`                       | 获取指定被监视主服务器的信息     |
| `SENTINEL slaves <master-name>`                       | 获取被监视主服务器的从服务器信息 |
| `SENTINEL sentinels <master-name>`                    | 获取其他`Sentinel`的相关信息     |
| `SENTINEL get-master-addr-by-name <master-name>`      | 获取给定主服务器的`IP`和端口     |
| `SENTINEL reset <pattern>`                            | 重置主服务器状态                 |
| `SENTINEL failover <master-name>`                     | 强制执行故障转移                 |
| `SENTINEL ckquorum <master-name>`                     | 检查可用`Sentinel`数量           |
| `SENTINEL flushconfig`                                | 强制写入配置文件                 |
| `SENTINEL monitor <master-name> <ip> <port> <quorum>` | 监视给定主服务器                 |
| `SENTINEL remove <master-name>`                       | 取消对给定主服务器的监视         |
| `SENTINEL set <master-name> <option> <value>`         | 修改`Sentinel`配置选项的值       |

### Sentinel配置文件

```bash
# Example sentinel.conf

# *** IMPORTANT ***
# 绑定IP地址
# bind 127.0.0.1 192.168.1.1
# 保护模式（是否禁止外部链接，除绑定的ip地址外）
# protected-mode no

# port <sentinel-port>
# 此Sentinel实例运行的端口
port 26379

# 默认情况下，Redis Sentinel不作为守护程序运行。 如果需要，可以设置为 yes。
daemonize no

# 启用守护进程运行后，Redis将在/var/run/redis-sentinel.pid中写入一个pid文件
pidfile /var/run/redis-sentinel.pid

# 指定日志文件名。 如果值为空，将强制Sentinel日志标准输出。守护进程下，如果使用标准输出进行日志记录，则日志将发送到/dev/null
logfile ""

# sentinel announce-ip <ip>
# sentinel announce-port <port>
#
# 上述两个配置指令在环境中非常有用，因为NAT可以通过非本地地址从外部访问Sentinel。
#
# 当提供announce-ip时，Sentinel将在通信中声明指定的IP地址，而不是像通常那样自动检测本地地址。
#
# 类似地，当提供announce-port 有效且非零时，Sentinel将宣布指定的TCP端口。
#
# 这两个选项不需要一起使用，如果只提供announce-ip，Sentinel将宣告指定的IP和“port”选项指定的服务器端口。
# 如果仅提供announce-port，Sentinel将通告自动检测到的本地IP和指定端口。
#
# Example:
#
# sentinel announce-ip 1.2.3.4

# dir <working-directory>
# 每个长时间运行的进程都应该有一个明确定义的工作目录。对于Redis Sentinel来说，/tmp就是自己的工作目录。
dir /tmp

# sentinel monitor <master-name> <ip> <redis-port> <quorum>
#
# 告诉Sentinel监听指定主节点，并且只有在至少<quorum>哨兵达成一致的情况下才会判断它 O_DOWN 状态。
#
#
# 副本是自动发现的，因此您无需指定副本。
# Sentinel本身将重写此配置文件，使用其他配置选项添加副本。另请注意，当副本升级为主副本时，将重写配置文件。
#
# 注意：主节点（master）名称不能包含特殊字符或空格。
# 有效字符可以是 A-z 0-9 和这三个字符 ".-_".
sentinel monitor mymaster 127.0.0.1 6379 2

# 如果redis配置了密码，那这里必须配置认证，否则不能自动切换
# Example:
#
# sentinel auth-pass mymaster MySUPER--secret-0123passw0rd

# sentinel down-after-milliseconds <master-name> <milliseconds>
#
# 主节点或副本在指定时间内没有回复PING，便认为该节点为主观下线 S_DOWN 状态。
#
# 默认是30秒
sentinel down-after-milliseconds mymaster 30000

# sentinel parallel-syncs <master-name> <numreplicas>
#
# 在故障转移期间，多少个副本节点进行数据同步
sentinel parallel-syncs mymaster 1

# sentinel failover-timeout <master-name> <milliseconds>
#
# 指定故障转移超时（以毫秒为单位）。 它以多种方式使用：
#
# - 在先前的故障转移之后重新启动故障转移所需的时间已由给定的Sentinel针对同一主服务器尝试，是故障转移超时的两倍。
#
# - 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时。
#
# - 取消已在进行但未生成任何配置更改的故障转移所需的时间
#
# - 当进行failover时，配置所有slaves指向新的master所需的最大时间。
#   即使过了这个超时，slaves依然会被正确配置为指向master。
#
# 默认3分钟
sentinel failover-timeout mymaster 180000

# 脚本执行
#
# sentinel notification-script和sentinel reconfig-script用于配置调用的脚本，以通知系统管理员或在故障转移后重新配置客户端。
# 脚本使用以下规则执行以进行错误处理：
#
# 如果脚本以“1”退出，则稍后重试执行（最多重试次数为当前设置的10次）。
#
# 如果脚本以“2”（或更高的值）退出，则不会重试执行。
#
# 如果脚本因为收到信号而终止，则行为与退出代码1相同。
#
# 脚本的最长运行时间为60秒。 达到此限制后，脚本将以SIGKILL终止，并重试执行。

# 通知脚本
#
# sentinel notification-script <master-name> <script-path>
#
# 为警告级别生成的任何Sentinel事件调用指定的通知脚本（例如-sdown，-odown等）。
# 此脚本应通过电子邮件，SMS或任何其他消息传递系统通知系统管理员 监控的Redis系统出了问题。
#
# 使用两个参数调用脚本：第一个是事件类型，第二个是事件描述。
#
# 该脚本必须存在且可执行，以便在提供此选项时启动sentinel。
#
# 举例:
#
# sentinel notification-script mymaster /var/redis/notify.sh

# 客户重新配置脚本
#
# sentinel client-reconfig-script <master-name> <script-path>
#
# 当主服务器因故障转移而变更时，可以调用脚本执行特定于应用程序的任务，以通知客户端，配置已更改且主服务器地址已经变更。
#
# 以下参数将传递给脚本：
#
# <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
#
# <state> 目前始终是故障转移 "failover"
# <role> 是 "leader" 或 "observer"
#
# 参数 from-ip, from-port, to-ip, to-port 用于传递主服务器的旧地址和所选副本的新地址。
#
# 举例:
#
# sentinel client-reconfig-script mymaster /var/redis/reconfig.sh

# 安全
# 避免脚本重置，默认值yes
# 默认情况下，SENTINEL SET将无法在运行时更改notification-script和client-reconfig-script。
# 这避免了一个简单的安全问题，客户端可以将脚本设置为任何内容并触发故障转移以便执行程序。
sentinel deny-scripts-reconfig yes

# REDIS命令重命名
#
#
# 在这种情况下，可以告诉Sentinel使用不同的命令名称而不是正常的命令名称。
# 例如，如果主“mymaster”和相关副本的“CONFIG”全部重命名为“GUESSME”，我可以使用：
#
# SENTINEL rename-command mymaster CONFIG GUESSME
#
# 设置此类配置后，每次Sentinel使用CONFIG时，它将使用GUESSME。 请注意，实际上不需要尊重命令案例，因此在上面的示例中写“config guessme”是相同的。
#
# SENTINEL SET也可用于在运行时执行此配置。
#
# 为了将命令设置回其原始名称（撤消重命名），可以将命令重命名为它自身：
#
# SENTINEL rename-command mymaster CONFIG CONFIG
```

# Redis 缓存穿透和雪崩

`Redis`缓存的使用，极大地提升了应用程序的性能和效率，特别是数据查询方面，但同时，它也带来了一些问题。其中，最要命的问题就是数据的一致性问题。从严格意义上讲，这个问题无解，如果对数据的一致性要求很高，那么就不能使用缓存。

另外的一些典型问题就是，缓存穿透、缓存雪崩和缓存击穿。目前，业界也有比较流行的解决方案。

## 缓存穿透

缓存穿透的概念很简单，用户想要查询一个数据，发现`redis`中没有，也就是缓存未命中，于是向持久层数据库查询，发现也没有，于是本次查询失败。当用户很多的时候，缓存都没有命中（比如秒杀），于是都去了持久层数据库就，这会给持久层数据库造成很大的压力，这时候就相当于出现了缓存穿透。

> 解决方案

**布隆过滤器**

布隆过滤器是一种数据结构，对所有可能查询的参数以`hash`的形式存储，在控制层进行校验，不符合则丢弃，从而避免了对底层存储系统的查询压力。

**缓存空对象**

当存储层不命中后，即使返回的空对象也将其缓存起来，同时会设置一个过期时间，之后在访问这个数据将会从缓存中读取，保护了后端数据源。

但是这种方法存在两个问题：

- 如果空值被缓存起来，这就意味着缓存需要更多的空间存储更多的键，因为这当中可能会有很多的空值的键。
- 即使对空值设置了过期时间，还是会存在缓存层和存储层的数据会有一段时间窗口的不一致，这对于需要保持一致性的业务会有影响。

## 缓存击穿

缓存击穿，是指一个`key`非常热点，大并发几种对着一个点进行访问，当这个`key`在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库，就像在屏障上凿开了一个洞。

当某个`key`过期的瞬间，有大量的请求并发访问，这类数据一般是热点数据，由于缓存过期，会同时访问数据库来查询最新数据，并且回写缓存，会导致数据库瞬间压力过大。

> 解决方案

**设置热点数据永不过期**

从缓存层面来看，没有设置过期时间，所以不会出现热点`key`过期后产生的问题。

**加互斥锁**

分布式锁：使用分布式锁，保证对于每个`key`同时只有一个线程去查询后端服务，其他线程没有获取分布式锁的权限，因此需要等待即可。这种方式将高并发的压力转移到了分布式锁，因此对分布式锁的考验很大。

## 缓存雪崩

缓存雪崩，是指在某一个时间段，缓存集中过期失效，或者`Redis`宕机，所有请求都会到达数据库，造成数据库挂掉的情况。

> 解决方案

**Redis高可用**

搭建`Redis`集群，多增设`Redis`服务器。

**限流降级**

在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量，比如对某个`key`只允许一个线程查询数据和写缓存，其他线程等待。

**数据预热**

在正式部署前，先把可能的数据预先访问一遍，这样部分可能大量访问的数据就会加载到缓存中。在即将发生大并发访问前手动互发加载缓存不同的`key`，设置不同的过期时间，让缓存失效的时间点尽量均匀。