---
title: Stateful Deployments
weight: 40
---

If you need stateful deployment, you can use a stateful set:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: verify-deployment
  labels:
    app.kubernetes.io/name: verify-app
    app.kubernetes.io/instance: verify-app
spec:
  serviceName: verify-service
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: verify-app
  template:
    metadata:
      labels:
        app.kubernetes.io/name: verify-app
        app.kubernetes.io/instance: verify-app
        netic.dk/network-ingress: "contour"
      annotations:
        backup.velero.io/backup-volumes: verify-volume
    spec:
      securityContext:
        runAsUser: 1000
        runAsGroup: 3000
        fsGroup: 2000
      containers:
        - name: verify-app
          image: registry.netic.dk/dockerhub/nginxinc/nginx-unprivileged:1.20
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 8080
          volumeMounts:
            - name: verify-volume
              mountPath: /etc/nginx
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop:
                - all
  volumeClaimTemplates:
    - metadata:
        name: verify-volume
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi
```

This creates one pod and a PVC with 1Gi of storage which is mounted automatically, at the specified mount path.

### Annotations

Backup of volumes are not enabled by default, however, this can be enabled by adding the following annotation to your pods that uses the PVCs you want to have backed-up (the above example utilises this):

```yaml
      annotations:
        backup.velero.io/backup-volumes: verify-volume
```

The backup is done to a local s3 storage and is maintained for 5 days. If you want longer retention this needs to be specified.
