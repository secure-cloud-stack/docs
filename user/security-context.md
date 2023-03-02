---
title: Security Context
weight: 20
---

By default a namespace is setup to adhere to the [Restricted Pod Security Standard](https://kubernetes.io/docs/concepts/security/pod-security-standards/#restricted).
Your deployment must be configured to adhere to this to be accepted for deployment otherwise the pods wont be created.

## Before you begin

The manifests for deploying the workload inside of the cluster is available.

## Adjusting Deployment

Having a deployment like so:

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: verify-deployment
  labels:
    app.kubernetes.io/name: verify-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: verify-app
  template:
    metadata:
      labels:
        app.kubernetes.io/name: verify-app
    spec:
      containers:
      - image: nginxinc/nginx-unprivileged:1.20
        name: verify-app
        ports:
        - containerPort: 8080
          name: http

```

You need to add a security context to the pod:

```yaml
      securityContext:
          runAsUser: 1000
          runAsGroup: 3000
          fsGroup: 2000
```

And to the container:

```yaml
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
              - all
```

Thus the deployment becomes:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: verify-deployment
  labels:
    app.kubernetes.io/name: verify-app
    app.kubernetes.io/instance: verify-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: verify-app
      app.kubernetes.io/instance: verify-app
  template:
    metadata:
      labels:
        app.kubernetes.io/name: verify-app
        app.kubernetes.io/instance: verify-app
    spec:
      securityContext:
          runAsUser: 1000
          runAsGroup: 3000
          fsGroup: 2000
      containers:
      - image: nginxinc/nginx-unprivileged:1.20
        name: verify-app
        ports:
        - containerPort: 8080
          name: http
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
              - all
```

## Patching Helm Output

If you are using standard Helm charts you may find that not everyone is running in a non-privileged
way. The cluster is using GitOps toolkit to reconcile the cluster and thus patching charts needs to
be done prior to the actual deployment, which means that the deployed charts needs to be secured
before deployment. There are probably many ways to do this. A simple way, which allows you to work
with the standard charts from the standard repos are to use the postrendering principle, where the
Helm chart is rendered prior to deployment using Kustomization.

Through the `HelmRelease` resource it is possible to add a path run as a post renderer. E.g.:

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: redis
spec:
  chart:
    spec:
      chart: redis
      version: 1.2.3
      sourceRef:
        kind: HelmRepository
        name: bitnami
        namespace: netic-gitops-system
  postRenderers:
    - kustomize:
        patchesStrategicMerge:
          - apiVersion: apps/v1
            kind: StatefulSet
            metadata:
              name: redis-master
              namespace: pb-k8s-app
            spec:
              selector:
                matchLabels:
                  app.kubernetes.io/name: redis
                  app.kubernetes.io/instance: redis
                  app.kubernetes.io/component: master
              template:
                metadata:
                  labels:
                    netic.dk/network-rules-egress: redis
                    netic.dk/network-rules-ingress: redis
                    netic.dk/network-component: redis
                spec:
                  securityContext:
                      runAsUser: 1001
                      runAsGroup: 3000
                      fsGroup: 2000

                  containers:
                    - name: redis
                      securityContext:
                        runAsUser: 1001
                        allowPrivilegeEscalation: false
                        capabilities:
                          drop:
                            - all
          - apiVersion: apps/v1
            kind: StatefulSet
            metadata:
              name: redis-replicas
              namespace: pb-k8s-app
            spec:
              selector:
                matchLabels:
                  app.kubernetes.io/name: redis
                  app.kubernetes.io/instance: redis
                  app.kubernetes.io/component: replica
              template:
                metadata:
                  labels:
                    netic.dk/network-rules-egress: redis
                    netic.dk/network-rules-ingress: redis
                    netic.dk/network-component: redis
                spec:
                  securityContext:
                      runAsUser: 1001
                      runAsGroup: 3000
                      fsGroup: 2000
                  containers:
                    - name: redis
                      securityContext:
                        runAsUser: 1001
                        allowPrivilegeEscalation: false
                        capabilities:
                          drop:
                            - all
```


## What's next

* [Ingress](/docs/user/ingress/)
