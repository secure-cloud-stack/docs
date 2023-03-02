---
title: Netic on Premise
weight: 10
---

Getting access to Netic managed and operated Kubernetes cluster on-prem requires a few steps.

## Before you begin

This guide expectes the following prerequisites:

  - A namespace has been created associated with a git repository for gitops based reconciliation
  - Access to a user authorized for the namespace/cluster
  - `kubectl` has [been installed](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
  - The kubelogin plugin [has been installed](https://github.com/int128/kubelogin)

## Access to Cluster

Access to a Kubernetes cluster requires a kubeconfig. Authentication and authorization is based on
OIDC and it is possible to download a kubeconfig file from your observability
dashboard at `https://<provider_name>.dashboard.netic.dk`. The downloaded
configuration depends on the kubelogin plugin to be installed. The plugin is
capable of requesting and caching an OAuth 2.0 access token.

When you sign into Grafana the first page you are met with shows you the
kubeconfig file for the clusters and namespaces you have access to.

It is possible to check access using `kubectl`

```console
kubectl auth can-i --list -n <namespace>
```

### Create kubeconfig manually

If you prefer, you can create the kubeconfig file manually.

Replacing the `<>`-tokens with their corresponding values, create the following
`kubeconfig.yaml` file:

```yaml
apiVersion: v1
kind: Config
preferences: {}
clusters:
  - name: default
    cluster:
      certificate-authority: <api-server>.crt
      server: https://<api-server:port>
users:
  - name: keycloak
    user:
      exec:
        apiVersion: client.authentication.k8s.io/v1beta1
        command: kubectl
        args:
          - oidc-login
          - get-token
          # This allows for authentication on, e.g., bastion host. Disabled on
          # local workstations.
          # - --grant-type=authcode-keyboard
          - --oidc-use-pkce
          - --oidc-issuer-url=https://keycloak.netic.dk/auth/realms/mcs
          - --oidc-client-id=<cluster_name>.<provider>.<cluster_type>.k8s.netic.dk
contexts:
  - context:
      cluster: default
      user: keycloak
    name: default
current-context: default
```

Then, get the certificate from the api server.

Again, replace `<>`-tokens with the proper values.

```console
true | openssl s_client -connect <api-server:port> -showcerts 2>/dev/null \
  | sed --quiet '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' \
  > <api-server>.crt
```

Using the configuration you can start using `kubectl`:

```console
kubectl --kubeconfig <api-server>.yaml get nodes
```

## What's next

* [Getting Started](/docs/user/getting-started/)
