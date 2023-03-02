---
title: Azure Kubernetes Service (AKS)
weight: 20
---

Getting access to Netic managed and operated Kubernetes cluster in Azure requires a few steps.

## Before you begin

This guide expectes the following prerequisites:

  - A namespace has been created associated with a git repository for gitops based reconciliation
  - Access to a user authorized for the namespace/cluster
  - `kubectl` has [been installed](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
  - The azure-kubelogin plugin (from k8s 1.24 onwards) has [been installed](https://github.com/Azure/kubelogin)

## Access to Cluster

Access to a Kubernetes cluster requires a kubeconfig. Authentication and
authorization is based on OIDC. The configuration depends on the Azure kubelogin
plugin to be installed. The plugin is capable of requesting and caching an OAuth
2.0 access token.

For Azure you can get the kubeconfig file for the clusters you have access to
using the following commands:

```console
az login

az account set --subscription <subscription id>

az aks get-credentials --resource-group <resource group name> --name <aks service name> -f <output file name>

```

It is possible to check access using `kubectl`

```console
kubectl --kubeconfig <output file name> auth can-i --list -n <namespace>
```

## What's next

* [Getting Started](/docs/user/getting-started/)
