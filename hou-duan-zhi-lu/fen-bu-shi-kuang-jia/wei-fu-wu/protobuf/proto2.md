# proto2

### 一.简介

新项目建议使用proto3，但是可能会有一些存量的项目仍然在使用proto2，这里主要介绍proto2和proto3的不同之处。

### 二.语法

对于每一个字段，必须由如下标签修饰：

* optional：表示字段0个或1个，会存储字段的存在信息。
* repeated：表示字段0个或多个。
* map：repeated的简写形式。
* required：表示字段必须存在，这个标签强烈不推荐使用。

proto2的字段前必有一个label，proto3可以不存在label。
