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

基本查询的语法如下：

```
SELECT 字段列表 
FROM 表名列表 
WHERE 条件列表
GROUP BY 分组字段列表 
HAVING 分组后条件列表
ORDER BY 排序字段列表 
LIMIT 分页参数
```

执行顺序如下：

* from --> where --> group by --> having --> select --> order by --> limit

#### 1.基础查询

* 查询多个字段

```sql
SELECT 字段1, 字段2, 字段3 ... FROM 表名;

SELECT * FROM 表名;
```

* 字段设置别名

```sql
SELECT 字段1 [ AS 别名1 ] , 字段2 [ AS 别名2 ] ... FROM 表名;

SELECT 字段1 [ 别名1 ] , 字段2 [ 别名2 ] ... FROM 表名;
```

* 去除重复记录

```sql
SELECT DISTINCT 字段列表 FROM 表名;
```

#### 2.条件查询

条件查询的语法如下，核心在于条件列表：

```sql
SELECT 字段列表 FROM 表名 WHERE 条件列表 ;
```

具体条件如下：

* 大小比较：大于`>`、大于等于`>=`、小于`<`、小于等于`<=`、等于`=`、不等于`<>`或`!=`。
* 在某个范围内（包含最大最小值）：`between .. and ...`。
* 在in之后列表的值：`in (...)`。
* 模糊匹配，`_`匹配单个字符，`%`匹配任意个字符：`like 占位符`。
* 是null：`is null`。

可以使用逻辑运算符组合多个条件：

* 且：and 或 &
* 或：or 或 ||
* 非：not 或 !

#### 3.聚合函数

聚合函数是将一列数据作为一个整体，进行纵向计算：

* count：统计数量
* max：最大值
* min：最小值
* avg：平均值
* sum：求和

语法如下（null值不参与运算）：

```sql
SELECT 聚合函数(字段列表) FROM 表名;
```

#### 4.分组查询

分组语法如下：

```sql
SELECT 字段列表
FROM 表名
[ WHERE 条件 ]
GROUP BY 分组字段名
[ HAVING 分组 后过滤条件 ];
```

本质上分组是挑选一个特定字段，将该字段在横向上分组，有去重效果，常和聚合函数搭配。

需要注意：

* 分组之后，查询的字段一般为聚合函数和分组字段，查询其他字段无任何意义。
* 执行顺序是where > 聚合函数 > having，having可以对聚合字段进行判断。
* 支持多字段分组, 具体语法为 : group by columnA,columnB。

#### 5.排序查询

语法如下：

```sql
SELECT 字段列表 FROM 表名 ORDER BY 字段1 排序方式1 , 字段2 排序方式2 ;
```

注意：

* ASC是升序（默认值），DESC是降序。
* 多字段排序，只有第一个字段值相同时，才会根据第二个字段进行排序。

#### 6.分页查询

语法如下：

```sql
SELECT 字段列表 FROM 表名 LIMIT 起始索引, 查询记录数;
```

注意：

* 起始索引从0开始，起始索引 = （查询页码 - 1）\* 每页显示记录数。
* 分页查询是数据库的方言，不同的数据库有不同的实现，MySQL中是LIMIT。
* 如果查询的是第一页数据，起始索引可以省略，直接简写为 limit 10。

### 七.DCL

