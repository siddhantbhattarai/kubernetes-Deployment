Here’s a complete step-by-step documentation combining all the instructions for setting up a Kubernetes cluster using `kubeadm` on Azure virtual machines (both master and worker nodes).

---

# Kubernetes Cluster Setup Using `kubeadm` on Azure VMs

This guide outlines the steps to install a Kubernetes cluster with one master node and one or more worker nodes using `kubeadm`. We will use CRI-O as the container runtime and Calico for network management.

## Prerequisites:
- **Ubuntu OS** (Xenial or later) on both master and worker nodes.
- **Azure VMs** with 8 GB RAM (BS series) for both master and worker nodes.
- **Root (sudo) privileges** on both nodes.
- **Port 6443** opened for Kubernetes API communication.

## Prepare the Azure Environment
1. **Security Group Configuration**:
   - Make sure both VMs (master and worker) are in the same network (Azure VNet) or allow communication between them.
   - Open port 6443 on the security group to allow worker nodes to communicate with the master node.
     - In Azure, configure the Network Security Group (NSG) to allow inbound traffic on port 6443 (Kubernetes API).
  
---

## Steps for Both Master and Worker Nodes

### 1. Disable Swap

Disable swap on both master and worker nodes to ensure Kubernetes functions properly.

```bash
sudo swapoff -a
```

### 2. Load Kernel Modules

Create a `.conf` file to load the required kernel modules on both nodes at boot:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

Then manually load the modules:

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

### 3. Configure Sysctl Parameters

Configure `sysctl` parameters required for Kubernetes networking on both nodes:

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```

Apply these parameters without rebooting:

```bash
sudo sysctl --system
```

### 4. Install CRI-O Container Runtime

1. Install the required packages:

```bash
sudo apt-get update -y
sudo apt-get install -y software-properties-common curl apt-transport-https ca-certificates gpg
```

2. Add the CRI-O repository and install it:

```bash
sudo curl -fsSL https://pkgs.k8s.io/addons:/cri-o:/prerelease:/main/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/cri-o-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/cri-o-apt-keyring.gpg] https://pkgs.k8s.io/addons:/cri-o:/prerelease:/main/deb/ /" | sudo tee /etc/apt/sources.list.d/cri-o.list
sudo apt-get update -y
sudo apt-get install -y cri-o
```

3. Start and enable the CRI-O service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable crio --now
sudo systemctl start crio.service
```

### 5. Install Kubernetes (`kubeadm`, `kubelet`, `kubectl`)

1. Add the Kubernetes repository and install the required packages:

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update -y
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

2. Enable and start `kubelet`:

```bash
sudo systemctl enable --now kubelet
sudo systemctl start kubelet
```

---

## Steps for the Master Node

### 6. Pull Kubernetes Control Plane Images

Before initializing the master node, pull the necessary Kubernetes images:

```bash
sudo kubeadm config images pull
```

### 7. Initialize the Master Node

Run the following command to initialize the Kubernetes control plane on the master node:

```bash
sudo kubeadm init
```

After initialization, you will receive a join command with a token that the worker node will use to join the cluster. Keep this information for later.

### 8. Configure `kubectl` on the Master Node

Set up `kubectl` for the non-root user on the master node to manage the cluster:

```bash
mkdir -p "$HOME"/.kube
sudo cp -i /etc/kubernetes/admin.conf "$HOME"/.kube/config
sudo chown "$(id -u)":"$(id -g)" "$HOME"/.kube/config
```

### 9. Install Calico Network Plugin

Install Calico, which will enable network communication between the Kubernetes pods:

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/calico.yaml
```

### 10. Generate the Join Command for Worker Nodes

Create a new join token and print the join command for the worker node(s):

```bash
kubeadm token create --print-join-command
```

The command will look something like this:

```bash
kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

Copy this command and use it on the worker node(s).

---

## Steps for Worker Node(s)

### 11. Perform Pre-flight Checks

On each worker node, reset any previous Kubernetes setup to ensure a clean installation:

```bash
sudo kubeadm reset pre-flight checks
```

### 12. Join the Worker Node to the Master Node

On each worker node, use the join command received from the master node. Make sure to append `--v=5` for verbose output:

```bash
sudo kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash> --v=5
```

This command joins the worker node to the Kubernetes cluster.

---

## Verification

### 13. Verify the Cluster on the Master Node

After the worker node(s) join the cluster, verify the setup on the master node by running:

```bash
kubectl get nodes
```

You should see both the master and worker nodes in `Ready` status.

---

This completes the Kubernetes cluster setup using `kubeadm` with CRI-O on both master and worker nodes.
