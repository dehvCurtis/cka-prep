# Workloads & Scheduling [ 15% ]

Table of Contents

- [Description](#Description)
- [Deployments](#Deployments)
  - Create Deployment
  - Scale Deployment
  - Rolling Updates
  - Rollbacks

- ConfigMaps and Secrets

# Deployments

## <u>Create Deployment</u>

### Reading

Overview of Deployments https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

### Objectives

- Create a namespace named `ngx`
- Create a deployment named `nginx-deploy` in the `ngx` namespace using `nginx:1.19` with three replicas
- Confirm the deployment rolled out successfully

### Practice

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

## <u>Scale Deployment</u>

### Reading

AutoScaling

- https://kubernetes.io/de/docs/tasks/run-application/horizontal-pod-autoscale/
- https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/

### Objectives

- Create scalable deployment
- Scale Deployment
- Auto-Scale Deployment

### Practice

Create deployment

```bash
# Create namespace & scalable deployment
kubectl create deployment test-deployment -n test --replicas=2 --image=nginx:1.9
```

Scale deployment

```bash
# Scale to a total of 5 pods
k scale deployment test-deployment -n test --replicas=5
```

Autoscale a deployment

- minimum of two pods
- maximum of six pods
- autoscales when cpu usage goes above 70%

```bash
```

