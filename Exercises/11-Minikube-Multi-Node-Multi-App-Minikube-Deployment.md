
# Multi-Node Kubernetes Cluster with Multiple Applications and Replicasets

## Storyboard

Let's consider developing an e-commerce platform with the product requirements:
1. **Product Catalog** (AppA) - Provides information about available products.
2. **Shopping Cart** (AppB) - Manages user shopping carts.

To ensure availability and fault tolerance, the Product Catalog will have 2 replicas and the Shopping Cart will have 3 replicas. 
Each replica of both services should be distributed across multiple nodes to handle traffic and provide redundancy.

## Use Case
An online store needs high availability for critical services like product browsing and cart management. 
To avoid service disruptions, we deploy each service with replicas distributed across multiple nodes. 
This setup ensures that even if one node fails, the services remain available for customers.

---
### Step 0: Clean up any existing minikube running

```
minikube stop
minikube delete
```
### Step 1a: Set Up Minikube with Multiple Nodes

To create a multi-node environment in Minikube, use the following command:

```bash
minikube start --nodes 3 -p devops-multinode --force
```

This command sets up Minikube with three nodes in a profile named "devops-multinode."
The "docker" driver should not be used with root privileges due to ensure security of the host.
But as we wish to continue as root, we use --force option.

---

### Step 1b: Set Up Minikube with Multiple Nodes

For multi-node cluster, we can't use eval $(minikube -p devops-multinode docker-env) directly to access the Docker environment. Instead, we have to use the Minikube registry add-on to share the images between the nodes in the multi-node setup.  Enable the registry add-on for the multi-node profile

```
minikube -p devops-multinode addons enable registry
```

The output would look like this:

```
minikube -p devops-multinode addons enable registry
💡  registry is an addon maintained by minikube. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
╭──────────────────────────────────────────────────────────────────────────────────────────────────────╮
│                                                                                                      │
│    Registry addon with docker driver uses port 32775 please use that instead of default port 5000    │
│                                                                                                      │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────╯
📘  For more information see: https://minikube.sigs.k8s.io/docs/drivers/docker
    ▪ Using image gcr.io/k8s-minikube/kube-registry-proxy:0.0.6
    ▪ Using image docker.io/registry:2.8.3
🔎  Verifying registry addon...
🌟  The 'registry' addon is enabled
```

This command should enable the registry add-on for the devops-multinode profile and allow us to load images into the registry once we build them using docker build (See Step 4)

### Step 2: Product Catalog (AppA) - Python Code

Create a simple Flask application for the **Product Catalog** service.
Save this file as `product_catalog.py`. This code provides an endpoint to list available products.

```
# product_catalog.py
from flask import Flask, jsonify

app = Flask(__name__)

products = [
    {"id": 1, "name": "Laptop", "price": 1200},
    {"id": 2, "name": "Phone", "price": 800},
    {"id": 3, "name": "Headphones", "price": 150},
]

@app.route("/products", methods=["GET"])
def get_products():
    return jsonify(products)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80)
```

### Step 3: Shopping Cart (AppB) - Python Code

Create a simple Flask application for the **Shopping Cart** service.
Save this file as `shopping_cart.py`. This code provides endpoints to view and add items to the shopping cart.

```
# shopping_cart.py
from flask import Flask, jsonify, request

app = Flask(__name__)

cart = []

@app.route("/cart", methods=["GET"])
def get_cart():
    return jsonify(cart)

@app.route("/cart", methods=["POST"])
def add_to_cart():
    item = request.json
    cart.append(item)
    return jsonify(cart), 201

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80)
```

### Step 4: Dockerize Applications

Create Dockerfiles for both applications.

**Product Catalog Dockerfile**

Save this file as: Dockerfile.product

```dockerfile
# Dockerfile for Product Catalog
FROM python:3.9-slim
WORKDIR /app
COPY product_catalog.py /app/
RUN pip install flask
CMD ["python", "product_catalog.py"]
```

**Shopping Cart Dockerfile**

Save this file as: Dockerfile.shopping

```dockerfile
# Dockerfile for Shopping Cart
FROM python:3.9-slim
WORKDIR /app
COPY shopping_cart.py /app/
RUN pip install flask
CMD ["python", "shopping_cart.py"]
```

Build the Docker images:

```bash
docker build -t product-catalog:latest -f Dockerfile.product .
docker build -t shopping-cart:latest -f Dockerfile.shopping .
```

Once the build is successful, we will have these docker images available locally

```
docker images
REPOSITORY              TAG        IMAGE ID       CREATED          SIZE
shopping-cart           latest     cdfedc6f31ff   8 seconds ago    137MB
product-catalog         latest     2e47c49941d7   53 seconds ago   137MB

```
### Step 5: Load images into registry

After building the image, we have to load it into Minikube's registry so that your Kubernetes cluster can access it. 

```
minikube -p devops-multinode image load product-catalog:latest
minikube -p devops-multinode image load shopping-cart:latest
```

Verify if the images are sucessfully loaded into the registry using

```
minikube -p devops-multinode ssh -- docker images
````

The output of the command should list product-catalog and shopping-cart images

```
REPOSITORY                                TAG                  IMAGE ID       CREATED          SIZE
shopping-cart                             latest               cdfedc6f31ff   17 minutes ago   137MB
product-catalog                           latest               2e47c49941d7   18 minutes ago   137MB
kindest/kindnetd                          v20240813-c6f155d6   12968670680f   3 months ago     85.8MB
registry.k8s.io/kube-scheduler            v1.31.0              1766f54c897f   3 months ago     67.4MB
registry.k8s.io/kube-controller-manager   v1.31.0              045733566833   3 months ago     88.4MB
registry.k8s.io/kube-apiserver            v1.31.0              604f5db92eaa   3 months ago     94.2MB
registry.k8s.io/kube-proxy                v1.31.0              ad83b2ca7b09   3 months ago     91.5MB
registry.k8s.io/etcd                      3.5.15-0             2e96e5913fc0   4 months ago     148MB
registry.k8s.io/pause                     3.10                 873ed7510279   6 months ago     736kB
gcr.io/k8s-minikube/kube-registry-proxy   <none>               38c5e506fa55   8 months ago     187MB
registry.k8s.io/coredns/coredns           v1.11.1              cbb01a7bd410   15 months ago    59.8MB
gcr.io/k8s-minikube/storage-provisioner   v5                   6e38f40d628d   3 years ago      31.5MB
```
---

### Step 6: Deploy Applications on Minikube

Use Kubernetes manifests to deploy the Product Catalog and Shopping Cart services.

**Product Catalog Deployment (ReplicaSet of 2)**

Save this file as: product_catalog_deployment.yaml

```yaml
# product_catalog_deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-catalog
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: product-catalog
  template:
    metadata:
      labels:
        app: product-catalog
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                app: product-catalog
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: product-catalog-container
        image: product-catalog:latest
        imagePullPolicy: Never
        ports:
        - containerPort: 80
```

**Shopping Cart Deployment (ReplicaSet of 3)**

Save this file as: shopping_cart_deployment.yaml

```yaml
# shopping_cart_deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shopping-cart
  namespace: devops-exercise
spec:
  replicas: 3
  selector:
    matchLabels:
      app: shopping-cart
  template:
    metadata:
      labels:
        app: shopping-cart
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                app: shopping-cart
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: shopping-cart-container
        image: shopping-cart:latest
        imagePullPolicy: Never
        ports:
        - containerPort: 80
```

Apply the configurations:

```bash
kubectl apply -f product_catalog_deployment.yaml
kubectl apply -f shopping_cart_deployment.yaml
```

---

### Step 7: Expose the Services

Expose both applications to make them accessible.

```yaml
# product_catalog_service.yaml
apiVersion: v1
kind: Service
metadata:
  name: product-catalog-service
  namespace: default
spec:
  selector:
    app: product-catalog
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: NodePort
```

```yaml
# shopping_cart_service.yaml
apiVersion: v1
kind: Service
metadata:
  name: shopping-cart-service
  namespace: default
spec:
  selector:
    app: shopping-cart
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: NodePort
```

Apply the service configurations:

```bash
kubectl apply -f product_catalog_service.yaml
kubectl apply -f shopping_cart_service.yaml
```
---

### Step 8: Verify Deployment and Distribution

List the pods and check their node assignment to ensure they are distributed as expected:

```bash
kubectl get pods -o wide 
```

Once the deployment is applied sucessfully, we should see these pods running.

```
kubectl get pods -o wide
NAME                               READY   STATUS    RESTARTS   AGE   IP           NODE                   NOMINATED NODE   READINESS GATES
product-catalog-5fdfc7f5fb-2jlnv   1/1     Running   0          29s   10.244.1.3   devops-multinode-m02   <none>           <none>
product-catalog-5fdfc7f5fb-7v556   1/1     Running   0          29s   10.244.2.4   devops-multinode-m03   <none>           <none>
shopping-cart-788c48675b-7l4lx     1/1     Running   0          25s   10.244.0.4   devops-multinode       <none>           <none>
shopping-cart-788c48675b-d48w2     1/1     Running   0          25s   10.244.1.4   devops-multinode-m02   <none>           <none>
shopping-cart-788c48675b-hml9r     1/1     Running   0          25s   10.244.2.5   devops-multinode-m03   <none>           <none>
```

Each pod for Product Catalog and Shopping Cart should be running on different nodes.

### Step 9: Access the Services

Use Minikube’s service forwarding to access the applications:

## Step 9a: Access Product-Catalog service

Open a new Linux or WSL Terminal

```bash
minikube -p devops-multinode service product-catalog-service
```
Output
```
minikube -p devops-multinode service product-catalog-service
|-----------|-------------------------|-------------|---------------------------|
| NAMESPACE |          NAME           | TARGET PORT |            URL            |
|-----------|-------------------------|-------------|---------------------------|
| default   | product-catalog-service |          80 | http://192.168.49.2:32316 |
|-----------|-------------------------|-------------|---------------------------|
🏃  Starting tunnel for service product-catalog-service.
|-----------|-------------------------|-------------|------------------------|
| NAMESPACE |          NAME           | TARGET PORT |          URL           |
|-----------|-------------------------|-------------|------------------------|
| default   | product-catalog-service |             | http://127.0.0.1:35855 |
|-----------|-------------------------|-------------|------------------------|
🎉  Opening service default/product-catalog-service in default browser...
👉  http://127.0.0.1:35855
❗  Because you are using a Docker driver on linux, the terminal needs to be open to run it.
```

## Step ba: Access Shopping-Cart service

Open a new Linux or WSL Terminal

```bash
minikube -p devops-multinode service shopping-cart-service
```

Output

```
|-----------|-----------------------|-------------|---------------------------|
| NAMESPACE |         NAME          | TARGET PORT |            URL            |
|-----------|-----------------------|-------------|---------------------------|
| default   | shopping-cart-service |          80 | http://192.168.49.2:32382 |
|-----------|-----------------------|-------------|---------------------------|
🏃  Starting tunnel for service shopping-cart-service.
|-----------|-----------------------|-------------|------------------------|
| NAMESPACE |         NAME          | TARGET PORT |          URL           |
|-----------|-----------------------|-------------|------------------------|
| default   | shopping-cart-service |             | http://127.0.0.1:38975 |
|-----------|-----------------------|-------------|------------------------|
🎉  Opening service default/shopping-cart-service in default browser...
👉  http://127.0.0.1:38975
❗  Because you are using a Docker driver on linux, the terminal needs to be open to run it.
```

These commands open the services in our default web browser or display URLs to access each service.

Note the URLs:
👉  http://127.0.0.1:35855  (Product Catalog service)
👉  http://127.0.0.1:38975  (Shopping Cart service)

---

### Step 10: Testing with `curl`

After deploying the services, we can test their functionality using `curl` commands.

#### Test Product Catalog Service

1. **Retrieve Product List**: Test if the Product Catalog service returns the list of products.

   ```
   curl  http://127.0.0.1:35855/products
   ```
   Expected response:
   ```json
   [
       {"id": 1, "name": "Laptop", "price": 1200},
       {"id": 2, "name": "Phone", "price": 800},
       {"id": 3, "name": "Headphones", "price": 150}
   ]
   ```

#### Test Shopping Cart Service

1. **Retrieve Cart**: Verify that the Shopping Cart service returns an empty cart initially.

   ```bash
   curl http://127.0.0.1:38975/cart
   ```

   Expected response:
   ```json
   []
   ```

2. **Add Item to Cart**: Test adding an item to the cart by sending a `POST` request with item data.

   ```bash
   curl -X POST http://127.0.0.1:38975/cart -H "Content-Type: application/json" -d '{"id": 1, "name": "Laptop", "quantity": 1}'
   ```

   Expected response:
   ```json
   [
       {"id": 1, "name": "Laptop", "quantity": 1}
   ]
   ```

---

## Summary
In this exercise, we deployed two Python applications (Product Catalog and Shopping Cart) on a multi-node Minikube cluster
with ReplicaSets, ensuring each pod is running on a different node. 
This setup maximizes availability and fault tolerance across the cluster.
