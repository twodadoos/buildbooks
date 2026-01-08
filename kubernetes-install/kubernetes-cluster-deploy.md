## Pre-deployment

1.  Backup /etc/fstab file
```
mkdir -p /fstab-backup/

cp /etc/fstab /fstab-backup/fstab.backup
```
2.  Disable Swap
```
swapoff -a

sed -i '/[[:space:]]swap[[:space:]]/ s/^\(.*\)$/#\1/g' /etc/fstab
```

3.  Load kernel modules and configure kernel parameters
```
touch /etc/modules-load.d/containerd.conf

printf "%soverlay\nbr_netfilter\n" > /etc/modules-load.d/containerd.conf

modprobe overlay

modprobe br_netfilter

touch /etc/sysctl.d/kubernetes.conf

cat > /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sysctl --quiet --system 2>/dev/null
```
4.  Install prerequisite applications
```
apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
```
5.  Configure Docker repository
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg

add-apt-repository -y "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```
## Deployment
1.  Install and configure containerd
```
apt update

apt install -y containerd.io

containerd config default > /etc/containerd/config.toml 2>&1

sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

systemctl restart containerd.service

systemctl enable containerd.service
```
2.  Install and configure kubernetes
```
curl -fsSL "${K8S_REPO}Release.key" | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

touch /etc/apt/sources.list.d/kubernetes.list

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] ${K8S_REPO} /" > /etc/apt/sources.list.d/kubernetes.list

apt update

apt install -y kubelet kubeadm kubectl

apt-mark hold kubelet kubeadm kubectl

systemctl enable kubelet.service
```
