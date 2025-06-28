# Microservices Kubernetes Deployment with Minikube

## Table of Contents

1. [Project Overview](#project-overview)  
2. [Application Architecture](#application-architecture)  
3. [System Requirements](#system-requirements)  
4. [Installation and Environment Setup](#installation-and-environment-setup)  
5. [Building Docker Images](#building-docker-images)  
6. [Loading Images into Minikube](#loading-images-into-minikube)  
7. [Kubernetes Deployment](#kubernetes-deployment)  
8. [Ingress Configuration](#ingress-configuration)  
9. [Testing Services](#testing-services)  
10. [Screenshots and Documentation](#screenshots-and-documentation)  
11. [Troubleshooting](#troubleshooting)  
12. [Project Structure](#project-structure)  

---

## Project Overview

This project involves deploying a containerized microservices-based Node.js application on a local Kubernetes cluster using Minikube. The deployment includes service discovery, health checks, resource limits, and an NGINX Ingress controller for unified external access.

---

## Application Architecture

The application consists of the following independent microservices:

| Service           | Port  | Description                       |
|-------------------|-------|-----------------------------------|
| user-service      | 3000  | Manages user-related data         |
| product-service   | 3001  | Handles product catalog           |
| order-service     | 3002  | Processes customer orders         |
| gateway-service   | 3003  | Acts as a reverse proxy/API hub   |

The `gateway-service` is exposed externally, and routes incoming HTTP requests to the respective services through Ingress.

---

## System Requirements

- Operating System: Linux (Ubuntu 20.04 or later) or WSL2 (Windows Subsystem for Linux)
- Docker: Installed and running
- Minikube: v1.36.0 or newer
- kubectl: Kubernetes CLI
- Node.js: Optional (for running services locally)

---

## Installation and Environment Setup

1. **Install Docker**

   Follow instructions: https://docs.docker.com/get-docker/

2. **Install kubectl**

   ```bash
   sudo apt update
   sudo apt install -y curl
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   chmod +x kubectl
   sudo mv kubectl /usr/local/bin/
   ```

3. **Install Minikube**

   ```bash
   curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
   sudo install minikube-linux-amd64 /usr/local/bin/minikube
   ```

4. **Start Minikube**

   ```bash
   minikube start --driver=docker --cpus=2 --memory=4096
   ```

5. **Enable Ingress Addon**

   ```bash
   minikube addons enable ingress
   ```

6. **Verify Ingress**

   ```bash
   kubectl get pods -n kube-system
   ```

---

## Building Docker Images

Navigate to each microservice directory and build the Docker image:

```bash
docker build -t user-service ./user-service
docker build -t product-service ./product-service
docker build -t order-service ./order-service
docker build -t gateway-service ./gateway-service
```

---

## Loading Images into Minikube

To ensure Minikube can access local images, use:

```bash
minikube image load user-service:latest
minikube image load product-service:latest
minikube image load order-service:latest
minikube image load gateway-service:latest
```

---

## Kubernetes Deployment

Apply the Kubernetes resources:

```bash
kubectl apply -f deployments/
kubectl apply -f services/
```

Check that all pods are running:

```bash
kubectl get pods
```

---

## Ingress Configuration

1. **Hosts File Setup**

   Add the following to your system's hosts file:

   - On Linux/macOS:
     ```bash
     sudo nano /etc/hosts
     ```
   - On Windows (as Administrator):
     ```
     C:\Windows\System32\drivers\etc\hosts
     ```

   Add this line (replace with your Minikube IP):

   ```
   <MINIKUBE_IP> micro.local
   ```

   Example:
   ```
   192.168.49.2 micro.local
   ```

2. **Apply Ingress Resource**

   ```bash
   kubectl apply -f ingress/ingress.yaml
   ```

3. **Ingress Routing Paths**

   | Path                    | Service           |
   |-------------------------|-------------------|
   | `/api/users/(.*)`       | user-service      |
   | `/api/products/(.*)`    | product-service   |
   | `/api/orders/(.*)`      | order-service     |
   | `/(.*)`                 | gateway-service   |

---

## Testing Services

1. **Test Ingress Endpoints**

   ```bash
   curl http://micro.local/api/users/users
   curl http://micro.local/api/products/products
   curl http://micro.local/api/orders/orders
   curl http://micro.local/
   ```

2. **Test via Port Forwarding**

   ```bash
   kubectl port-forward svc/user-service 3000:3000
   curl http://localhost:3000/users
   ```

3. **Check Logs**

   ```bash
   kubectl logs deploy/gateway-service
   ```

---

## Screenshots and Documentation

The screenshots are included in the `screenshots/` folder

---

## Troubleshooting

- **Pods not pulling images**: Ensure `imagePullPolicy: IfNotPresent` is set.
- **Ingress not routing**: Check for correct annotations and `rewrite-target` rules.
- **No response from services**: Port-forward to confirm service availability.
- **DNS issue**: Confirm that `/etc/hosts` or Windows hosts file contains correct Minikube IP mapping.

---

## Project Structure

```
submission/
├── deployments/
│   ├── user-service.yaml
│   ├── product-service.yaml
│   ├── order-service.yaml
│   └── gateway-service.yaml
├── services/
│   ├── user-service.yaml
│   ├── product-service.yaml
│   ├── order-service.yaml
│   └── gateway-service.yaml
├── ingress/
│   └── ingress.yaml
├── screenshots/
└── README.md
```

---

## Conclusion

This project demonstrates a complete microservices deployment pipeline on Kubernetes using Minikube. It includes containerized services, declarative configuration with YAML manifests, and NGINX Ingress routing. This setup simulates a production-like environment suitable for development, testing, and learning Kubernetes orchestration.
