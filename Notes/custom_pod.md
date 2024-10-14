Now suppose that you have your custom Docker images in DockerHub, let's deploy them as Pods in your Kubernetes cluster. I’ll show you how to create individual Pods for each of your Docker images with examples.

You can follow this general approach:

### Step-by-Step Guide to Deploy Custom Docker Images as Pods

#### Step 1: Define the Pod Manifest

For each of your custom images, you need to create a Kubernetes **YAML manifest**. Let’s create YAML files for each of your Docker images.

---

### Example 1: Deploy `react-education-website`

Create a YAML manifest file (`react-education-website-pod.yaml`):
```yaml
# react-education-website-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: react-education-website-pod
  labels:
    app: react-education-website
spec:
  containers:
  - name: react-education-website
    image: siddhantbhattarai/react-education-website:latest
    ports:
    - containerPort: 80
```

**Explanation:**
- **apiVersion**: The Kubernetes API version.
- **kind**: We are defining a Pod here.
- **metadata**: The name and labels for the Pod.
- **spec**: This defines the containers and the Docker image to be used (your image from DockerHub).
- **ports**: The exposed port (assuming your React app runs on port 80).

Apply this manifest to your cluster:
```bash
kubectl apply -f react-education-website-pod.yaml
```

Check the status:
```bash
kubectl get pods
```

---

### Example 2: Deploy `react-resturant-app`

Now, create another YAML manifest for the restaurant app (`react-resturant-app-pod.yaml`):
```yaml
# react-resturant-app-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: react-resturant-app-pod
  labels:
    app: react-resturant-app
spec:
  containers:
  - name: react-resturant-app
    image: siddhantbhattarai/react-resturant-app:latest
    ports:
    - containerPort: 80
```

Apply this manifest:
```bash
kubectl apply -f react-resturant-app-pod.yaml
```

Verify:
```bash
kubectl get pods
```

---

### Example 3: Deploy `todo-list-app`

Next, create a YAML manifest for the **Todo List** app (`todo-list-app-pod.yaml`):
```yaml
# todo-list-app-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: todo-list-app-pod
  labels:
    app: todo-list-app
spec:
  containers:
  - name: todo-list-app
    image: siddhantbhattarai/todo-list-app:latest
    ports:
    - containerPort: 80
```

Apply this manifest:
```bash
kubectl apply -f todo-list-app-pod.yaml
```

Check if the Pod is running:
```bash
kubectl get pods
```

---

### Step 2: Expose the Pods (Optional)

If you want to access these applications externally (outside the cluster), you’ll need to expose the Pods using a **Service**.

#### Expose `react-education-website` via NodePort:

```bash
kubectl expose pod react-education-website-pod --type=NodePort --port=80
```

To find out which NodePort Kubernetes assigned, run:
```bash
kubectl get services
```

You’ll get output like this:
```
NAME                        TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
react-education-website-pod NodePort   10.104.101.115   <none>        80:30001/TCP   1m
```

Now, you can access the app using your node’s IP and the assigned port `30001`:
```bash
http://<NodeIP>:30001
```

Repeat the same steps to expose the **restaurant app** and **todo-list app**:
```bash
kubectl expose pod react-resturant-app-pod --type=NodePort --port=80
kubectl expose pod todo-list-app-pod --type=NodePort --port=80
```

---

### Step 3: Scaling the Applications (Optional)

If you want to scale any of your applications (run multiple replicas), it's better to use a **Deployment** instead of a single Pod.

For example, let’s create a **Deployment** for the **react-education-website** app.

```yaml
# react-education-website-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react-education-website-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: react-education-website
  template:
    metadata:
      labels:
        app: react-education-website
    spec:
      containers:
      - name: react-education-website
        image: siddhantbhattarai/react-education-website:latest
        ports:
        - containerPort: 80
```

- **replicas**: We are telling Kubernetes to run 3 instances (replicas) of the app.
- The rest of the fields are similar to the Pod definition.

Apply this deployment:
```bash
kubectl apply -f react-education-website-deployment.yaml
```

Check the running Pods:
```bash
kubectl get pods
```

You will see 3 Pods running for this application.

---

### Summary:

- **Pods**: We created individual Pods for each of your custom Docker images using a YAML manifest.
- **Services**: Exposed the Pods to allow external access using NodePort.
- **Scaling**: You can use **Deployments** to run multiple replicas of your applications.

This approach gives you flexibility to deploy any custom Docker image to your Kubernetes cluster.