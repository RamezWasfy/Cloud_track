Build your practice Cluster (not included in exam)
---------------------------------------------------

- 1 master node. (Kubernetes API) 
- 2 Worker nodes. (To put containers on)

Servers size = (2 CPU - 4GB RAM)
- Kube-master 
- Kube-worker1
- Kube-worker2 

Commands to run on the 3 servers
--------------------------------
# Step 1 - Kubernetes Installation

In this first step, we will prepare those 3 servers for Kubernetes installation, so run all commands on the master and node servers.

We will prepare all servers for Kubernetes installation by changing the existing configuration on servers, and also installing some packages, including docker-ce and kubernetes itself.

* Configure Hosts

$ vim /etc/hosts

10.0.15.10      Kube-master
10.0.15.21      Kube-worker1
10.0.15.22      Kube-worker2

- Disable SELinux

$ setenforce 0
$ sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux

- Enable br_netfilter Kernel Module

The br_netfilter module is required for kubernetes installation. Enable this kernel module so that the packets traversing the bridge are processed by iptables for filtering and for port forwarding, and the kubernetes pods across the cluster can communicate with each other.

$ modprobe br_netfilter
$ echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables

- Disable SWAP

Disable SWAP for kubernetes installation by running the following commands.

$ swapoff -a

- Install Docker CE
- Install the package dependencies for docker-ce.

$ yum install -y yum-utils device-mapper-persistent-data lvm2

- Add the docker repository to the system and install docker-ce using the yum command.

$ yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
$ yum install -y docker-ce

- Install Kubernetes
- Add the kubernetes repository to the centos 7 system by running the following command.

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

- Now install the kubernetes packages kubeadm, kubelet, and kubectl using the yum command below.

$ yum install -y kubelet kubeadm kubectl

- After the installation is complete, restart all those servers.

$ sudo reboot

- Log in again to the server and start the services, docker and kubelet.

$ systemctl start docker && systemctl enable docker
$ systemctl start kubelet && systemctl enable kubelet

- Change the cgroup-driver
- We need to make sure the docker-ce and kubernetes are using same 'cgroup'.
- Check docker cgroup using the docker info command.

$ docker info | grep -i cgroup

- And you see the docker is using 'cgroupfs' as a cgroup-driver.
- Now run the command below to change the kuberetes cgroup-driver to 'cgroupfs'.

$ sed -i 's/cgroup-driver=systemd/cgroup-driver=cgroupfs/g' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

- Reload the systemd system and restart the kubelet service.

$ systemctl daemon-reload
$ systemctl restart kubelet


# Step 2 - Kubernetes Cluster Initialization

In this step, we will initialize the kubernetes master cluster configuration.
Move the shell to the master server 'kube-master' and run the command below to set up the kubernetes master.

$ kubeadm init --apiserver-advertise-address=10.0.15.10 --pod-network-cidr=10.244.0.0/16

- Note:

--apiserver-advertise-address = determines which IP address Kubernetes should advertise its API server on.

--pod-network-cidr = specify the range of IP addresses for the pod network. We're using the 'flannel' virtual network. If you want to use another pod network such as weave-net or calico, change the range IP address.


- Note:

Copy the 'kubeadm join ... ... ...' command to your text editor. The command will be used to register new nodes to the kubernetes cluster.

Now in order to use Kubernetes, we need to run some commands as on the result.

Create new '.kube' configuration directory and copy the configuration 'admin.conf'.

$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

- Next, deploy the flannel network to the kubernetes cluster using the kubectl command.

$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

- The flannel network has been deployed to the Kubernetes cluster.
- Wait for a minute and then check kubernetes node and pods using commands below.

$ kubectl get nodes
$ kubectl get pods --all-namespaces

- And you will get the 'k8s-master' node is running as a 'master' cluster with status 'ready', and you will get all pods that are needed for the cluster, including the 'kube-flannel-ds' for network pod configuration.
- Make sure all kube-system pods status is 'running'.
- Kubernetes cluster master initialization and configuration has been completed.

# Step 3 - Adding node01 and node02 to the Cluster

- Connect to the node01 server and run the kubeadm join command as we copied on the top.

$ kubeadm join 10.0.15.10:6443 --token vzau5v.vjiqyxq26lzsf28e --discovery-token-ca-cert-hash sha256:e6d046ba34ee03e7d55e1f5ac6d2de09fd6d7e6959d16782ef0778794b94c61e

- Connect to the node02 server and run the kubeadm join command as we copied on the top.

$ kubeadm join 10.0.15.10:6443 --token vzau5v.vjiqyxq26lzsf28e --discovery-token-ca-cert-hash sha256:e6d046ba34ee03e7d55e1f5ac6d2de09fd6d7e6959d16782ef0778794b94c61e

- Wait for some minutes and back to the 'kube-master' master cluster server check the nodes and pods using the following command.

$ kubectl get nodes
$ kubectl get pods --all-namespaces

- Now you will get node01 and node02 has been added to the cluster with status 'ready'.
- node01 and node02 have been added to the kubernetes cluster.

# Step 4 - Testing Create First Pod

- In this step, we will do a test by deploying the Nginx pod to the kubernetes cluster. A pod is a group of one or more containers with shared storage and network that runs under Kubernetes. A Pod contains one or more containers, such as Docker container.
- Login to the 'kube-master' server and create new deployment named 'nginx' using the kubectl command.

$ kubectl create deployment nginx --image=nginx
- To see details of the 'nginx' deployment sepcification, run the following command.

$ kubectl describe deployment nginx

- And you will get the nginx pod deployment specification.
- Next, we will expose the nginx pod accessible via the internet. And we need to create new service NodePort for this.
- Run the kubectl command below.

$ kubectl create service nodeport nginx --tcp=80:80

- Make sure there is no error. Now check the nginx service nodeport and IP using the kubectl command below.

$ kubectl get pods
$ kubectl get svc

- Now you will get the nginx pod is now running under cluster IP address '10.160.60.38' port 80, and the node main IP address '10.0.15.x' on port '30691'.

- From the 'kube-master' server run the curl command below.

$ curl node01:30691
$ curl node02:30691

- The Nginx Pod has now been deployed under the Kubernetes cluster and it's accessible via the internet.
- Now access from the web browser.

http://10.0.15.10:30691/ 

- And you will get the Nginx default page.

- On the node02 server - http://10.0.15.11:30691/



Reference: https://www.howtoforge.com/tutorial/centos-kubernetes-docker-cluster/