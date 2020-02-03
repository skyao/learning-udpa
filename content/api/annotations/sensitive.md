---
date: 2020-02-02T17:00:00+08:00
title: Sensitive注解
menu:
  main:
    parent: "api-annotations"
weight: 312
description : "Sensitive annotation"
---

定义来自：https://github.com/cncf/udpa/blob/master/udpa/annotations/sensitive.proto

### SensitiveAnnotation 说明

Sensitive注解将某个字段标记为“敏感”字段。

### SensitiveAnnotation定义

SensitiveAnnotation 可以用于 Field：

```protobuf
extend google.protobuf.FieldOptions {
  // 此文件中的 Magic number 源自 "udpa.annotations.sensitive" 的SHA256摘要的前28位。
  // 设置为 true 时，“敏感/sensitive”表示该字段包含敏感数据，
  // 例如个人身份信息，密码或私钥，并且应通过了解此注释的工具对其进行编辑以进行显示。 
  // 注意，这对标准的 Protobuf 函数没有影响，例如 `TextFormat::PrintToString` 。
  bool sensitive = 76569463;
}
```

### 相关issue

- [implement SensitiveAnnotation](https://github.com/cncf/udpa/pull/17)
- [Make the "sensitive" annotation a message rather than a raw bool. ](https://github.com/cncf/udpa/pull/18): 这个issue提交后没有merge，很快就直接close了。



