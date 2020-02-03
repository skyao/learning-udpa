---
date: 2020-02-02T17:00:00+08:00
title: Migrate注解
menu:
  main:
    parent: "api-annotations"
weight: 311
description : "Migrate annotation"
---

定义来自：https://github.com/cncf/udpa/blob/master/udpa/annotations/migrate.proto

### MigrateAnnotation 说明

MigrateAnnotation 注解用于标记在前后版本中的和迁移相关的API变更。

### MigrateAnnotation 定义

MigrateAnnotation 可以用于 Message / Field / Enum / Enum Value / File：

```protobuf
// 此文件中的 Magic number 源自 "udpa.annotation.migrate" 的SHA256摘要的前28位。
extend google.protobuf.MessageOptions {
  MigrateAnnotation message_migrate = 171962766;
}

extend google.protobuf.FieldOptions {
  FieldMigrateAnnotation field_migrate = 171962766;
}

extend google.protobuf.EnumOptions {
  MigrateAnnotation enum_migrate = 171962766;
}

extend google.protobuf.EnumValueOptions {
  MigrateAnnotation enum_value_migrate = 171962766;
}

extend google.protobuf.FileOptions {
  FileMigrateAnnotation file_migrate = 171962766;
}
```

### 相关的消息定义

```protobuf
message MigrateAnnotation {
  // 在下一个版本中重命名 message/enum/enum value
  string rename = 1;
}

message FieldMigrateAnnotation {
  // 在下一个版本中重命名字段
  string rename = 1;

  // 在下一版本中，将该字段添加到命名的oneof。 
  // 如果已经存在，则该字段将加入oneof，否则将使用给定名称创建一个新的oneof。
  string oneof_promotion = 2;
}

message FileMigrateAnnotation {
  // 将文件中的所有类型移动到另一个包，这意味着更改 proto 文件路径。
  string move_to_package = 2;
}
```

### 相关issue

- [add migrate annotations](https://github.com/cncf/udpa/pull/13)：加了 rename
- [Add package move annotation](https://github.com/cncf/udpa/pull/20/files)： 加了 move_to_package 
- [Oneof promotion field migration annotation](https://github.com/cncf/udpa/pull/21/files): 加了 oneof_promotion 

