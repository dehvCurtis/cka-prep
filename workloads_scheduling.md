# Workloads & Scheduling [ 15% ]

Table of Contents

- [Description](#Description)
- [Deployments](#Deployments)
  - Create Deployment
  - Scale Deployment
  - Rolling Updates
  - Rollbacks

- ConfigMaps and Secrets

# Description

Understand deployments and how to perform rolling update and rollbacks

# Deployments

## Reading

Overview of Deployments https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

## Create Deployment

### Objectives

- Create a namespace named `ngx`
- Create a deployment named `nginx-deploy` in the `ngx` namespace using `nginx:1.19` with three replicas
- Confirm the deployment rolled out successfully

### Practice


```bash
# Create the namespace
kubectl create ns test

# Create deployment & replicas
kubectl create deployment test-deployment -n test --image=nginx --replicas=3
```

- Confirm deployment is up:

```bash
# check rollout status
kubectl -n test rollout status deployment/test-deployment

# check deployment
kubectl -n test get deployment/test-deployment
```

Check the pods from the deployment:

```bash
kubectl -n test get pods
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-57767fb8cf-fjtls   1/1     Running   0          29s
nginx-deploy-57767fb8cf-krp4m   1/1     Running   0          29s
nginx-deploy-57767fb8cf-xvz8l   1/1     Running   0          29s
```

## Scale Deployment

### Objectives

- Confirm deployment
- Scale Deployment
- Auto-Scale Deployment

### Practice

```bash
```

