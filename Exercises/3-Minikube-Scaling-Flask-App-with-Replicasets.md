# Exercise: Scaling Flask App on Single Node using Replicasets

## Objective:
- Understand ReplicaSets and Pods
- Scale Flask App deployment
- Observe pod distribution

## Step 1: Clean up previous minikube does

If you already have a running cluster, you'll need to stop and delete it before starting a new one

```
minikube stop
minikube delete
```
## Step 2: Start minikube with a single node

```
minikube start --nodes=1
```

**Output**

```
😄  minikube v1.34.0 on Ubuntu 24.04 (amd64)
    ▪ MINIKUBE_ACTIVE_DOCKERD=minikube
✨  Automatically selected the docker driver
📌  Using Docker driver with root privileges
👍  Starting "minikube" primary control-plane node in "minikube" cluster
🚜  Pulling base image v0.0.45 ...
🔥  Creating docker container (CPUs=2, Memory=2400MB) ...
🐳  Preparing Kubernetes v1.31.0 on Docker 27.2.0 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

Check nodes
```
kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   42s   v1.31.0
```

## Step 3: Create a new file replicaset.yaml with the following content:

File Name: replicaset.yaml 
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: flask-app-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
      - name: flask-app
        image: flask-app:latest
        imagePullPolicy: Never
        ports:
        - containerPort: 15000
```

## Step 4: Apply the ReplicaSet configuration:

```
kubectl apply -f replicaset.yaml
```

Output
```
replicaset.apps/flask-app-rs created
```

## Step 5: Initialize minikube, build Imange and Verify the ReplicaSet:

```
minikube docker-env
eval $(minikube docker-env)
docker build -t flask-app .
```

## Step 6: Verify the pods:

```
kubectl get pods

NAME                 READY   STATUS    RESTARTS   AGE
flask-app-rs-4nr6q   1/1     Running   0          3m35s
flask-app-rs-84v7x   1/1     Running   0          3m35s
flask-app-rs-rbwr4   1/1     Running   0          3m35s

kubectl get rs

NAME           DESIRED   CURRENT   READY   AGE
flask-app-rs   3         3         0       40s
```

## Step 7: Scale the ReplicaSet to 5 replicas:

```
kubectl scale rs flask-app-rs --replicas=5

replicaset.apps/flask-app-rs scaled
```

## Step 8: Verify the updated ReplicaSet:

```
kubectl get rs

NAME           DESIRED   CURRENT   READY   AGE
flask-app-rs   5         5         5       7m38s
```

## Step 9: Verify the updated pods:

```
kubectl get pods

NAME                 READY   STATUS    RESTARTS   AGE
flask-app-rs-4nr6q   1/1     Running   0          7m55s
flask-app-rs-84v7x   1/1     Running   0          7m55s
flask-app-rs-nsmlx   1/1     Running   0          32s
flask-app-rs-rbwr4   1/1     Running   0          7m55s
flask-app-rs-wfbb4   1/1     Running   0          32s
```

## Step 10: Delete one pod:
Command to delete a pod: kubectl delete pod <pod-name>

```
kubectl delete pod flask-app-rs-84v7x
pod "flask-app-rs-84v7x" deleted
```

## Step 11: Verify the pods:

```
kubectl get pods

NAME                 READY   STATUS    RESTARTS   AGE
flask-app-rs-4nr6q   1/1     Running   0          9m19s
flask-app-rs-hqtm7   1/1     Running   0          51s
flask-app-rs-nsmlx   1/1     Running   0          116s
flask-app-rs-rbwr4   1/1     Running   0          9m19s
flask-app-rs-wfbb4   1/1     Running   0          116s
```

View pod distribution across nodes
```
kubectl get pods -o wide
NAME                 READY   STATUS    RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES
flask-app-rs-4nr6q   1/1     Running   0          10m     10.244.0.9    minikube   <none>           <none>
flask-app-rs-hqtm7   1/1     Running   0          109s    10.244.0.12   minikube   <none>           <none>
flask-app-rs-nsmlx   1/1     Running   0          2m54s   10.244.0.11   minikube   <none>           <none>
flask-app-rs-rbwr4   1/1     Running   0          10m     10.244.0.7    minikube   <none>           <none>
flask-app-rs-wfbb4   1/1     Running   0          2m54s   10.244.0.10   minikube   <none>           <none>
```

# Q&A based on the exercise


**Q1. What is the initial number of replicas in the ReplicaSet?**

Answer: 3

**Q2. How many pods are running after applying the ReplicaSet configuration?**

Answer: 3

**3. What happens when you scale the ReplicaSet to 5 replicas?**

Answer: Kubernetes creates 2 additional pods to meet the desired number of replicas (5). The ReplicaSet now has 5 running pods.

**4. What happens when you delete one pod?**

Answer: Kubernetes automatically creates a new pod to replace the deleted one, maintaining the desired number of replicas (5).

**5. How does Kubernetes maintain the desired number of replicas?**

Answer: Kubernetes continuously monitors the number of running pods and compares it to the desired number of replicas. If there's a discrepancy, Kubernetes creates or deletes pods to maintain the desired state.

**6. How many nodes are running?**

Answer: 1

**7. Where are the pods running with respect to nodes?**

Answer:

Node 1:

- Pod 1 (flask-app-<id>)
- Pod 2 (flask-app-<id>)
- Pod 3 (flask-app-<id>)
- Pod 4 (flask-app-<id>)
- Pod 5 (flask-app-<id>)

All 5 pods are running on the single node.


## Additional Challenges:

- Update the replicaset.yaml file to use a different image.
- Create a Deployment instead of a ReplicaSet.
- Use kubectl describe to inspect the ReplicaSet and pods.

## Tips and Variations:

- Use kubectl get pods -o wide to see pod distribution across nodes.
- Use kubectl logs to view pod logs.
- Use kubectl exec to access a pod's container.
