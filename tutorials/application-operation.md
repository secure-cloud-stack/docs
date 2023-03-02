---
title: Application Operations
weight: 50
---

Netic will do operations and management of Kubernetes as well as cluster wide components referred to
as technical operations and management. However, monitoring and reacting to events from the deployed
applications are not covered by this - this is referred to as application operations.

## Getting Started

Application operations is basically setup following the below steps:

1. Define incident scenarios which requires human interaction and write the standard operating procedures to be followed
2. Identify and ensure metrics are available to detect the incident scenarios
3. Develop alerting rules based on the metrics
4. Handover alerting rules and standard operating procedures to Netic for verification and activation

Often new scenarios are discovered along the way extending the set of alerting rules over time.

## Incident Scenarios

The failure scenarios may vary a lot from application to application. Strart out by trying to identify the most critical
points in the application and how to detect failure in these. It is important to write a standard operating procedure
to go together with every failure scenario describing the steps to be taken - even if this is to wake up the developer
on duty.

## Metrics

Metrics can be either specific application metrics such as rate of caught exceptions, http response error rates or other
similar but metrics can also be core Kubernetes metrics such as memory or cpu consumption. The monitoring must be
based on metrics rather than log patterns since metrics is much more robust to change. If the application provides
application specific metrics the application must expose these see [Observability](../observability/).

## Alerting Rules

[Alerting rules](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/) are written in
[PromQL](https://prometheus.io/docs/prometheus/latest/querying/basics/) as the rules will be evaluated by the Prometheus
rule engine. The rule expression can be testet against the real metrics using the Grafana Explore mode. Note that it
is also important to consider the robustness of a rule with respect to pod restarts, flapping etc.

## Handover

Once standard operating procedure and alerting rule expression is in place it can be handed over to Netic. Both will be
validated and reviewed such that the 24x7 operation center is able to follow the operational procedure and such that the
concrete alerting rule is capable of triggering a correct alert. This may cause a few iterations the first times.
