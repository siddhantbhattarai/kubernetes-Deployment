### Kubernetes Notes: Key Concepts and Components

Kubernetes (often abbreviated as **K8s**) is an open-source platform designed for **automating deployment, scaling, and managing containerized applications**. Below is an overview of the key concepts and components that you need to know to work with Kubernetes effectively.

---

### 1. **Core Concepts**

#### **Containers**
- **Containers** are lightweight, portable units that package an application and its dependencies.
- Common container runtimes include **Docker**, but Kubernetes supports other runtimes through the Container Runtime Interface (CRI).

#### **Pods**
- A **Pod** is the smallest, most basic deployable object in Kubernetes. 
- It can contain one or more containers that share resources like storage and network.
- Containers within the same Pod can communicate with each other easily, while Pods communicate over a network.

#### **Nodes**
- **Node** is a machine (either physical or virtual) in the Kubernetes cluster where **Pods** are deployed.
- A **Node** runs the necessary services like `kubelet`, container runtime, and `kube-proxy`.

#### **Cluster**
- A **Cluster** is a set of **Nodes** managed by Kubernetes, which work together to run containerized applications.
- It consists of at least one **Master Node** and multiple **Worker Nodes**.

---

### 2. **Kubernetes Architecture**

#### **Master Node (Control Plane)**
The **Control Plane** is responsible for managing the state of the Kubernetes cluster. Key components include:

1. **API Server (`kube-apiserver`)**
   - Serves as the entry point for all administrative tasks.
   - It processes requests from users, the command-line tool (`kubectl`), and other components.

2. **etcd**
   - A highly available, consistent, and distributed key-value store that stores all cluster data.
   - It holds the configuration data, including the desired state of the cluster.

3. **Controller Manager (`kube-controller-manager`)**
   - Monitors the state of the cluster and takes corrective action to achieve the desired state.

4. **Scheduler (`kube-scheduler`)**
   - Assigns Pods to available Nodes based on resource availability, constraints, and policies.

---

### 3. **Kubernetes Objects**

#### **ReplicaSet**
- Ensures that a specified number of Pod replicas are running at all times.
- It maintains the desired number of replicas even if Pods fail.

#### **Deployment**
- A **Deployment** manages ReplicaSets and provides declarative updates to Pods.
- Used to perform rolling updates, scaling, and rollback of applications.

#### **Service**
- A **Service** is an abstraction that defines a logical set of Pods and a policy by which to access them.
- It enables Pods to be exposed to the outside world (or within the cluster) through a stable IP address.
  - **ClusterIP**: Exposes the Service within the cluster.
  - **NodePort**: Exposes the Service on each Node’s IP and static port.
  - **LoadBalancer**: Integrates with cloud provider’s load balancers for external traffic.

#### **ConfigMap**
- Allows you to decouple configuration artifacts from application images.
- It provides configuration data in key-value pairs and can be mounted as files or environment variables inside containers.

#### **Secret**
- Stores sensitive information, such as passwords, OAuth tokens, and SSH keys, in an encrypted format.

#### **Volume**
- **Volumes** provide persistent storage to Pods.
- Kubernetes supports multiple volume types like **emptyDir**, **hostPath**, and external storage systems (e.g., NFS, AWS EBS, GCP Persistent Disk).

#### **StatefulSet**
- Used for managing stateful applications.
- Ensures that Pods are created in a deterministic and sequential order, with stable network identifiers.

---

### 4. **Networking in Kubernetes**

#### **CNI (Container Network Interface)**
- Kubernetes uses **CNI** plugins to manage network connectivity.
- Popular networking solutions include **Calico**, **Flannel**, **Weave**, and **Cilium**.

#### **Kube-Proxy**
- A component that runs on each Node, responsible for maintaining network rules to allow communication between Pods and Services.

#### **Ingress**
- **Ingress** is an API object that manages external access to services, typically through HTTP/S routes.
- It provides load balancing, SSL termination, and name-based virtual hosting.

---

### 5. **Scaling and Auto-scaling**

#### **Horizontal Pod Autoscaler (HPA)**
- Automatically adjusts the number of Pods based on CPU utilization or other select metrics.
  
#### **Vertical Pod Autoscaler (VPA)**
- Adjusts the CPU and memory requests/limits for containers in a Pod.

#### **Cluster Autoscaler**
- Scales the number of worker Nodes in the cluster when there are not enough resources to schedule new Pods.

---

### 6. **Security**

#### **RBAC (Role-Based Access Control)**
- **RBAC** is a method to define permissions and access policies within the cluster.
- **Roles** and **RoleBindings** define which users or services can perform specific actions on particular resources.

#### **Network Policies**
- **Network Policies** allow you to control the network traffic flow to and from Pods.

#### **Pod Security Standards (PSS)**
- Security policies that define how Pods should be configured (e.g., no root user, read-only file system).

#### **Service Accounts**
- A **Service Account** is used to provide an identity for Pods to interact with the Kubernetes API or external services securely.

---

### 7. **Kubernetes Tools and Ecosystem**

#### **kubectl**
- **kubectl** is the command-line tool used to interact with Kubernetes clusters.
  - Example commands:
    - `kubectl get pods`: List all Pods.
    - `kubectl describe pod <pod-name>`: Get detailed information about a specific Pod.
    - `kubectl apply -f <file.yaml>`: Apply a configuration file to the cluster.

#### **Helm**
- **Helm** is a package manager for Kubernetes, used for deploying and managing applications in the form of "Helm Charts."

#### **Operators**
- **Kubernetes Operators** extend the Kubernetes API to manage custom resources and handle application-specific tasks, like database management, backups, and upgrades.

#### **Kustomize**
- A tool that allows customizing Kubernetes YAML manifests without modifying them directly.

---

### 8. **Observability and Monitoring**

#### **Prometheus**
- **Prometheus** is a monitoring tool often integrated with Kubernetes to collect metrics and alert based on resource usage.

#### **Grafana**
- A visualization tool that works alongside Prometheus to provide dashboards for real-time monitoring of cluster and application health.

#### **Logging**
- Kubernetes does not provide a native logging solution but can integrate with tools like **Fluentd**, **Elasticsearch**, and **Kibana** for centralized logging.

---

### 9. **CI/CD Integration**

- Kubernetes can be integrated with **Continuous Integration/Continuous Deployment (CI/CD)** pipelines using tools like **Jenkins**, **GitLab CI**, **Argo CD**, or **Tekton**.
  
- **GitOps** practices (such as with Argo CD or Flux) ensure that Kubernetes applications are always in sync with a version-controlled repository.

---

### 10. **Common Kubernetes Commands**

- **Check cluster status:**  
  `kubectl cluster-info`
  
- **Get list of nodes:**  
  `kubectl get nodes`
  
- **Get detailed Pod info:**  
  `kubectl describe pod <pod-name>`
  
- **Scale a deployment:**  
  `kubectl scale deployment <deployment-name> --replicas=<number>`
  
- **Delete resources (Pod, Service, etc.):**  
  `kubectl delete pod <pod-name>`

---

### 11. **Kubernetes Deployment Strategies**

1. **Rolling Updates**
   - Gradually replaces old Pods with new ones without downtime.
   
2. **Blue/Green Deployment**
   - Runs two environments simultaneously (Blue = current version, Green = new version). Switches traffic to the new environment when it's ready.

3. **Canary Deployment**
   - Gradually shifts a small percentage of traffic to a new version to test its stability before full rollout.

---

These notes cover the essential Kubernetes concepts and components, helping you get started or deepen your understanding of managing containerized applications.