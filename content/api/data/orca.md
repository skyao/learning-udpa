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


