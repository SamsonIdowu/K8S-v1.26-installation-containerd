# Kubernetes V1.26 Installation with containerd container engine - Multi Node Cluster setup


```console
Step a : Create 2 VMs with Ubuntu 20.04 LTS OS. VM1: k8s-controlplane, VM2: k8s-workernode-1

$$ On k8s-controlplane node & workernodes  - perform below steps
Step b : containerd-installation
Step c : kubelet,kubectl,kubeadm installation
Step d : Reboot the VM

## Control Plane steps
Step e : Run kubeadm init and setup the control plane

$$ On k8s-workernode-1 - perform below steps
Step f : containerd-installation
Step g : kubelet,kubectl,kubeadm installation
Step h : Reboot the VM
Step i : Run kubeadm join command

```
# On k8s-controlplane & K8s-workernodes : Until step d
## Step b : containerd-installation
## How to install Containerd on Ubuntu OS

```console
Step 1 : Download & unpack containerd package
Step 2 : Install runc
Step 3 : Download & install CNI plugins
Step 4 : Configure containerd
Step 5 : Start containerd service
```

### Step 1 : Download & unpack containerd package

Containerd versions can be found in this location :  https://github.com/containerd/containerd/releases

### Download :
```bash
wget https://github.com/containerd/containerd/releases/download/v1.6.14/containerd-1.6.14-linux-amd64.tar.gz
```
### Unpack : 
```bash
sudo tar Cxzvf /usr/local containerd-1.6.14-linux-amd64.tar.gz
```

### Step 2 : Install runc

Runc is a standardized runtime for spawning and running containers on Linux according to the OCI specification
```bash
wget https://github.com/opencontainers/runc/releases/download/v1.1.3/runc.amd64
sudo install -m 755 runc.amd64 /usr/local/sbin/runc
```

### Step 3: Download and install CNI plugins :

```bash
wget https://github.com/containernetworking/plugins/releases/download/v1.1.1/cni-plugins-linux-amd64-v1.1.1.tgz
sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.1.1.tgz
```

### Step 4: Configure containerd

#### Create a containerd directory for the configuration file
#### config.toml is the default configuration file for containerd
#### Enable systemd group . Use sed command to change the parameter in config.toml instead of using vi editor 
####  Convert containerd into service
```bash
sudo mkdir /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo curl -L https://raw.githubusercontent.com/containerd/containerd/main/containerd.service -o /etc/systemd/system/containerd.service
```

### Step 5: Start containerd service
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now containerd
sudo systemctl status containerd    
```
##### To come out of the prompt , press q

## Step c : kubelet,kubectl,kubeadm installation

```bash
sudo apt update
sudo apt-get install -y apt-transport-https ca-certificates curl
curl -L https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add
sudo touch /etc/apt/sources.list.d/kubernetes.list
sudo chmod 666 /etc/apt/sources.list.d/kubernetes.list
sudo echo deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://apt.kubernetes.io/ kubernetes-xenial main | tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

  Validation :
service kubelet status
kubeadm   # this command should produce an output . Incase if it says command not found, check the installation steps again
kubectl   # this command should produce an output . Incase if it says command not found, check the installation steps again
```

## Step d: Reboot the VM

### NOTE: Before running step e, enable IP forwarding by changing the value of *ip_forward* parameter from 0 to 1 and enable the *bridge-nf-call-iptables* using the below commands:

```bash
sudo sysctl net.ipv4.ip_forward=1   # enables IP forwarding.
sysctl net.ipv4.ip_forward          # this command verifies that the parameter is now set to 1.

sudo modprobe br_netfilter                          # loads the necessary kernel modules.
sudo sysctl net.bridge.bridge-nf-call-iptables=1    # sets the "bridge-nf-call-iptables" parameter to 1.
cat /proc/sys/net/bridge/bridge-nf-call-iptables    # this command verifies that the parameter is now set to 1.
```
*These settings are required for kubernetes networking to work properly.*

## Step e: Run kubeadm init and setup the control plane
```bash
sudo kubeadm init
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl get nodes
kubectl get pods --all-namespaces
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
kubectl get pods --all-namespaces
kubectl get nodes
```


## On k8s-wn-1 node - perform below steps
 ``` console
Step f : containerd-installation  [ Follow above containerd installation steps ]
Step g : kubelet,kubectl,kubeadm installation  [ Follow above steps - BUT PLZ DO NOT RUN KUBEADM INIT ON WORKERNODES . ITS ONLY ON MASTER NODE ]
Step h : Reboot the VM
Step i : Run kubeadm join command  [ Get the join command from master node ]
```
#### INSTALLATION COMPLETE

To join nodes to the cluster subsequently, use the following [resource](https://computingforgeeks.com/join-new-kubernetes-worker-node-to-existing-cluster/).
