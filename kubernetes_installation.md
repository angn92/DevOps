## Kubernetes Instalation Instructions (Single node - Cluster)


# This setup install: [Kubernetes doc](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
- containerd (container runtime)
- kubeadm (command to bootstrap the cluster)
- kubelet (component that runs on all of the machines in cluster )
- kubectl (command line tool to talk with cluster)


# 1. Prepare Linux system
Update packages:
```bash
sudo apt update && sudo apt upgrade -y
```

Disable swap (required by Kubernetes)
```bash
sudo swapoff -a
```

Also necessary to comment out swap in file ```bash /etc/fstab```

To verify:
```bash
free -h
```

# 2. Enable required kernel modules

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

Load them:
```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

Configure sysctl:
```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF
```

Apply settings:
```bash
sudo sysctl --system
```

# 3. Install containerd

Install dependencies:

```bash
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
```

Install containerd:
```bash
sudo apt install -y containerd
```

Generate default config:
```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

Enable Systemd cgroup driver:
```bash
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
```

Verify:
```bash
grep SystemdCgroup /etc/containerd/config.toml
```

Result should be: ```bash SystemdCgroup = true```


Restart containerd:
```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```

Verify:
```bash
sudo systemctl status containerd
```


# 4. Install Kubernetes packages 

Add Kubernetes repository key: Before executing verify version on documentation
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Add the appropriate Kubernetes apt repository.
```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Install packages:
```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl # Prevent automatic upgrades:
```

(Optional) Enable the kubelet service before running kubeadm:
```bash
sudo systemctl enable --now kubelet
```

Verify:
```bash
kubeadm version
kubectl version
```


# 5. Initialize the Kubernetes control plane

CIDR should be compatible with CNI configuration like (Cilium, Flannel, Calico)

Initialize cluster:
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16

kubectl cluster-info

kubectl get nodes -o wide
```

When finished, configure kubectl for your user:
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Test:
```bash
kubectl get nodes
```

# 6. Install a CNI network plugin
Kubernetes needs pod networking.

Calico:
```bash
curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.32.0/manifests/calico.yaml
kubectl apply -f calico.yaml

kubectl get pods -n kube-system
```

# 7. Allow workloads on single-node clusters (only for lab/test not for production)

```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

To verify:
```bash
sudo systemctl status kubelet
sudo systemctl status containerd
```

Verify crictl:
```bash
sudo crictl version
```
If not installed run: [crictl](https://github.com/kubernetes-sigs/cri-tools/releases)

```bash
VERSION="v1.36.0"

wget https://github.com/kubernetes-sigs/cri-tools/releases/download/${VERSION}/crictl-${VERSION}-linux-amd64.tar.gz

sudo tar zxvf crictl-${VERSION}-linux-amd64.tar.gz -C /usr/local/bin

rm crictl-${VERSION}-linux-amd64.tar.gz
```
```bash
sudo crictl version
```

Create configuration for containerd:
```bash
sudo tee /etc/crictl.yaml <<EOF
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF
```

Install Helm: [Helm](https://helm.sh/docs/intro/install/)


# 9. Runnig Nginx to test cluster

First create new namespace
```bash
kubectl create namespace dev;
```

Create deploymnet yaml file

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

```bash
kubectl apply -f deployment.yaml
```


Service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: dev
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

```bash
kubectl apply -f service.yaml
```