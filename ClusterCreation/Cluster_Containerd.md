## Azure Cli commands to create RG and 2 VM's (one Master and one Worker)

# Create RG
```bash
az group create --name k8s-rg --location eastus
```

# Create Master Node
 ```bash
 az vm create --resource-group k8s-rg --admin-username azureuser  --authentication-type password --name master --image Ubuntu2204 --public-ip-address "" --size  Standard_D2s_v3
 ```

# Create Worker Node
```bash
az vm create --resource-group k8s-rg --admin-username azureuser  --authentication-type password --name worker-01 --image Ubuntu2204 --public-ip-address "" --size  Standard_D2s_v3
```

# Install Packages ( The following steps must be performed on all two nodes)

sudo apt update
sudo apt upgrade
sudo swapoff -a(disable swapoff)
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Add Kernel Parameters

sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

# Configure the critical kernel parameters for Kubernetes using the following

sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# reload the changes 

sudo sysctl --system

# Install Containerd Runtime

sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates


# Enable the Docker repository

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# Update the package list and install containerd

sudo apt update
sudo apt install -y containerd.io

# Configure containerd to start using systemd as cgroup

containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

# Restart and enable the containerd service

sudo systemctl restart containerd
sudo systemctl enable containerd