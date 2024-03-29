# Services & Networking (20%)

Table of Contents

- [Networking configuration on the cluster nodes](#networking-configuration-on-the-cluster-nodes)
- [Understand connectivity between Pods](#Understand-connectivity-between-Pods)
  - ClusterIP
  - NodePort
  - LoadBalancer
- [Ingress controllers and Ingress resources](#Ingress_controllers_and_Ingress_resources)
- [Configure and Use CoreDNS](#configure-and-use-CoreDNS)

Understand ClusterIP, NodePort, LoadBalancer service types and endpoints

Know how to use Ingress controllers and Ingress resources

Know how to configure and use CoreDNS

Choose an appropriate container network interface plugin

## Networking configuration on the cluster nodes

https://kubernetes.io/docs/concepts/cluster-administration/networking/

## Understand connectivity between Pods

https://kubernetes.io/docs/concepts/cluster-administration/networking/

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

Deploy `hello-world` app

```shell
kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0
```

Create `ingress.yaml` from the following file

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: hello-world.info
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 8080
```

Create the Ingress object

```shell
kubectl apply -f ingress.yaml
```

Verify the IP address is set:

```shell
kubectl get ingress
```

Add the IP resolution to `/etc/hosts` file 

```
<ip-address> hello-world.info
```

Verify that the Ingress controller is directing traffic:

```shell
curl hello-world.info
```

## Configure and Use CoreDNS

https://kubernetes.io/docs/concepts/services-networking/dns-pod-service

https://kubernetes.io/docs/tasks/administer-cluster/coredns
