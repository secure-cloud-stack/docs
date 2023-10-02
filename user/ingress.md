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

The most portable way to configure ingress is using the Kubernetes `Ingress` resource as below.

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

### TLS Termination

It is possible to issue certificates based on [Let's Encrypt](https://letsencrypt.org/) by annotating the
ingress resource. Certificates are also automatically renewed. Note the Let's Encrypt limits if doing
a lot of deployments.
The annotation: `cert-manager.io/cluster-issuer: letsencrypt` means that it will uses a cluster-issuer called letsencrypt,
which is configured to use the [ACME DNS Challenge](https://datatracker.ietf.org/doc/html/rfc8555#section-8.4) to issue the certificate.
This cluster-issuer requires that Netic manages DNS for the domain to be issued.
If it is not possible to have Netic manage DNS, then it is also possible to use [ACME HTTP Challenge](https://datatracker.ietf.org/doc/html/rfc8555#section-8.3),
this does require the cluster to be publicly available for letsencrypt to validate.


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

### Ingress DNS

When a ingress resource is created a DNS A record i created that points the host to the public IP of the cluster,
but only if the host in the ingress resouce is on the configured allow list.
For this feature to work, Netic must manage the DNS for the host/domain.

It is possible to have Netic manage domain/subdomains, contact Netic for more information. 