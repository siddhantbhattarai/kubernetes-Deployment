Here’s a step-by-step guide for setting up a Kubernetes cluster using `kubeadm` on Azure VMs with one master node and one worker node. 
### Prerequisites:
- Ubuntu OS (Xenial or later) on both nodes.
- Azure VMs with 8GB RAM, BS series.
- Root (sudo) privileges.
- Open port 6443 (Kubernetes API server port) between the nodes.
- Swap is disabled on both machines.

### Step 1: Prepare the Azure Environment
1. **Security Group Configuration**:
   - Make sure both VMs (master and worker) are in the same network (Azure VNet) or allow communication between them.
   - Open port 6443 on the security group to allow worker nodes to communicate with the master node.
     - In Azure, configure the Network Security Group (NSG) to allow inbound traffic on port 6443 (Kubernetes API).

### Step 2: Disable Swap on Both Nodes
Disable swap to ensure proper Kubernetes operation. This must be done on both master and worker nodes.

```bash
sudo swapoff -a
```

### Step 3: Install Container Runtime (Containerd)

1. Update the package list and install required dependencies:
   
   ```bash
   sudo apt-get update
   sudo apt-get install -y apt-transport-https ca-certificates curl
   ```

2. Install the container runtime (containerd in this case):

   ```bash
   sudo apt-get install -y containerd
   ```

3. Configure containerd to start using systemd as the cgroup driver:

   ```bash
   sudo mkdir -p /etc/containerd
   containerd config default | sudo tee /etc/containerd/config.toml
   sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
   sudo systemctl restart containerd
   ```

### Step 4: Install kubeadm, kubelet, and kubectl

1. Install the Kubernetes packages on both nodes:

   ```bash
   sudo curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
   sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
   sudo apt-get update
   sudo apt-get install -y kubelet kubeadm kubectl
   ```

2. Mark the packages to ensure they are not automatically updated:

   ```bash
   sudo apt-mark hold kubelet kubeadm kubectl
   ```

### Step 5: Initialize the Master Node

1. On the master node, initialize the Kubernetes cluster with `kubeadm`:

   ```bash
   sudo kubeadm init --pod-network-cidr=192.168.0.0/16
   ```

   - This command initializes the Kubernetes control plane.
   - The `--pod-network-cidr` is required for certain CNI plugins (e.g., Calico).

2. Set up kubectl for the `ubuntu` user (or any non-root user) on the master:

   ```bash
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```

### Step 6: Install a Pod Network Add-on (on Master Node)
Install a network add-on for communication between the pods. Here, we’ll use Calico.

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

### Step 7: Join the Worker Node to the Master Node

1. After initializing the master node, you will see a command that looks like this:

   ```bash
   kubeadm join <master-node-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
   ```

2. Copy this command and run it on the worker node to join it to the master.

   Example:

   ```bash
   sudo kubeadm join 192.168.0.10:6443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:1234567890abcdef1234567890abcdef
   ```

### Step 8: Verify the Nodes

On the master node, verify that the worker node has joined the cluster:

```bash
kubectl get nodes
```

You should see both the master and worker nodes listed, with the status `Ready`.

### Step 9: Optional – Test a Demo Pod

You can test the setup by deploying a simple pod:

```bash
kubectl run hello-world-pod --image=busybox --restart=Never --command -- sh -c "echo 'Hello, World' && sleep 3600"
```

Verify that the pod is running:

```bash
kubectl get pods
```

This completes the basic Kubernetes setup using `kubeadm` on Azure VMs.

### Summary of Key Steps:
1. Disable swap on both nodes.
2. Install container runtime (containerd), `kubeadm`, `kubelet`, and `kubectl` on both nodes.
3. Initialize the master node and install a network plugin (Calico).
4. Join the worker node to the master using the `kubeadm join` command.
5. Verify the cluster setup by checking the nodes and deploying a test pod.
