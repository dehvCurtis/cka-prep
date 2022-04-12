# Services & Networking (20%)

Table of Contents

- [Networking configuration on the cluster nodes](#networking-configuration-on-the-cluster-nodes)
- [Understand connectivity between Pods](#Understand-connectivity-between-Pods)

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

### LoadBalancer

[`LoadBalancer`](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer): Exposes the Service externally using a cloud provider's load balancer. `NodePort` and `ClusterIP` Services, to which the external load balancer routes, are automatically created.

**Create `NodePort` & `LoadBalancer`**

*If on minikube, start tunnel*

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

