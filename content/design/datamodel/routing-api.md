---
date: 2020-02-01T11:00:00+08:00
title: Routing API设计
menu:
  main:
    parent: "design-datamodel"
weight: 521
description : "UDPA Routing API设计"
---



备注：Routing API 的设计还在进行中:

- 内容来自 issue 讨论 [[WiP] L7 routing straw man](https://github.com/cncf/udpa/pull/4) 
- 和提交的文件 [udpa/config/v1alpha/routing.proto](https://github.com/cncf/udpa/pull/4/files#diff-40a6d8a368e77b5a9046c367cb47182b)

## API 定义

ConfigProvider：

```protobuf
message ConfigProvider {}
```

RegexDescriptor：

```protobuf
// 正则表达式匹配器设计用于与不受信任的输入一起使用时的安全性
message RegexDescriptor {
  // Google's `RE2 <https://github.com/google/re2>`_ 正则匹配引擎.
  // 正则表达式字符串必须遵守已记录的`syntax <https://github.com/google/re2/wiki/Syntax>`_。 
  // 该引擎旨在在线性时间内完成执行并限制使用的内存量。
  message GoogleRE2 {
    // 此字段控制RE2的“program size/程序大小”
    // “program size”是对已编译正则表达式要评估的复杂程度的粗略估计。 
    // program size大于配置值的正则表达式将无法编译。 
    // 在这种情况下，可以增加配置的最大program size，或者可以简化正则表达式。 
    // 如果未指定，则默认值为100。
    google.protobuf.UInt32Value max_program_size = 1;
  }

  oneof engine_type {
    option (validate.required) = true;

    // Google's RE2 正则表达式引擎
    GoogleRE2 google_re2 = 1 [(validate.rules).message.required = true];
  }
}
```

SimpleStringMatchDescriptor：

```protobuf
message SimpleStringMatchDescriptor {
  message ExactDescriptor {}
  message PrefixDescriptor {}
  message SuffixDescriptor {}

  oneof match_pattern {
    option (validate.required) = true;

    // 输入字符串必须与此处指定的字符串完全匹配。
    //
    // Examples:
    //
    // * *abc* only matches the value *abc*.
    ExactDescriptor exact = 1;

    // 输入字符串必须具有此处指定的前缀。
	 // 注意：不允许使用空前缀，请改用正则表达式。
    //
    // Examples:
    //
    // * *abc* matches the value *abc.xyz*
    PrefixDescriptor prefix = 2;

    // 输入字符串必须具有此处指定的后缀。
	 // 注意：不允许使用空后缀，请改用正则表达式。
    //
    // Examples:
    //
    // * *abc* matches the value *xyz.abc*
    SuffixDescriptor suffix = 3;
  }
}
```

StringMatchDescriptor:

```protobuf
// 指定匹配字符串的方式
message StringMatchDescriptor {
  oneof match_pattern {
    option (validate.required) = true;

    SimpleStringMatchDescriptor simple_match = 1;

    // 输入字符串必须与此处指定的正则表达式匹配。
    RegexDescriptor regex = 2;
  }
}
```

Int64Range:

```protobuf
// 使用半开间隔的语义 [start, end) 指定int64范围的开始和结束。
message Int64Range {
  // 范围的开始 (包括)
  int64 start = 1;

  // 范围的结束 (不包括)
  int64 end = 2;
}
```

HeaderValue 和 HeaderValueOption:

```protobuf
// Header name/value pair.
message HeaderValue {
  // Header name.
  string key = 1 [(validate.rules).string = {min_bytes: 1, max_bytes: 16384}];

  // Header value.
  //
  // The same :ref:`format specifier <config_access_log_format>` as used for
  // :ref:`HTTP access logging <config_access_log>` applies here, however
  // unknown header values are replaced with the empty string instead of `-`.
  string value = 2 [(validate.rules).string.max_bytes = 16384];
}

// Header name/value 对加上附加的控制行为选项。
message HeaderValueOption {
  // Header name/value pair that this option applies to.
  HeaderValue header = 1 [(validate.rules).message.required = true];

  // 是否应该附加值？ 如果为true（默认值），则该值将附加到现有值之后。
  google.protobuf.BoolValue append = 2;
}
```

Transform：

```protobuf
// Action是请求转发时的请求转换
message Transform {
  // 指定应添加到连接管理器编码的每个响应中的HTTP header列表。 
  // 在此级别指定的 header 应用于来自任何附带的：`envoy_api_msg_route.VirtualHost`或 `envoy_api_msg_route.RouteAction`的 header。 
  // 有关更多信息，包括 header 值语法的详细信息，请参阅自定义请求 header <config_http_conn_man_headers_custom_request_headers>上的文档。
  repeated HeaderValueOption response_headers_to_add = 4
      [(validate.rules).repeated .max_items = 1000];

  // 指定应从连接管理器编码的每个响应中删除的HTTP header列表。
  repeated string response_headers_to_remove = 5;

  // 指定应添加到HTTP连接管理器路由的每个请求中的HTTP heaer列表 
  // Headers specified at this level are
  // applied after headers from any enclosed :ref:`envoy_api_msg_route.VirtualHost` or
  // :ref:`envoy_api_msg_route.RouteAction`. For more information, including details on
  // header value syntax, see the documentation on :ref:`custom request headers
  // <config_http_conn_man_headers_custom_request_headers>`.
  repeated HeaderValueOption request_headers_to_add = 6
      [(validate.rules).repeated .max_items = 1000];

  // 指定应从HTTP连接管理器路由的每个请求中删除的HTTP标头列表。
  repeated string request_headers_to_remove = 8;
}
```

Action：

```protobuf
// Action 可以是 重定向/redirect, 路由/route (转发/forward) 或者其他 match-action 树级别.
message Action {
  // 重定向/Redirect 是 Envoy v2 xDS envoy.api.v2.route.RedirectAction 消息的复制+粘贴
  // 尽管为清晰起见，可以对其进行重大重写，但其思想是此处的行为似乎非常的和实现无关。 
  // TODO（htuch）：那里的所有 DPLB 实现都可以实现这个 Redirect 吗？
  message Redirect {
    // 进行 scheme 重定向时，以下规则适用：
    //  1. 如果来源 URI scheme 是 `http` 而端口被显式设置为 80，则重定向之后端口将被移除
    //  2. 如果来源 URI scheme 是 `https` 而端口被显式设置为 443，则重定向之后端口将被移除 
    oneof scheme_rewrite_specifier {
      // URL的 scheme 部分将被替换为"https"。
      bool https_redirect = 4;
      // URL的 schema 部分将被替换为此值。
      string scheme_redirect = 7;
    }

    // URL的 host 部分将与被替换为此值。
    string host_redirect = 1;
    // URL的 port 部分将与被替换为此值。
    uint32 port_redirect = 8;

    oneof path_rewrite_specifier {
      // URL的 path 部分将与被替换为此值。
      string path_redirect = 2;

      // 指示在重定向期间，应将匹配的前缀（或路径）替换为此值。 
      // 此选项允许根据请求动态创建重定向URL。
      // .. attention::
      //
      //   Pay attention to the use of trailing slashes as mentioned in
      //   :ref:`RouteAction's prefix_rewrite <envoy_api_field_route.RouteAction.prefix_rewrite>`.
      string prefix_rewrite = 5;
    }

    enum RedirectResponseCode {
      // Moved Permanently HTTP Status Code - 301.
      MOVED_PERMANENTLY = 0;

      // Found HTTP Status Code - 302.
      FOUND = 1;

      // See Other HTTP Status Code - 303.
      SEE_OTHER = 2;

      // Temporary Redirect HTTP Status Code - 307.
      TEMPORARY_REDIRECT = 3;

      // Permanent Redirect HTTP Status Code - 308.
      PERMANENT_REDIRECT = 4;
    }

    // 在重定向响应中使用的HTTP状态代码。 
    // 默认响应代码为 MOVED_PERMANENTLY（301）。
    RedirectResponseCode response_code = 3 [(validate.rules).enum.defined_only = true];

    // 指示在重定向过程中，URL的查询部分将被删除。 
    // 默认值为false。
    bool strip_query = 6;
  }

  // 从 Envoy 的v2 xDS envoy.api.v2.route.RouteAction 大幅精简而成的。 
  // 关键是我们可能要做的最基本的通用操作就是指向给定的服务。 
  // 除此之外，还需要逐个特征进行分析。
  message Route {
    // service name
    string service_name = 1;

    // 请求转换描述
    // 即转发时我们要对请求做什么？
    Transform transform = 2;

    // TODO(htuch): 已知的DPLB还共享RouteAction的其他哪些部分？

    // DPLB特定行为，例如 流量镜像，速率限制，请求对冲（request hedging），亲和性（affinity），超时，重试策略，CORS，跟踪都可以在此处进行编码。
    google.protobuf.Any dplb_extensions = 3;
  }

  oneof action_specifier {
    Redirect redirect = 1;
    Route route = 2;
    // TODO(htuch): add DirectResponseAction
    Matcher match = 3;
  }
}
```

HeaderValueExtractor：

```protobuf
// header匹配器需要能够触及 header 内
// 例如 :path或Cookie，并提取感兴趣的值，例如查询参数。 
// StringExtractor提供此功能。
message HeaderValueExtractor {
  oneof extractor_specifier {
    option (validate.required) = true;

    // header值的幂等(Idempotent)返回。
    google.protobuf.Empty exact = 1;

    // path 部分, 例如 ？之前的内容
    google.protobuf.Empty path = 2;

    // 单个查询参数
    string query_parameter = 3;

    // TODO(htuch): should this be further generalized to support things like
    // extracting values embedded in header values with separators such as , or
    // :?
  }
}
```

各种 Matcher：



Matcher:

```protobuf
message Matcher {
  oneof matcher_specifier {
    option (validate.required) = true;

    LinearMatcher linear_matcher = 1;

    HierarchicalMatcher hierarchical_matcher = 2;
  }
}
```

LinearMatcher:

```protobuf
message LinearMatcher {
  message MatchAction {
    message HeaderMatcher {
      // 指定请求中 header 的名称
      string name = 1 [(validate.rules).string.min_bytes = 1];

      // 如果指定，匹配结果将在检查前反转。 
      // 默认为false。
      //
      // Examples:
      //
      // * The regex *\d{3}* does not match the value *1234*, so it will match
      //   when inverted.
      // * The range [-10,0) will match the value -1, so it will not match when
      //   inverted.
      bool invert_match = 2;

      // 如何提取标头值以进行匹配
      HeaderValueExtractor value_extractor = 3;

      // 字符串值的匹配器。
      message StringMatcher {
        StringMatchDescriptor match_descriptor = 1;
        string value = 2;
      }

      // 整数范围的匹配器。
      message IntegerRangeMatcher {
        Int64Range range = 1;
      }

      oneof header_matcher_specifier {
        option (validate.required) = true;

        StringMatcher string_matcher = 4;

        IntegerRangeMatcher integer_range_matcher = 5;
      }
    }

    // 提供匹配条件的 header 匹配器列表。 
    // 所有HeaderMatcher必须与请求匹配； 执行 HeaderMatcher结果的逻辑与（And）。
    repeated HeaderMatcher header_matchers = 1;

    // 当所有header_matchers匹配时要执行的操作。
    Action action = 2;
  }
  repeated MatchAction match_actions = 1;
}
```

HierarchicalMatcher：

```protobuf
// 有序匹配器是分层的树结构，在每个节点上指定单个匹配条件。
message HierarchicalMatcher {
  message HeaderMatcher {
    // 指定请求中 header 的名称。
    string name = 1 [(validate.rules).string.min_bytes = 1];

    // 如何提取标头值以进行匹配
    HeaderValueExtractor value_extractor = 2;

    // String 值 的 Match-action 表
    message StringMatcher {
      // 字符串值将如何匹配？ 正则表达式匹配器不允许
      //
      // 优化实现的预期性能为：
      // - exact: O(m)
      // - prefix: O(m)  [longest prefix wins]
      // - suffix: O(m)  [longest suffix wins]
      // 其中，n是匹配表中的条目数，m是要查找的值的长度。（没有看到n啊？）
      SimpleStringMatchDescriptor match_descriptor = 1;

      message InlineTable {
        map<string, Action> inline_table = 1;
      }

      oneof table_specifier {
        // Inline match-action table.
        InlineTable inline_table = 2;

        // 支持类似VHDS的行为，按需获取 header 路由表。 
        // 资源类型是Action，资源名称是要使用的字符串值匹配。 
        // 例如。 可以动态传递一个名为 "*-foo.com" 的 Action。 
        // 不能与正则表达式匹配描述符一起使用。
        ConfigProvider config_provider = 3;
      }
    }

    // Match-action table for integer ranges.
    message IntegerRangeMatcher {
      message IntegerRangeMatchAction {
        Int64Range range = 1;
        Action action = 2;
      }
      // Ranges must not overlap.
      repeated IntegerRangeMatchAction match_actions = 1;
    }

    oneof header_match_specifier {
      // String value match-action.
      StringMatcher string_matcher = 3;

      // Integer value match-action.
      IntegerRangeMatcher integer_range_matcher = 4;
    }
  }

  // 当前仅支持 header 匹配器。
  oneof matcher_specifier {
    option (validate.required) = true;
    HeaderMatcher header_matcher = 1;
  }

  // Fall-thru action，如果没有其他匹配发生
  Action default_action = 2;
}
```

TODO：需要和 xDS v2 / v3 的 API 对照看一下，看具体有哪些变化。


