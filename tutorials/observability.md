---
title: Application Observability
---

The secure cloud stack comes with a readymade observability setup to collect logs, metrics and traces and
gain insights into application health and performance. While the platform as such is polyglot and works
independt of specific programming languges, there are some recommendations with respect to development.

## Before you begin

This guide assumes some familarity with the concepts of cloud native observability, i.e., logs, metrics,
and traces as well as the chosen programming language.

## Logs

Basically there is no requirements on logging. All that is output to stdout/stderr will be forwarded to the
log indexing solution. However, it is recommended to use a logging framework to make sure the log output is
consistent and allowing for more easy log parsing afterwards. Below are examples of common logging frameworks
for a few languages.

It is worth mentioning that [OpenTelemetry](https://opentelemetry.io/) is also working on standardizing logging
across languages however only alpha support currently exists for a few languages.

{{< note >}}
If the application is also providing traces it is recommended to add trace ids to the log output.
{{< /note>}}

### .NET

The .NET framework comes with logging interfaces built in and a number of 3rd party solution can be hooked
into to support controlling the log output. Examples are:

* [SeriLog](https://serilog.net/)
* [NLog](https://nlog-project.org/)

### Go

The standard Go libraries for logging is very seldom sufficient and a number of logging frameworks exists. Popular
ones are:

* [Logrus](https://github.com/sirupsen/logrus)
* [Zerolog](https://github.com/rs/zerolog)
* [Zap](https://pkg.go.dev/go.uber.org/zap)

### Java

Java also comes with built in logging support in the `java.util.logging` (jul) package though a number of 3rd frameworks
are also very popular. Interoperational bridges exists between these and also between these and the built-in Java support.

* [log4j](https://logging.apache.org/log4j/2.x/)
* [slf4j](https://www.slf4j.org/)

## Metrics and Traces

While metrics and traces are different concepts there are some overlap. Metrics are a quantitative measure aggregating data, i.e.,
a counter of requests or a histogram of latencies. Distributed traces are a qualitative measure recording the exact execution
path of a specific transaction through the system. However, often it is desired to record metrics in almost the same places as
a span is added to a trace. This makes a natural coupling between traces and metrics. Also support is coming for enriching the
aggregated metrics with trace ids representing examples, like a histogram bucket of a high latency may be reported along with
an trace id of a transaction with high latency.

While the platform does not put any constraints on trace or metrics frameworks by default it is recommended to use and follow
the [OpenTelemetry](https://opentelemetry.io/) recommendations. The [OpenTelemetry](https://opentelemetry.io/) project both support
libraries for multiple languages and also standadizes recommendations on naming, labels etc. This allows for more easy reuse
of dashboards, alerts, and more across applications. The instrumentation libraries implement the standard metrics.

### .NET

* [OpenTelemetry](https://opentelemetry.io/docs/instrumentation/net/)
* [prometheus-net](https://github.com/prometheus-net/prometheus-net)

### Go

* [OpenTelemetry](https://opentelemetry.io/docs/instrumentation/go/)
* Prometheus official [client_golang](https://github.com/prometheus/client_golang)

### Java

* [OpenTelemetry](https://opentelemetry.io/docs/instrumentation/java/)
* Prometheus official [client_java](https://github.com/prometheus/client_java)

## What's next

* Activate telemetry collection - see [Observability](/docs/user/observability/)
