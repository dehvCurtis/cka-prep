# Workloads & Scheduling [ 15% ]

Table of Contents

- [Description](#Description)
- [Deployments](#Deployments)
  - Deployments, Rolling Updates and Rollbacks
  - ConfigMaps and Secrets
  - Know how to scale applications
  - Understand the primitives used to create robust, self-healing, application deployments
  - Understand how resource limits can affect Pod scheduling 
  - Awareness of manifest management and common templating tools
- ConfigMaps and Secrets

# Deployments

## Deployments, Rolling Updates and Rollbacks

### Description

Understand deployments and how to perform rolling update and rollbacks

### Reading

Overview of Deployments https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

### Creating Deployments

Create Deployment


```bash
# Create the namespace
kubectl create ns test

# Create deployment & replicas
kubectl create deployment test-deployment -n test --image=nginx:1.19 --replicas=3
```

Confirm deployment is functional

```bash
# check deployment
kubectl -n test get deployment test-deployment

# check pods
kubectl -n test get pods
```

### Reading

Overview of Rolling Update: https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/

### Performing a Rolling Update

Scale to 5 replicas

```bash
kubectl scale deployment test-deployment -n test --replicas=5 
```

Update to newer image

```bash
kubectl edit deployment test-deployment -n test
```

### Practice Performing a Rollback

Check the history of the deployment

```bash
kubectl rollout history deployment test-deployment -n test
```

Rollback to previous revision

```bash
# Check rollout history
kubectl rollout history deployment test-deployment -n test

# Undo changes
kubectl rollout undo deployment test-deployment -n test

# Confirm ReplicaSet
kubectl get rs -n test

# Confirm pods
kubectl get pods -n test

# Check pod version
kubectl describe pod <pod-name> -n test
```

## ConfigMaps and Secrets

### Description

Use ConfigMaps and Secrets to configure applications

### Reading

https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/

### Practice Environment Variables
