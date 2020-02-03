---
date: 2020-02-02T17:00:00+08:00
title: Status注解
menu:
  main:
    parent: "api-annotations"
weight: 313
description : "Status annotation"
---

定义来自：https://github.com/cncf/udpa/blob/master/udpa/annotations/status.proto

### StatusAnnotation 说明

Status注解标记状态，比如将某个文件标记为“work_in_progress/进行中”。

### StatusAnnotation定义

StatusAnnotation 可以用于 File：

```protobuf
// 此文件中的 Magic number 源自 "udpa.annotation.status" 的SHA256摘要的前28位。
extend google.protobuf.FileOptions {
  StatusAnnotation file_status = 222707719;
}
```

相关的消息定义

```protobuf
message StatusAnnotation {
  // 该实体正在进行中，并且可能会发生重大更改。
  bool work_in_progress = 1;
}
```

### 相关的issue

- [annotations: add WiP status annotations](https://github.com/cncf/udpa/pull/22): 加了 work_in_progress 

