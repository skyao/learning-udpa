---
date: 2020-02-02T17:00:00+08:00
title: OpenRcaService
menu:
  main:
    parent: "api-service"
weight: 321
description : "OpenRcaService"
---

定义来自：https://github.com/cncf/udpa/blob/master/udpa/service/orca/v1/orca.proto

### OpenRcaService说明

未位于请求路径中的额外负载报告代理的带外（Out-of-band/OOB）负载报告服务。 

定期以足够的频率对报告进行采样，以提供与请求的关联。 OOB报告弥补了带内（in-band）报告的局限性，因为它揭示了无法提供稳定遥测流的后端（例如长期运行的流操作和零QPS的服务）的成本。 

这是一项服务器流服务，客户端需要终止当前的RPC并发起新的调用以更改后端报告频率。

### OpenRcaService 定义

```protobuf
service OpenRcaService {
  rpc StreamCoreMetrics(OrcaLoadReportRequest) returns (stream udpa.data.orca.v1.OrcaLoadReport);
}
```



### OrcaLoadReportRequest定义

```protobuf
message OrcaLoadReportRequest {
  // 生成Open RCA 核心度量标准响应的时间间隔。
  google.protobuf.Duration report_interval = 1;

  // 要收集的请求开销。如果为空，则将返回负载报告代理跟踪的所有已知请求开销。
  // 这为客户提供了一个机会来有选择地获得跟踪开销的子集。
  repeated string request_cost_names = 2;
}
```

### Out-of-band (OOB) reporting

See section `Out-of-band (OOB) reporting` of the design document in
https://github.com/envoyproxy/envoy/issues/6614