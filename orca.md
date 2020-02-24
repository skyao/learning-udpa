---
date: 2020-02-02T22:00:00+08:00
title: OrcaLoadReport
menu:
  main:
    parent: "api-data"
weight: 331
description : "OpenRcaService"
---

定义来自：https://github.com/cncf/udpa/blob/master/udpa/service/orca/v1/orca.proto

### OrcaLoadReport说明

ORCA load report format

https://github.com/envoyproxy/envoy/issues/6614

摘录并翻译部分内容：

> 如今，在Envoy中，可以通过考虑后端负载（例如CPU）的本地或全局知识来做出简单的负载均衡决策。更复杂的负载均衡决策可能需要借助特定于应用的知识，例如队列深度，或组合多个指标。
> 
> 这对于可能在多个维度上受到资源限制的服务（例如，CPU和内存都可能成为瓶颈，取决于所应用的负载和执行环境，无法确定是哪个先触及瓶颈），以及这些维度不在预定类别中的位置时很有用（例如，资源可能是“池中的可用线程数”，磁盘IOPS等）。
> 
> https://docs.google.com/document/d/1NSnK3346BkBo1JUU3I9I5NYYnaJZQPt8_Z_XCBCI3uA/edit# 为开放请求成本汇总（ORCA）标准提供了设计建议，以在Envoy和上游等代理之间传递此信息。我们建议作为UDPA的标准部分，并得到Envoy的支持。

有关 Orca 的更详细的信息，请见设计文档 [Open Request Cost Aggregation (ORCA)](https://docs.google.com/document/d/1NSnK3346BkBo1JUU3I9I5NYYnaJZQPt8_Z_XCBCI3uA/) 。

### OrcaLoadReport定义

```protobuf
message OrcaLoadReport {
  // CPU利用率表示为可用CPU资源的一部分。 应该来自最新的样本或测量。
  double cpu_utilization = 1 [(validate.rules).double.gte = 0, (validate.rules).double.lte = 1];

  // 内存利用率表示为可用内存资源的一部分。 应该来自最新的样本或测量。
  double mem_utilization = 2 [(validate.rules).double.gte = 0, (validate.rules).double.lte = 1];

  // 端点已服务的总RPS。 应该涵盖端点负责的所有服务。
  uint64 rps = 3;

  // 特定于应用程序的请求成本。 
  // 每个值都是与该请求相关联的绝对成本（例如3487字节的存储空间）。
  map<string, double> request_cost = 4;

  // 资源利用率值。
  // 每个值都表示为从最新样本或度量得出的可用总资源的一部分。
  map<string, double> utilization = 5
      [(validate.rules).map.values.double.gte = 0, (validate.rules).map.values.double.lte = 1];
}
```



### 和LRS的关系

看到 issue  https://github.com/envoyproxy/envoy/issues/6614 中的问答：

CodingSinger:

> 大家好，我有一个问题，orca_load_report和原始LRS有什么区别？ 我的理解是orca_load_report是将负载信息传递给envoy的后端服务器，而LRS是在envoy和管理服务器之间传递信息的？

SecurityInsanity:

> 目前，ORCA实际上将在实施后集成到LRS中。 它将提供更丰富的信息。(备注：这里没理解，到底是谁集成到谁？TODO：去详细翻译下LRS的定义)
>
> 目前，LRS仅提供有关请求数量，路由到谁以及何时发送的负载信息。 ORCA通过允许服务报告请求成本多少来补充该信息。 例如，某服务可以说“处理此请求，我占用了20％的CPU”。
>
> 服务可以通过两种方式将此信息报告给Envoy：
>
> - 通过响应中的header。
> - 通过单独的带外报告（band reporting）机制。
> 
> 我们的目标是首先读取响应header。 诚然，我已经做了很多事情，所以情况有所下降，但是我希望在未来几周内有所改善。

CodingSinger:

> 感谢您的回复。 但我认为现在似乎可以分为 LoadReportingService 和 OrcaLoadReport。 根据您的答复，ORCA和LRS在后端服务器和Envoy之间是否都在起作用？
> 但是我在LoadReportingService的评论中发现
>
> ```
> // Independently, Envoy will initiate a StreamLoadStats bidi stream with a
>    // management serve
> ```

SecurityInsanity:

> ORCA指标将被添加到 （而不是替换） LoadReportingService 统计信息中，因为我们认为它在这里也很有用，但是实际的 ORCA 统计信息是在 Envoy正在发送请求的事物和  Envoy 之间发送的。