---
title: Image Automation
weight: 45
---

Flux is able to scan image-registries for new versions of images, such that upgrades automatically can be committed
directly to your Git repository. An `ImageRepository` is used to scan the registry for updates, an `ImagePolicy` is
used to sorting the tags for the latest version, and an `ImageUpdateAutomation` commits it to Git:

```yaml
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageRepository
metadata:
  name: pb-k8s-app
spec:
  image: registry.netic.dk/dockerhub/nginxinc/nginx-unprivileged
  interval: 1m0s
  secretRef:
    name: registry-secret
---
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImagePolicy
metadata:
  name: pb-k8s-app
spec:
  imageRepositoryRef:
    name: pb-k8s-app
  policy:
    semver:
      range: 1.x
---
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageUpdateAutomation
metadata:
  name: pb-k8s-app
spec:
  interval: 1m0s
  sourceRef:
    kind: GitRepository
    name: sync
  git:
    checkout:
      ref:
        branch: main
    commit:
      author:
        email: fluxcdbot@users.noreply.github.com
        name: fluxcdbot
      messageTemplate: '{{range .Updated.Images}}{{println .}}{{end}}'
    push:
      branch: main
```

In order for Flux to know where to make the change to your manifests, a comment is required in the deployment:

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: verify-deployment
spec:
  replicas: 1
  selector:
    ...
  template:
    metadata:
      labels:
        ...
    spec:
      containers:
      - image: registry.netic.dk/dockerhub/nginxinc/nginx-unprivileged:1.20 # {"$imagepolicy": "pb-k8s-app:pb-k8s-app"}
        name: verify-app
...
```

See [here](https://fluxcd.io/docs/components/image/) for documentation.
