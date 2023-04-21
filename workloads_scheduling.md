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
- [Assigning Pods to Nodes](#Assigning-Pods-to-Nodes)
  - [DaemonSets](#daemonSets)
  - [Node Selectors](#label-selectors)
  - [Affinity](#Affinity)
  - [Anti-Affinity](#Anti-Affinity)
  - [Taints & Tolerations](#taints--tolerations)
  - [Scheduler](#Scheduler)
- [Manifest Management & Common Templating Tools](#Manifest-Management--Common-Templating-Tools)

# Deployments

## Deployments, Rolling Updates and Rollbacks

### Creating Deployments

Overview of Deployments https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

Create namespace

```shell
# Create a namespace
kubectl create ns test
```

**Create Deployment - Imperative**

Create *.yaml for deployment

```shell
# Create deployment with output flag and dryrun flag
kubectl create deployment test-deploy --dry-run --image=nginx:1.19 -o yaml > /tmp/deployment.yaml
```

Deploy from `*.yaml` file

```
kubectl apply -f /tmp/deployment.yaml
```

**Create Deployment - Imperative**


```bash
# Create deployment & replicas - Imperative
kubectl create deployment test-deployment -n test --image=nginx:1.19 --replicas=3 -o yaml
```

```yaml
# Create deployment & replicas - Declarative
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app-gcr-deployment
  labels:
    app: hello-app-gcr
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-app-gcr
  template:
    metadata:
      labels:
        app: hello-app-gcr
    spec:
      containers:
      - name: hello-app-gcr
        image: gcr.io/google-samples/hello-app:2.0
        ports:
        - containerPort: 80
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

Create a pod with an environment variable

```bash
# Imperative
kubectl run pod-test -n test --image=nginx --env=TEST=test
```

Exec into pod to confirm environment variable

```bash
kubectl exec pod-test -n test -- env | grep TEST
```

```yaml
# Declarative
apiVersion: v1
kind: Pod
metadata:
  name: envvar-test
  labels:
    purpose: test-envars
spec:
  containers:
  - name: envvar-test-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: TEST
      value: "this is a test"
    - name: DEMO
      value: "this is also a test"
```

```bash
kubectl exec envar-test -n test -- env | grep TEST
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
kubectl create secret generic creds --from-literal=testkey='testvalue'

# Confirm secrets
kubectl get secret creds -o jsonpath='{.data}'

# Decode secret
kubectl get secret creds -o jsonpath={'.data.testkey'} | base64 --decode
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

```yaml
# Declarative

---
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: K8S_USERNAME
        valueFrom:
          secretKeyRef:
            name: creds
            key: username
            optional: false
      - name: K8S_PASSWORD
        valueFrom:
          secretKeyRef:
            name: creds
            key: password
            optional: false
```

Confirm Secret as Env Var

```bash
k exec secret-env-pod -- env | sort
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
# Autoscale a deployment
kubectl create deployment autoscalable --image=nginx:latest
kubectl autoscale deployment autoscalable --min=2 --max=6 --cpu-percent=70

# Check HPA and pods
kubectl get hpa
kubectl get pods
```

## Assigning Pods to Nodes

https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/

You can use any of the following methods to choose where Kubernetes schedules specific Pods:

 - nodeSelector field matching against node labels
 - Affinity and anti-affinity
 - nodeName field
 - Pod topology spread constraints

### DaemonSets

A `DaemonSet` ensures that all (or some) Nodes run a copy of a Pod.

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

Deploy `DaemonSet`

```bash
kubectl apply -f daemonset.yaml
```

### Node Selector

https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/

nodeSelector is the simplest recommended form of node selection constraint. You can add the nodeSelector field to your Pod specification and specify the node labels you want the target node to have. Kubernetes only schedules the Pod onto nodes that have each of the labels you specify.

#### Assign pod to node with `kind=special` node selector

Add the following to the `spec` of  your pod `*.yaml`

```yaml
nodeSelector:
  kind: special
```

Create pod
```bash
k apply -f nginx-alpine.yaml
```

Check pods
```bash
kubectl get pods
```

Pods should be `Pending`

Add the label to the node
```bash
k label node <node-name> kind=special
```

Check pods
```bash
kubectl get pods
```

### Affinity & Anti-Affinity

https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity

Affinity and anti-affinity expands the types of constraints you can define using `nodeSelector`. 

**Affinity**

There are two types of node affinity:

    - `requiredDuringSchedulingIgnoredDuringExecution`: The scheduler can't schedule the Pod unless the rule is met. This functions like nodeSelector, but with a more expressive syntax.
    - `preferredDuringSchedulingIgnoredDuringExecution`: The scheduler tries to find a node that meets the rule. If a matching node is not available, the scheduler still schedules the Pod.

You can specify node affinities using the `.spec.affinity.nodeAffinity` field in your Pod spec.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In
            values:
            - antarctica-east1
            - antarctica-west1
  containers:
  - name: with-node-affinity
    image: registry.k8s.io/pause:2.0
```

In this example, the following rule applies:

The node must have a label with the key `topology.kubernetes.io/zone` and the value of that label must be either `antarctica-east1` or `antarctica-west1`.

After creating this pod, it will remain in the pending state

Check your pods, add one of the labels to a node and confirm the pod is now in a running state.

```shell
kubectl get pods
kubectl label nodes <node> topology.kubernetes.io/zone=antarctica-east1
kubectl get pods
```
**Anti-Affinity**



### Taints & Tolerations

https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/


### Tolerations

Add to an example pod yaml. You specify a toleration for a pod in the PodSpec.

```yaml
tolerations:
- key: "key1"
  operator: "Exists"
  effect: "NoSchedule"
```

**Taints**

Add taint to Node.

```bash
kubectl taint node <node> <key>=<value>:<taint>
```

Remove taint from Node

```bash
kubectl taint node <node> <key>=<value>:<taint>-
```

<u>Example</u>

`NoSchedule` taint to node. `NoSchedule` means no pod will be able to schedule onto the node unless it has matching toleration.

```bash
kubectl taint node node1 key1=value1:NoSchedule
```

Remove taint from Node

```bash
kubectl taint node <node> key1=value1:NoSchedule-
```

### Scheduler

The Kubernetes scheduler is a control plane process which assigns Pods to Nodes. 

https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/

https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/

https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/



## Manifest Management & Common Templating Tools

- Helm: https://helm.sh/docs/intro/quickstart/
- Kustomize: https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/
