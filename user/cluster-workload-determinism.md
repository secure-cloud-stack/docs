---
title: Cluster Workload Determinism
weight: 45
---

## Before you begin

It is possible to specify which workloads need to have priority over other workloads e.g. in a situation where e.g. a central back-end service serves all the front-end services. This could mean that the back-end service may be more important the front-end services, and thus it would be necessary to tell that to kubernetes in order for that to be able to make the right decision when pre-empting Pods. Kubernetes has an Object Type called PriorityClasses for exactly that purpose. Kubernetes itself uses these PriorityClasses internally for ensuring its own ability to run run node and system workloads, and the Secure Cloud Platform uses that same mechanism for ensuring that Technical Operations etc. is running and we can deliver the promised services.

Applications deployed on the Secure Cloud Stack may have the same need for this as seen from the example above with the front-end and back-end service, and a number of PriorityClasses has been created for that purpose:

```
  secure-cloud-stack-tenant-namespace-application-critical
  secure-cloud-stack-tenant-namespace-application-less-critical
  secure-cloud-stack-tenant-namespace-application-lesser-critical
  secure-cloud-stack-tenant-namespace-application-non-critical
```

## Configuring an Application to use PriorityClasses

An application enables the use of a PriorityClass by using the PriorityClassName under the Pod Specification, underneath this is exemplified for a burstable deployment based on cpu request being set and limit not set. 
As explained above this may lead to an overcommit for cpu seen from a node and cluster perspective:

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: a-customer-critical-deployment
  labels:
    app.kubernetes.io/name: back-end-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app.kubernetes.io/name: back-end-deployment
  template:
    metadata:
      labels:
        app.kubernetes.io/name: back-end-deployment
    spec:
      terminationGracePeriodSeconds: 10 # short grace period - default is 30 seconds
      priorityClassName: "secure-cloud-stack-tenant-namespace-application-critical"
      containers:
      - image: nginxinc/nginx-unprivileged:1.20
        name: back-end-deployment
        resources:
            requests:
              memory: 990M
              cpu: 5m
            limits:
              memory: 990M
        ports:
        - containerPort: 8080
          name: http
```

If nothing is specified for the application pods, the default assigned PriorityClassName is  `secure-cloud-stack-tenant-namespace-application-non-critical`. This is something supported by kubernetes itself. 

The default grace period for a pod is 30 seconds, which means the pods gets preempted at that point - ready or not. If you want to ensure that lower priority pods are [preemted](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/) faster, you may adjust the `terminationGracePeriodSeconds` to a feasible number of seconds lower than the default.

Please note that there may be derived classes in some situations, where e.g. an operator is used, or a sidecar is used etc. which also need to have the `priorityClassName` set in order for that not to be assigned default priority.
