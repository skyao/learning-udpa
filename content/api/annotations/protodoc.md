---
date: 2020-02-02T17:00:00+08:00
title: Protodoc注解（已移除）
menu:
  main:
    parent: "api-annotations"
weight: 315
description : "Protodoc annotation"
---

ProtodocAnnotation 最终没有加入到UDPA，而是保留在 Envoy 当中。详细见 issue：

https://github.com/cncf/udpa/pull/14/

### ProtodocAnnotation定义

```protobuf
message ProtodocAnnotation {
  // Not implemented yet annotation. This will hide the
  bool not_implemented_hide = 1;
}
```

### 讨论过程

- htuch：

	因此，这实际上是一个关键问题。 UDPA通常应与客户端无关，因此不应反映Envoy的实现状态。 我认为其他注释是通用API的演变，但这个注解是Envoy特有的。 因此，我建议我们暂时将其放入Envoy 仓库中。 对于Envoy以外的UDPA字段，我们将不得不弄清楚一些故事，但这可能超出了我们对v3关心的范围。

- Lizan：

	好吧，我认同 “is this feature implemented” 是特定于每种实现的，而不是特定于Envoy的，因此这就是我在此处发布注释的原因。 另一个原因是我认为我们将在这里结束移动我们的 proto工具（包括 protoxform/protodoc）。 关键问题是如何标记未实现的UDPA消息（对于每个实现），而不是该注解所在的位置。

- htuch：

	我认为使用大量项目的项目必须使用一些基准工具才能使用Protos和UDPA格式的通用文档，然后在生成文档之前叠加自己的其他自定义设置。 我实际上认为这里有许多可行的选择，例如 我们可以让每个文档使用共同的资源来产生自己的文档树，或者我们可以让各种项目提供覆盖并生成一组中央文档，这些文档具有联邦责任来传达实现的内容。

	无论哪种方式，每次添加或删除新字段时，我们都不会提交UDPA存储库。

	我将其添加到下一个UDPA-WG的议程中，我们应该讨论更多。 同时，我建议将此注解移回Envoy。 CC @ mattklein123

- martklein123:

	我同意这是一个非常混乱的话题，我们必须深思。 现在，我同意我们应该将其移回，直到我们了解情况。

> 补充：个人意见，我比较赞成Lizan的说法，应该有一个方式来标记未实现的UDPA消息，而这个定义应该是对所有UDPA实现都通用的，因此定义在envoy这种单个实现中（意味着每个UDPA实现都要做这个事情）是不合适的。

### 后续跟进

- Envoy 的 xDS v3 中是否有这个注解？待确定中。