# SQL

### 一.通用语法

* 单行或多行书写，以分号结尾。
* 可以使用空格或者缩进来增加可读性。
* 不区分大小写。
* 单行注释使用`--`或者`#`，多行注释使用`/* */`

### 二.分类

<table><thead><tr><th width="328">名称</th><th>说明</th></tr></thead><tbody><tr><td>Data Definition Language（DDL）</td><td>数据定义语言，用来定义数据库对象(数据库，表， 字段)</td></tr><tr><td>Data Manipulation Language（DML）</td><td>数据操作语言，用来对数据库表中的数据进行增删改</td></tr><tr><td>Data Query Language（DQL）</td><td>数据查询语言，用来查询数据库中表的记录</td></tr><tr><td>Data Control Language（DCL）</td><td>数据控制语言，用来创建数据库用户、控制数据库的 访问权限</td></tr></tbody></table>

### 三.数据类型

#### 1.数值

* tinyint，1byte，有符号范围是(-128，127)，无符号范围是(0，255)。
* smallint，2bytes，有符号范围是(-32768，32767)，无符号范围是(0，65535)。
* mediumint，3bytes，有符号范围是(-8388608，8388607)，无符号范围是(0，16777215)。
* int/integer，4bytes，有符号范围是(-2147483648， 2147483647)，无符号范围是(0，4294967295)。
* bigint，8bytes，有符号范围是(-2^63，2^63-1)，无符号范围是(0，2^64-1)。
* float，4bytes，有符号范围是(-3.402823466 E+38， 3.402823466351 E+38)，无符号范围是0 和 (1.175494351 E- 38，3.402823466 E+38)。
* double，8bytes，有符号范围是(-1.7976931348623157 E+308， 1.7976931348623157 E+308)，无符号范围是0 和 (2.2250738585072014 E-308， 1.7976931348623157 E+308)。
* decimal，范围依赖于M(精度)和D(标度) 的值，表示限定小数位和整数位的小数。

#### 2.字符串

* char，0-255 bytes，定长字符串(需要指定长度)。
* varchar，0-65535 bytes，变长字符串(需要指定长度)。
* tinyblob，0-255 bytes，不超过255个字符的二进制数据。
* tinytext，0-255 bytes，短文本字符串。
* blob，0-65535 bytes，二进制形式的长文本数据。
* text，0-65535 bytes，长文本数据。
* mediumblob，0-16777215 bytes，二进制形式的中等长度文本数据。
* mediumtext，0-16777215 bytes，中等长度文本数据。
* longblob，0-4294967295 bytes，二进制形式的极大文本数据。
* longtext，0-4294967295 bytes，极大文本数据。

#### 3.日期时间

* date，范围 1000-01-01 至 9999-12-31。
* time，范围 -838:59:59 至 838:59:59。
* year，范围 1901 至 2155。
* datetime，范围 1000-01-01 00:00:00 至 9999-12-31 23:59:59。
* timestamp，范围 1970-01-01 00:00:01 至 2038-01-19 03:14:07。

### 四.DDL

#### 1、定义DB

* 查询所有DB：

{% code fullWidth="false" %}
```sql
show database;
```
{% endcode %}

* 查询当前数据库：

```sql
select database();
```

* 创建数据库：

```sql
create database [ if not exists ] 数据库名 [ default charset 字符集 ] [ collate 排序 规则 ] ;
```

* 删除数据库：

```sql
drop database [ if exists ] 数据库名;
```

* 切换数据库：

```sql
use 数据库名;
```

#### 2.定义table

* 查询表

```sql
show tables;
```

* 查看表结构

```sql
desc 表名;
```

* 查询建表语句

```sql
show create table 表名;
```

* 创建表

```sql
create table 表名(
    字段1 字段1类型 [comment 字段1注释],
    字段2 字段2类型 [comment 字段2注释],
    字段3 字段3类型 [comment 字段3注释],
    ...... 字段n 字段n类型 [comment 字段n注释]
) [comment 表注释];
```

* 删除表

```sql
drop table [ if exists ] 表名;
```

* 删除指定表，并创建新表

```sql
truncate table 表名;
```

#### 3.修改table

* 添加字段

```sql
alter table 表名 add 字段名 类型 (长度) [ comment 注释 ] [ 约束 ];
```

* 修改数据类型

```sql
alter table 表名 modify 字段名 新数据类型 (长度);
```

* 修改字段名和字段类型

```sql
alter table 表名 change 旧字段名 新字段名 类型 (长度) [ comment 注释 ] [ 约束 ];
```

* 删除字段

```sql
alter table 表名 drop 字段名;
```

* 修改表名

```sql
alter table 表名 rename to 新表名;
```

### 五.DML

#### 1、添加数据

* 给指定字段添加数据

```sql
insert into 表名 (字段名1, 字段名2, ...) values (值1, 值2, ...);
```

* 给全部字段添加数据

```sql
insert into 表名 values (值1, 值2, ...);
```

* 批量添加数据

```sql
insert into 表名 (字段名1, 字段名2, ...) values (值1, 值2, ...), (值1, 值2, ...), (值 1, 值2, ...);

insert into 表名 values (值1, 值2, ...), (值1, 值2, ...), (值1, 值2, ...);
```

#### 2、修改数据

```sql
update 表名 set 字段名1 = 值1 , 字段名2 = 值2 , .... [ where 条件 ];
```

#### 3、删除数据

```sql
delete from 表名 [ where 条件 ];
```

### 六.DQL

