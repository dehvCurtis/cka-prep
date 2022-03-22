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

### Reading



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

## ConfigMaps, Environment Variables and Secrets

### Description

Use the `kubectl create configmap` command to create ConfigMaps from [directories](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#create-configmaps-from-directories), [files](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#create-configmaps-from-files), or [literal values](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#create-configmaps-from-literal-values)

### Reading

https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

### Reading

https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/

### Practice Environment Variables

Create a pod using latest `nginx` image with an environment variable

```bash
kubectl run pod-test -n test --image=nginx --env="TEST=test"
```

Exec into pod to confirm environment variable

```bash
kubectl exec pod-test -n test -- env | grep TEST
```

### Reading

https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

### Practice ConfigMaps

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

### Reading

https://kubernetes.io/docs/concepts/configuration/secret/

### Practice Secrets

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

Create secret as env variable

```bash
```

