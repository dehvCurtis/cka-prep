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

## Practice

- Create a namespace named `ngx`
- Create a deployment named `nginx-deploy` in the `ngx` namespace using `nginx:1.19` with three replicas
- Confirm the deployment rolled out successfully

<details><summary>Solution</summary>
<p>

```bash
# Create the template from kubectl
kubectl -n ngx create deployment nginx-deploy --replicas=3 --image=nginx:1.19 --dry-run=client -o yaml > nginx-deploy.yaml

# Create the namespace first
kubectl create ns ngx
kubectl apply -f nginx-deploy.yaml
```

Check that the deployment has rolled out and that it is running:

```bash
kubectl -n ngx rollout status deployment/nginx-deploy
deployment "nginx-deploy" successfully rolled out

kubectl -n ngx get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   3/3     3            3           44s
```

Check the pods from the deployment:

```bash
kubectl -n ngx get pods
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-57767fb8cf-fjtls   1/1     Running   0          29s
nginx-deploy-57767fb8cf-krp4m   1/1     Running   0          29s
nginx-deploy-57767fb8cf-xvz8l   1/1     Running   0          29s
```


