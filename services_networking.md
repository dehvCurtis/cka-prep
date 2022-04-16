# Services & Networking (20%)

Table of Contents

- [Networking configuration on the cluster nodes](#networking-configuration-on-the-cluster-nodes)
- [Understand connectivity between Pods](#Understand-connectivity-between-Pods)
  - ClusterIP
  - NodePort
  - LoadBalancer

- [Ingress controllers and Ingress resources](#Ingress_controllers_and_Ingress_resources)

Understand ClusterIP, NodePort, LoadBalancer service types and endpoints

Know how to use Ingress controllers and Ingress resources

Know how to configure and use CoreDNS

Choose an appropriate container network interface plugin

## Networking configuration on the cluster nodes

https://kubernetes.io/docs/concepts/cluster-administration/networking/

## Understand connectivity between Pods

https://kubernetes.io/docs/concepts/cluster-administration/networking/

## 

## Understand ClusterIP, NodePort, LoadBalancer service types and endpoints

### ClusterIP

https://kubernetes.io/docs/concepts/services-networking/service/

`ClusterIP`: Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default `ServiceType`.

### NodePort

https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport

`NodePort`: Exposes the Service on each Node's IP at a static port (the `NodePort`). A `ClusterIP` Service, to which the `NodePort` Service routes, is automatically created. You'll be able to contact the `NodePort` Service, from outside the cluster, by requesting `<NodeIP>:<NodePort>`

Create a Deployment:

```shell
kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0
```

Expose Deployment:

```bash
kubectl expose deployment web --type=NodePort --port=8080
```

Verify Service is created and is available on a node port:

```shell
kubectl get svc web
```

if on `minikube`, retrieve Service via NodePort :

```shell
minikube svc web --url
```

The output is similar to:

```bash
Hello, world!
Version: 1.0.0
Hostname: web-55b8c6998d-8k564
```

You can now access the sample app via the Minikube IP address and NodePort. The next step lets you access the app using the Ingress resource

### LoadBalancer

[`LoadBalancer`](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer): Exposes the Service externally using a cloud provider's load balancer. `NodePort` and `ClusterIP` Services, to which the external load balancer routes, are automatically created.

**Create `NodePort` & `LoadBalancer`**

*If on `minikube`, start tunnel*

```bash
minikube tunnel --cleanup
```

Create a deployment with the latest nginx image and two replicas

```bash
kubectl create deployment test-deployment --image=nginx:latest --replicas=2
```

Expose it's port 80 through a service of type `NodePort` or `LoadBalancer`

```bash
# NodePort
kubectl expose deployment test-deployment --type=NodePort --port=80 --target-port=80

# LoadBalancer
kubectl expose deployment hello-minikube2 --type=LoadBalancer --port=8080
```

Confirm service & Retrieve `Endpoint` and `NodePort`

```bash
kubectl describe svc test-deployment
```

Confirm `NodePort` via `curl` or browser

```bash
curl <ip-address>

minikube service --url test-deployment
```

## Ingress controllers and Ingress resources

https://kubernetes.io/docs/concepts/services-networking/ingress/

https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/

https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/

**Create Ingress Controller**

*note: if using `minikube` you need to enable ingress*

```bash
minikube addons enable ingress	
```

