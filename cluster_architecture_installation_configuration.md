# Cluster Architecture, Installation & Configuration (25%)

Table of Contents

- [Use KubeADM to install a basic cluster](#Use-KubeADM-to-install-a-basic-cluster)
- [Perform a Upgrade](#Perform-a-version-upgrade-on-a-Kubernetes-cluster-using-KubeADM)
- [Check Certificate Information](#Check-Certificate-Information)
- [Backup Restore etcd cluster](#Backup-Restore-etcd-cluster)
- [Manage role-based access control (RBAC)](#Manage-role-based-access-control-RBAC)

## Use KubeADM to install a basic cluster

### Install container runtime

Doc: https://kubernetes.io/docs/setup/production-environment/container-runtimes/

Do this on all three nodes:

```bash 
# disable swap (required by cluster)
swapoff -as

# containerd preinstall configuration
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Setup required sysctl params, these persist across reboots.
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

# Install containerd
## Set up the repository
### Install packages to allow apt to use a repository over HTTPS
sudo apt-get update
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

## Add Docker’s official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

## Add Docker apt repository.
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

## Install packages
sudo apt-get update
sudo apt-get install -y \
  containerd.io

# Configure containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

# Restart containerd
sudo systemctl restart containerd
```

### Install kubeadm, kubelet and kubectl

Doc: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

Do this on all three nodes:

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet=1.20.7-00 kubeadm=1.20.7-00 kubectl=1.20.7-00
sudo apt-mark hold kubelet kubeadm kubectl
```

### Create a cluster with KubeADM

Setup alias `k` and `kubectl` auto-complete

https://kubernetes.io/docs/tasks/tools/included/optional-kubectl-configs-bash-linux/

```bash
source /usr/share/bash-completion/bash_completion
echo 'source <(kubectl completion bash)' >>~/.bashrc
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc
source ~/.bashrc
```

Doc: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

Make sure the nodes have different hostnames.

On controlplane node:
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --cri-socket unix:///run/containerd/containerd.sock
```

Run the output of the init command on worker nodes:
```bash
sudo kubeadm join 172.16.1.11:6443 --token h8vno9.7eroqaei7v1isdpn \
    --discovery-token-ca-cert-hash sha256:44f1def2a041f116bc024f7e57cdc0cdcc8d8f36f0b942bdd27c7f864f645407 --cri-socket unix:///run/containerd/containerd.sock
```

On master node:
```bash
# Configure kubectl access
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Deploy Flannel as a network plugin
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Confirm Contexts
```bash
kubectl config view
```

If not context exist on node, perform the following

```bash
# Copy the config from the master
cat $HOME/.kube/config

# Create config on the node
# confirm contexts
kubectl config view

# make config
mkdir -p $HOME/.kube

# paste config from mastercontrol plane
vi $HOME/.kube/config

# test config
kubectl config view
kubectl get nodes
```

### Check that your nodes are running and ready

```bash
retrieve ip address of master control plane
ip add | grep inet

# check control plane connectivity
nc -z -v <control-plane-ip> 6443

kubectl get nodes
NAME               STATUS   ROLES                  AGE     VERSION
k8s-controlplane   Ready    control-plane,master   2m35s   v1.20.7
k8s-node-1         Ready    <none>                 2m7s    v1.20.7
k8s-node-2         Ready    <none>                 117s    v1.20.7
```

## Perform a version upgrade on a Kubernetes cluster using KubeADM

Doc: https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

https://github.com/kubernetes/kubernetes/releases

On controlplane node:

```bash
# Upgrade kubeadm
sudo apt-mark unhold kubeadm
sudo apt-get update
sudo apt-get install -y kubeadm=1.21.1-00
sudo apt-mark hold kubeadm

# Upgrade controlplane node
sudo kubeadm upgrade plan
kubectl drain k8s-controlplane --ignore-daemonsets
sudo kubeadm upgrade apply v1.21.1

# Update Flannel
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# Upgrade kubelet and kubectl
sudo apt-mark unhold kubelet kubectl
sudo apt-get update && sudo apt-get install -y kubelet=1.21.1-00 kubectl=1.21.1-00
sudo apt-mark hold kubelet kubectl
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# Make master node reschedulable
kubectl uncordon k8s-controlplane
```

On worker nodes:

```bash
# Upgrade kubeadm
sudo apt-mark unhold kubeadm
sudo apt-get update && sudo apt-get install -y kubeadm=1.21.1-00
sudo apt-mark hold kubeadm

# Upgrade worker node
kubectl drain k8s-node-1 --ignore-daemonsets
sudo kubeadm upgrade node

# Upgrade kubelet and kubectl
sudo apt-mark unhold kubelet kubectl
sudo apt-get update && sudo apt-get install -y kubelet=1.21.1-00 kubectl=1.21.1-00
sudo apt-mark hold kubelet kubectl
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# Make worker node reschedulable
kubectl uncordon k8s-node-1
```

Verify that the nodes are upgraded to v1.21:

```yaml
kubectl get nodes
NAME               STATUS   ROLES                  AGE   VERSION
k8s-controlplane   Ready    control-plane,master   21m   v1.21.1
k8s-node-1         Ready    <none>                 21m   v1.21.1
k8s-node-2         Ready    <none>                 21m   v1.21.1
```

### Facilitate operating system upgrades

When having a one master node in you cluster, you cannot upgrade the OS system (with reboot) without loosing temporarily access to your cluster.

Here we will upgrade our worker nodes:

```bash
# Hold kubernetes from upgrading
sudo apt-mark hold kubeadm kubelet kubectl

# Upgrade node
kubectl drain k8s-node-1 --ignore-daemonsets
sudo apt update && sudo apt upgrade -y # Be careful about container runtime (e.g., docker) upgrade.

# Reboot node if necessary
sudo reboot

# Make worker node reschedulable
kubectl uncordon k8s-node-1
```

## Check Certificate Information

```shell
# get pods
kubectl -n kube-system get pods

# retrieve cert info via describe pod
kubectl -n kube-system describe pod <etc-pod>

# retrieve cert info via kubenetes manifest
find /etc/kubernetes/manifests
cat /etc/kubernetes/manifests/etcd.yaml

# get cert expirey info
openssl x509 -text -in <server-cert>
```

## Backup Restore etcd cluster

Doc: https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster

Check the version of your etcd cluster depending on how you installed it.

```bash
kubectl exec -it -n kube-system etcd-k8s-controlplane -- etcd --version
etcd Version: 3.4.13
Git SHA: ae9734ed2
Go Version: go1.12.17
Go OS/Arch: linux/amd64
```

```shell
# Download/Install etcd client
wget https://github.com/etcd-io/etcd/releases/download/v3.4.13/etcd-v3.4.13-linux-amd64.tar.gz
tar xzvf etcd-v3.4.13-linux-amd64.tar.gz
sudo mv etcd-v3.4.13-linux-amd64/etcdctl /usr/local/bin

# Gather control-plane endpoint IP
kubectl get endpoints

# Get etcd pod name from kube-system namespace
k -n kube-system get pods

# grep to output cert info
k -n kube-system describe pod <etcd-pod> | grep "\-\-"

# output needed arguments
etcdctl --help | grep -A 25 OPTIONS

# etcdctl api env var
export ETCDCTL_API=3

# create etcd snapshot
etcdctl --endpoints=<endpoint-ip>:2379 \
--cert="" \
--cacert="" \
--key="" \
snapshot save <snapshot-name>

# View snapshot
sudo etcdctl snapshot status <snapshot-name> --write-out=table 
```


### Restore an etcd cluster from a snapshot

Doc: https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster

```shell
# Restore from endpoint location
ETCDCTL_API=3 etcdctl --endpoints <ip-address>:2379 snapshot restore <snapshot-name>

# Restore from directory
ETCDCTL_API=3 etcdctl --data-dir <data-dir-location> snapshot restore <snapshot-name>
```

## Manage role-based access control (RBAC)

Documentation and Resources:

- [Kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [A Practical Approach to Understanding Kubernetes Authorization](https://thenewstack.io/a-practical-approach-to-understanding-kubernetes-authorization/)

RBAC is handled by roles (permissions) and bindings (assignment of permissions to subjects):

| Object               | Description                                                  |
| -------------------- | ------------------------------------------------------------ |
| `Role`               | Permissions within a particular namespace                    |
| `ClusterRole`        | Permissions to non-namespaced resources; can be used to grant the same permissions as a Role |
| `RoleBinding`        | Grants the permissions defined in a role to a user or set of users |
| `ClusterRoleBinding` | Grant permissions across a whole cluster                     |

### Lab Practice

Create the `test-ns` namespace.

`kubectl create namespace test-ns`

---

Create a deployment in the `test-ns` namespace using the image of your choice:

1. `kubectl create deployment hello-node --image=k8s.gcr.io/echoserver:1.4 -n test-ns`
1. `kubectl create deployment busybox --image=busybox -n test-ns -- sleep 2000`

You can view the yaml file by adding `--dry-run=client -o yaml` to the end of either deployment.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: hello-node
  name: hello-node
  namespace: test-ns
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-node
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: hello-node
    spec:
      containers:
        - image: k8s.gcr.io/echoserver:1.4
          name: echoserver
          resources: {}
```

---

Create the `pod-reader` role in the `test-ns` namespace.

`kubectl create role pod-reader --verb=get --verb=list --verb=watch --resource=pods -n test-ns`

> Alternatively, use `kubectl create role pod-reader --verb=get --verb=list --verb=watch --resource=pods -n test-tens --dry-run=client -o yaml` to output a proper yaml configuration.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: null
  name: pod-reader
  namespace: test-ns
rules:
  - apiGroups:
      - ""
    resources:
      - pods
    verbs:
      - get
      - list
      - watch
```

---

Create the `read-pods` rolebinding between the role named `pod-reader` and the user `spongebob` in the `test-ns` namespace.

`kubectl -n test-ns create rolebinding read-pods --role=pod-reader --user=spongebob`

> Alternatively, use `kubectl -n test-ns create rolebinding --role=pod-reader --user=spongebob read-pods --dry-run=client -o yaml` to output a proper yaml configuration.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: null
  name: read-pods
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-reader
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: spongebob
```

---

Create the `cluster-secrets-reader` clusterrole.

`kubectl create clusterrole cluster-secrets-reader --verb=get,list,watch --resource=secrets`

> Alternatively, use `kubectl create clusterrole cluster-secrets-reader --verb=get,list,watch --resource=secrets --dry-run=client -o yaml` to output a proper yaml configuration.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  creationTimestamp: null
  name: cluster-secrets-reader
rules:
- apiGroups:
  - ""
  resources:
  - secrets
  verbs:
  - get
  - list
  - watch
```
---

Create the `cluster-read-secrets` clusterrolebinding between the clusterrole named `cluster-secrets-reader` and the user `gizmo`.

`kubectl create clusterrolebinding --clusterrole=cluster-secrets-reader --user=gizmo cluster-read-secrets`

> Alternatively, use `kubectl create clusterrolebinding --clusterrole=cluster-secrets-reader --user=gizmo cluster-read-secrets --dry-run=client -o yaml` to output a proper yaml configuration.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  creationTimestamp: null
  name: cluster-read-secrets
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-secrets-reader
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: gizmo
```

Test to see if this works by running the `auth` command.

`kubectl auth can-i get secrets --as=gizmo`

Attempt to get secrets as the `gizmo` user.

`kubectl get secrets --as=gizmo`

```bash
NAME                  TYPE                                  DATA   AGE
default-token-lz87v   kubernetes.io/service-account-token   3      7d1h
```
