# 🚀 Kubernetes Cluster Setup Using `kubeadm` on Azure VMs

Kubernetes is a powerful tool for container orchestration, making it easy to deploy, manage, and scale containerized applications. In this guide, we will set up a Kubernetes cluster using `kubeadm` on Azure VMs, with **CRI-O** as the container runtime and **Calico** as the network plugin.

## 🔍 Overview

### Tools You Will Use:
- **`kubeadm`**: A tool to bootstrap a Kubernetes cluster.
- **`kubelet`**: An agent that runs on each node and makes sure that containers are running as expected.
- **`kubectl`**: A command-line tool to interact with the Kubernetes cluster.

### Cluster Components:
- **Master Node**: Manages the Kubernetes control plane.
- **Worker Node(s)**: Where the containers (pods) will run.

---

## 🛠️ Prerequisites

Before you begin, ensure that the following prerequisites are met:

- **Operating System**: Ubuntu 16.04 (Xenial) or later on both master and worker nodes.
- **VM Resources**: Each VM (master and worker nodes) should have at least 8 GB of RAM (Azure B-series VMs are recommended).
- **Network Access**: Both VMs should be in the same Azure VNet, or you should configure them to communicate with each other.
- **Open Port 6443**: This port needs to be opened for API server communication between the master and worker nodes.
- **Root (sudo) privileges** on both nodes.

---

## 🏗️ Step 1: Prepare the Azure Environment

### 1.1 Configure Azure Network Security Group (NSG)

To allow proper communication between your nodes, ensure that both VMs are in the same **Azure VNet** or can communicate through their public IP addresses. Additionally, open **port 6443** in the Network Security Group (NSG) for inbound traffic, as this port is required for the Kubernetes API server to communicate with worker nodes.

You can configure this using Azure Portal or Azure CLI. For CLI users:

```bash
az network nsg rule create --nsg-name <your-nsg-name> --resource-group <your-resource-group> --name k8s-api-rule --protocol Tcp --priority 1000 --destination-port-ranges 6443 --access Allow
```

---

## ⚙️ Step 2: Setup on Both Master and Worker Nodes

For these steps, perform them **on both the master and worker nodes**.

### 2.1 Disable Swap

Kubernetes does not support running with swap enabled. Disabling swap ensures Kubernetes runs correctly.

```bash
sudo swapoff -a
```

To make this permanent, comment out or remove the swap line in `/etc/fstab`.

```bash
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

---

### 2.2 Load Kernel Modules

Kubernetes requires specific kernel modules to manage networking and container storage. These modules should be loaded both at boot and immediately.

1. **Create a config file to load kernel modules at boot:**

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

2. **Manually load the required modules:**

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

---

### 2.3 Configure Sysctl for Kubernetes Networking

Kubernetes networking relies on specific kernel parameters. These parameters need to be set up to enable proper traffic routing between pods.

1. **Create a sysctl config file:**

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```

2. **Apply the sysctl settings immediately without rebooting:**

```bash
sudo sysctl --system
```

---

### 2.4 Install CRI-O Container Runtime

Kubernetes uses container runtimes to run the actual containers. Here, we will use **CRI-O**, a lightweight container runtime optimized for Kubernetes.

#### Step-by-step Breakdown:

1. **Update your package list and install necessary dependencies:**

```bash
sudo apt-get update -y
sudo apt-get install -y software-properties-common curl apt-transport-https ca-certificates gpg
```

2. **Add the CRI-O repository GPG key to your system:**

```bash
sudo curl -fsSL https://pkgs.k8s.io/addons:/cri-o:/prerelease:/main/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/cri-o-apt-keyring.gpg
```

3. **Add the CRI-O repository to your system:**

```bash
echo "deb [signed-by=/etc/apt/keyrings/cri-o-apt-keyring.gpg] https://pkgs.k8s.io/addons:/cri-o:/prerelease:/main/deb/ /" | sudo tee /etc/apt/sources.list.d/cri-o.list
```

4. **Update your package list and install CRI-O:**

```bash
sudo apt-get update -y
sudo apt-get install -y cri-o
```

5. **Enable and start the CRI-O service:**

```bash
sudo systemctl daemon-reload
sudo systemctl enable crio --now
sudo systemctl start crio.service
```

---

### 2.5 Install Kubernetes (`kubeadm`, `kubelet`, `kubectl`)

These tools are necessary for setting up and managing the Kubernetes cluster:

- **`kubeadm`**: Bootstrap the cluster.
- **`kubelet`**: Ensure that containers are running on nodes.
- **`kubectl`**: Interact with the cluster.

#### Step-by-step Breakdown:

1. **Add the Kubernetes GPG key to your system:**

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

2. **Add the Kubernetes repository to your sources list:**

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

3. **Update your package list and install Kubernetes tools:**

```bash
sudo apt-get update -y
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

4. **Enable and start the `kubelet` service:**

```bash
sudo systemctl enable --now kubelet
sudo systemctl start kubelet
```

---

## 🖥️ Step 3: Configure the Master Node

Now, follow these steps **only on the master node**.

### 3.1 Pull Kubernetes Control Plane Images

Before initializing the control plane on the master node, Kubernetes images need to be pulled.

```bash
sudo kubeadm config images pull
```

---

### 3.2 Initialize the Master Node

Initialize the Kubernetes control plane on the master node using the following command:

```bash
sudo kubeadm init
```

After the initialization, you will be given a join command with a token. This token will be needed to join the worker nodes to the master.

---

### 3.3 Configure `kubectl` for the Master Node

After the master node is initialized, you need to set up `kubectl` to interact with the cluster from the master node.

```bash
mkdir -p "$HOME"/.kube
sudo cp -i /etc/kubernetes/admin.conf "$HOME"/.kube/config
sudo chown "$(id -u)":"$(id -g)" "$HOME"/.kube/config
```

---

### 3.4 Install Calico Network Plugin

Kubernetes needs a networking plugin to manage pod communication. Here, we will install **Calico** as the network plugin.

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/calico.yaml
```

---

### 3.5 Generate the Join Command for Worker Nodes

If you didn’t save the initial join token, you can generate a new one with the following command:

```bash
kubeadm token create --print-join-command
```

Copy this join command as it will be needed on the worker nodes.

---

## 🖥️ Step 4: Configure the Worker Node(s)

Perform these steps **on each worker node**.

### 4.1 Reset Pre-flight Checks

Before joining the worker node to the master, reset any previous Kubernetes setup to ensure a clean installation:

```bash
sudo kubeadm reset pre-flight checks
```

---

### 4.2 Join the Worker Node to the Master Node

Use the join command provided by the master node to add the worker node to the cluster:

```bash
sudo kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash> --v=5
```

---

## 🚀 Conclusion

You’ve now successfully set up a Kubernetes cluster using `kubeadm` on Azure VMs with CRI-O and Calico for networking. You can use `kubectl get nodes` on the master node to verify that your worker nodes have successfully joined the cluster.

This basic setup is ready for deploying applications, scaling your services, and exploring Kubernetes further!

