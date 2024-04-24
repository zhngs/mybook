# 字段存在性

### 一.简介

字段存在性分两种情况：

* 不存储存在信息（no presence），pb生成的代码只会存储值，不会存储字段是否设置的信息。
* 存储存在信息（explicit presence），pb生成的代码不仅会存储值，也会存储字段是否设置的信息。

### 二.语义区别

不存储存在信息（no presence）：

* 默认值不会进行序列化。
* 默认值不会被合并。
* 想要清理一个字段，设置成默认值即可。
* 字段是默认值意味着：
  * 字段被显式设置为默认值。
  * 字段被clear（通过设置成默认值）。
  * 字段没有被设置过。

存储存在信息（explicit presence）：

* 字段总是会进行序列化。
* 未设置的字段不会被合并。
* 显式设置字段（包含默认值）会被合并。
* 会有类似`has_foo`的方法来表示字段是否被设置。
* 会有类似`clear_foo`的方法来清理字段。

### 三.判断方法

对于proto2来说：

| Field type             | 是否存储存在信息 |
| ---------------------- | -------- |
| Singular field         | yes      |
| Singular message field | yes      |
| Field in a oneof       | yes      |
| Repeated field & map   | no       |

对于proto3来说（v3.15版本以后）：

| Field type             | 是否存储存在信息                    |
| ---------------------- | --------------------------- |
| _Other_ singular field | `optional`设置了为true，否则为flase |
| Singular message field | yes                         |
| Field in a oneof       | yes                         |
| Repeated field & map   | no                          |

### 四.例子

#### 1.c++例子

不存储存在信息（no presence）

```cpp
Msg m = GetProto();
if (m.foo() != 0) {
  // "Clear" the field:
  m.set_foo(0);
} else {
  // Default value: field may not have been present.
  m.set_foo(1);
}
```

存储存在信息（explicit presence）

```cpp
Msg m = GetProto();
if (m.has_foo()) {
  // Clear the field:
  m.clear_foo();
} else {
  // Field is not present, so set it.
  m.set_foo(1);
}
```

#### 2.go例子

不存储存在信息（no presence）

```go
m := GetProto()
if m.Foo != 0 {
  // "Clear" the field:
  m.Foo = 0
} else {
  // Default value: field may not have been present.
  m.Foo = 1
}
```

存储存在信息（explicit presence）

```go
m := GetProto()
if m.Foo != nil {
  // Clear the field:
  m.Foo = nil
} else {
  // Field is not present, so set it.
  m.Foo = proto.Int32(1)
}
```
