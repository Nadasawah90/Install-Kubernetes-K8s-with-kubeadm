# Install-Kubernetes-K8s-with-kubeadm
## Architecture 
we wnat to set up a 5-node Kubernetes cluster on CentOS 9, with:

2 Master nodes (control plane, for HA)

3 Worker nodes

1 HAProxy to configure HAProxy on an external node . 

Using kubeadm as the bootstrap tool

## Prerequisites for the cluster :
Firstly install VM with Hproxy configuration as belwow : 

1- OS for all nodes : CentOS 9 

Hostnames set (master1, master2, worker1, worker2, worker3)

Proper /etc/hosts resolution between nodes

2- Disable swap (required by kubelet):
sudo swapoff -a
sed -i '/swap/d' /etc/fstab
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system

3- Firewall / SELinux

Either disable firewalld or allow required ports:

sudo systemctl disable --now firewalld
sudo setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config


4- Install container runtime (containerd is recommended)

sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y containerd.io
yum install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd

5- Install kubeadm, kubelet, kubectl

cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/repodata/repomd.xml.key
EOF

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable --now kubelet

### if happen issue on etcd can not connect with another etcd master i fixed it by 
Resetting Kubernetes Cluster (on any node)

Run kubeadm reset

sudo kubeadm reset -f


This will remove the cluster configuration, kubelet state, and certificates.

Clean up CNI networking
If you used Flannel, Calico, Weave, or Cilium, clear CNI configs:

sudo rm -rf /etc/cni/net.d


Remove Kubernetes configs

sudo rm -rf ~/.kube
sudo rm -rf /etc/kubernetes


Restart container runtime (containerd in your case)

sudo systemctl restart containerd


(Optional) Flush iptables
If you want a really clean state:

sudo iptables -F && sudo iptables -t nat -F && sudo iptables -t mangle -F
sudo iptables -X


Re-initialize the cluster
Now you can run:

sudo kubeadm init --control-plane-endpoint <VIP_or_IP>:6443 --pod-network-cidr=10.244.0.0/16


(replace with your HA IP if you’re using a load balancer, otherwise use master IP)

Set up kubeconfig again

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

