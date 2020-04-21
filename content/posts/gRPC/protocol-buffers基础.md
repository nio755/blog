---
title: Protocol Buffers 使用
date: 2020-04-20T09:08:15+08:00
lastmod: 2020-04-20T09:08:15+08:00
author: nio
cover: /img/grpc-cover.png
categories: ["gRPC"]
tags: ["rpc"]
# showcase: true
draft: true
---

- [定义消息类型](#%e5%ae%9a%e4%b9%89%e6%b6%88%e6%81%af%e7%b1%bb%e5%9e%8b)
  - [分配字段编号](#%e5%88%86%e9%85%8d%e5%ad%97%e6%ae%b5%e7%bc%96%e5%8f%b7)
  - [指定字段规则](#%e6%8c%87%e5%ae%9a%e5%ad%97%e6%ae%b5%e8%a7%84%e5%88%99)
  - [添加注释](#%e6%b7%bb%e5%8a%a0%e6%b3%a8%e9%87%8a)
  - [.proto 产生了什么?](#proto-%e4%ba%a7%e7%94%9f%e4%ba%86%e4%bb%80%e4%b9%88)
  - [默认值](#%e9%bb%98%e8%ae%a4%e5%80%bc)
- [标量值类型(基本类型)](#%e6%a0%87%e9%87%8f%e5%80%bc%e7%b1%bb%e5%9e%8b%e5%9f%ba%e6%9c%ac%e7%b1%bb%e5%9e%8b)
- [保留字段](#%e4%bf%9d%e7%95%99%e5%ad%97%e6%ae%b5)
- [枚举(Enumerations)](#%e6%9e%9a%e4%b8%beenumerations)
  - [枚举保留值](#%e6%9e%9a%e4%b8%be%e4%bf%9d%e7%95%99%e5%80%bc)
- [使用其他消息类型](#%e4%bd%bf%e7%94%a8%e5%85%b6%e4%bb%96%e6%b6%88%e6%81%af%e7%b1%bb%e5%9e%8b)
  - [导入定义](#%e5%af%bc%e5%85%a5%e5%ae%9a%e4%b9%89)
  - [使用 proto2 消息类型](#%e4%bd%bf%e7%94%a8-proto2-%e6%b6%88%e6%81%af%e7%b1%bb%e5%9e%8b)
- [嵌套类型](#%e5%b5%8c%e5%a5%97%e7%b1%bb%e5%9e%8b)
- [更新消息类型](#%e6%9b%b4%e6%96%b0%e6%b6%88%e6%81%af%e7%b1%bb%e5%9e%8b)
- [未知字段](#%e6%9c%aa%e7%9f%a5%e5%ad%97%e6%ae%b5)
- [Any](#any)
- [Oneof](#oneof)
  - [oneof 的功能](#oneof-%e7%9a%84%e5%8a%9f%e8%83%bd)
  - [向后兼容问题](#%e5%90%91%e5%90%8e%e5%85%bc%e5%ae%b9%e9%97%ae%e9%a2%98)
    - [标签重用问题](#%e6%a0%87%e7%ad%be%e9%87%8d%e7%94%a8%e9%97%ae%e9%a2%98)
- [Maps](#maps)
  - [向后兼容](#%e5%90%91%e5%90%8e%e5%85%bc%e5%ae%b9)
- [Packages](#packages)
- [定义 Service](#%e5%ae%9a%e4%b9%89-service)
- [JSON Mapping](#json-mapping)
  - [JSON 选项](#json-%e9%80%89%e9%a1%b9)
- [Options](#options)
  - [自定义 Options](#%e8%87%aa%e5%ae%9a%e4%b9%89-options)
- [生成对应语言的代码](#%e7%94%9f%e6%88%90%e5%af%b9%e5%ba%94%e8%af%ad%e8%a8%80%e7%9a%84%e4%bb%a3%e7%a0%81)
- [proto 文件风格](#proto-%e6%96%87%e4%bb%b6%e9%a3%8e%e6%a0%bc)
- [Encoding](#encoding)
  - [Base 128 Varints](#base-128-varints)
  - [message 结构](#message-%e7%bb%93%e6%9e%84)
  - [更多](#%e6%9b%b4%e5%a4%9a)
- [参考](#%e5%8f%82%e8%80%83)

**当前基于 proto3**

## 定义消息类型

```proto
syntax = "proto3"; //指定使用proto3语法，不指定将默认为proto2

//定义消息，每一个字段为type+name
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

### 分配字段编号

消息定义中的每个字段都有一个唯一的编号。
field number 范围 [1,2^29-1],1 到 15 的范围需要一个字节进行编码，16 打 2047 需要两个字节，对常出现的字段使用 1 到 15，不能使用[19000,19999],这些是保留字段号

### 指定字段规则

消息字段可以是：

- singular：格式正确的 message 可以有`零个或一个`这样的字段
- `repeated`：格式正确的 message，此字段可以重复任意次（包括零次）。重复值的顺序将保留。

在 proto3 中，默认情况下，标量数字类型的`repeated`字段使用`packed`编码。

### 添加注释

使用 C / C ++样式`//`和`/* ... */`语法。

```proto
/* SearchRequest represents a search query, with pagination options to
 * indicate which results to include in the response. */

message SearchRequest {
  string query = 1;
  int32 page_number = 2;  // Which page number do we want?
  int32 result_per_page = 3;  // Number of results to return per page.
}
```

### .proto 产生了什么?

| 语言        | 生成文件                                                                                                                              |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| C++         | .h 和.cc；文件中描述的每种消息类型都有一个类                                                                                          |
| Java        | .java；以及用于创建消息类实例的特殊 Builder 类                                                                                        |
| Python      | Python 编译器会生成一个模块，该模块带有.proto 中每种消息类型的静态描述符,然后与元类一起使用，以在运行时创建必要的 Python 数据访问类。 |
| go          | 生成一个.pb.go 文件，其中包含文件中每种消息类型的类型                                                                                 |
| Ruby        | 将使用包含您的消息类型的 Ruby 模块生成一个.rb 文件                                                                                    |
| Objective-C | 从每个.proto 生成一个 pbobjc.h 和 pbobjc.m 文件，并为文件中描述的每种消息类型提供一个类                                               |
| C#          | 从每个.proto 生成一个.cs 文件，并为文件中描述的每种消息类型提供一个类                                                                 |
| Dart        | 生成一个.pb.dart 文件，其中包含文件中每种消息类型的类。                                                                               |

### 默认值

解析 message 时，如果编码的 message 不包含特定的单数元素，则已解析对象中的相应字段将设置为该字段的默认值。这些默认值是特定于类型的：

| 类型     | 默认值                                                                                                                          |
| -------- | ------------------------------------------------------------------------------------------------------------------------------- |
| string   | 空字符串                                                                                                                        |
| bytes    | 空 byte 数组                                                                                                                    |
| bool     | false                                                                                                                           |
| 数组类型 | 0                                                                                                                               |
| enum     | 默认值是第一个定义的枚举值，必须为 0。                                                                                          |
| message  | 未设置该字段。它的确切值取决于语言，参考[API Reference](https://developers.google.com/protocol-buffers/docs/reference/overview) |

重复字段的默认值为空（通常为相应语言的空列表）。

注意，对于标量消息字段，解析消息后，就无法判断字段是否已明确设置为默认值(例如，布尔值是否设置为 false)或根本不设置：定义消息类型时应牢记这一点。还要注意，如果将标量消息字段设置为其默认值，则该值将不会被序列化。

## 标量值类型(基本类型)

参考具体内容，查阅[官方文档](https://developers.google.com/protocol-buffers/docs/proto3#scalar)

- double、float、int32、int64、uint32、uint64
- sint32、sint64 - 使用可变长度编码。有符号的 int 值。与常规 int32、int64 相比，它们更有效地编码负数
- fixed32 - 始终为四个字节。如果值通常大于 2^28，则比 uint32 更有效。
- fixed64 - 始终为八个字节。如果值通常大于 2^56，则比 uint64 更有效。
- sfixed32、sfixed64 - 有符号的 fixed32 和 fixed64
- bool、string、bytes

## 保留字段

如果通过完全删除一个字段或将其注释掉来更新消息类型，则将来的用户在自己对该类型进行更新时可以重用该字段号。如果它们以后加载同一.proto 的旧版本，则可能导致严重的问题，包括数据损坏，隐私错误等。为了避免这些情况的发生，可以使用保留字段`reserved`，如果将来有任何用户尝试使用这些字段标识符，则协议缓冲区编译器会报错。

```proto
message Foo {
  string field2 = 2;
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

output=> `Field "field2" uses reserved number 2.`

注意：**不能在同一保留语句中混合使用字段名和字段号**。如`reserved 2, "foo";`

## 枚举(Enumerations)

```proto
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
  }
  Corpus corpus = 4;
}
```

Corpus 枚举的第一个常量映射为零：每个枚举定义必须包含一个映射为零的常量作为其第一个元素。这是因为：

- 必须有一个零值，以便我们可以使用 0 作为数字默认值。
- 零值必须是第一个元素，以便与 proto2 语义兼容，其中第一个枚举值始终是默认值。

您可以通过将相同的值分配给不同的枚举常量来定义别名。为此，您需要将`allow_alias`选项设置为`true`，否则，当找到别名时，协议编译器将生成错误消息。

```proto
enum EnumAllowingAlias {
  option allow_alias = true;
  UNKNOWN = 0;
  STARTED = 1;
  RUNNING = 1;
}
enum EnumNotAllowingAlias {
  UNKNOWN = 0;
  STARTED = 1;
  // RUNNING = 1;  // Uncommenting this line will cause a compile error inside Google and a warning message outside.
}
```

- 枚举常量必须在 32 位整数范围内；
- 负值的效率不高，不建议使用负值；
- 可以在消息内部定义枚举，或消息外部定义枚举，以便重复使用；
- 还可以使用语法 MessageType.EnumType 将在一条消息中声明的枚举类型用作另一条消息中的字段类型；
- 两个全局枚举类型，不能出现相同的枚举值类型，如上的 EnumAllowingAlias 和 EnumNotAllowingAlias 同时出现在全局枚举中，编译出错。

```proto
//将在一条消息中声明的枚举类型用作另一条消息中的字段类型
message SearchRequest {
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
  }
  Corpus corpus = 4;
}
message SearchRequest1 {
  SearchRequest.Corpus src = 1;
}
```

在反序列化期间，无法识别的枚举值将保留在消息中，尽管在反序列化消息时如何表示该值取决于语言。 在支持具有超出指定符号范围的值的开放式枚举类型的语言（例如 C++和 Go）中，未知的枚举值仅存储为其基础整数表示形式。 在具有封闭枚举类型的语言（例如 Java）中，枚举中的大小写用于表示无法识别的值，并且可以使用特殊的访问器访问基础整数。 无论哪种情况，如果消息被序列化，则无法识别的值仍将与消息一起序列化。

### 枚举保留值

可以使用 max 关键字指定保留的数值范围达到最大可能值。

```proto
enum Foo {
  reserved 2, 15, 9 to 11, 40 to max;
  reserved "FOO", "BAR";
}
```

## 使用其他消息类型

您可以使用其他消息类型作为字段类型。 例如，假设您要在每条 SearchResponse 消息中包括结果消息-为此，您可以在同一`.proto`中定义结果消息类型，然后在 SearchResponse 中指定结果类型的字段：

```proto
message SearchResponse {
  repeated Result results = 1;
}

message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}
```

### 导入定义

如上，“Result”消息类型与 SearchResponse 定义在同一文件中，如果要使用其他文件的消息类型，可以通过导入其他.proto 文件使用它们的定义：

```proto
import "myproject/other_protos.proto";
```

默认情况下，您只能使用直接导入的.proto 文件中的定义。 但是，有时您可能需要将.proto 文件移动到新位置。 现在，您可以直接在原始位置放置一个虚拟的.proto 文件，而不是直接移动.proto 文件并一次更改所有 call sites，而是使用`import public`将所有导入转发到新位置。

```proto
// new.proto
// All definitions are moved here
```

```proto
// old.proto
// This is the proto that all clients are importing.
import public "new.proto";
import "other.proto";
```

```proto
// client.proto
import "old.proto";
// You use definitions from old.proto and new.proto, but not other.proto
```

编译器使用`-I/-proto_path`标志在编译器命令行中指定的一组目录中搜索导入的文件。 如果未给出任何标志，它将在调用编译器的目录中查找。 通常，应将--proto_path 标志设置为项目的根目录，并对所有导入使用完全限定的名称。

### 使用 proto2 消息类型

可以导入 proto2 消息类型并在 proto3 消息中使用它们，反之亦然。 但是，不能直接在 proto3 语法中使用 proto2 枚举。

## 嵌套类型

```proto
message SearchResponse {
  message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
  }
  repeated Result results = 1;
}

message SomeOtherMessage {
  SearchResponse.Result result = 1;
}

message Outer {  // Level 0
  message MiddleAA {  // Level 1
    message Inner {   // Level 2
      int64 ival = 1;
      bool  booly = 2;
    }
  }
  message MiddleBB {  // Level 1
    message Inner {   // Level 2
      int32 ival = 1;
      bool  booly = 2;
    }
  }
}
```

## 更新消息类型

如果现有消息类型不再满足您的所有需求（例如，您希望消息格式具有一个额外的字段），但是您仍然希望使用以旧格式创建的代码，请不要担心！ 在不破坏任何现有代码的情况下更新消息类型非常简单。 只要记住以下规则：

- 不要更改任何现有字段的字段编号。
- 如果添加新字段，则仍可以使用新生成的代码来解析使用“旧”消息格式通过代码序列化的任何消息。 您应记住这些元素的默认值，以便新代码可以与旧代码生成的消息正确交互。 同样，由新代码创建的消息可以由旧代码解析：旧的二进制文件在解析时只会忽略新字段。 有关详细信息，请参见“未知字段”部分。
- 只要在更新的消息类型中不再使用字段号，就可以删除字段。 您可能想要重命名该字段，或者添加前缀“ OBSOLETE\_”，或者保留该字段编号，以使.proto 的将来用户不会意外重用该编号。
- int32，uint32，int64，uint64 和 bool 都是兼容的–这意味着您可以将字段从这些类型中的一种更改为另一种，而不会破坏向前或向后的兼容性。 如果解析出一个不适合相应类型的数字，则将获得与在 C++中将数字强制转换为该类型一样的效果（例如，如果将 64 位数字读为 int32， 它将被截断为 32 位）。
- sint32 和 sint64 彼此兼容，但与其他整数类型不兼容。
- string 和 bytes 是兼容的，只要字节是有效的 UTF-8。
- 如果 bytes 包含消息的编码版本，则嵌入式消息与 bytes 兼容。
- fixed32 与 sfixed32 兼容，fixed64 与 sfixed64 兼容。
- enum 在格式方面与 int32，uint32，int64 和 uint64 兼容（请注意，如果值不合适，该值将被截断）。 但是请注意，客户端代码在反序列化消息时可能会以不同的方式对待它们：例如，无法识别的 proto3 枚举类型将保留在消息中，但是在反序列化消息时如何表示这取决于语言。 Int 字段始终只是保留其值。
- 将单个值更改为新的`oneof`的成员是安全且二进制兼容的。 如果您确定没有代码一次设置多个字段，那么将多个字段移动到一个新的`oneof`中可能是安全的。 将任何字段移动到现有的`oneof`中都是不安全的。

## 未知字段

未知字段是格式正确的序列化数据，表示解析器无法识别的字段。例如，当旧二进制文件解析具有新字段的新二进制文件发送的数据时，这些新字段将成为旧二进制文件中的未知字段。

最初，proto3 消息在解析过程中总是丢弃未知字段，但是在版本 3.5 中，重新引入了保留未知字段以匹配 proto2 行为的功能。 在版本 3.5 和更高版本中，未知字段将在解析期间保留并包含在序列化输出中。

## Any

Any 消息类型使您可以将消息用作嵌入式类型，而无需它们的 `.proto` 定义，`Any`包含任意序列化的消息（以字节为单位）以及`URL`，URL 作为该消息的类型并解析为该消息的类型的全局唯一标识符。要使用`Any`类型，您需要`import google/protobuf/any.proto`。

```proto
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```

给定消息类型的默认类型 URL 为`type.googleapis.com/packagename.messagename`。

不同的语言实现将支持运行时库帮助程序以类型安全的方式打包和解压缩 Any 值-例如，在 Java 中，Any 类型将具有特殊的 pack()和 unpack()访问器，而在 C++中则具有 PackFrom()和 UnpackTo()方法：

```c++
// Storing an arbitrary message type in Any.
NetworkErrorDetails details = ...;
ErrorStatus status;
status.add_details()->PackFrom(details);

// Reading an arbitrary message from Any.
ErrorStatus status = ...;
for (const Any& detail : status.details()) {
  if (detail.Is<NetworkErrorDetails>()) {
    NetworkErrorDetails network_error;
    detail.UnpackTo(&network_error);
    ... processing network_error ...
  }
}
```

**Currently the runtime libraries for working with Any types are under development.**

## Oneof

如果您的消息包含多个字段，并且最多同时设置一个字段，则可以使用 oneof 功能强制执行此行为并节省内存。

Oneof 字段类似于常规字段，除了在 oneof 中的所有字段共享内存，并且同一时间只能设置一个字段。设置 oneof 中的任何成员会自动清除所有其他成员。您可以使用特殊方法检查 oneof 设置了哪个值，使用**case()**或**WhichOneof()**方法，依赖于当前使用的语言。

```proto
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```

在 oneof 中可以添加任何类型的字段，但不能使用`repeated`的字段。在生成的代码中，oneof 字段具有与常规字段相同的 getter 和 setter。

### oneof 的功能

- 设置 oneof 字段将自动清除 oneof 的所有其他成员。因此，如果您设置了几个字段，则只有您设置的最后一个字段仍具有值。

```c++
SampleMessage message;
message.set_name("name");
CHECK(message.has_name());
message.mutable_sub_message();   // Will clear name field.
CHECK(!message.has_name());
```

- 如果解析器在线路上遇到同一个对象的多个成员，则在解析的消息中仅使用最后看到的成员
- oneof 不能使用`repeated`
- 反射 API 适用于其中一个字段。
- 如果将 oneof 字段设置为默认值（例如将 int32 的 oneof 字段设置为 0），则将设置该 oneof 字段的“case”，并且该值将在线路上序列化。
- 如果您使用的是 C++，请确保您的代码不会导致内存崩溃。以下示例代码将崩溃，因为通过调用 set_name()方法已经删除了 sub_message。

```c++
SampleMessage message;
SubMessage* sub_message = message.mutable_sub_message();
message.set_name("name");      // Will delete sub_message
sub_message->set_...            // Crashes here
```

- 同样，在 C++中，如果您用`Swap()`交换`oneof`的两条消息，则每条消息将以另一个的`oneof`的 case 结尾：在下面的示例中，`msg1`将具有`sub_message`，而`msg2`将具有`name`。

```c++
SampleMessage msg1;
msg1.set_name("name");
SampleMessage msg2;
msg2.mutable_sub_message();
msg1.swap(&msg2);
CHECK(msg1.has_sub_message());
CHECK(msg2.has_name());
```

### 向后兼容问题

添加或删除一个字段时要小心。如果检测一个 oneof 的值返回`None/NOT_SET`，这可能意味着 oneof 未被设置或它已设置为另一个版本的字段。由于无法知道线上的未知字段是否是 oneof 的成员，因此无法分辨出差异。

#### 标签重用问题

- 将字段移入或移除 oneof：message 被序列化和解析后，您可能会丢失一些信息（某些字段将被清除）。但是，您可以安全地将单个字段移动到新的 oneof 中，并且如果已知只设置了一个字段，则可以移动多个字段。
- 删除一个 oneof 字段并将其添加回来：在 message 被序列化和解析之后，这可能会清除您当前设置的 oneof 字段。
- 拆分或合并 oneof：这与移动常规字段有类似的问题。

## Maps

```proto
map<key_type, value_type> map_field = N;
```

key_type 可以是任何整数或字符串类型(除了浮点类型和 bytes 之外)。枚举不是有效的`key_type`。 `value_type`可以是除另一个映射以外的任何类型。

```proto
map<string, Project> projects = 3;
```

- map 字段不能是`repeated`
- map 的格式排序和迭代顺序是不定的，因此不能依赖于 map 项的特定顺序。
- 为`.proto`生成文本格式时，map 按 key 排序。数字 keys 按数字排序
- 解析或合并时，如果存在重复的 map keys，则使用最后获取的 key。当从文本格式解析 map，如果 keys 重复，解析可能会失败。
- 如果为 map 字段提供 key 但没有 value，则序列化字段时的行为取决于语言。在 C ++，Java 和 Python 中，类型的默认值是序列化的，而在其他语言中，则没有序列化的值。

### 向后兼容

map 的语法和以下语法等效，因此不支持 map 的 protobuf 仍可以处理数据：

```proto
message MapFieldEntry {
  key_type key = 1;
  value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```

任何支持 map 的 protobuf 实现都必须产生并接受可以被上述定义接受的数据。

## Packages

您可以将可选的`package`说明符添加到`.proto`文件中，以防止协议消息类型之间的名称冲突。

```proto
package foo.bar;
message Open { ... }
```

然后，可以在定义消息类型的字段时使用包说明符：

```proto
message Foo {
  ...
  foo.bar.Open open = 1;
  ...
}
```

包说明符影响生成的代码的方式取决于您选择的语言，如 C++ 为 foo::bar

## 定义 Service

如果要将消息类型与 RPC 系统一起使用，则可以在`.proto`文件中定义 RPC 服务接口，并且编译器将以您选择的语言生成服务接口代码和存根 stub。例如，如果要使用接收 SearchRequest 并返回 SearchResponse 的方法来定义 RPC 服务，则可以在.proto 文件中进行如下定义：

```proto
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```

## JSON Mapping

Proto3 支持 JSON 中的规范编码，从而使在系统之间共享数据更加容易。

如果 JSON 编码的数据中缺少某个值，或者该值为 null，则在解析时，它将被解析为适当的默认值。 如果字段中具有默认值，则默认情况下会在 JSON 编码数据中将其省略以节省空间。一个 implementation 可以提供选项，以在 JSON 编码的输出中发出具有默认值的字段。

相关参考[JSON Mapping](https://developers.google.com/protocol-buffers/docs/proto3#json)

### JSON 选项

一个 proto3 JSON implementation 可以提供以下选项:

- 发出具有默认值的字段：默认情况下，proto3 JSON 输出中会省略具有默认值的字段。一个 implementation 可以提供一个选项，以使用其默认值覆盖此行为和输出字段。
- 忽略未知字段：Proto3 JSON 解析器默认情况下应拒绝未知字段，但可以提供在解析中忽略未知字段的选项。
- 使用 proto field name 代替 lowerCamelCase name：默认情况下，proto3 JSON printer 应将字段名称转换为 lowerCamelCase 并将其用作 JSON 名称。一个 implementation 可以提供一个选项，改为使用原型字段名称作为 JSON 名称。Proto3 JSON 解析器必须接受转换后的 lowerCamelCase 名称和原型字段名称。
- 将枚举值作为整数而不是字符串发送：默认情况下，JSON 输出中使用枚举值的名称。可以提供一个选项使用枚举值的数字值来代替。

## Options

.proto 文件中的各个声明可以使用 option 进行注释。选项不会改变声明的整体含义，但可能会影响在特定上下文中处理声明的方式。可用选项的完整列表在`google/protobuf/descriptor.proto`中定义。

以下是一些最常用的选项：

- `java_package`（file option）
- `java_multiple_files` (file option)：使顶消息，枚举和服务在包级别定义，而不是在以`.proto`文件命名的外部类内部定义
- `java_outer_classname`（file option）：要生成的最外层 Java 类的类名，如果为显式指定，则用驼峰式大小写来构造类名（foo_bar.proto 变为 FooBar.java）
- `optimize_for`（file option）：可以设置为 `SPEED`, `CODE_SIZE`或 `LITE_RUNTIME`，这会通过以下方式影响 C ++和 Java 代码生成器（可能还有第三方生成器）：
  - `SPEED` (default)：编译器将生成代码，用于对消息类型进行序列化，解析和执行其他常见操作。此代码已高度优化。
  - `CODE_SIZE`：编译器将生成最少的类，并将依赖于基于反射的共享代码来实现序列化，解析和其他各种操作。 因此，生成的代码将比使用 SPEED 的代码小得多，但是操作会更慢。 类仍将实现与 SPEED 模式下完全相同的公共 API。 此模式在包含大量.proto 文件且不需要所有文件都快速达到要求的应用程序中最有用。
  - `LITE_RUNTIME`：编译器将生成仅依赖于“精简版”运行时库的类（libprotobuf-lite 而非 libprotobuf）。 精简版运行时比完整库要小得多（大约小一个数量级），但省略了某些功能，例如描述符和反射。 这对于在受限平台（例如手机）上运行的应用程序特别有用。 编译器仍将像在 SPEED 模式下一样快速生成所有方法的实现。 生成的类将仅以每种语言实现 MessageLite 接口，该接口仅提供完整 Message 接口方法的子集。
- `cc_enable_arenas`（file option）：为 C ++生成的代码启用 arena 分配
- `objc_class_prefix`（file option）
- `deprecated`（file option）：如果设置为 true，则表明该字段已弃用，并且不应由新代码使用。如果想阻止他人使用该字段，考虑使用保留字段(reserved)

```proto
option java_package = "com.example.foo";
option java_multiple_files = true;
option java_outer_classname = "Ponycopter";
option optimize_for = CODE_SIZE;
int32 old_field = 6 [deprecated=true];
```

### 自定义 Options

参考[Custom Options](https://developers.google.com/protocol-buffers/docs/proto#customoptions)

## 生成对应语言的代码

`protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto`

- `IMPORT_PATH`指定解析导入指令时要在其中查找.proto 文件的目录。 如果省略，则使用当前目录。可以通过多次传递`--proto_path`选项来指定多个导入目录。`-I = IMPORT_PATH`可以用作`--proto_path`的缩写。
- 可以提供一个或多个输出指令：`--cpp_out`,`--java_out`,`--go_out`等
- 为了更加方便，如果 DST_DIR 以.zip 或.jar 结尾，则编译器会将输出写入具有给定名称的单个 ZIP 格式的存档文件中。 根据 Java JAR 规范的要求，还将为.jar 输出提供清单文件。 注意，如果输出归档文件已经存在，它将被覆盖； 编译器无法将文件添加到现有存档中。
- 您必须提供一个或多个.proto 文件作为输入。 可以一次指定多个.proto 文件。即使这些文件是相对于当前目录命名的，但是每个文件都必须位于 IMPORT_PATH 之一中，以便编译器可以确定其规范名称。

## proto 文件风格

- 每行 80 字符
- 使用 2 个空格作为缩进
- 文件应命名为 lower_snake_case.proto

- 文件结构如下

```txt
1. License header (if applicable)
2. File overview
3. Syntax
4. Package
5. Imports (sorted)
6. File options
7. Everything else
```

- 软件包名称应为小写，并且应与目录层次结构相对应。例如，如果文件位于`my/package/`中，则包名称应为`my.package`。
- message 和字段命名：message 命名使用 CamelCase，字段使用下划线，如果字段名称包含数字，则该数字应出现在字母之后而不是下划线之后

```proto
message SongServerRequest {
  string song_name = 1;
  string song_name1 = 2;
}
```

- 对重复的字段使用复数名称。

```proto
  repeated string keys = 1;
  repeated MyMessage accounts = 17;
```

- enum：使用 CamelCase（以大写字母开头）作为枚举类型名称，并使用 CAPITALS_WITH_UNDERSCORES 作为值名称。每个枚举值都应以分号（而不是逗号）结尾。优先为枚举值添加前缀，而不是将其包含在封闭 message 中。零值枚举应具有后缀 UNSPECIFIED。

```proto
enum Foo {
  FOO_UNSPECIFIED = 0;
  FOO_FIRST_VALUE = 1;
  FOO_SECOND_VALUE = 2;
}
```

- 定义了 RPC 服务，则应使用 CamelCase（以大写字母开头）作为服务名称和任何 RPC 方法名称。

```proto
service FooService {
  rpc GetSomething(FooRequest) returns (FooResponse);
}
```

- 需要避免的事：
  - Required fields (only for proto2)
  - Groups (only for proto2)

## Encoding

创建 message

```proto
message Test1 {
  optional int32 a = 1;
}
```

在应用程序中，创建 Test1 消息并将 a 设置为 150。然后将消息序列化为输出流。如果您能够检查编码后的消息，则会看到三个字节：`08 96 01`

### Base 128 Varints

Varint 是一种使用一个或多个字节序列化整数的方法。较小的数字占用较少的字节数。

除了最后一个字节外，varint 中的每个字节都有最高有效位（msb）设置——这表明还会有更多字节。每个字节的低 7 位用于以 7 位为一组存储数字的二进制补码表示，最低有效组优先存储。

因此，例如，这里是数字 1——它是一个字节，因此未设置 msb：`0000 0001`

这是 300 –这有点复杂：`1010 1100 0000 0010`

如何确定这是 300？首先，您从每个字节中删除 msb，因为这是在告诉我们是否到达数字的末尾（如您所见，它设置在第一个字节中，因为 varint 中有多个字节） ：

```txt
 1010 1100 0000 0010
→ 010 1100  000 0010
```

反转两组 7 位，varint 将数字从最低有效位开始存储。然后，将它们连接起来以获得最终值：

```txt
000 0010  010 1100
→  000 0010 ++ 010 1100
→  100101100
→  256 + 32 + 8 + 4 = 300
```

### message 结构

protobuf 消息是一系列键值对，消息的二进制版本仅使用字段的编号作为 key–每个字段的名称和声明的类型只能在解码端通过引用消息类型的定义来确定。

对消息进行编码时，键和值被串联到一个字节流中。对消息进行解码时，解析器需要能够跳过无法识别的字段，这样，可以将新字段添加到消息中，而不会破坏不知道它们的旧程序。为此，每对消息中的“key”实际上是两个值——`.proto`文件中的字段编号以及提供仅提供足够信息以查找值长度的 wire 类型。在大多数语言实现中，此键称为标签。

可用的 wire 类型如下：

| Type | Meaning          | Used For                                                 |
| ---- | ---------------- | -------------------------------------------------------- |
| 0    | Varint           | int32, int64, uint32, uint64, sint32, sint64, bool, enum |
| 1    | 64-bit           | fixed64, sfixed64, double                                |
| 2    | Length-delimited | string, bytes, embedded messages, packed repeated fields |
| 3    | Start group      | groups (deprecated)                                      |
| 4    | End group        | groups (deprecated)                                      |
| 5    | 32-bit           | fixed32, sfixed32, float                                 |

流式消息中的每个 key 都是具有值`(field_number << 3) | wire_type`的 varint——换句话说，数字的最后三位存储 wire 类型。

如下例子，流中的第一个数字始终是 varint 键，这是 08，或者(删除 MSB)`000 1000`

您使用最后三位获得 wire 类型（0），然后右移三位以获得字段编号（1）。因此，您现在知道字段号为 1 且值为 varint。使用 varint 解码知识，您可以看到接下来的两个字节存储值 150。

```txt
96 01 = 1001 0110  0000 0001
       → 000 0001  ++  001 0110 (drop the msb and reverse the groups of 7 bits)
       → 10010110
       → 128 + 16 + 4 + 2 = 150
```

### 更多

参考：[Encoding](https://developers.google.com/protocol-buffers/docs/encoding)

## 参考

> - [Protocol Buffers](https://developers.google.com/protocol-buffers)
