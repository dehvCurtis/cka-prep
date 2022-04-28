# Cluster Architecture, Installation & Configuration (25%)

# Cluster Architecture, Installation & Configuration (25%)

## Manage role-based access control (RBAC)

Doc: https://kubernetes.io/docs/reference/access-authn-authz/rbac/

## Use KubeADM to install a basic cluster

<details><summary>Solution</summary>
<p>
### Install container runtime

<details><summary>Solution</summary>
<p>

Doc: https://kubernetes.io/docs/setup/production-environment/container-runtimes/

Do this on all three nodes:

```bash
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

</p>
</details>

### Install kubeadm, kubelet and kubectl

<details><summary>Solution</summary>
<p>

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

</p>
</details>

### Create a cluster with KubeADM

<details><summary>Solution</summary>
<p>

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

On master node again:
```bash
# Configure kubectl access
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Deploy Flannel as a network plugin
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

</p>
</details>

### Check that your nodes are running and ready

<details><summary>Solution</summary>
<p>

```bash
kubectl get nodes
NAME               STATUS   ROLES                  AGE     VERSION
k8s-controlplane   Ready    control-plane,master   2m35s   v1.20.7
k8s-node-1         Ready    <none>                 2m7s    v1.20.7
k8s-node-2         Ready    <none>                 117s    v1.20.7
```

</p>
</details>

</p>
</details>

## Provision underlying infrastructure to deploy a Kubernetes cluster

<details><summary>Solution</summary>
<p>

You can use any cloud provider (AWS, Azure, GCP, OpenStack, etc.) and multiple tools to provision nodes for your Kubernetes cluster.

We will deploy a three node cluster, with one master node and two worker nodes.

Three Libvirt/KVM nodes (or any cloud provider you are using):
- k8s-controlplane: 2 vCPUs, 4GB RAM, 40GB Disk, 172.16.1.11/24
- k8s-node-1: 2 vCPUs, 2GB RAM, 40GB Disk, 172.16.1.21/24
- k8s-node-2: 2 vCPUs, 2GB RAM, 40GB Disk, 172.16.1.22/24

OS description:

```bash
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 20.04.4 LTS
Release:	    20.04
Codename:	    focal
```

We will use a local libvirt/KVM baremetal node with terraform (v0.15.3) to provision the three node cluster described above.

```bash
mkdir terraform
cd terraform
wget https://raw.githubusercontent.com/alijahnas/CKA-practice-exercises/master/terraform/cluster-infra.tf
terraform init
terraform plan
terraform apply
```
w
</p>
</details>

## Perform a version upgrade on a Kubernetes cluster using KubeADM

<details><summary>Solution</summary>
<p>

Doc: https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

After installing Kubernetes v1.20 here: [install](https://github.com/alijahnas/CKA-practice-exercises/blob/CKA-v1.20/cluster-architecture-installation-configuration.md#use-kubeadm-to-install-a-basic-cluster)

We will now upgrade the cluster to v1.21.

On controlplane node:

```bash
# Upgrade kubeadm
sudo apt-mark unhold kubeadm
sudo apt-get update && sudo apt-get install -y kubeadm=1.21.1-00
sudo apt-mark hold kubeadm

# Upgrade controlplane node
kubectl drain k8s-controlplane --ignore-daemonsets
sudo kubeadm upgrade plan
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

</p>
</details>

### Facilitate operating system upgrades

<details><summary>Solution</summary>
<p>

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

</p>
</details>

## Implement etcd backup and restore

<details><summary>Solution</summary>
<p>

### Backup etcd cluster

<details><summary>Solution</summary>
<p>

Doc: https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster

Check the version of your etcd cluster depending on how you installed it.

```bash
kubectl exec -it -n kube-system etcd-k8s-controlplane -- etcd --version
etcd Version: 3.4.13
Git SHA: ae9734ed2
Go Version: go1.12.17
Go OS/Arch: linux/amd64
```

```bash
# Download etcd client
wget https://github.com/etcd-io/etcd/releases/download/v3.4.13/etcd-v3.4.13-linux-amd64.tar.gz
tar xzvf etcd-v3.4.13-linux-amd64.tar.gz
sudo mv etcd-v3.4.13-linux-amd64/etcdctl /usr/local/bin

# save etcd snapshot
sudo ETCDCTL_API=3 etcdctl snapshot save --endpoints 172.16.1.11:2379 snapshot.db --cacert /etc/kubernetes/pki/etcd/server.crt --cert /etc/kubernetes/pki/etcd/ca.crt --key /etc/kubernetes/pki/etcd/ca.key

# View the snapshot
ETCDCTL_API=3 sudo etcdctl --write-out=table snapshot status snapshot.db 
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| 4056f9fc |    18821 |        809 |     4.1 MB |
+----------+----------+------------+------------+
```

</p>
</details>

### Restore an etcd cluster from a snapshot

<details><summary>Solution</summary>
<p>

Doc: https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster

</p>
</details>

</p>
</details>
