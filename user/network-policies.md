---
title: Network Policies
weight: 25
---

The network policies restricts communication within the cluster to mitigate effects should
a pod get compromised. A number of network policies will be deployed into a namespace by default.

## Default policies

A default policy is in place denying all communication.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

Besides this normally a default egress policy would also be applied.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-egress
spec:
  egress:
  - ports:
    - port: 53
      protocol: TCP
    - port: 53
      protocol: UDP
    - port: 443
      protocol: TCP
    - port: 4317
      protocol: TCP
  podSelector: {}
  policyTypes:
  - Egress
```

## Ingress policies

A few opt-in policies exists to be activated on a pod to pod basis. Allowing ingress
into a pod requires specifying the label `netic.dk/network-ingress: contour` which
activates the policy below.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: contour-ingress
spec:
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: netic-ingress-system
    ports:
    - port: http
      protocol: TCP
  podSelector:
    matchLabels:
      netic.dk/network-ingress: contour
  policyTypes:
  - Ingress
```

{{< note >}}
The port used for ingress in the pod must have the logic name `http` no matter what the numeric port assignment is.
{{< /note >}}

If metrics is exposed and observability is set up there is a label to allow Prometheus
scrape `netic.dk/allow-prometheus-scraping: "true"` activating the below policy.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: prometheus-scrape-ingress
spec:
  ingress:
  - from:
    - namespaceSelector: {}
      podSelector:
        matchLabels:
          app.kubernetes.io/name: prometheus
    ports:
    - port: metrics
      protocol: TCP
    - port: http
      protocol: TCP
  podSelector:
    matchLabels:
      netic.dk/allow-prometheus-scraping: "true"
  policyTypes:
  - Ingress
```

{{< note >}}
The port used to expose metrics in the pod must have the logic name `http` og `metrics` no matter what the numeric port assignment is.
{{< /note >}}

## Additional network policies

Components inside of a namespace may also require to communicate. Defining these is requrested as a serviced definition and will then
be applied by Netic.
