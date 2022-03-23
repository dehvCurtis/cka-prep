# Workloads & Scheduling [ 15% ]

Table of Contents

- [Deployments](#Deployments)
  - [Creating Deployments](#creating-deployments)
  - [Performing a Rolling Update](#performing-a-rolling-update)
  - [Performing a Rollback](#performing-a-rollback)
- [ConfigMaps, Environment Variables and Secrets](#configmaps-environment-variables-and-secrets)

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

### Performing a Rolling Update

Overview of Rolling Update: https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/

Scale to 5 replicas

```bash
kubectl scale deployment test-deployment -n test --replicas=5 
```

Update to newer image

```bash
kubectl edit deployment test-deployment -n test
```

### Performing a Rollback

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

### Practice Environment Variables

https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/

Create a pod using latest `nginx` image with an environment variable

```bash
kubectl run pod-test -n test --image=nginx --env="TEST=test"
```

Exec into pod to confirm environment variable

```bash
kubectl exec pod-test -n test -- env | grep TEST
```

### Practice ConfigMaps

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

### Practice Secrets

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
