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

### 三.拓展

拓展可以定义message中的field，但是不必和message在同一个文件，主要作用是减少依赖。

#### 1.拓展例子

如下是在kittens/video\_ext.proto文件中定义的一个拓展：

```protobuf
// file kittens/video_ext.proto

import "kittens/video.proto";
import "media/user_content.proto";

package kittens;

// This extension allows kitten videos in a media.UserContent message.
extend media.UserContent {
  // Video is a message imported from kittens/video.proto
  repeated Video kitten_videos = 126;
}
```

被定义拓展的media/user\_content.proto文件中的message需要留出拓展号，并且写上相应的声明：

```protobuf
// file media/user_content.proto

package media;

// A container message to hold stuff that a user has created.
message UserContent {
  extensions 100 to 199 [
    declaration = {
      number: 126,
      full_name: ".kittens.kitten_videos",
      type: ".kittens.Video",
      repeated: true
    },
   // Ensures all field numbers in this extension range are declarations.
    verification = DECLARATION];
}
```

以下几点需要注意：

* media/user\_content.proto不需要导入kittens/video\_ext.proto中的拓展定义。
* 拓展字段在序列化的时候和正常字段没区别，将拓展字段移动到message中或者从message里抽出字段到拓展是兼容的。
* 拓展字段和标准字段生成的api有区别，拓展字段需要用特殊的api来读取。
* 拓展字段不能是oneof或者map。

下面是c++中操作拓展字段的例子：

```cpp
UserContent user_content;
user_content.AddExtension(kittens::kitten_videos, new kittens::Video());
assert(1 == user_content.GetExtensionCount(kittens::kitten_videos));
user_content.GetExtension(kittens::kitten_videos, 0);
```

### 四.option

#### 1.自定义option

下面定义了一个message级别的option，方法是拓展MessageOptions：

```protobuf
import "google/protobuf/descriptor.proto";

extend google.protobuf.MessageOptions {
  optional string my_option = 51234;
}

message MyMessage {
  option (my_option) = "Hello world!";
}
```

这样定义后在c++中可以这样读取option，option本质上是个字段而已，用来附加一些相关信息：

```cpp
string value = MyMessage::descriptor()->options().GetExtension(my_option);
```

下面是个所有option级别都支持的例子：

```protobuf
import "google/protobuf/descriptor.proto";

extend google.protobuf.FileOptions {
  optional string my_file_option = 50000;
}
extend google.protobuf.MessageOptions {
  optional int32 my_message_option = 50001;
}
extend google.protobuf.FieldOptions {
  optional float my_field_option = 50002;
}
extend google.protobuf.OneofOptions {
  optional int64 my_oneof_option = 50003;
}
extend google.protobuf.EnumOptions {
  optional bool my_enum_option = 50004;
}
extend google.protobuf.EnumValueOptions {
  optional uint32 my_enum_value_option = 50005;
}
extend google.protobuf.ServiceOptions {
  optional MyEnum my_service_option = 50006;
}
extend google.protobuf.MethodOptions {
  optional MyMessage my_method_option = 50007;
}

option (my_file_option) = "Hello world!";

message MyMessage {
  option (my_message_option) = 1234;

  optional int32 foo = 1 [(my_field_option) = 4.5];
  optional string bar = 2;
  oneof qux {
    option (my_oneof_option) = 42;

    string quux = 3;
  }
}

enum MyEnum {
  option (my_enum_option) = true;

  FOO = 1 [(my_enum_value_option) = 321];
  BAR = 2;
}

message RequestType {}
message ResponseType {}

service MyService {
  option (my_service_option) = FOO;

  rpc MyMethod(RequestType) returns(ResponseType) {
    // Note:  my_method_option has type MyMessage.  We can set each field
    //   within it using a separate "option" line.
    option (my_method_option).foo = 567;
    option (my_method_option).bar = "Some string";
  }
}
```
