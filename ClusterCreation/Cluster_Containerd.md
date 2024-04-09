## Azure Cli commands to create RG and 2 VM's (one Master and one Worker):

# Create ResourceGroup:
```bash
az group create --name k8s-rg --location eastus
```

# Create Master Node:
 ```bash
 az vm create --resource-group k8s-rg --admin-username azureuser  --authentication-type password --name master --image Ubuntu2204 --public-ip-address "masterpub" --size  Standard_D2s_v3
 ```

# Create Worker Node:
```bash
az vm create --resource-group k8s-rg --admin-username azureuser  --authentication-type password --name worker-01 --image Ubuntu2204 --public-ip-address "worker01-pub" --size  Standard_D2s_v3
```

# Install Packages ( The following steps must be performed on all two nodes):
```bash
sudo apt update
sudo apt upgrade
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

# Add Kernel Parameters
```bash
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
```

# Configure the critical kernel parameters for Kubernetes using the following:
```bash
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```
# Reload the changes:
```bash
sudo sysctl --system
```
# Install Containerd Runtime:
```bash
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
```

# Enable the Docker repository:
```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```
# Update the package list and install containerd:
```bash
sudo apt update
sudo apt install -y containerd.io
```
# Configure containerd to start using systemd as cgroup:
```bash
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```

# Restart and enable the containerd service:
```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```
# Installing kubeadm, kubelet and kubectl:
```bash
sudo apt-get update
```
# apt-transport-https may be a dummy package; if so, you can skip that package:
```bash
sudo apt-get install -y apt-transport-https ca-certificates curl
```
# Download the public signing key for the Kubernetes package repositories. The same signing key is used for all repositories so you can disregard the version in the URL:
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.27/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```


# Add the appropriate Kubernetes apt repository. Please note that this repository have packages only for Kubernetes 1.27
```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.27/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

# Update the apt package index, install kubelet, kubeadm and kubectl, and pin their version:
```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```


# Initialize Kubernetes Cluster with Kubeadm (master node):

sudo kubeadm init

# Set kubectl access:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

# Test access to cluster:
kubectl get nodes

# Join the Worker Nodes to the Cluster(run this cmd on master node)

kubeadm token create --print-join-command

example:- kubeadm join 10.0.0.4:6443 --token 5b74ci.fy1p46z095xizzmr --discovery-token-ca-cert-hash sha256:984bfad396cde3ad762ff416b74e3f0c9a1fb164efd5c4db57ec63de1e869f0c


# In the worker nodes, paste the kubeadm join command to join the cluster. Use sudo to run it as root:

kubeadm join 10.0.0.4:6443 --token 5b74ci.fy1p46z095xizzmr --discovery-token-ca-cert-hash sha256:984bfad396cde3ad762ff416b74e3f0c9a1fb164efd5c4db57ec63de1e869f0c

# Install Kubernetes Network Plugin (master node):

kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml


# Check status of the Cluster(master node):
kubectl get nodes