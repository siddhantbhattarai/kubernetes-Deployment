Let’s explore **Persistent Volumes** (PV) in Kubernetes, which are crucial for managing storage in a way that survives beyond the lifecycle of individual Pods.

### What are Persistent Volumes?

**Persistent Volumes (PV)** are a way to provide durable storage in Kubernetes. They allow you to decouple storage from the Pods that use it, enabling your data to persist even if the Pod is terminated or restarted. This is especially important for databases that need to retain their data across restarts.

### Key Concepts:

1. **Persistent Volume (PV)**: A piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes. It's a resource in the cluster.
2. **Persistent Volume Claim (PVC)**: A request for storage by a user. A PVC can specify size, access modes, and storage classes. Kubernetes will bind a PVC to a suitable PV.
3. **Storage Class**: Provides a way to describe the "classes" of storage you offer. For example, you might have SSDs and HDDs with different performance characteristics.

### Step-by-Step Guide to Using Persistent Volumes

#### Step 1: Create a Persistent Volume (PV)

Let’s create a Persistent Volume for our MySQL database. Here is an example YAML manifest for a PV:

```yaml
# mysql-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 5Gi # Specify the size of the volume
  accessModes:
    - ReadWriteOnce # Indicates that the volume can be mounted as read-write by a single node
  hostPath:
    path: /mnt/data/mysql # Specify the path on the host where data will be stored
```

> **Note**: The `hostPath` type is good for development and testing but is not suitable for production. In production, you might use cloud storage options like AWS EBS, GCE Persistent Disks, or NFS.

Apply the Persistent Volume manifest:

```bash
kubectl apply -f mysql-pv.yaml
```

#### Step 2: Create a Persistent Volume Claim (PVC)

Next, create a Persistent Volume Claim that will request storage for our MySQL database:

```yaml
# mysql-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi # Requesting 5Gi of storage
```

Apply the Persistent Volume Claim manifest:

```bash
kubectl apply -f mysql-pvc.yaml
```

#### Step 3: Update the MySQL Deployment to Use the PVC

Now, we need to update the MySQL Pod to use the Persistent Volume Claim. Modify your `mysql-pod.yaml` to include the volume definition:

```yaml
# Updated mysql-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod
  labels:
    tier: database
spec:
  containers:
  - name: mysql-container
    image: mysql:5.7
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: rootpassword
    - name: MYSQL_DATABASE
      value: mydb
    - name: MYSQL_USER
      value: user
    - name: MYSQL_PASSWORD
      value: password
    ports:
    - containerPort: 3306
    volumeMounts:
    - mountPath: /var/lib/mysql # Where MySQL stores its data
      name: mysql-storage
  volumes:
  - name: mysql-storage
    persistentVolumeClaim:
      claimName: mysql-pvc
```

**Explanation of Changes**:
- **volumeMounts**: This specifies where in the container to mount the volume (MySQL stores its data in `/var/lib/mysql`).
- **volumes**: This connects the `mysql-storage` to the `mysql-pvc`.

#### Step 4: Apply Changes

Now, update your Pod with the new configuration:

```bash
kubectl apply -f mysql-pod.yaml
```

### Step 5: Verify Persistent Volume Usage

Check if the PVC is bound to the PV:

```bash
kubectl get pvc
```

You should see the status of your PVC as **Bound**, indicating that it is successfully connected to a PV.

### Step 6: Verify Data Persistence

To test that your data persists, you can do the following:
1. Create a few tables or insert data into the MySQL database.
2. Delete the MySQL Pod (it will be recreated if managed by a Deployment).
3. Connect to the MySQL service again and check if the data still exists.

You can delete the Pod with:
```bash
kubectl delete pod mysql-pod
```

Then check the data again by connecting to the MySQL service to see if your data is still intact.

---

### Advanced Networking in Kubernetes

Now, let’s touch on some advanced networking concepts that you can apply when setting up your 3-tier application.

#### 1. **ClusterIP vs. NodePort vs. LoadBalancer**

- **ClusterIP**: The default service type, which exposes the service on a cluster-internal IP. Only reachable from within the cluster.
- **NodePort**: Exposes the service on each node’s IP at a static port (the NodePort). You can access the service externally via `<NodeIP>:<NodePort>`.
- **LoadBalancer**: Creates an external load balancer (if supported by the cloud provider) and assigns a fixed, external IP to the service.

#### 2. **DNS in Kubernetes**

Kubernetes automatically assigns DNS names to services. You can use service names to communicate between Pods without needing to know IP addresses. For example, the backend can connect to MySQL using the name `mysql-service`, and the frontend can connect to the backend using `backend-service`.

#### 3. **Network Policies**

Kubernetes allows you to create network policies that define how Pods communicate with each other. This is useful for enhancing security.

Here’s a simple example of a network policy that only allows the frontend to communicate with the backend:

```yaml
# network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-to-backend
spec:
  podSelector:
    matchLabels:
      tier: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: frontend
```

Apply the network policy:
```bash
kubectl apply -f network-policy.yaml
```

### Summary

- **Persistent Volumes** allow you to decouple storage from Pods, providing durable storage for databases or applications.
- **Persistent Volume Claims** request storage from available PVs and bind to them.
- **Advanced Networking** includes using different service types, leveraging DNS for service discovery, and implementing network policies for security.

This should give you a good foundation for working with persistent storage and advanced networking in Kubernetes.