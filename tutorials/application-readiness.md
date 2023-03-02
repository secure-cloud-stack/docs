---
title: Application Readiness
weight: 55
---

Netic recommends for running workloads inside of Kubernetes some which is enforced by the
policies of the secure cloud stack and some which is best-practice running Kubernetes. These
recommendations are always valid but especially so if Netic is to provide [application operations](../application-operation/).

## Security

The containers must be able to run under the following security constraints also enforced by the pod and container
security context (see also [Security Context](../../user/security-context/)).

- Running without Linux capabilities
- Running as unprivileged
- Impossible to do privilege escalation

## Stability

The following concerns the ability to run the an application stable on Kubernetes even when the cluster is undergoing
maintenance.

- Number of replica must be >1
- Readiness and liveness probes should be properly set
- Resource requests and limits must be set according to expected load
- Pod disruption budget should be present and allow for maintenance to be carried out (min available < replicas)
- If applicable persistent volumes should be annotated for backup

## Documentation

The following concerns recommended (and required) documentation.

- Define requirements for backup; retention and restore
- Well-defined restore procedure including possibly acceptable dataloss
- Alerting rules and associated standard operating procedures must be defined

## Resilience and Robustness

The following concerns the resilience, robustness and compliance.

- Cross-Origin Resource Sharing (CORS) headers are _not_ automatically set - remember if applicable
- Observe correct use of http protocol with respect to idempodency etc. to allow for retries and other outage mitigation
- Utilize fault injection to make sure clients are resilient to unexpected conditions
- Beware to avoid sensitive log information (GDPR and otherwise)

## Testing Application Operational Readiness

Prior to engaging in application operations Netic offers a workshop to assess the operational readiness of the
application based on the outlined points.
