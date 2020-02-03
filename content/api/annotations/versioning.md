---
date: 2020-02-02T17:00:00+08:00
title: Versioning注解
menu:
  main:
    parent: "api-annotations"
weight: 314
description : "Versioning annotation"
---

定义来自：https://github.com/cncf/udpa/blob/master/udpa/annotations/versioning.proto

### VersioningAnnotation 说明

VersioningAnnotation注解用于记录版本信息，比如当前message在上一个版本中的类型。

### VersioningAnnotation定义

VersioningAnnotation 可以用于 Message：

```protobuf
extend google.protobuf.MessageOptions {
  // Magic number 源自xds： 0x78 ('x') 0x44 ('D') 0x53 ('S')
  VersioningAnnotation versioning = 7881811;
}
```

### 相关的消息定义

```protobuf
message VersioningAnnotation {
  // 跟踪以前的消息类型。 
  // 例如：此消息可能是 udpa.foo.v3alpha.Foo ，以前是 udpa.bar.v2.Bar。 
  // UDPA通过 proto 描述符使用此信息。
  string previous_message_type = 1;
}
```

### 相关issue

- [api: added versioning annotations](https://github.com/cncf/udpa/pull/9)

