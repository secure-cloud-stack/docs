---
title: Ingress
weight: 24
---

Ingress is normally handled by [Contour](https://projectcontour.io/) so it is possible to
define ingress by either standard Kubernetes `Ingress` resources or Contour custom resource
definition `HTTPProxy`.

## Before you begin

Automation is set up for both TLS certificates and DNS entries. Before hand you need to agree on which
DNS domains the setup should be enabled for.

## Configuring Ingress

The most portable way to configure ingress is using the Kubernetes `Ingress` resource as below. This will
automatically create a DNS `A` records for the given host pointing to the public IP address of the cluster.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: verify-ingress
spec:
  tls:
    - secretName: pb-sample-netic-dev-tls
      hosts:
        - pb.sample.netic.dev
  rules:
    - host: pb.sample.netic.dev
      http:
        paths:
          - path: /verify
            pathType: Prefix
            backend:
              service:
                name: verify-service
                port:
                  name: http
```

{{< note >}}
For the ingress controller (Contour) to be able to pass requests to a pod the pod must be labelled with
`netic.dk/network-ingress: "contour"` as this activates the network policy allowing ingress to the port
named `http`.
{{< /note >}}

## TLS Handling

It is possible to issue certificates based on [Let's Encrypt](https://letsencrypt.org/) by annotating the
ingress resource. Certificates are also automatically renewed. Note the Let's Encrypt limits if doing
a lot of deployments.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: verify-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt
    kubernetes.io/tls-acme: "true"
spec:
  tls:
    - secretName: pb-sample-netic-dev-tls
      hosts:
        - pb.sample.netic.dev
  rules:
    - host: pb.sample.netic.dev
      http:
        paths:
          - path: /verify
            pathType: Prefix
            backend:
              service:
                name: verify-service
                port:
                  name: http
```
