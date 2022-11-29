### Kubernetes
Kubernetes is a free and open-source container orchestration tool which is also known as k8s. With the help of Kubernetes, we can automate the deployment, scaling and management of containerized applications.
A Kubernetes cluster consists of master and worker nodes where application workload is deployed on worker nodes and master nodes are used to manage worker nodes and pods in the cluster.

In this tutorial, we are using one master node and two worker nodes. Following are system requirements on each node,

Minimal install Ubuntu 22.04 <br>
Minimum 2GB RAM or more <br>
Minimum 2 CPU cores / or 2 vCPU <br>
20 GB free disk space on /var or more <br>
sudo user with root privilege <br>

We are going to setup lab environment on Linode as below:
### Lab Setup
```
Master Node:    – k8smaster.ramesht.com.np 
Worker Node 1:  – k8sworker1.ramesht.com.np 
Worker Node 2:  – k8sworker2.ramesht.com.np 
```
![1](https://user-images.githubusercontent.com/11027110/204564852-2f3dd4e6-6fbc-433f-8e40-5087fb283333.jpg)

### Installation Steps

### 1. Set hostname and add entries in the hosts file
Login to master node and set hostname using hostnamectl command as below:
```
$ sudo hostnamectl set-hostname "k8smaster.ramesht.com.np"
$ exec bash
```
Login to worker nodes and set hostnames as below:
```
$ sudo hostnamectl set-hostname "k8sworker1.ramesht.com.np"
$ sudo hostnamectl set-hostname "k8sworker2.ramesht.com.np"
$ exec bash
```
Add the following entries in /etc/hosts file on each node
```
Master node's IP    k8smaster.ramesht.com.np  k8smaster 
Worker node1 IP     k8sworker1.ramesht.com.np k8sworker1 
Worker node2 IP     k8sworker2.ramesht.com.np k8sworker1 
```

### 2. Disable swap & add kernel settings
Run following commands to disable swap:
```
$ sudo swapoff -a
$ sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
Load the following kernel modules on all nodes:
```
$ sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
$ sudo modprobe overlay
$ sudo modprobe br_netfilter
```

Set following Kernel parameters for Kubernetes executing following command:
```
$ sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF 
```
Reload the above changes, running following command:
```
$ sudo sysctl --system
```

### 3. Install containerd run time
In this tutorial, we are going to use containerd run time for our Kubernetes cluster. So, we need to install dependencies before installing containerd:
```
$ sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
```
Now, we need to enable docker repository:

```
$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```
Now, run following command to install containerd: 
```
$ sudo apt update
$ sudo apt install -y containerd.io
```
Configure containerd to start it using systemd as cgroup:
```
$ containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
$ sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```
Restart and enable containerd service with following command:
```
$ sudo systemctl restart containerd
$ sudo systemctl enable containerd
```
### 4. Add apt repository for Kubernetes
Execute following commands to add apt repository for Kubernetes:
```
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
$ sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```
###### Note: Please check latest kubernetes repository from here : https://packages.cloud.google.com/apt/dists

### 5. Install Kubernetes components Kubectl, kubeadm & kubelet
Install Kubernetes components like kubectl, kubelet and Kubeadm utility on all the nodes running following commands:
```
$ sudo apt update
$ sudo apt install -y kubelet kubeadm kubectl
$ sudo apt-mark hold kubelet kubeadm kubectl
```
### 6. Initialize Kubernetes cluster with Kubeadm command
Now, we are all set to initialize Kubernetes cluster. Run the following Kubeadm command from the master node only:
```
$ sudo kubeadm init --control-plane-endpoint=k8smaster.ramesht.com.np
```
![11](https://user-images.githubusercontent.com/11027110/204588706-24e0a162-0fb9-4f34-a0c6-3bda65444f12.jpg)


