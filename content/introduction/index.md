---
date: 2020-02-01T11:00:00+08:00
title: UDPA介绍
weight: 100
description : "UDPA介绍"
---



## 介绍

UDPA 是 "Universal Data Plane API / 通用数据平面API"的缩写。

而 Universal Data Plane API Working Group (UDPA-WG) 是CNCF下的一个工作组。

### UDPA的目标

UDPA的目标，来自 https://github.com/cncf/udpa 的描述：

> The objective of the Universal Data Plane API Working Group (UDPA-WG) is to bring together parties across the industry interested in a common control and configuration API for data plane proxies and load balancers.
>
> 通用数据平面API工作组（UDPA-WG）的目标是召集对数据平面代理和负载均衡器的通用控制和配置API感兴趣的业界人士。 

### UDPA的愿景

> The vision of the Universal data Plane API (UDPA) is articulated at https://blog.envoyproxy.io/the-universal-data-plane-api-d15cec7a. We will pursue a set of APIs that provide the de facto standard for L4/L7 data plane configuration, similar to the role played by OpenFlow at L2/L3/L4 in SDN.
>
> 通用数据平面API（UDPA）的愿景在 https://blog.envoyproxy.io/the-universal-data-plane-api-d15cec7a 中阐明。 我们将寻求一组API，它们为L4/L7数据平面配置提供事实上的标准，类似于SDN中L2/L3/L4的OpenFlow所扮演的角色。
>
> The APIs will be defined canonically in proto3 and incrementally evolve from [existing Envoy xDS APIs](https://github.com/envoyproxy/data-plane-api) via a well defined [stable API versioning policy](https://docs.google.com/document/d/1xeVvJ6KjFBkNjVspPbY_PwEDHC7XPi0J5p1SqUXcCl8/edit#). APIs will cover service discovery, load balancing assignments, routing discovery, listener configuration, secret discovery, load reporting, health check delegation, etc.
>
> 这些API将在proto3中规范定义，并通过定义良好的 稳定API版本控制策略，从现有的Envoy xDS API逐步演进。 API将涵盖服务发现，负载均衡分配，路由发现，监听器配置，安全发现，负载报告，运行状况检查委托等。
>
> We will evolve and mold the APIs to support client-side lookaside load balancing (e.g. gRPC-LB), data plane proxies beyond Envoy, hardware LB, mobile clients and beyond. We will strive to be vendor and implementation agnostic to the degree possible while not regressing on support for projects that have committed to UDPA in production (Envoy & gRPC-LB so far).
>
> 我们将对API进行改进和成型，以支持客户端后备负载均衡（例如gRPC-LB），Envoy之外的数据平面代理，硬件LB，移动客户端以及其他范围。 我们将努力成为尽可能与供应商和实现无关的公司，同时坚持支持已投入生产的UDPA的项目（到目前为止，Envoy和gRPC-LB）。

### UDPA的成员

> Initial members will include representatives from the Envoy and gRPC projects. This will include Google and Lyft based maintainers, as well as members from Microsoft and Amazon. We are soliciting the wider data plane proxy community for additional interest in initial membership, since we feel that a truly universal API should reflect a diverse set of projects, organizations and individuals.
>
> 初始成员包括Envoy和gRPC项目的代表。 这将包括基于Google和Lyft的维护者，以及来自Microsoft和Amazon的成员。 我们正在征集更广泛的数据平面代理社区，以增加对初始成员资格的兴趣，因为我们认为真正的通用API应该反映各种项目，组织和个人。
>
> We would like to keep the working group small and tightly focussed on efficiently balancing incrementally improving the API while pursuing longer term strategic evolution. We will meet once every two weeks on Zoom and communicate via the [udpa-wg@lists.cncf.io](https://lists.cncf.io/g/udpa-wg/) mailing list.
>
> 我们希望使工作组保持较小规模，并专注于有效平衡增量改进的API，同时追求长期的战略发展。 我们每两周在Zoom上开会一次，并通过 udpa-wg@lists.cncf.io 邮件列表进行交流。