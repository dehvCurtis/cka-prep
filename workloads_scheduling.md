# Workloads & Scheduling [ 15% ]

Table of Contents

- [Deployments](#Deployments)
  - [Creating Deployments](#creating-deployments)
  - [Rolling Update](#rolling-update)
  - [Rollback](#rollback)
- [ConfigMaps, Environment Variables and Secrets](#configmaps-environment-variables-and-secrets)
  - [ConfigMaps](#configmaps)
  - [Environment Variables](#environment-variables)
  - [Secrets](#secrets)
- [Scale Applications](#scale-applications)
  - [Scale Application](#scale-application)
  - [Auto-Scale Application](#auto-scale-application)
- [Robust, Self-Healing, Application Deployments](#robust-self-healing-application-deployments)
  - [ReplicaSets](#replicasets)
  - [DaemonSets](#daemonSets)
  - [Resource Limits](#resource-limits)

# Deployments

## Deployments, Rolling Updates and Rollbacks

### Creating Deployments

Overview of Deployments https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

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

### Rolling Update

Overview of Rolling Update: https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/

Update Image

```bash
kubectl edit deployment test-deployment -n test

# change image version and save
```

### Rollback

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

# Confirm pods
kubectl get pods -n test

# Check pod version
kubectl describe pod <pod-name> -n test
```

## ConfigMaps, Environment Variables and Secrets

### ConfigMaps

https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

Create ConfigMap from files

```bash	
# Download example config files
mkdir configmaps && cd configmaps
wget https://kubernetes.io/examples/configmap/game.properties
wget https://kubernetes.io/examples/configmap/ui.properties

# Create ConfigMap
kubectl create configmap test-config --from-file=game.properties

# Confirm ConfigMap
kubectl decsribe configmap test-config
kubectl get configmap test-config -o yaml
```

Define the key when creating ConfigMap from file

```bash
kubectl create configmap test-config-2 --from-file TEST_KEY=configmaps/game.properties
```

Create ConfigMap from literal values

```bash
kubectl create configmap test-config-3 --from-literal blue=blueberry
```

### Environment Variables

https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/

Create a pod using latest `nginx` image with an environment variable

```bash
kubectl run pod-test -n test --image=nginx --env=TEST=test
```

Exec into pod to confirm environment variable

```bash
kubectl exec pod-test -n test -- env | grep TEST
```

### Secrets

https://kubernetes.io/docs/concepts/configuration/secret/

Create files that contain secrets

```bash
echo -n 'admin' > /tmp/admin.txt
echo -n 'passw0rd' > /tmp/pass.txt
```

Create secret from file

```bash 
# Create secrets from file
kubectl create secret generic user-pass-secret --from-file=username=/tmp/admin.txt --from-file=password=/tmp/pass.txt

# Confirm secrets
kubectl get secret user-pass-secret -o jsonpath='{.data}'
kubectl get secret user-pass-secret -o jsonpath='{.data.username}'
kubectl get secret user-pass-secret -o jsonpath='{.data.password}'
```

Create secret from literal

```bash
# Create secret using literals
kubectl create secret generic test-secret --from-literal=testkey='testvalue'

# Confirm secrets
kubectl get secret test-secret -o jsonpath='{.data}'

# Decode secret
kubectl get secret test-secret -o jsonpath={'.data.testkey'} | base64 --decode
```

Create secret from `.env` file

```bash
# Create `.env` file
echo -n "TESTVAR=testenvvar" > .env

# Create secret with .env file
kubectl create secret generic testsecret --from-env-file=.env

# Confirm secret
kubectl get secret testsecret -o jsonpath={.data}
```

Editing secrets

```bash
# Create new base64 secret
echo -n 'test-secret' | base64

# Edit secret
kubectl edit secret test-secret
```

Use secret as environment variables

```bash
```

## Scale Applications

### Scale Application

```bash
kubectl scale deployment test-deployment --replicas=5
```

### Auto-Scale Application

https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/

https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/

```bash
# Configure metrics server
git clone https://github.com/kubernetes-sigs/metrics-server

# Deploy the metrics server
kubectl apply -k metrics-server/manifests/base/

# Autoscale a deployment
kubectl create deployment autoscalable --image=nginx:latest
kubectl autoscale deployment autoscalable --min=2 --max=6 --cpu-percent=70

# Check HPA and pods
kubectl get hpa
kubectl get pods
```

## Robust, Self-Healing, Application Deployments

### ReplicaSets

https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/

A deployment uses a replicaset object to maintain the right number of desired replicas of a pod. See section [Deployments](#Deployments) above to see how deployments handle replicaset for updating.

### DaemonSets

https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

Create DaemonSet with latest busybox image and see that it runs on all nodes.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    type: daemon
  name: daemontest
spec:
  selector:
    matchLabels:
      run: daemon
  template:
    metadata:
      labels:
        run: daemon
      name: daemonpod
    spec:
      containers:
      - image: busybox:latest
        name: daemonpod
        args:
          - sleep
          - "3600"
```

Deploy DaemonSet

```bash
kubectl apply -f daemonset.yaml
```

### Resource Limits

https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/

Create `busybox` pod with the following:

request

- 1GB memory (1Gi)
- half CPU (500m)

limit

- 2GB memory (2Gi)
- whole CPU (1)

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: podquota
  name: podquota
spec:
  containers:
  - image: busybox:latest
    name: podquota
    args:
      - sleep
      - "3600"
    resources:
      requests:
        memory: "1Gi"
        cpu: "500m"
      limits:
        memory: "2Gi"
        cpu: "1"
```

