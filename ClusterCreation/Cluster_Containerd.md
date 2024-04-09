## Azure Cli commands to create RG and 2 VM's (one Master and one Worker):

# Create ResourceGroup:
```bash
az group create --name k8s-rg --location eastus
```

# Create Master Node:
 ```bash
 az vm create --resource-group k8s-rg --admin-username azureuser  --authentication-type password --name master --image Ubuntu2204 --public-ip-address "" --size  Standard_D2s_v3
 ```

# Create Worker Node:
```bash
az vm create --resource-group k8s-rg --admin-username azureuser  --authentication-type password --name worker-01 --image Ubuntu2204 --public-ip-address "" --size  Standard_D2s_v3
```

# Install Packages ( The following steps must be performed on all two nodes):
```bash
sudo apt update
sudo apt upgrade
sudo swapoff -a(disable swapoff)
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


