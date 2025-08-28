# Kubernetes Hands-On Exercise Series

Welcome to the first set of **Kubernetes (K8s) exercises**!  
These activities will help you understand the basics of how Kubernetes runs and manages containerized applications.  

## Exercise 1: Hello Pod

**Goal:** Run your first app inside Kubernetes and access it.

### Steps:
1. Start a local Kubernetes cluster with **Minikube**:
   ```bash
   minikube start
   
2. Create your first Pod (using Nginx image):
   ```bash
   kubectl run hello-k8s --image=nginx --port=80

3. Verify the Pod is running:
   ```bash
   kubectl get pods

4. Expose the Pod as a Service:
   ```bash
   kubectl expose pod hello-k8s --type=NodePort --port=80

5. Open the app in your browser:
   ```bash
   minikube service hello-k8s

You should see the Nginx welcome page. 
Congratulations, you just deployed your first container in Kubernetes!

## System Internal Details


