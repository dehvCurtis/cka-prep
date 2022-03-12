# Workloads & Scheduling [ 15% ]
- How to Perform Deployment
- How to Perform Rolling Update 
- How to Perform Rollbacks

## Deployments

### Create Deployment
```bash
kubectl create deployment test-deployment --image=alpine
```

### Scale Deployment
```bash
kubectl scale deployment test-deployment --replicas=4 # Scale Up / Down
```

### Reference
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

## Rolling Updates

### Update Image
```bash
kubectl set image deployment test-deployment <name-of-container>=<new-image-name>
```

### Reference

## Rollback
```bash

```

### Reference
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment

## Use Config Maps and Secrets to Configure applications
Kubernetes `configmaps` are used to store non-critical data in kv pair format. They can also be used to inject env vars into pods.


```bash
kubectl create cm test-configmap --from-file=<file>
```

```bash
kubectl create cm test-configmap --from-literal=key1=value1 
```

## Secrets
Secrets are used to store sensitive data in kv pair format. They can also be used to inject env vars into pods.

```bash
kubectl create secrets test-secret --from-file=hello.txt 
```

```bash
kubectl create secrets test-secret --from-file=hello.txt 
```
