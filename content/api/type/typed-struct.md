---
date: 2020-02-02T17:00:00+08:00
title: Typed Struct
menu:
  main:
    parent: "api-type"
weight: 311
description : "Typed Struct"
---

定义来自：https://github.com/cncf/udpa/blob/master/udpa/type/v1/typed_struct.proto

### TypedStruct说明

TypedStruct包含任意 JSON 序列化后的 protocol buffer 消息，以及一个描述序列化消息类型的URL。 这与 `google.protobuf.Any` 非常相似，它使用 `google.protobuf.Struct` 作为值，而不是使用 protocol buffer 二进制。

该消息旨在嵌入到 Any 中，因此不应直接从其他 UDPA 消息中引用该消息。

打包不透明的扩展配置时，出于效率考虑，最好将期望的类型打包到 Any 中。 仅当 proto 描述符不可用时才使用TypedStruct，例如：

- 控制平面以人类可读的格式（例如JSON或YAML）发送不透明消息，该消息最初来自外部来源。
- 控制平面不了解 protocol buffer 格式，因此无法以 protocol buffer 二进制格式序列化该消息。
- DPLB不了解其插件或扩展使用的 protocol buffer 格式。 这必须在 DPLB 能力协商中指出。

当 DPLB 收到存放在 Any 中的 TypedStruct 时，它应该：

- 检查 TypedStruct 的 type_url 是否与扩展所期望的类型匹配。
- 将值转换为 type_url 中描述的类型并执行验证。

	  TODO（lizan）：找出 TypeStruct 如何与未将 protobuf 描述符与DPLB本身链接的 DPLB 扩展一起使用（例如，gRPC LB插件，Envoy WASM扩展）。

### TypedStruct 定义

```protobuf
message TypedStruct {
  // 用于唯一标识序列化 protocol buffer 消息的类型的URL
  // 这与 google.protobuf.Any 中描述的语义和格式相同：
  // https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/any.proto
  string type_url = 1;

  // 上述指定类型的JSON表示形式。
  google.protobuf.Struct value = 2;
}
```

### 相关issue

- [issue：Add TypedStruct](https://github.com/cncf/udpa/pull/7)
- [re-deprecate opaque Struct config](https://github.com/envoyproxy/envoy/issues/8403)

