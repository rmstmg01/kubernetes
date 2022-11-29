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

From the output above, we see that control-plane has been initialize successfully. In the output, we can find the set of commands for interacting the cluster and also the command for worker node to join the cluster.

So, to start interacting with the cluster, run following commands from the master node:
```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Now, run following kubectl commands to view cluster and node status:
```
$ kubectl cluster-info
$ kubectl get nodes
```

![13](https://user-images.githubusercontent.com/11027110/204589992-7d167fdc-f383-4162-8077-3aa0234f6eb8.jpg)

![not](https://user-images.githubusercontent.com/11027110/204591113-362159f3-bb73-4132-a774-72a8d6b8702b.png)

Join both the worker nodes to the cluster, we need to run the command as below:
```
sudo kubeadm join k8smaster.ramesht.com.np:6443 --token 1so3re.alopyer4tz5329ut   --discovery-token-ca-cert-hash sha256:d69eed5765929759f33082f08c930b7f975027fe812a0f74b1300b0ff4859e1b
```
Output from both the worker nodes are shown in the screenshot below:
![16](https://user-images.githubusercontent.com/11027110/204592060-77285176-d594-4c76-88d4-79596c709801.jpg)

![17](https://user-images.githubusercontent.com/11027110/204592114-84f66f7d-f3c1-4497-adb4-e797a56f2f79.jpg)

Check the nodes status from master node using kubectl command:
```
$ kubectl get nodes
```
![18](https://user-images.githubusercontent.com/11027110/204592750-9036526f-cecb-416a-839d-eeea0a5232be.jpg)

We can see the nodes status is ‘NotReady’, so to make it active and running, we must install CNI (Container Network Interface) or network add-on plugins like Calico, Flannel and Weave-net.

### 6. Install Calico Pod Network Add-on
Run curl and kubectl command to install Calico network plugin from the master node:
```
$ curl https://projectcalico.docs.tigera.io/manifests/calico.yaml -O
$ kubectl apply -f calico.yaml
```
![19](https://user-images.githubusercontent.com/11027110/204594515-1a7b4f0a-9b3f-470d-b7d1-beac87d58289.jpg)

Verify the status of pods in kube-system namespace:
```
$ kubectl get pods -n kube-system
```
![20](https://user-images.githubusercontent.com/11027110/204594703-c48677f3-4b5e-4be4-9094-7c4e2330e78e.jpg)

Now, check the nodes status again with following command:
```
$ kubectl get nodes
```
![22](https://user-images.githubusercontent.com/11027110/204597153-1eb544aa-c261-4f79-baec-8a8969e25863.jpg)

Now, we see that nodes are active and we can say that our Kubernetes cluster is functional.
### 7. Test Kubernetes Installation
To test Kubernetes installation, we are trying to deploy nginx based application and try to access it.
```
$ kubectl create deployment nginx-app --image=nginx --replicas=2
```
Check the status of nginx-app deployment
```
$ kubectl get deployment nginx-app
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
nginx-app   2/2     2            2           72s
$
```
Expose the deployment as NodePort,
```
$ kubectl expose deployment nginx-app --type=NodePort --port=80
service/nginx-app exposed
$
```
Run following commands to view service status
```
$ kubectl get svc nginx-app
$ kubectl describe svc nginx-app
```
Output of above commands:

![25](https://user-images.githubusercontent.com/11027110/204612455-1f1ad0e5-7575-492e-8b70-66a208892058.jpg)

You can access your nginx based application browsing worker node hostname/ip address and the port as below:

![26](https://user-images.githubusercontent.com/11027110/204612889-dbaa065f-aea1-4da3-a732-4af7df9253a9.jpg)

![27](https://user-images.githubusercontent.com/11027110/204612940-cb87bcfa-0d21-43f0-b2f5-35a769079557.jpg)

We can see the default web page of nginx accessible from a browser which confirms that we have successfully deployed the web application on kubernetes cluster.
