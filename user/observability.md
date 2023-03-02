---
title: Observability
weight: 30
---

A lot of observability information are collected at the cluster level. All cluster level observability is accessible
through the relevant dashboards. Cluster level data includes data such as pod memory and cpu consumption etc. However,
it is possible to subscribe to application level observablity consisting of the collection of metrics, traces and logs.

It is recommended that application metrics and traces created using the libraries from the [OpenTelemetry](https://opentelemetry.io/)
project. This ensures a uniform application instrumentation even acroess programming languages.

## Before you begin

The application is capable of providing telemetry data:

* The application should be logging to stdout
* The application should expose Prometheus style metrics ([OpenMetrics](https://openmetrics.io/)) using [OpenTelemetry](https://opentelemetry.io/) is recommended
* If collection of traces is desired the application should be able to push traces in Jaeger or [OpenTelemetry](https://opentelemetry.io/) format

See also [Application Observability](/docs/tutorials/observability/).

## Log collection

By default all output from stdout will be captured and indexed.

{{< note >}}
Additional fees may apply with large volumes of log.
{{< /note>}}

## Metric collection

Enabling metrics collection is done by deploying a `ServiceMonitor` resource with instructions on
how Prometheus should scrape metrics off the application. Typically as below.

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    app.kubernetes.io/instance: hello-service
    app.kubernetes.io/managed-by: Kustomize
    app.kubernetes.io/name: hello-service
    app.kubernetes.io/version: latest
    netic.dk/monitoring: <scope>
  name: hello-service
spec:
  endpoints:
  - interval: 15s
    port: http
  selector:
    matchLabels:
      app.kubernetes.io/instance: hello-service
      app.kubernetes.io/name: hello-service
```

## Trace collection

An OpenTelemetry Collector sidecar can be injected for trace collection by annotating
the pod with `sidecar.opentelemetry.io/inject: "true"`. This will allow the application
to push to localhost either as OpenTelemetry or Jaeger format.
