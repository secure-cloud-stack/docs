---
title: Vault and Secrets
weight: 35
---

The secure cloud stack includes a secrets management service to store sensitive key/value pairs to be used
in the cluster. Secrets, such as credentials, usually have a lifecycle different from the lifecycle of the
source code. Therefore it makes sense to handle crendentials and the like through another channel.

## Before you begin

There is a requirement for some sensitive data to be provided to the workloads running inside of the cluster.

## Access Data

If you want to access sensitive data from the cluster, go to the correct namespace area in the vault and create
a new secret in key-value-format. Using `external-secrets`, you can synchronize this data into a secret resource
in the cluster. In the following example, the secret is called 'vault-secret', and contains the key 'pb-secret-key':

``` yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: vault-secret
spec:
  dataFrom:
    - key: k8s/prod1/<namespace>/vault-secret
  refreshInterval: 60s
  secretStoreRef:
    kind: SecretStore
    name: vault
  target:
    name: "vault-secret"
```

Using `dataFrom`, all key-value pairs are synced onto the secret called "vault-secret". Assuming the secret
contain only one key the result should be as seen below.

``` yaml  
apiVersion: v1
kind: Secret
metadata:
  name: vault-secret
type: Opaque
data:
  pb-secret-key: dmVyeS1zZWNyZXQ=
```

You can check the secret and the value inside your namespace with:

```bash
kubectl get secrets vault-secret -n <namespace> -o jsonpath='{.data.pb-secret-secret}' | base64 -D
```
and your should get the result: `very-secret`
