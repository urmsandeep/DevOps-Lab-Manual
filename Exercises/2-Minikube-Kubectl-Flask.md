# Exercise: Deploy a Flask app on Minikube using kubectl and yaml

## Objective
Learn Kubernetes basics using Minikube to set up a single-node cluster and deploy Python applications. 
This exercise involves deploying a Flask app using minikube.

## Prerequisites
- The Illustrated **Children’s Guide to Kubernetes**
https://www.cncf.io/phippy/the-childrens-illustrated-guide-to-kubernetes/
- Video: https://www.youtube.com/watch?v=3I9PkvZ80BQ
- Install Minikube https://minikube.sigs.k8s.io/docs/
- Follow the Minikube installation guide for your operating system (Windows/Linux/MacOS).
- **Watch:** Origins of Kubernetes with Tim Hockin of Google https://www.youtube.com/watch?v=xSztxKexDXM
- **Watch:** Keynote: A Vision for Vision (in the era of AI) - Kubernetes in Its Second Decade - Tim Hockin https://www.youtube.com/watch?v=WqeShpaztZY

## Minikube cheat sheet - frequently used commands
```
# 1. Starting and Stopping Minikube
minikube start                            # Start Minikube with default settings
minikube start --kubernetes-version=v1.21.2  # Start with a specific Kubernetes version
minikube stop                             # Stop Minikube without deleting
minikube delete                           # Delete Minikube cluster

# 2. Checking Status and Information
minikube status                           # Check Minikube status
kubectl cluster-info                      # Display cluster information

# 3. Accessing Services
minikube service <service-name>           # Open specified service in browser
minikube service list                     # List URLs of all services
minikube tunnel                           # Create a network tunnel to access LoadBalancer services

# 4. Docker with Minikube
eval $(minikube docker-env)               # Use Minikube's Docker daemon
docker build -t my-image .                # Build image directly in Minikube
minikube image load my-image              # Load a local image into Minikube

# 5. Debugging and Logs
minikube logs                             # View Minikube logs
minikube dashboard                        # Open the Kubernetes dashboard
minikube ssh                              # SSH into the Minikube VM
```

## Step 1: Start minkube
```
minikube start
```

## Step 2: Create a Flask Application
File Name: app.py
```
from flask import Flask
app = Flask(__name__)

@app.route('/')
def home():
    return "Hello from Flask on Kubernetes!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=15000)
```

## Step 3: Create a Dockerfile for Flask Application:
File Name: Dockerfile
```
FROM python:3.8-slim
WORKDIR /app
COPY . /app
RUN pip install flask
CMD ["python", "app.py"]
```

## Step 4: Build the Docker Image with Minikube’s Docker Daemon:
Run the following comamnd to display what variable would be set. 
```
minikube docker-env
```
This set of variable export statements 
will configure your (operating systems's) Docker CLI to use Minikube’s Docker daemon. 
```
eval $(minikube docker-env)
docker build -t flask-app .
```

## Step 5: Create a Kubernetes Deployment YAML File:

**Note: About imagePullPolicy: Never** 

Kubernetes has three image pull policies:

1. **Always:** Kubernetes always pulls the image from the registry, even if it exists locally.
2. **IfNotPresent:** Kubernetes pulls the image only if it doesn't exist locally.
3. **Never:** Kubernetes never pulls the image from the registry; it only uses local images.

By setting imagePullPolicy: Never, we are making Kubernetes to:
- Use the local flask-app:latest image.
- Not pull the image from a remote or external registry like Docker Hub

File Name: flask-deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
spec:
  replicas: 1
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

## Step 6: Deploy the application
Use kubectl to apply the YAML file and deploy the Flask application:

```
kubectl apply -f flask-deployment.yaml
```
## Step 7: Check Deployment Status
To verify that the deployment was created successfully and check the number of replicas:
```
kubectl get deployments
```
Output: You should see your flask-app deployment with its current status and number of replicas.
```
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
flask-app    1/1     1            1           5s
```
## Step 8: Verify Pods Created by the Deployment
To check if the deployment created the expected pods:
```
kubectl get pods -l app=flask-app
```
Output: The output will list the pods created by the flask-app deployment. Ensure the STATUS shows Running.

```
NAME                          READY   STATUS    RESTARTS   AGE
flask-app-3443a-213a          1/1     Running   0          5s
```
## Step 9: Describe the Deployment
This command provides detailed information about the deployment, 
including events, number of replicas, and any issues during deployment.
```
kubectl describe deployment flask-app

Name:                   flask-app
Namespace:              default
CreationTimestamp:      Mon, 11 Nov 2024 16:52:41 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=flask-app
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=flask-app
  Containers:
   flask-app:
    Image:         flask-app:latest
    Port:          15000/TCP
    Host Port:     0/TCP
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   flask-app-b8cd75b6f (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  34s   deployment-controller  Scaled up replica set flask-app-b8cd75b6f to 1

```
## Step 10: View Deployment Logs
To see the logs of the Flask application (useful to verify if the app is running as expected):
Use **kubectl logs <pod-name>**

```
kubectl logs flask-app-b8cd75b6f-tpdpr

 * Serving Flask app 'app'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:15000
 * Running on http://10.0.0.210:15000
Press CTRL+C to quit

```
The logs shows that Flask is running and listening on the specified port, 
indicating that the application is successfully deployed.

## Step 11: Check Services
This will show the flask-service if created, with details like the service type 
(NodePort), PORT, and any external IPs if applicable.
```
kubectl get services

NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   7d11h
```
## Step 12: Try access the app.py python flask service running on port: 15000

Use command **curl  http://127.0.0.1:15000** to access the python flask service.

```
curl: (7) Failed to connect to 127.0.0.1 port 15000 after 1 ms: Couldn't connect to server
```
You will notice that the curl fails to connect to 127.0.0.1:15000. This is because:
1. Minikube runs in a virtual environment.
2. The app exposes port 15000 internally, but not externally.

## Step 13: Update deployment file with Service section to Access port 15000

The "Service" section specifies port:15000 and targetPort:15000

Updated: flask-deployment.yaml file.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
spec:
  replicas: 1
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
---
apiVersion: v1
kind: Service
metadata:
  name: flask-app-service
spec:
  selector:
    app: flask-app
  ports:
  - port: 15000
    targetPort: 15000
  type: NodePort
```
Apply the changed deployment file

```
kubectl apply -f flask-deployment.yaml
deployment.apps/flask-app unchanged
service/flask-app-service created
```

**Explanation:** 

**port (Service Port):**
- The port number exposed by the Kubernetes Service.
- This is the port that external users will use to access python Flask application.
- In this case, port: 15000 means the Service is exposed on port 15000.

**targetPort (Container Port):**
- The port number on which the container is listening.
- This is the port that the Flask application is using inside the container.
- In this case, targetPort: 15000 means the container is listening on port 15000.

By setting port: 15000 and targetPort: 15000, we are creating a mapping:

**External Request (port 15000) → Service (port 15000) → Container (port 15000)**

Here's what happens:
1. External clients send requests to http://localhost:15000.
2. The Kubernetes Service receives the request on port 15000.
3. The Service forwards the request to the container on port 15000 (targetPort).
4. The Flask application, running inside the container, receives the request on port 15000.

Now run **minikube service flask-app-service --url** command 
1. minikube service: This command interacts with Kubernetes services running in Minikube.
2. flask-app-service: This specifies the name of the service ("flask-app-service") that we want to access.
3. --url: This flag instructs Minikube to provide the URL for accessing the service.

When we run this command:
1. Minikube checks if the flask-app-service is running and exposed.
2. If the service is available, Minikube generates a URL for accessing the service.
3. The URL is displayed in the terminal.

```
minikube service flask-app-service --url
http://127.0.0.1:36157
❗  Because you are using a Docker driver on linux, the terminal needs to be open to run it.
```
Keep the window running and in a *new* terminal (WSL) run the following command to reach the service

```
curl http://127.0.0.1:36157
Hello from Flask on Kubernetes!
```
The Flask application is now accessible !!

# Q&A based on the exercise   

**Q1: What is the purpose of minikube service flask-app-service --url?**
A1: To provide the URL for accessing the flask-app-service running in Minikube.

**Q2: What happens when you run minikube service flask-app-service --url?**
A2: Minikube checks if the service is running, generates a URL, and displays it in the terminal.

**Q3: Why is targetPort used in Kubernetes Service configuration?**
A3: To specify the port number on which the container is listening.

**Q4: What is the difference between port and targetPort in Kubernetes Service configuration?**
A4: port is the exposed Service port, while targetPort is the container port.

**Q5: How do you access a Flask application running in Minikube?**
A5: Use minikube service <service-name> --url to get the access URL.

**Q6: Why does the terminal need to remain open when using Docker driver on Linux with Minikube?**
A6: Because the Docker driver requires the terminal to stay open to maintain the Minikube connection.

**Q7: What is the benefit of using --url flag with minikube service command?**
A7: It provides a convenient way to access the service without manually constructing the URL.

**Q8: What command is used to expose a service in Kubernetes?**   
**A8:** kubectl expose or kubectl apply -f <service.yaml> is used to expose a service in Kubernetes.

**Q9: How does Minikube help in local Kubernetes testing?**  
**A9:** Minikube runs a single-node Kubernetes cluster locally, making it easier to test deployments without needing a full Kubernetes environment.

**Q10: What is the role of kubectl in this setup?**
**A10:** kubectl is the Kubernetes CLI tool used to interact with the Kubernetes cluster. It allows you to deploy applications, scale them, and manage resources.

