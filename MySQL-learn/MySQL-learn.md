## 登录MySQL

使用登录命令

```bash
mysql -hlocalhost -P3306 -uroot -pxxxx
```

`-h`代表地址，`-P`代表端口，`-u`用户名，`-p`密码。如果是本机默认可以省略`-h`和`-P`两个参数，`-p`可以不直接输入密码就回车，在后面输入。

## 修改密码

### SET PASSWORD命令

```mysql
set password for username@localhost=password('new password');
#例子
set password for root@localhost=password('123');
```

### mysqladmin

```mysql
mysqladmin -u用户名 -p旧密码 password 新密码;
#例子
mysqladmin -uroot -p123 password 456;
```



### 用UPDATE直接编辑user表

首先登录`MySQL`。

```mysql
use mysql;

update user set password=password('123') where user='root' and host='localhost';

flush privileges;
```



### 忘记root密码修改方法

以`windows`为例：

1. 关闭正在运行的`MySQL`服务。

2. 打开`DOS`窗口，转到`mysql/bin`目录。

3. 输入` mysqld --skip-grant-tables `回车。 `--skip-grant-tables`的意思是启动`MySQL`时跳过权限表认证。

4. 再开一个DOS窗口，输入`mysql`回车，连接权限数据库 `use mysql`。

5. 改密码： `user set password=password('123') where user='root'；`(别忘了最后加分号)。

6. 刷新权限： `flush privileges;`

7. 退出 `quit`。

8. 注销系统，再进入，使用刚才的用户名和修改后的密码登录。





##  语句分类

`MySQL`语句分为以下几种：

- `DQL（数据查询语言）` ：查询语句，所有`select`语句都是`DQL`。
- `DML（数据操作语言）`： `insert`、`delete`、`update`，对表当中的数据进行增删改。
- `DDL（数据定义语言）`： `create`、`drop`、`alter`，对表结构的增删改。
- `TCL（事务控制语言）`：`commit`提交事务，`rollback`回滚事务。
- `DCL（数据控制语言）`：`grant`授权、`revoke`撤销权限等。

每条语句用分号`;`分隔，一定要在命令或语句后加分号，否则会因为命令无法结束而没有效果，要想结束一条语句，必须 使用分号。

语句不区分大小写，但建议关键字大写，表名列名等小写。

### 一些基础命令和语句

- 查看`MySQL`版本：在命令行输入`mysql --version;`或在`MySQL sell`中`SELECT VERSION();`。
- 退出`MySQL sell`： `EXIT`或者`QUIT`。
- 查看所有数据库： `SHOW DATABASES;`。
- 切换数据库： `USE 数据库名;`。
- 查看所在数据库的表： `SHOW TABLES;`。
- 查看某个数据库的表： `SHOW TABLES FROM 数据库名`。
- 查看自己在哪个数据库： `SELECT DATABASE()`。
- 创建一个表：`CRATE TABLE hello(
      -> id int,
      -> name varchar(20));`
- 查看表的结构： `DESC 表名;`。
- 查看表的所有数据: `SELECT * FROM 表名;`。
- 表里插入数据: `INSERT INTO 表名 (id,name) VALUES(1,'mexion');`。
- 修改表里的数据: `UPDATE 表名 SET name='小明' WHERE id=1;`。
- 删除表里的数据: `DELETE FROM 表名 WHERE id=1;`。

### 注释

注释有三种方式：

- 单行注释： `#注释内容`
- 单行注释： `-- 注释内容` （注意--和内容之间有空格）
- 多行注释： `/* 注释文字 */`



## 创建切换和删除数据库

### 创建和切换数据库

首先登录数据库：

```mysql
mysql -uroot -p123
```

查看所有数据库：

```mysql
SHOW DATABASES; #这个不是SQL语句，是MySQL自己的命令
```

这时会显示出`MySQL`自带的一些数据库：

```mysql
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
```

创建自己的数据库：

```mysql
CREATE DATABASE mydata; #这个不是SQL语句，是MySQL自己的命令
```

切换数据库：

```mysql
USE mydata;
```

查看当前数据库中有哪些表：

```mysql
SHOW TABLES;
```

因为现在还没有任何数据，所以会显示`Empty set`。

也可以查看其他数据库中的表：

```mysql
SHOW TABLES FROM 数据库名;
```



### 删除数据库

如果需要删除一个数据库，可以使用：

```mysql
DROP DATABASE mydata;
```



## SOURCE执行sql脚本

执行`sql脚本`：

```mysql
source ‪C:\Users\Whatif\Desktop\studydata.sql
```

使用`source`命令 ，后面是需要执行的`sql脚本`路径（当一个文件的拓展名是`.sql`，并且该文件中编写了大量`sql`语句，这样的文件就是`sql脚本`）。

再次查询，就可以看到新增的数据。

### 查看表结构

可以使用`DESC 表名;`查看表结构，如：

```mysql
DESC dept;
```

输出如下：

```mysql
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| deptno | int(10)     | NO   | PRI | NULL    |       |
| dname  | varchar(14) | YES  |     | NULL    |       |
| loc    | varchar(13) | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
3 rows in set (0.00 sec)
```

这是一个存储部门信息的表，`deptno`是部门编号，`int`类型，`dname`部门名字，字符类型，`loc`部门位置，字符类型。



## 数据查询语句

### 简单的查询语句

#### 查询某个字段

语法：

```mysql
SELECT 字段名1,字段名2,字段名3... FROM 表名;
```

基于导入的数据，比如要查询所有员工的姓名：

```mysql
SELECT ename FROM emp;
```

会得到：

```mysql
+--------+
| ename  |
+--------+
| SIMITH |
| ALLEN  |
| WARD   |
| JONES  |
| MARTIN |
| BLAKE  |
| CLARK  |
| SCOTT  |
| KING   |
| TURNER |
| ADAMS  |
| JAMES  |
| FORD   |
| MILLER |
+--------+
14 rows in set (0.00 sec)
```

要查询员工编号和姓名：

```mysql
SELECT empno,ename FROM emp;
```

得到：

```mysql
+-------+--------+
| empno | ename  |
+-------+--------+
|  7369 | SIMITH |
|  7499 | ALLEN  |
|  7521 | WARD   |
|  7566 | JONES  |
|  7654 | MARTIN |
|  7698 | BLAKE  |
|  7782 | CLARK  |
|  7788 | SCOTT  |
|  7839 | KING   |
|  7844 | TURNER |
|  7876 | ADAMS  |
|  7900 | JAMES  |
|  7902 | FORD   |
|  7934 | MILLER |
+-------+--------+
14 rows in set (0.00 sec)
```

同理，想要查询员工的月薪很简单，这个表中`sal`字段为员工的月薪，只要将查询字段改为`sal`即可。那么如果想要查询员工的年薪怎么办呢？按照正常的思维，`年薪 = 月薪 * 12`，所以有如下语句：

```mysql
SELECT ename,sal*12 FROM emp;
```

可以得到：

```mysql
+--------+----------+
| ename  | sal*12   |
+--------+----------+
| SIMITH |  9600.00 |
| ALLEN  | 19200.00 |
| WARD   | 15000.00 |
| JONES  | 35700.00 |
| MARTIN | 15000.00 |
| BLAKE  | 34200.00 |
| CLARK  | 29400.00 |
| SCOTT  | 36000.00 |
| KING   | 60000.00 |
| TURNER | 18000.00 |
| ADAMS  | 13200.00 |
| JAMES  | 11400.00 |
| FORD   | 36000.00 |
| MILLER | 15600.00 |
+--------+----------+
14 rows in set (0.00 sec)
```

所以，**查询到的结果是可以参与数学运算的**。

但是问题又来了，可以看到查询到的结果虽然没错，但是字段名却变成了`sal*12`，这个显然观感很不好，所以**可以使用`AS`关键字给查询结果的字段名重命名（可以省略，用空格代替）**：

```mysql
SELECT ename,sal*12 AS yearsal FROM emp;
```

但是**重命名不能直接使用中文，如果实在要使用中文可以用单引号将中文引起来（只要是字符串格式，就要使用引号）**，如：

```mysql
SELECT ename,sal*12 AS '年薪' FROM emp;
```

#### 查询所有字段

要查询一张表中的所有字段，可以使用：

```mysql
SELECT * FROM emp;
```

这样就可以查询到表中的所有数据。

但是，实际开发中不建议使用`*`来进行查询，因为有性能问题，效率较低。

#### 去重

语法：

```mysql
SELECT DISTINCT 查询内容 FROM 表名;
```

**`DISTINCT`只能出现在所有语句的最前方。**

### 条件查询

如果需要在查询时设定条件，比如查询月薪大于`3000`的员工，就要使用到`WHERE`语句，它相当于编程语言中的`if`。

需要注意的是：**`WHERE`必须放到`FROM`语句的后面，而不能放到前面**。

在`WHERE`语句中，支持如下运算符：

| 运算符                   | 说明                                                         |
| ------------------------ | ------------------------------------------------------------ |
| =                        | 等于                                                         |
| <>或!=                   | 不等于                                                       |
| <                        | 小于                                                         |
| <=                       | 等于                                                         |
| >                        | 大于                                                         |
| >=                       | 大于等于                                                     |
| BETWEEN... AND ...       | 两者之间（包含两者），等同于 >=  AND  <=，可以作用于字符，取首字母（ASCII），但是区别在于结果不包括后者 |
| IS NULL                  | 为 NULL                                                      |
| IS NOT NULL              | 不为 NULL                                                    |
| AND                      | 并且（优先级高于OR，优先级不确定时用括号将先判断的括起来）   |
| OR                       | 或者                                                         |
| IN(参数一，参数二...)    | 包含，查找的结果是In函数中参数的某一个                       |
| NOT IN(参数一,参数二...) | 不包含，查找的结果不是In函数中参数的某一个                   |
| NOT                      | NOT可以取非，主要用在 IS 或 IN 中                            |
| LIKE                     | LIKE成为模糊查询，支持%或下划线匹配（%表示任意多个字符，包括0个，_ 表示任意一个字符）。如果要把通配符当成普通的字符，要用\转义。或者自定义转义字符，比如'\_$_%' ESCAPE '$' 使用ESCAPE关键字表明$是转义符号。 |

所以一条简单的条件查询语句如下：

```mysql
SELECT 字段,字段... FROM 表名 WHERE 条件;
```

执行顺序为：先`FROM`，然后`WHERE`，最后`SELECT`。

回到最初的问题：查询月薪大于`3000`的员工的姓名和月薪？

```mysql
SELECT ename,sal FROM emp WHERE sal>3000;
```

可得：

```mysql
+-------+---------+
| ename | sal     |
+-------+---------+
| KING  | 5000.00 |
+-------+---------+
1 row in set (0.00 sec)
```

可以看到结果只有一人，名为`KING`，月薪为`5000`。

要查询名为`SIMITH`的员工的工资：

```mysql
SELECT sal FROM emp WHERE ename='SIMITH';
```

可得：

```mysql
+--------+
| sal    |
+--------+
| 800.00 |
+--------+
1 row in set (0.00 sec)
```

### 排序

现在有了新的需求：找出员工名和月薪，并按照工资升序排列？

排序使用`ORDER BY`关键字，具体语句为：

```mysql
SELECT 字段 FROM 表名 WHERE 条件 ORDER BY 排序字段 ASC|DESC;
```

其中，`ASC`为升序，`DESC`为降序，不写默认为`ASC`。

所以，上面的需求：

```mysql
SELECT ename,sal FROM emp ORDER BY sal;
```

如果需求为：按照工资的降序排列，当工资相同的时候再按照名字的升序排列：

```mysql
SELECT ename,sal FROM emp ORDER BY sal desc, ename ASC;
```

多个字段排序时，排序字段越靠前的字段起的作用越大，前面的字段无法排序时才会启用后面的字段，所以，当工资相同时，就按照后面的字段排。



排序可以按照别名进行排序，如下：

```mysql
SELECT ename,sal AS salary FROM emp ORDER BY salary;
```

### 函数

#### 分组函数

所有的分组（聚合）函数都是对一组数据进行操作，最终输出一个数（分组函数会忽略空数据）。**分组函数只能在分组后才能使用，语句中没有`GROUP BY`，默认也会分组，分组在`WHERE`语句之后，所以分组函数不能直接用在`WHERE`的子句中。**

| 函数        | 说明                     |
| ----------- | ------------------------ |
| SUM(data)   | 对一组数据求和           |
| AVG(data)   | 对一组数据求平均值       |
| MAX(data)   | 对一组数据求最大值       |
| MIN(data)   | 对一组数据求最小值       |
| COUNT(data) | 对一组数据求个数（非空） |

比如对员工的工资进行求和：

```mysql
SELECT SUM(sal) FROM emp;
```

可得：

```mysql
+----------+
| SUM(sal) |
+----------+
| 29025.00 |
+----------+
1 row in set (0.00 sec)
```

这些函数可以支持`DISTINCT`，在字段前加`DISTINCT`关键字即可。

#### 单行函数

##### 字符函数

一些常见的字符函数：

| 函数                      | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| LENGTH(str)               | 返回参数的字节长度                                           |
| CONCAT(str1,str2,...)     | 对多个字符串进行连接                                         |
| UPPER(str)                | 返回字符串的大写                                             |
| LOWER(str)                | 返回字符串的小写                                             |
| SUBSTR(str,start,length?) | 截取参数（sql索引从1开始，不是0，包括长度最后一位）          |
| INSTR(str, str2)          | 返回str2在str中的第一次出现的索引，没有则返回0               |
| TRIM()                    | 去除前后空格，也可以去除前后任意字符，比如TRIM('a' FROM 'aaaabbbbaaaa')可以将前后的a都去掉 |
| LPAD(str, length, str2)   | 如果str字符数不足length，则在前面补str2直到str满length       |
| RPAD(str, length, str2)   | 如果str字符数不足length，则在后面补str2直到str满length       |
| REPLACE(str,str2,str3)    | 将str中的str2替换为str3                                      |

##### 数学函数

| 函数                | 说明                                                |
| ------------------- | --------------------------------------------------- |
| CEIL(num)           | 向上取整                                            |
| ROUND(num,digit)    | 对数字进行四舍五入，ROUND第二个参数是保留的小数位数 |
| FLOOR(num)          | 向下取整                                            |
| TRUNCATE(num,digit) | 截断，比如TRUNCATE(1.65,1)会返回1.6                 |
| MOD(num, num2)      | 取余，比如MOD(10,3)返回1                            |

##### 日期函数

| 函数                      | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| NOW()                     | 回当前日期和时间                                             |
| CURDATE()                 | 返回当前日期，不包含时间                                     |
| CURTIME()                 | 返回当前时间，不包含日期                                     |
| YEAR(date)                | 返回日期的年份                                               |
| MONTH(date)               | 返回日期的月份                                               |
| MONTHNAME(date)           | 返回日期的英文月份（后面的天时分秒都一样，就不一一列出来了） |
| STR_TO_DATE(str, format)  | 将日期格式的字符串转换成指定格式的日期，如STR_TO_DATE('9-13-1999','%Y-%m-%d')会转换成1999-09-13 |
| DATE_FORMAT(date, format) | 将日期转换成字符，如DATE_FORMAT('2018/6/6','%Y年%m月%d日')会返回 2018年06月06日 |

格式如下:
`%Y`表示四位数年份，`%y`表示两位数年份，`%m`月份(不足两位会自动补`0`)，`%c`月份不自动补`0`，`%d`日，`%H24`小时，`%h`12小时，`%i`分钟，`%s`秒。

##### 其他函数

| 函数       | 说明           |
| ---------- | -------------- |
| VERSION()  | 返回版本       |
| DATABASE() | 返回当前数据库 |
| USER()     | 返回当前用户   |

##### 流程控制函数

| 函数                  | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| IF(expr1,expr2,expr3) | 使用表达式1进行判断，如果为真，值是表达式2的值，否则是表达式3的值 |
| IFNULL(expr1,expr2)   | 如果表达式一不为NULL返回表达式一的值，否则返回表达式二的值   |
| CASE                  | CASE是一个语句，不是函数，但是放在这里。CASE可以实现编程语言中的switch case的效果。 |

`CASE`语句的语法：

```mysql
CASE 要判断的字段或表达式
WHEN 常量1 THEN 要显示的值1或语句1
WHEN 常量2 THEN 要显示的值2或语句2
ELSE 要显示的值n或语句n
END
```

`CASE`后面可以不写判断，可以在`WHEN`里进行判断。



**只要运算中有`NULL`参与运算，那么结果就必为`NULL`**。

### 分组

如果我们要找出工资高于平均工资的员工，会想到如下语句：

```mysql
SELECT ename,sal FROM emp WHERE sal>avg(sal);
```

但是这条语句会直接报错，由此我们可以得到：

**`SQL`语句当中有一个语法规则，分组函数不能直接使用在`WHERE`子句中。**



`GROUP BY` 语句可以按照字段对相同字段的数据进行分组操作(可以有多个字段，用逗号分隔)。如果要在分组查询后再进行筛选，要在`GROUP BY` 之后加 `HAVING `查询条件。

**如果一条语句有`GROUP BY`，那么这条语句的`SELECT`子句中只能有分组函数和参加分组的字段，虽然加其他字段在`MySQL`中可能也可以通过，但是数据混乱毫无意义。**

比如要中找出每个工作岗位的最高薪资：

```mysql
SELECT MAX(sal) FROM emp GROUP BY job;
```

分组函数一般会和`GROUP BY`联合使用，这也是为什么它被称为分组函数的原因，任何一个分组函数都是在`GROUP BY`执行结束之后才会执行的。当一条`SQL`语句没有`GROUP BY`的话，整张表的数据自成一组。

**分组函数之所以不能直接使用在`WHERE`子句中，是因为`GROUP BY`语句运行在`WHERE`语句之后，分组函数必须在分完组之后才能用，而`WHERE`时还没有分组。**

查询语句的执行顺序：

```mysql
SELECT #5
FROM	#1
WHERE	#2
GROUP BY	#3
HAVING	#4
ORDER BY #6
LIMIT #7
```



案例：找出每个部门不同工作岗位的最高薪资？

```mysql
SELECT deptno,job,MAX(sal) FROM emp GROUP BY deptno,job;
```

可以看到这里使用了联合分组，只有同时满足部门和工作岗位都相同的条件时，才会被分成一组。



回归到开始的 问题，要找出工资高于平均工资的员工，可以使用子查询，而不能直接在`WHERE`中用分组函数：

```mysql
SELECT ename,sal FROM emp WHERE sal > (SELECT AVG(sal) FROM emp);
```



案例： 找出平均薪资大于2000的部门编号和薪资：

```mysql
SELECT deptno,AVG(sal) FROM emp GROUP BY deptno HAVING AVG(sal) > 2000;
```

### 连接查询

在实际的开发中，大部分的情况下都不是从单表中查询数据，因为一般一个业务都会对应多张表，比如学生信息和班级信息，分开两张表进行存储，所以一般都是多张表联合查询取出最终的结果。

 在表的连接查询方面有一种现象被称为：笛卡尔乘积现象。

 案例:找出每一个员工的部门名称，要求显示员工名和部门名。

这个案例中由于部门信息存储在`dept`表中，所要连表查询。
假如这样查找:

```mysql
SELECT ename,dname FROM emp,dept;
```

`emp`有`14`条数据，`dept`有`4`条数据，这样查询时`emp`中的每条数据都会匹配`dept`中的每条数据，最后一共会产生`56`条数据，这就是笛卡尔乘积现象。

> 当两张表进行联合查询时，没有任何条件限制，结果的条数为两张表数据数的乘积。


要避免笛卡尔乘积现象，就要给查询加条件进行过滤。但是在数据库的底层依然会拿着`emp`中的结果去一一匹配`dept`的数据，依然还是匹配`56`次，只是展示出来的数据是筛选过的，所以，即使加了条件，笛卡尔乘积现象依然存在，并不能提高搜索效率。
加条件后语句如下:

```mysql
SELECT e.ename,d.dname FROM emp e,dept d WHERE e.deptno = d.deptno;
```

可以看到这条语句使用了重命名，最后通过判断两个部门名是否一致来判断，只有两者一致的条目才会成为最终的查询结果(查询次数没变，依然是`56`次，只是最终显示的经过了筛选)。

#### 内连接
##### 等值连接

> 所谓等值连接，就是通过两张表共同的字段来连接，当两张表有可以对比的字段，这两张表就可以联系起来，数据的某个字段的值相等时，这两个数据就可以连接上。

比如上面的:
```mysql
SELECT e.ename,d.dname FROM emp e,dept d WHERE e.deptno = d.deptno;
```

这就是等值连接，表一表二都有部门编号`deptno`这个字段，当数据的`deptno`相等时，这两个数据就可以被筛选出来。
但上面的写法是老式的`92语法`，一般不用，`99语法`如下:

```mysql
SELECT e.ename,d.dname FROM emp e INNER JOIN dept d ON e.deptno = d.deptno;
````

可以看到两张表的连接部分由逗号换成了`INNER JOIN（INNER可省略）`，而条件换成了`ON`。`99语法`的结构更清晰一些，并且将表连接的条件和真正的筛选条件分离开了，也就是说`JOIN...ON...`之后还可以加`WHERE`语句来进行真正的数据筛选。

##### 非等值连接

案例：找出每个员工的工资等级，要求显示员工名、工资、工资等级。

已知的是，员工信息和工资在`emp`表中，员工工资等级在`salgrade`表中，`grade`字段为工资等级，`losal`为该等级最低薪资，`hisal`为最高薪资，如图：

```mysql
mysql> SELECT * FROM salgrade;
+-------+-------+-------+
| grade | losal | hisal |
+-------+-------+-------+
|     1 |   700 |  1200 |
|     2 |  1201 |  1400 |
|     3 |  1401 |  2000 |
|     4 |  2001 |  3000 |
|     5 |  3001 |  5000 |
+-------+-------+-------+
5 rows in set (0.00 sec)

```

两张表中没有相同的字段，如果进行查询没有任何限制，笛卡尔乘积`14*5`会得到`70`条结果，回归到案例之中，要进行匹配的条件就是工资在`losal`和`hisal`之间就可以得到该员工的工资等级，可以进行如下连接查询：

```mysql
SELECT
	e.ename,e.sal,s.grade
FROM
	emp e
JOIN
	salgrade s 
ON
	e.sal BETWEEN s.losal AND s.hisal;
```

可得：

```mysql
+--------+---------+-------+
| ename  | sal     | grade |
+--------+---------+-------+
| SIMITH |  800.00 |     1 |
| ALLEN  | 1600.00 |     3 |
| WARD   | 1250.00 |     2 |
| JONES  | 2975.00 |     4 |
| MARTIN | 1250.00 |     2 |
| BLAKE  | 2850.00 |     4 |
| CLARK  | 2450.00 |     4 |
| SCOTT  | 3000.00 |     4 |
| KING   | 5000.00 |     5 |
| TURNER | 1500.00 |     3 |
| ADAMS  | 1100.00 |     1 |
| JAMES  |  950.00 |     1 |
| FORD   | 3000.00 |     4 |
| MILLER | 1300.00 |     2 |
+--------+---------+-------+
14 rows in set (0.00 sec)
```

这样查询后笛卡尔乘积现象会使每个员工都对应五个工资等级，但是`ON`后面的条件会根据`sal`选取在适合区间的`s.grade`的条目，其他的舍弃。

因为`ON`后面的条件不是等量关系，所以这就是非等量链接。

##### 自连接

> 自连接是一张表看做两张表，自己连自己。

案例：找出每个员工的上级领导，要求显示员工名和对应的领导名。

```mysql
SELECT
	a.ename,b.ename leadername
FROM
	emp a
JOIN	
	emp b
ON
	a.mgr = b.empno;
```

可得：

```mysql
+--------+------------+
| ename  | leadername |
+--------+------------+
| SIMITH | FORD       |
| ALLEN  | BLAKE      |
| WARD   | BLAKE      |
| JONES  | KING       |
| MARTIN | BLAKE      |
| BLAKE  | KING       |
| CLARK  | KING       |
| SCOTT  | JONES      |
| TURNER | BLAKE      |
| ADAMS  | SCOTT      |
| JAMES  | BLAKE      |
| FORD   | JONES      |
| MILLER | CLARK      |
+--------+------------+
13 rows in set (0.00 sec)
```

来分析下这个查询语句：我们把`emp`表一表当两表用，起了别名，可以将`a`看成员工表，`b`看成领导表，然后自己连接自己，条件是`a`的`mgr（上级领导编号）`等于`b`的员工编号。我们将领导名`b.ename`起了别名`leadername`，最后输出。

#### 外连接

什么是外连接，和内连接有什么区别？

- 内连接： 假设`A`和`B`两张表进行内连接，凡是`A`表和`B`表能够匹配上的记录都会查询出来，`AB`没有主副之分，两张表是平等的关系。
- 外连接： 假设`A`和`B`两张表进行外连接，`AB`两表有一张是主表，一张是副表，主要查询主表的数据，捎带链接副表，当附表中的数据没有和主表中的数据匹配时，附表会自动模拟出`NULL`与之匹配。

外连接的分类：
- 左外连接：左边的表是主表。
- 右外连接：右边的表是主表。
左连接有右连接的写法，右连接也会有对应的左连接的写法。

依然是上面的案例： 找出每个员工的上级领导（所有员工必须都查询出来，即使他没有上级领导）。

内连接中，因为`KING`没有上级领导所以查询出来只有`13`条数据，因为内连接只会匹配可以匹配的，`KING`没有上级领导，自然就不会匹配。而如果使用外连接，假设`A`表为主表，那么`A`的所有数据都必须要有匹配项，即使没有匹配项，副表也要使用`NULL`来满足匹配，使`A`的所有数据都要有查询结果。
所以，这个案例的外连接如下：
```mysql
SELECT
	a.ename,b.ename leadername
FROM
	emp a
LEFT JOIN	
	emp b
ON
	a.mgr = b.empno;
```
可以看到，只在`JOIN`前增加了`LEFT`关键字，表明左边的表为主表，所以左边的员工表`a`的所有项都必须要全部匹配上，即使`KING`没有上级，可得如下结果：
```mysql
+--------+------------+
| ename  | leadername |
+--------+------------+
| SIMITH | FORD       |
| ALLEN  | BLAKE      |
| WARD   | BLAKE      |
| JONES  | KING       |
| MARTIN | BLAKE      |
| BLAKE  | KING       |
| CLARK  | KING       |
| SCOTT  | JONES      |
| KING   | NULL       |
| TURNER | BLAKE      |
| ADAMS  | SCOTT      |
| JAMES  | BLAKE      |
| FORD   | JONES      |
| MILLER | CLARK      |
+--------+------------+
14 rows in set (0.00 sec)
```

上面的是左连接，它的右连接形式是：

```mysql
SELECT
	a.ename,b.ename leadername
FROM
	emp b
RIGHT JOIN	
	emp a
ON
	a.mgr = b.empno;
```

可以看到，使用了`RIGHT`关键字表明了右表是主表，如果依然要全部员工都要匹配，那么需要将`a`表放到右边，得到的结果依然相同。

其实，`LEFT`和`RIGHT`后面可以有`OUTER`关键字，表示外连接，但是可以省略。

所以，`INNER`和`OUTER`不是关键，外连接和内连接在语法上就只是外连接增加了`LEFT`或者`RIGHT`而已。

**在实际的开发中，外连接使用居多，因为很多时候都是主要查一张表，另一张表是捎带查的，如果匹配不上，主表也要保持全面，不能因为匹配不上就使主表匹配不上的数据丢失掉。**



最后，有一个需求：查询没有员工的部门名称？

员工表`emp`中的`deptno`表示部门编号，部门表`dept`中的`deptno`也表示部门编号。

分析：可以将部门表`dept`作为主表进行外连接，查询结果中员工信息为`NULL`的部门没有员工：

```mysql
SELECT
	d.dname
FROM
	dept d
LEFT JOIN
	emp e
ON
	e.deptno = d.deptno
WHERE
	e.ename IS NULL;
```

可得：

```mysql
+------------+
| dname      |
+------------+
| OPERATIONS |
+------------+
1 row in set (0.00 sec)
```

#### 三张表的连接查询

案例：找出每一个员工的部门名称以及工资等级。

可以写出如下语句：

```mysql
SELECT
	e.ename,d.dname,s.grade
FROM
	emp e
JOIN
	dept d
ON
	e.deptno = d.deptno
JOIN
	salgrade s
ON
	e.sal BETWEEN s.losal AND s.hisal;
```

可得：

```mysql
+--------+-------------+-------+
| ename  | dname       | grade |
+--------+-------------+-------+
| SIMITH | RESEARCHING |     1 |
| ALLEN  | SALES       |     3 |
| WARD   | SALES       |     2 |
| JONES  | RESEARCHING |     4 |
| MARTIN | SALES       |     2 |
| BLAKE  | SALES       |     4 |
| CLARK  | ACCOUNTING  |     4 |
| SCOTT  | RESEARCHING |     4 |
| KING   | ACCOUNTING  |     5 |
| TURNER | SALES       |     3 |
| ADAMS  | RESEARCHING |     1 |
| JAMES  | SALES       |     1 |
| FORD   | RESEARCHING |     4 |
| MILLER | ACCOUNTING  |     2 |
+--------+-------------+-------+
14 rows in set (0.01 sec)
```

可以看到，在连接完`dept`表后，再次对`salgrade`表进行了连接，最后得到最终的结果。

修改一下案例：找出每一个员工的部门名称、工资等级和上级领导？

可以在上面的语句上修改为下：

```mysql
SELECT
	e.ename,d.dname,s.grade,e1.ename leadername
FROM
	emp e
JOIN
	dept d
ON
	e.deptno = d.deptno
JOIN
	salgrade s
ON
	e.sal BETWEEN s.losal AND s.hisal
LEFT JOIN
	emp e1
ON
	e.mgr = e1.empno;
```

可得：

```mysql
+--------+-------------+-------+------------+
| ename  | dname       | grade | leadername |
+--------+-------------+-------+------------+
| SIMITH | RESEARCHING |     1 | FORD       |
| ADAMS  | RESEARCHING |     1 | SCOTT      |
| JAMES  | SALES       |     1 | BLAKE      |
| WARD   | SALES       |     2 | BLAKE      |
| MARTIN | SALES       |     2 | BLAKE      |
| MILLER | ACCOUNTING  |     2 | CLARK      |
| ALLEN  | SALES       |     3 | BLAKE      |
| TURNER | SALES       |     3 | BLAKE      |
| JONES  | RESEARCHING |     4 | KING       |
| BLAKE  | SALES       |     4 | KING       |
| CLARK  | ACCOUNTING  |     4 | KING       |
| SCOTT  | RESEARCHING |     4 | JONES      |
| FORD   | RESEARCHING |     4 | JONES      |
| KING   | ACCOUNTING  |     5 | NULL       |
+--------+-------------+-------+------------+
14 rows in set (0.00 sec)
```

在原来的基础之上，再次连接查询，自连接并且进行外连接，得到最终结果。

#### 子查询

> SELECT 语句当中可以嵌套SELECT语句，被嵌套的SELECT语句是子查询。

`SELECT`语句可以出现在以下几个位置：

```mysql
SELECT
	...(可以有SELECT)
FROM
	...(可以有SELECT)
WHERE
	...(可以有SELECT)
```

##### WHERE子句中使用子查询

案例：找出高于平均薪资的员工信息？

这个案例之前已经说过了，因为不能再`WHERE`子句后直接使用分组函数，所以可以使用子查询：

```mysql
SELECT
	*
FROM
	emp
WHERE
	sal > (SELECT AVG(sal) FROM emp);
```

这样就在`WHERE`子句中使用了子查询。

##### FROM之后嵌套子查询

案例：找出每个部门平均薪资的薪资等级？

这个可以进行拆解成小步骤：

第一步，找出每个部门的平均薪资（按照部门编号分成组，再求平均值）：

```mysql
SELECT deptno,AVG(sal) AS avgsal FROM emp GROUP BY deptno;
```

得到：

```mysql
+--------+-------------+
| deptno | avgsal      |
+--------+-------------+
|     10 | 2916.666667 |
|     20 | 2175.000000 |
|     30 | 1566.666667 |
+--------+-------------+
3 rows in set (0.00 sec)
```

第二步，将以上的查询结果当做临时表`t`，让`t`表和`salgrade s`表进行连接，条件为`t.avgsal BETWEEN s.losal AND s.hisal`：

```mysql
SELECT
	t.*, s.grade
FROM
	(SELECT deptno,AVG(sal) AS avgsal FROM emp GROUP BY deptno) t
JOIN
	salgrade s
ON
	t.avgsal BETWEEN s.losal AND s.hisal;
```

可得：

```mysql
+--------+-------------+-------+
| deptno | avgsal      | grade |
+--------+-------------+-------+
|     10 | 2916.666667 |     4 |
|     20 | 2175.000000 |     4 |
|     30 | 1566.666667 |     3 |
+--------+-------------+-------+
3 rows in set (0.00 sec)
```

##### SELECT之后嵌套子查询

案例： 找出每个员工所在的部门名称，要求显示员工名和部门名？

这个案例可以直接用连接查询解决，但是也可以用下面的方式：

```mysql
SELECT
	e.ename,(SELECT d.dname FROM dept d WHERE e.deptno = d.deptno) AS dname
FROM
	emp e;
```

可以看到在`SELECT`之后嵌套了一个子查询，查询出对应的`dname`。

##### EXISTS子查询（相关子查询）

语法：

```mysql
EXISTS(完整的查询语句)
```

这个语句的查询结果可能会返回两个值，一个值是`0`表示查询结果不存在，一个值是`1`表示查询结果存在。

#### UNION

> UNION可以将查询结果集相加。

案例：找出工作岗位是`SALESMAN`和`MANAGER`的员工？

这个案例有几种写法：

第一种使用`OR`：

```mysql
SELECT ename,job FROM emp WHERE job = 'MANAGER' OR job = 'SALESMAN';
```

第二种使用`IN()`：

```mysql
SELECT ename,job FROM emp WHERE job IN('MANAGER', 'SALESMAN');
```

第三种就是使用`UNION`：

```mysql
SELECT ename,job FROM emp WHERE job = 'MANAGER'
UNION
SELECT ename,job FROM emp WHERE job = 'SALESMAN';
```

可得：

```mysql
+--------+----------+
| ename  | job      |
+--------+----------+
| JONES  | MANAGER  |
| BLAKE  | MANAGER  |
| CLARK  | MANAGER  |
| ALLEN  | SALESMAN |
| WARD   | SALESMAN |
| MARTIN | SALESMAN |
| TURNER | SALESMAN |
+--------+----------+
7 rows in set (0.00 sec)
```

可以看到，两张标的数据结合在了一起。但是`UNION`需要两个查询的查询字段数量（列数）相同，否则会报错。

#### LIMIT

> LIMIT可以实现只获取一定数量的查询结果（取结果集中的部分数据）。LIMIT语句不是`SQL`的通用语句，其他的数据库可能没有或有其他的实现语法，比如Oracle中的ROWNUM。

 `LIMIT`的语法：

```mysql
LIMIT startIndex, length
```

`startIndex`表示起始位置（从`0`开始数，不是`1`），`length`表示取几个，如果`startIndex`省略，则默认从`0`开始。



案例：找出工资前五名的员工？

这个案例的思路在于降序取前五个，`SQL`语句如下：

```mysql
SELECT ename,sal FROM emp ORDER BY sal DESC LIMIT 0,5;
```

可得：

```mysql
+-------+---------+
| ename | sal     |
+-------+---------+
| KING  | 5000.00 |
| FORD  | 3000.00 |
| SCOTT | 3000.00 |
| JONES | 2975.00 |
| BLAKE | 2850.00 |
+-------+---------+
5 rows in set (0.00 sec)
```

> LIMIT是SQL语句中最后执行的环节。

## 数据定义和操作语句

### 创建表

建表语法：

```mysql
CREATE TABLE 表名称
(
字段1 数据类型,
字段2 数据类型,
字段3 数据类型,
....
);
```

表名在数据库中一般建议以：`t_`或者`tb_`开头。

创建时可以判断不存在才创建：

```mysql
CREATE TABLE IF NOT EXISTS 表名称
(
字段1 数据类型,
字段2 数据类型,
字段3 数据类型,
....
);
```



创建表时可以为数据指定默认值：

```mysql
CREATE TABLE 表名称
(
字段1 数据类型 DEFAULT 默认值,
字段2 数据类型,
字段3 数据类型,
....
);
```

在插入数据时如果没有显式插入该字段，会用默认值自动填充，假如没有设定默认值且没有进行约束则会用`NULL`进行填充。



##### 常见数据类型

| 数据类型 | 说明                                           |
| -------- | ---------------------------------------------- |
| INT      | 整形                                           |
| BIGINT   | 长整型                                         |
| FLOAT    | 浮点型                                         |
| CHAR     | 定长字符串                                     |
| VARCHAR  | 可变长字符串                                   |
| DATE     | 日期类型                                       |
| BLOB     | 二进制大对象（存储图片、视频等流媒体信息）     |
| CLOB     | 字符大对象（存储大文本，可以存储4个G的字符串） |

 **整数类型后面的宽度与你存储的数据字符长度`无关`，与显示有关**：在设定数据类型时，比如`int(3)`，不是说`int`只能存三个字符，而是在显示时只会显示`3`个字符，它不代表存储的数据范围，具体范围只和数据类型本身的范围有关。
**字符类型后面的宽度与你存储数据类型的字符长度`有关`：**在设定数据类型时，比如`varchar(20)`， 如果你输入了一个`21`个字符的密码，那么保存和显示的只会是前`20`个字符，你将丢失一个字符信息，`char`同理。 



`char`和`varchar`的选择：

在实际开发中，当某个字段中的数据长度不发生改变时，例如性别、生日等都才使用`char`。

当一个字段的长度不确定，例如简介、姓名等都是采用`varchar`。

比如，创建一个班级学生的信息表：

```mysql
 CREATE TABLE t_student (
    -> no BIGINT,
    -> name VARCHAR(255),
    -> sex CHAR(1),
    -> classno VARCHAR(255),
    -> birth CHAR(10)
    -> );
Query OK, 0 rows affected (0.10 sec)
```



### 表的修改

#### 修改列名

```mysql
ALTER TABLE 表名 CHANGE COLUMN 旧列名 新列名 新数据类型;
```

#### 修改列的类型或约束

```mysql
ALTER TABLE 表名 MODIFY COLUMN 列名 新数据类型;
```

#### 增加新列

```mysql
ALTER TABLE 表名 ADD COLUMN 列名 数据类型;
```

#### 删除列

```mysql
ALTER TABLE 表名 DROP COLUMN 列名;
```

#### 修改表名

```mysql
ALTER TABLE 表名 RENAME TO 新表名;
```



### 表的删除

```mysql
DROP TABLE 表名;
```

如果没有这个表，删除就会报出错误，可以检查是否存在：

```mysql
DROP TABLE IF EXISTS 表名; 
```

这样就会存在该表才会删除。



### 插入数据

语法格式：

```mysql
INSERT INTO 表名(字段名1,字段名2,字段名3,...) VALUES(值1,值2,值3,...);
```
字段和值一一对应，数量相同，且数据类型要对应相同。

往上面建的学生表插入数据：

```mysql
INSERT INTO t_student(no,name,sex,classno,birth) VALUES(1,'zhangsan','1','gaosan1ban','1949-10-1');
```

可以插入成功：

```mysql
mysql> SELECT * FROM t_student;
+------+----------+------+------------+-----------+
| no   | name     | sex  | classno    | birth     |
+------+----------+------+------------+-----------+
|    1 | zhangsan | 1    | gaosan1ban | 1949-10-1 |
+------+----------+------+------------+-----------+
1 row in set (0.00 sec)
```



如果只插入一个字段：

```mysql
INSERT INTO t_student(name) VALUES('lisi');
```

这样也是可以插入成功的，只不过其他的字段都自动插入`NULL`：

```mysql
mysql> SELECT * FROM t_student;
+------+----------+------+------------+-----------+
| no   | name     | sex  | classno    | birth     |
+------+----------+------+------------+-----------+
|    1 | zhangsan | 1    | gaosan1ban | 1949-10-1 |
| NULL | lisi     | NULL | NULL       | NULL      |
+------+----------+------+------------+-----------+
2 rows in set (0.00 sec)
```



插入数据时也可以这么写：

```mysql
INSERT INTO t_student VALUES(3,'wangmazi','0','gaosan2ban','1947-11-11');
```

可以看到省略了全部的字段名称，这样插入数据需要插入值的顺序要与数据库中字段的顺序要一致，并且不能省略任何一个字段。



还可以一次插入多行数据：

```mysql
INSERT INTO t_student
	(no,name,sex,classno,birth)
VALUES
	(4,'rose','0','gaoyi2ban','1949-12-12'),(5,'jack','1','gaoyi2ban','1949-12-12');
```

多行数据一起插入，数据之间用逗号隔开。

#### 表的复制

如果只是复制表的结构：

```mysql
CREATE TABLE 新表名 LIKE 旧表名;
```

如果只要复制部分结构：

```mysql
CREATE TABLE 新表名 SELECT 需要的字段1,字段2 FROM 旧表名 WHERE 0;
```

如果没有条件会将数据和结构都复制过来，但是条件设置为一个不可能成立的值就可以只复制结构了，设置为`0`永远都为`falsy`值，所以会只复制结构。



要复制结构和数据，语法如下：

```mysql
CREATE TABLE emp1 SELECT * FROM emp;
```

这个语句将后面的查询结果当做表进行创建。

也可以不全部查询：

```mysql
CREATE TABLE emp2 SELECT ename,empno FROM emp;
```

这样创建出来的`emp2`只会有`ename`、`empno`两个字段。

还可以加条件：

```mysql
CREATE TABLE emp3 SELECT ename,empno FROM emp WHERE 条件;
```

这样只有符合条件的才会复制过来。



#### 将查询结果插入到一张表中

```mysql
INSERT INTO emp1 SELECT * FROM emp;
```

这样会将搜索结果全部插入到`emp1`中，当然，这两张表的字段和顺序要相同（这里`emp1`是`emp`的复制）。

### 修改表中的数据
语法格式：
```mysql
UPDATE 表名 SET 字段名1=值1,字段名2=值2,字段名3=值3,... WHERE 条件;
```
**当没有条件时整张表的数据将全部更新。**

案例：将部门`10`的`LOC`改为`SHANGHAI`，部门名称改为`RENSHIBU`？

先复制一张表用于学习：

```mysql
CREATE TABLE dept1 SELECT * FROM dept;
```

然后进行修改：

```mysql
UPDATE dept1 SET loc='SHANGHAI',dname='RENSHIBU' WHERE deptno=10;
```

可得：

```mysql
+--------+-------------+----------+
| deptno | dname       | loc      |
+--------+-------------+----------+
|     10 | RENSHIBU    | SHANGHAI |
|     20 | RESEARCHING | DALLAS   |
|     30 | SALES       | CHICAGO  |
|     40 | OPERATIONS  | BOSTON   |
+--------+-------------+----------+
4 rows in set (0.00 sec)
```

### 删除数据

语法格式：

```mysql
DELETE FROM 表名 WHERE 条件;
```

**当没有条件时全部删除。**

案例： 删除编号为`10`的部门？

```mysql
DELETE FROM dept1 WHERE deptno = 10;
```



>  **如何删除大表？**

语法格式：

```mysql
TRUNCATE TABLE 表名;
```

但需要注意的是：

- `TRUNCATE`不支持条件删除，只能删除整个表。
- **`TRUNCATE`不支持回滚，所以必须小心使用**。
- 由于`DELETE`删除时没删除一行会在事务日志中记录，所以速度会慢，但是`TRUNCATE`由于不支持回滚，所以速度快。

### 约束

在创建表时，可以给表的字段添加对应的约束，添加约束的目的是为了保证表中数据的合法性、有效性、完整性。

常见的约束有：

- 非空约束（not null）：约束的字段不能为`NULL`。
- 唯一约束（unique）：约束的字段不能重复（可以为`NULL`，可以有多个`NULL`）。
- 默认约束（default）：给字段添加默认值。
- 主键约束（primary key）：约束的字段既不能为`NULL`，也不能重复，主键是一条数据的唯一标识，一张表只能有一个字段有主键约束，表的设计中要求每张表都要有主键，主键不建议和业务挂钩（主键不该用业务数据做主键，因为业务数据改变可能会改变），用单独的比如`id`来当主键，主键值可以设置自增，只要写成`PRIMARY KEY AUTO_INCREMENT`即可，主键字段会自动维护一个自增的数字，从`1`开始，在插入时不需要插入主键字段的值。
- 外键约束（foreign key）：用于限制两个表的关系，用于保证该字段的值必须来自主表的关联列的值，在从表添加外键约束，用于引用主表中某列的值，如果添加外键约束的字段插入的数据不属于 引用表字段的某个值（外键约束的值可以为`NULL`），就会报错。
- 检查约束（check）：插入时进行检查，符合条件才允许插入，`MySQL`不支持。

约束的添加分类：

- 列级约束：语法上都支持，但外键约束没有效果。
- 表级约束：除了非空、默认约束，其他都支持。

#### 列级约束

```mysql
CREATE TABLE major(
	id INT PRIMARY KEY,
    majorName VARCHAR(20)
)

CREATE TABLE stuinfo(
	id INT PRIMARY KEY, #主键
    stuName VARCHAR(20) NOT NULL, #非空
    gender CHAR(1) CHECK(gender='男' OR gender='女'), #检查
    seat INT UNIQUE, #唯一
    age INT DEFAULT 18, #默认约束
    majorId INT FOREIGN KEY REFERENCES major(id) #外键
);

```

外键约束中 `major(id)`，`major`是引用的表，`id`是表中的字段。

#### 表级约束

```mysql
DROP TABLE IF EXISTS stuinfo;
CREATE ATBLE stuinfo(
	id INT,
    stuname VARCHAR(20),
    gender CHAR(1),
    seat INT,
    age INT,
    majorid INT,
    
    CONSTRAINT pk PRIMARY KEY(id), #主键
    CONSTRAINT uq UNIQUE(seat), #唯一约束
    CONSTRAINT ck CHECK(gender='男' OR gender='女'), #检查
    CONSTRAINT fk_stuinfo_major FOREIGN KEY(majorid) REFERENCES major(id) #外键
);
```

可以看到，表级约束在所有字段的最下面定义，语法：

```mysql
CONSTRAINT 约束名（不可以重复） 约束类型(要约束的字段)
```

其中，`CONSTRAINT`和`约束名`可以省略。

表级约束中，可以多个字段联合添加一个约束：

```mysql
UNIQUE(字段1,字段2)
```

这时，只有两个字段都重复时，才会受到约束。

#### 修改表时添加约束

```mysql
ALTER TABLE 表名 MODIFY COLUMN 列名 类型 约束;
```

```mysql
ALTER TABLE 表名 ADD 约束类型(字段);
```

## 事务

什么是事务？

> 一个事务是一个完整的业务逻辑单元，不可再分。

比如：银行转账，从`A`账户向`B`账户转账`10000`，需要执行两条`UPDATE`语句：
```mysql
UPDATE t_act SET balance = balance - 10000 WHERE actno = 'act-001';
UPDATE t_act SET balance = balance + 10000 WHERE actno = 'act-002';
```
以上两条`DML`语句必须同时成功，或者同时失败，而不允许出现一条成功，一条失败。想要保证两条`DML`语句同时成功同时失败，就需要使用数据库的“事务机制”。

**只有`DML`语句（`INSERT DELETE UPDATE`）才支持事务。**因为这三个操作都是直接和数据库表中的数据直接相关的，事务的存在就是为了保证数据的完整性、安全性。



事务的特性？

事务包括四大特性（`ACID`）：

- `A`：原子性： 事务是最小的工作单位，不可拆分。
- `C`：一致性： 事务必须保证多条`DML`语句同时成功或者同时失败。
- `I`：隔离性： 事务之间具有隔离。
- `D`：持久性：持久性说的是最终数据必须持久化到硬盘文件中，事务才算成功的结束。



事务的隔离性存在隔离级别，理论上包括`4`个：

- 第一级别：读未提交（`Read Uncommitted`）。对方事务还没有提交，我们当前事务就可以读取到对方未提交的数据。读未提交存在脏读（`Dirty Read`）现象：表示读到了脏的数据。
- 第二级别：读已提交（`Read Committed`）。对方事务提交之后的数据我方可以读取到。这种隔离级别解决了脏读现象，但是存在的问题是不可重复读。
- 第三级别：可重复读（`Repeatable Read`）。这种隔离级别解决了不可重复读的问题，但是存在的问题是读取到的数据是幻象。
- 第四级别：序列化读/串行化读。解决了上面的所有问题，但是效率低，需要事务排队。

`Oracle`数据库默认隔离级别是：读已提交。`MySQL`默认是可重复读。



`MySQL`事务默认情况下是自动提交的（只要执行任意一条`DML`语句就提交一次）。怎么关闭？使用`START TRANSACTION`：

```mysql
DROP TABLE IF EXISTS t_uer;
CREATE TABLE t_user(
	id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(255)
);
```

当我们给表中插入数据时，会自动提交，使用`ROLLBACK（使用ROLLBACK事务会结束）`无法回滚。

```mysql
INSERT INTO t_user(username) VALUES('zhangsan');
START TRANSACTION;
INSERT INTO t_user(username) VALUES('lisi');
SELECT * FROM t_user;
```

会插入两条数据：

```mysql
+----+----------+
| id | username |
+----+----------+
|  1 | zhangsan |
|  2 | lisi     |
+----+----------+
2 rows in set (0.00 sec)
```

接下来回滚：

```mysql
ROLLBACK;
SELECT * FROM t_user;
```

得到：

```mysql
+----+----------+
| id | username |
+----+----------+
|  1 | zhangsan |
+----+----------+
1 row in set (0.00 sec)
```

发现可以成功回滚。

接下来再次开启事务：

```mysql
START TRANSACTION;
INSERT INTO t_user(username) VALUES('wangwu');
INSERT INTO t_user(username) VALUES('rose');
INSERT INTO t_user(username) VALUES('jack');
#提交事务
COMMIT;
#回滚
ROLLBACK;
SELECT * FROM t_user;
```

得到：

```mysql
+----+----------+
| id | username |
+----+----------+
|  1 | zhangsan |
|  3 | wangwu   |
|  4 | rose     |
|  5 | jack     |
+----+----------+
4 rows in set (0.00 sec)
```

发现回滚无效，所以**回滚只能回滚到上一次的提交点，而`COMMIt`已经将数据持久化到硬盘之中了。**

可以使用`SAVEPOINT`来设置保存点：

```mysql
START TRANSCATION;
DELETE FROM account WHERE id=25;
#设置保存点
SAVEPOINT a;
DELETE FROM account WHERE id=30;
#回滚到保存点
ROLLBACK TO a;
```





## 索引

什么是索引？有什么用？

> 索引相当于一本书的目录，通过目录可以快速地找到对应的资源。在数据库中，查询一张表有两种检索方式，第一种是全表扫描，第二种是根据索引检索（效率很高），索引缩小了扫描的范围。

索引虽然可以提高检索效率，但是不能随意添加索引，因为索引也是数据库当中的对象，也需要数据库不断维护，有维护成本。比如表中的数据经常被修改，这样就不适合添加索引，因为数据一旦修改，索引就要重新排序，进行维护。

什么时候考虑给字段添加索引：

- 数据量庞大
- 该字段很少进行`DML`操作
- 该字段经常出现在`WHERE`子句中



**主键和具有`UNIQUE`约束的字段会自动添加索引。所以根据主键查询效率较高，尽量根据主键检索。**

索引在模糊查询且第一个字符是通配符且是`%`时无效。

可以使用在查询语句增加`explain`，返回的结果中的`type`来判断该次查询时全表扫描还是索引扫描，如果是`all`则是全表扫描，`ref`是索引扫描。

### 创建索引

```mysql
CREATE INDEX 索引名 ON 表名(字段名);
```

这条语句就会给对应表的字段添加上索引。

### 删除索引

```mysql
DROP INDEX 索引名 ON 表名;
```



## 视图

什么是视图？

> 站在不同的角度去看待数据（同一张表的数据，通过不同的角度去看待）。

如何创建和删除视图？

```mysql
#创建
CREATE VIEW myview AS SELECT empno,ename FROM emp;
#删除
DROP VIEW myview;
```

对视图进行增删改查，会影响到原表数据（通过视图影响数据，不是直接操作的原表），可以对视图进行`CRUD`。

视图的作用？

视图可以隐藏表的实现细节。保密级别较高的系统，数据库只对外提供相关的视图，程序员只对视图进行`CRUD`。

## DBA命令

### 新建用户

```mysql
CREATE USER 用户名 IDENTIFIED BY'密码';
```

### 授权

```mysql
GRANT ALL PRIVILEGES ON 库名.表名 to '用户名'@'登录ip' IDENTIFIED BY '密码' WITH GRANT OPTION;
```

- `*`表示所有数据库
- `*`表示所有表
- `%`表示任何`ip`
- `WITH GRANT OPTION`表示该用户还可以授权给其他用户



> 细粒度授权

首先以`root`用户进入`MySQL`，然后键入命令：

```mysql
GRANT SELECT,INSERT,UPDATE,DELETE ON *.* to 用户名@localhost IDENTIFIED BY '密码';
```

如果希望该用户能够在任何机器上登录`MySQL`，将`localhost`改为`%`。

> 粗粒度授权

```mysql
GRANT ALL PRIVILEGES ON *.* TO '用户名'@'%' IDENTIFIED BY '密码';
```

这条命令授权的用户不能给其他用户授权，如果想要可以给其他用户授权，可以在后面加`WITH GRANT OPTION`。

`PRIVILEGES`包括：

- ALTER：修改数据库的表
- CREATE：创建新的数据库或表
- DELETE：删除表数据
- DROP：删除数据库、表
- INDEX：创建、删除索引
- INSERT：添加表数据
- SELECT：查询表数据
- UPDATE：更新表数据
- ALL：允许任何操作
- USAGE：只允许登录



### 回收权限

```mysql
REVOKE PRIVILEGES ON 库名[.表名] FROM 用户名;
```

```mysql
REVOKE ALL PRIVILEGES ON *.* FROM 用户名;
```



### 导入导出

#### 导出

在`windows`的`dos`命令行窗口执行：

```bash
mysqldump 库名>导出位置 -u用户名 -p密码
```

比如：

```mysql
mysqldump studydata>D:\studydata.sql -umyname -p123
```

也可以只导出指定表：

```mysql
mysqldump 库名 表名>导出位置 -u用户名 -p密码
```



#### 导入

首先要创建数据库，然后转到数据库中，使用`source`命令使用`sql脚本即可`：

```mysql
CREATE DATABASE studydata;
USE studydata;
SOURCE D:\studydata.sql
```



## 数据库设计三范式

什么是设计范式？

> 设计表的依据，按照这个范式设计的表不会出现数据冗余。

三范式都是那些？

- 第一范式：任何一张表都应该有主键，并且每一个字段原子性不可再分。
- 第二范式：在第一范式基础上，所有非主键字段完全依赖主键，不能产生部分依赖（多对多，三张表，关系表里有两个外键）。
- 第三范式：在第二范式基础上，所有非主键字段直接依赖主键，不能产生传递依赖（一对多，两张表，多数据的表加外键）。

实际开发中，以满足客户需求为主，有的时候会拿冗余换执行速度。



一对一的设计中有两种方案：

- 主键共享：两张表共用一个主键，一张表的主键的外键为另一张表的主键。
- 外键唯一：让一张表的一个字段外键为另一张表的主键，并且这个外键字段用唯一约束。