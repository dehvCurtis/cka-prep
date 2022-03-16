# Workloads & Scheduling [ 15% ]

Table of Contents

- [Description](#Description)
- [Deployments](#Deployments)
  - Understand deployments and how to perform rolling update and rollbacks
  - Use ConfigMaps and Secrets to configure applications
  - Know how to scale applications
  - Understand the primitives used to create robust, self-healing, application deployments
  - Understand how resource limits can affect Pod scheduling 
  - Awareness of manifest management and common templating tools
- ConfigMaps and Secrets

# Deployments

### Description

Understand deployments and how to perform rolling update and rollbacks

### Reading

Overview of Deployments https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

### Objectives

- Create a namespace named `ngx`
- Create a deployment named `nginx-deploy` in the `ngx` namespace using `nginx:1.19` with three replicas
- Confirm the deployment rolled out successfully

### Practice Creating Deployments

Create Deployment


```bash
# Create the namespace
kubectl create ns test

# Create deployment & replicas
kubectl create deployment test-deployment -n test --image=nginx --replicas=3
```

Confirm deployment is functional:

```bash
# check rollout status
kubectl -n test rollout status deployment/test-deployment

# check deployment
kubectl -n test get deployment/test-deployment
```

Check pods from deployment:

```bash
kubectl -n test get pods
```

### Practice Performing a Rolling Update

Scale to 5 replicas

```bash
kubectl scale deployment test-deployment -n test --replicas=5 
```

Update to newer image



### Practice Performing a Rollback

