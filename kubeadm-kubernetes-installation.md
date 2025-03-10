```bash
hostnamectl set-hostname "k8s-master01" && exec bash
echo "192.168.122.15 k8s-master01" >> /etc/hosts
echo "192.168.122.168 k8s-worker01" >> /etc/host
systemctl stop firewalld
systemctl disable firewalld
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/sysconfig/selinux
echo -e "overlay\nbr_netfilter" | sudo tee /etc/modules-load.d/containerd.conf && sudo modprobe overlay && sudo modprobe br_netfilter
echo -e "net.bridge.bridge-nf-call-iptables = 1\nnet.ipv4.ip_forward = 1\nnet.bridge.bridge-nf-call-ip6tables = 1" | sudo tee -a /etc/sysctl.d/k8s.conf
sysctl --system
```
```bash
dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
dnf install containerd.io -y
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
systemctl restart containerd
systemctl enable containerd
systemctl status containerd
```
Create the repo file for kubernetes
```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
```bash
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.32/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.32/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF
```
Register and install the package on all nodes.
```bash
subscription-manager register
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable --now kubelet
kubeadm init --control-plane-endpoint=k8s-master01
```

```bash
 44  mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
mkdir -p $HOME/.kube && bash -c "cp -i /etc/kubernetes/admin.conf $HOME/.kube/config && chown $(id -u):$(id -g) $HOME/.kube/config"
kubectl get nodes
```

Edit below file and the lines under command section.

vi /etc/kubernetes/manifests/kube-controller-manager.yaml
    - --allocate-node-cidrs=true
    - --cluster-cidr=10.244.0.0/16

Check if CIDR is added to the node and if flannel pods are started
```bash
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
kubectl get pods -n kube-flannel
kubectl get nodes -o custom-columns="NAME:.metadata.name,POD_CIDR:.spec.podCIDR"
```
