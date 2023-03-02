---
title: Getting Started
linkTitle: "Getting Started"
weight: 9
---

This section introduces some of the core concepts to utilizing Kubernetes with secure cloud stack on top.

## Before you begin

This guide expectes the following prerequisites:

  - Access to a user authorized for the namespace - see [Getting Access](/docs/user/getting-access)
  - Familiar with core concepts of gitops

## Verifying Access

Deployment is based on git and gitops - specifically [Flux](https://fluxcd.io/). A namespace must already
have been setup. It is possible to find the specific reconciliation setup for a namespace using `kubectl`.

Getting the `gitrepo` resource will display the repository associated with the namespace as well as 
the status for pulling in changes.

```console
kubectl get -n <namespace> gitrepo
```

Getting the `Kustomization` reousource will display status of applying resources in the cluster.
The specific path within the git repo used for reconciliation can also be found in the
`Kustomization` resource.

```console
kubectl get -n <namespace> kustomization
```

You are now ready to deploy by pushing commits to the git repository.

## What's next

* [Security Context](/docs/user/security-context/)
* [Ingress](/docs/user/ingress/)
