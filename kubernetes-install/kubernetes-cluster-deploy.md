# For Control Plane and Worker nodes

## Pre-deployment

1.  Backup /etc/fstab file
```
sudo mkdir -p /fstab-backup/ && cp /etc/fstab /fstab-backup/fstab.backup
```
2.  Disable Swap
```
sudo swapoff -a && sed -i '/[[:space:]]swap[[:space:]]/ s/^\(.*\)$/#\1/g' /etc/fstab
```
3.  Load kernel modules and configure kernel parameters
```
sudo bash -c 'cat > /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
sudo modprobe overlay br_netfilter
'
```

```
sudo bash -c 'cat > /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sysctl --quiet --system 2>/dev/null
'
```
4.  Install prerequisite applications
```
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
```
5.  Configure Docker repository
```
sudo bash -c 'curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/trusted.gpg.d/docker.gpg
echo "deb [arch=amd64 signed-by=/etc/apt/trusted.gpg.d/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
'
```
6.  Configure Kubernetes repository
```
sudo bash -c '
LATEST_K8S_VERSION=$(curl -sL https://dl.k8s.io/release/stable.txt | sed "s/^v//")
K8S_MAJOR_MINOR=$(echo "$LATEST_K8S_VERSION" | cut -d "." -f 1,2)
K8S_REPO="https://pkgs.k8s.io/core:/stable:/v${K8S_MAJOR_MINOR}/deb/"
curl -fsSL "${K8S_REPO}Release.key" | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
touch /etc/apt/sources.list.d/kubernetes.list
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] ${K8S_REPO} /" > /etc/apt/sources.list.d/kubernetes.list
'
```
7.  Update apt packages
```
apt update -y
```
## Deployment
1.  Install and configure containerd
```
apt install -y containerd.io
```

```
containerd config default > /etc/containerd/config.toml 2>&1
```

```
sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```

```
systemctl enable containerd.service
```

```
systemctl restart containerd.service
```
2.  Install and configure kubernetes
```
apt install -y kubelet kubeadm kubectl
```

```
apt-mark hold kubelet kubeadm kubectl
```

```
systemctl enable kubelet.service
```

# For Control Plane node
1.  Initialize control plane, for example:
```
sudo kubeadm init --control-plane-endpoint=controlNode.example.com
```
2.  Copy kubeconfig file to home directory
```
sudo bash -c '
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
'
```
3.  Install network plugin
```
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/calico.yaml
```
4.  Verify installation and status
```
kubectl cluster-info
kubectl get nodes
```

# For Worker nodes
1.  Join workers to control plane

Example:
```
kubeadm join controlNode.example.com:6443 --token xtn3zi.bro5vmh1br5ut7sd \
        --discovery-token-ca-cert-hash sha256:5d3243fc472921d4b958b4e167d3febdb9a26a8b0a8f2b36b7ee91a497967dfb
```

# Validate cluster
1.  Ensure control plane is running
```
kubectl cluster-info
```
2.  All nodes should be ready
```
kubectl get nodes
```
3.  All pods in kube-system namespace should be ready 1/1 and running
```
kubectl get pods -n kube-system
```