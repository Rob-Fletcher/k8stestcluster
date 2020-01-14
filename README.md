# k8stestcluster

## 1. Install [Docker CE for Ubuntu 18.04](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

add the esri user to the docker group
`sudo usermod -aG docker $USER`

configure docker to start on book
`sudo systemctl enable docker`

## 2. Install kubeadm, kubelet, kubectl and initialize master.

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

> Swap disabled. You MUST disable swap in order for the kubelet to work properly.
`sudo swapoff -a`

`sudo vi /etc/fstab`
then comment out the first line and save

> [Certain ports are open on your machines.](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports)

Control plane 
```
sudo ufw allow 6443/tcp
sudo ufw allow 2379:2380/tcp
sudo ufw allow 10250:10252/tcp
```

worker nodes
```
sudo ufw allow 10250/tcp
sudo ufw allow 30000:32767/tcp
```

Install kubeadm, kubelet, kubectl
```
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

Use flannel as the pod network. 
```
sudo sysctl net.bridge.bridge-nf-call-iptables=1`
`sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

Make kubectl work for your non-root user
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
`kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml`

test: `kubectl get pods --all-namespaces` look for coreDNS status

copy the kube config to your local machine:
`scp esri@k8cluster1:.kube/config ~/.kube/`


Repeat steps to install Docker and kubeadm on each worker, then:
on master:
`kubeadm token create --print-join-command`

on worker
`sudo kubeadm join 10.49.53.69:6443 --token <the token>     --discovery-token-ca-cert-hash sha256:<the hash>`

bonus notes:

find ubuntu version
`lsb_release -a`

is nftables installed?
look in /usr/sbin
`apt list --installed | grep nftables`

sudo!! (repeats last command with sudo prepended)

ctrl+A goes to beginning of line
ctrl+E goes to end of line
