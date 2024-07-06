# Kubernetes
## Creating a K8s cluser by installing Kubeadm
### Prerequisites
* Create 2 ec2 instances(ubuntu) each with 2 vCPUs and 4GB RAM (t2.medium)
### Step-1: Install Docker on both nodes
* Follow below commands to install docker
```bash
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install docker-ce -y
sudo systemctl status docker
sudo usermod -aG docker ${USER}
exit 
# log into the server
docker info
docker --version
```
![preview](images/k8s1.png)

### Step-2: Installing Kubeadm, Kubelet,Kubectl
* Follow the offical documentaion for latest version [kubeadm install](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
* I install 1.30 version. Mentioning commands below.
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
```
 
  ![preview](images/k8s2.png)

### Step-3: Configuring CRI runtime
* We can get the updated version in the offical page [ReferHere](https://github.com/Mirantis/cri-dockerd/releases) for packages.
* Followed below commands.
```bash
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.14/cri-dockerd_0.3.14.3-0.ubuntu-jammy_amd64.deb
sudo dpkg -i cri-dockerd_0.3.14.3-0.ubuntu-jammy_amd64.deb
```
  ![preview](images/k8s3.png)

### Step-4: Initialize the Kubernetes Master
* Run below commands on master node.
```bash
sudo -i
kubeadm init --cri-socket unix:///var/run/cri-dockerd.sock
```
* After that you will get few instructions to follow and copy the kubeadm join token command
```bash
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.31.87.114:6443 --token 1ak0ki.0cya6pxkgmvh5yd9 --discovery-token-ca-cert-hash sha256:52199f3c6bf6ef3a6085e0bee095efb5c624555877cd8a32c0cb251c345d94a5  --cri-socket unix:///var/run/cri-dockerd.sock
```

 ![preview](images/k8s4.png)
 ![preview](images/k8s5.png)

### Step-7: Join Worker Nodes to the Cluster
* On each worker node, use the kubeadm join command provided at the end of the kubeadm init output on the master node. It will look something like this:
```bash
sudo -i
kubeadm join 172.31.87.114:6443 --token 1ak0ki.0cya6pxkgmvh5yd9 --discovery-token-ca-cert-hash sha256:52199f3c6bf6ef3a6085e0bee095efb5c624555877cd8a32c0cb251c345d94a5 --cri-socket unix:///var/run/cri-dockerd.sock
```

  ![preview](images/k8s6.png)

### Step-6:  Install a Pod Network Add-on(on master)
* You need a pod network add-on to enable communication between pods. Install fannel
```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```
* Before and after install fannel in master node
  
  ![preview](images/k8s7.png)