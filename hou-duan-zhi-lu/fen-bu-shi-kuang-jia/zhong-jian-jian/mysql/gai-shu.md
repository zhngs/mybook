# 概述

### 一.数据库基本概念

首先明确以下三个概念：

* DataBase（DB）：数据库，存储数据的仓库。
* DataBase Management System (DBMS)：数据库管理系统，操作DB的大型软件。
* Structured Query Language (SQL)：操作关系型数据库的编程语言，同时也是所有关系型数据库的统一标准。

### 二.关系型数据库

关系型数据库除了MySQL，还有SQL Server、PostgreSQL、SQLLite等。

关系型数据库主要有如下特点：

* 使用二维表存储数据。DBMS可以创建多个DB，每个DB里可以有多张表，每张表有多条记录。
* 使用SQL操作数据。更准确地说是通过MySQL客户端操作DBMS，然后DBMS操作DB。
