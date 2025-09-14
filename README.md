# Install-Kubernetes-with-kubeadm
## Architecture 
We want to set up a 5-node Kubernetes cluster on CentOS 9, with:

2 Master nodes (control plane, for HA) & 3 Worker nodes Using kubeadm as the bootstrap tool.

1 HAProxy to configure HAProxy on an external node . 

OS for all nodes : CentOS 9 

## Steps :

### HAproxy 
Set up HAProxy on CentOS to load balance traffic across 3 Kubernetes master nodes .

1-  Install VM with HAproxy " lB" configuration using vagrant file with IP : 192.168.56.14

sudo yum install -y haproxy

sudo systemctl enable haproxy

sudo systemctl start haproxy

Edit the HAProxy config file:

2- sudo vi /etc/haproxy/haproxy.cfg

frontend kubernetes-frontend

    bind *:6443

    mode tcp

    default_backend kubernetes-backend

backend kubernetes-backend

    mode tcp

    balance roundrobin

    option tcp-check

    server master01 192.168.56.15:6443 

    server master02 192.168.56.16:6443 

    server master03 192.168.56.17:6443 


<img width="650" height="214" alt="image" src="https://github.com/user-attachments/assets/dccd053a-1378-4f79-8dc4-8c4f05f362a3" />

Hint : Hashing all the belwo lines to be not make the conflict with the new configuration which adding above : 

<img width="742" height="580" alt="image" src="https://github.com/user-attachments/assets/964d92ba-487f-416e-ace4-81a37604e881" />

<img width="913" height="570" alt="image" src="https://github.com/user-attachments/assets/9605dc94-51bf-489e-82a6-3d425bd51865" />


3- Validate config 

sudo haproxy -c -f /etc/haproxy/haproxy.cfg

4- sudo systemctl restart haproxy

<img width="650" height="214" alt="image" src="https://github.com/user-attachments/assets/ea1a207e-ce09-4c33-b879-c85f5548d3c6" />

•	Check that port 6443 is open:

•	ss -ntlp | grep 6443

At this point, HAProxy should be distributing Kubernetes API requests to your 3 masters

## K8s betweeen nodes 

Hostnames set (master1, master2, worker1, worker2, worker3)

Proper /etc/hosts resolution between nodes using :

 $ vagrant plugin install vagrant-hostmanager

1- update OS 

sudo yum install -y 

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

### Now can inialize the Cluster  :

## on master01 " 192.168.56.15"

1- kubeadm init --control-plane-endpoint "192.168.56.14:6443" --upload-certs --pod-network-cidr=10.244.0.0/16

This ensures ETCD certs include HA IP and ETCD will listen on HA IP for peers.


and Verify etcd is listening 

ss -tlnp | grep 2379

nc -zv 192.168.56.14 2379

shoudl appear ==> 2379 open on HA IP " 192.168.56.14"

<img width="975" height="558" alt="image" src="https://github.com/user-attachments/assets/aa8a57ea-42f3-438f-af56-6b0dde2aa9a1" />

2- Install Flannel:

kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

3- Copy your admin kubeconfig:

mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config

## on Master02 

kubeadm join 192.168.56.14:6443 --token dcbil8.9i3jmt2trcrq6b4q --discovery-token-ca-cert-hash sha256:ac5a11b7426958fc6000d4c41f18803f7b8c4aa2f2ccb08e73071b8e1f662575 --control-plane   --certificate-key 0b84f3882a2419d7f6c62946249d8e40ac89a364a190b22384bd66a05acacf60   --apiserver-advertise-address=192.168.56.16

should be add apiserver with the ip of master node which will be joined to be using VIP of HAproxy .

## on Master03 

kubeadm join 192.168.56.14:6443 --token dcbil8.9i3jmt2trcrq6b4q --discovery-token-ca-cert-hash sha256:ac5a11b7426958fc6000d4c41f18803f7b8c4aa2f2ccb08e73071b8e1f662575 --control-plane   --certificate-key 0b84f3882a2419d7f6c62946249d8e40ac89a364a190b22384bd66a05acacf60   --apiserver-advertise-address=192.168.56.17

## on worker01  & worker02 

kubeadm join 192.168.56.14:6443 --token dcbil8.9i3jmt2trcrq6b4q --discovery-token-ca-cert-hash sha256:ac5a11b7426958fc6000d4c41f18803f7b8c4aa2f2ccb08e73071b8e1f662575


## Verify HA cluster : 

kubectl get nodes

kubectl get pods -n kube-system -o wide

All control-plane pods (etcd, kube-apiserver, controller-manager, scheduler) should be Running on both masters.


<img width="975" height="201" alt="image" src="https://github.com/user-attachments/assets/7ff656c1-ab15-4482-a4c5-78e3b6f09313" />

<img width="975" height="201" alt="image" src="https://github.com/user-attachments/assets/371f5cf0-17b0-46d1-8785-29d42e3f499c" />

<img width="975" height="389" alt="image" src="https://github.com/user-attachments/assets/48202fbe-b02e-4018-9ab8-41988a2b70a9" />

### Testing pods and create container nginx for testing and found the pods take from CIDR network pod-network-cidr=10.244.0.0/16 as belwo  : 


<img width="975" height="380" alt="image" src="https://github.com/user-attachments/assets/a4a21f31-f807-4723-ba86-dc1de35b1186" />


### when add the second master and can not connect with etcd of the first master , we can fixed it by : 

==> Resetting Kubernetes Cluster on the master .

Run kubeadm reset

sudo kubeadm reset -f

==> This will remove the cluster configuration, kubelet state, and certificates.

==> Clean up CNI networking " Flannel "

sudo rm -rf /etc/cni/net.d

==> Remove Kubernetes configs

sudo rm -rf ~/.kube

sudo rm -rf /etc/kubernetes

==> Restart container runtime 

sudo systemctl restart containerd

and after that 

### Re-initialize the cluster on master01 

1- sudo kubeadm init --control-plane-endpoint <VIP_or_IP>:6443 --pod-network-cidr=10.244.0.0/16

(replace with your HA IP if you’re using a load balancer, otherwise use master IP)

2- Set up kubeconfig again

mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config


###  issue with private ip change it from NAT to private :

How to add the private IP

Edit the file:

on workers 

sudo nano /var/lib/kubelet/kubeadm-flags.env


Change the line to include --node-ip=<your-private-ip> (e.g. 192.168.56.18 & 192.168.56.19):

KUBELET_KUBEADM_ARGS="--container-runtime-endpoint=unix:///var/run/cri-dockerd.sock --pod-infra-container-image=registry.k8s.io/pause:3.9 --node-ip=192.168.56.18"

sudo systemctl daemon-reexec

sudo systemctl daemon-reload

sudo systemctl restart kubelet

<img width="1448" height="128" alt="image" src="https://github.com/user-attachments/assets/192fa027-9469-4d29-bf9d-e4a04c2bf881" />

