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
Master Node:    – k8smaster.ramesht.com.np <br>
Worker Node 1:  – k8sworker1.ramesht.com.np <br>
Worker Node 2:  – k8sworker2.ramesht.com.np <br>
![1](https://user-images.githubusercontent.com/11027110/204564852-2f3dd4e6-6fbc-433f-8e40-5087fb283333.jpg)

### Installation Steps

###### 1. Set hostname and add entries in the hosts file
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

<Master node's IP >   k8smaster.ramesht.com.np k8smaster <br>
<Worker node1 IP>     k8sworker1.ramesht.com.np k8sworker1 <br>
<Worker node2 IP>     k8sworker2.ramesht.com.np k8sworker1 <br>
###### 2. Disable swap & add kernel settings
Run following commands to disable swap:
```
$ sudo swapoff -a
$ sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
Load the following kernel modules on all the nodes:
```
$ sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
$ sudo modprobe overlay
$ sudo modprobe br_netfilter
```





