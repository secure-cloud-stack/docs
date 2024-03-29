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

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: vault-secret
spec:
  dataFrom:
    - extract:
        key: k8s/prod1/<namespace>/vault-secret
  refreshInterval: 60s
  secretStoreRef:
    kind: SecretStore
    name: vault
  target:
    name: "vault-secret"
```

Using `dataFrom`, all key-value pairs are synced onto the secret called "vault-secret". Assuming the secret
contain only one key the result should be as seen below.

```yaml
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

## Vault Policies

Vault is setup with an area reserved for each namespace following a structure like `k8s/<cluster>/<namespace>`. As described above
external-secrets will be able to read these secrets. By default users are only able to create or overwrite secrets in the Vault
to reduce the risk of secret data being leaked. This is the same principle as you cannot retrive your old password only set a new
one. Furthermore Vault is not authoritative of any secrets therefore it should be possible to re-create the secrets inside of Vault.

However, on rare occasions larger configuration structures may need to be stored inside of Vault and it can be tedious to maintain
such structures when not able to read back the original data. To support this use case it is possible to indicate classification of
the secret contents by using the folder structure defined below.

| Path | Example | Purpose  |
|------|---------|----------|
| `k8s/<cluster>/<namespace>/restricted` | `k8s/prod1/my-awesomne-app/restricted/my-password` | *Purpose*: This folder contains secrets that are semi-automatically maintained and can be listed, created, updated, and deleted, but cannot be read by humans.<br>*Example*: Examples of secrets that could be stored here include temporary access tokens, session keys, and other data that is generated by machines and should not be accessible by humans. |
| `k8s/<cluster>/<namespace>/automated` | `k8s/prod1/my-awesomne-app/automated/ssh-key` | *Purpose*: This folder contains secrets that are automatically maintained and can only be listed, but cannot be read, created, updated, or deleted by humans.<br>*Example*: Examples of secrets that could be stored here include machine-generated encryption keys, service account credentials, and other data that is automatically managed by machines and should not be accessible by humans.|
| `k8s/<cluster>/<namespace>/unrestricted` | `k8s/prod1/my-awesomne-app/unrestricted/my-config` | *Purpose*: This folder contains secrets that are manually maintained and can be listed, created, updated, deleted, and read by humans.<br>*Example*: Examples of secrets that could be stored here include passwords, API keys, and other sensitive data that humans need to access. |
| `k8s/<cluster>/<namespace>/<app>/restricted` | `k8s/prod1/my-awesomne-app/svc1/restricted/my-password` | *Purpose*: Same as with the general restricted folder but allows for a sub-division into application "spaces".|
| `k8s/<cluster>/<namespace>/<app>/automated` | `k8s/prod1/my-awesomne-app/svc1/automated/ssh-key` | *Purpose*: Same as with the general automated folder but allows for a sub-division into application "spaces".|
| `k8s/<cluster>/<namespace>/<app>/unrestricted` | `k8s/prod1/my-awesomne-app/svc1/unrestricted/my-config` | *Purpose*: Same as with the general unrestricted folder but allows for a sub-division into application "spaces".|

All secrets located in the path `k8s/<cluster>/<namespace>` will be considered "restricted" following the description under `restricted` sub-folder.
