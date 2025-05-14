# Shopper App Deployment Guide with Docker Hub and Kubernetes HPA

This guide will walk you through the process of deploying a three-component Shopper application (frontend, backend, and admin) to Docker Hub and implementing Horizontal Pod Autoscaling (HPA) in a Minikube Kubernetes cluster.

## Project Structure

```
shopper-app/
├── frontend/         # User-facing website
│   ├── Dockerfile
│   └── (frontend source code)
├── backend/          # API server
│   ├── Dockerfile
│   └── (backend source code)
├── admin/            # Admin dashboard
│   ├── Dockerfile
│   └── (admin source code)
├── k8s/              # Kubernetes manifests
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── admin-deployment.yaml
│   ├── admin-service.yaml
│   ├── hpa-frontend.yaml
│   ├── hpa-backend.yaml
│   └── hpa-admin.yaml
```

## Step 1: Build and Push Docker Images to Docker Hub

First, log in to Docker Hub:

```bash
docker login
```

Build and push each component:

### Frontend
```bash
cd frontend
docker build -t <your-dockerhub-username>/shopper-frontend:latest .
docker push <your-dockerhub-username>/shopper-frontend:latest
```

### Backend
```bash
cd ../backend
docker build -t <your-dockerhub-username>/shopper-backend:latest .
docker push <your-dockerhub-username>/shopper-backend:latest
```

### Admin
```bash
cd ../admin
docker build -t <your-dockerhub-username>/shopper-admin:latest .
docker push <your-dockerhub-username>/shopper-admin:latest
```

## Step 2: Create Kubernetes Deployment and Service Manifests

### Frontend Component

**frontend-deployment.yaml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shopper-frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: shopper-frontend
  template:
    metadata:
      labels:
        app: shopper-frontend
    spec:
      containers:
      - name: shopper-frontend
        image: <your-dockerhub-username>/shopper-frontend:latest
        ports:
        - containerPort: 5173
        resources:
          requests:
            cpu: "100m"
          limits:
            cpu: "500m"
```

**frontend-service.yaml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: shopper-frontend-service
spec:
  type: NodePort
  selector:
    app: shopper-frontend
  ports:
    - protocol: TCP
      port: 5173
      targetPort: 5173
```

### Backend Component

**backend-deployment.yaml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shopper-backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: shopper-backend
  template:
    metadata:
      labels:
        app: shopper-backend
    spec:
      containers:
      - name: shopper-backend
        image: <your-dockerhub-username>/shopper-backend:latest
        ports:
        - containerPort: 5050
        resources:
          requests:
            cpu: "100m"
          limits:
            cpu: "500m"
        env:
        - name: PORT
          value: "5050"
        - name: MONGO_URI
          value: "mongodb+srv://savalkaranjali6:0PWoqbtNNbiCmaIK@cluster0.kny7zzh.mongodb.net/e-commerce"
        - name: JWT_SECRET
          value: "secret_ecom"
```

**backend-service.yaml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: shopper-backend-service
spec:
  type: ClusterIP
  selector:
    app: shopper-backend
  ports:
    - protocol: TCP
      port: 5050
      targetPort: 5050
```

### Admin Component

**admin-deployment.yaml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shopper-admin
spec:
  replicas: 1
  selector:
    matchLabels:
      app: shopper-admin
  template:
    metadata:
      labels:
        app: shopper-admin
    spec:
      containers:
      - name: shopper-admin
        image: <your-dockerhub-username>/shopper-admin:latest
        ports:
        - containerPort: 5174
        resources:
          requests:
            cpu: "100m"
          limits:
            cpu: "500m"
```

**admin-service.yaml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: shopper-admin-service
spec:
  type: NodePort
  selector:
    app: shopper-admin
  ports:
    - protocol: TCP
      port: 5174
      targetPort: 5174
```

## Step 3: Create Horizontal Pod Autoscaling (HPA) Manifests

### Frontend HPA

**hpa-frontend.yaml**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-frontend
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: shopper-frontend
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

### Backend HPA

**hpa-backend.yaml**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-backend
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: shopper-backend
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

### Admin HPA

**hpa-admin.yaml**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-admin
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: shopper-admin
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

## Step 4: Enable Metrics Server for HPA

HPA requires the metrics server to be enabled in your Minikube cluster:

```bash
minikube addons enable metrics-server
```

Verify the metrics server is running:

```bash
kubectl get deployment metrics-server -n kube-system
```

## Step 5: Deploy to Minikube

Apply all the Kubernetes manifests:

```bash
# Deploy the applications
kubectl apply -f k8s/frontend-deployment.yaml
kubectl apply -f k8s/frontend-service.yaml
kubectl apply -f k8s/backend-deployment.yaml
kubectl apply -f k8s/backend-service.yaml
kubectl apply -f k8s/admin-deployment.yaml
kubectl apply -f k8s/admin-service.yaml

# Apply the HPAs
kubectl apply -f k8s/hpa-frontend.yaml
kubectl apply -f k8s/hpa-backend.yaml
kubectl apply -f k8s/hpa-admin.yaml
```

## Step 6: Verify the Deployment

Check if all deployments, services, and HPAs are running:

```bash
# Check deployments
kubectl get deployments

# Check services
kubectl get svc

# Check HPAs
kubectl get hpa

# Check pods
kubectl get pods
```

## Step 7: Accessing the Services

To access your services in the browser:

```bash
# Get the Minikube IP
minikube ip

# Get the NodePort for each service
kubectl get svc
```

Then, access your services using:
- Frontend: `http://<minikube-ip>:<frontend-nodeport>`
- Admin: `http://<minikube-ip>:<admin-nodeport>`
- Backend (if exposed via NodePort): `http://<minikube-ip>:<backend-nodeport>`

Alternatively, use Minikube service command:

```bash
minikube service shopper-frontend-service
minikube service shopper-admin-service
```

## Step 8: Test the Autoscaler

To test the HPA, generate load on your services:

### Option 1: Create a Load Generator Pod

```bash
kubectl run -i --tty load-generator --rm --image=busybox /bin/sh

# Inside the pod, run:
while true; do wget -q -O- http://shopper-backend-service:5050/; done
```

### Option 2: If Load Generator Pod Already Exists

```bash
# Delete existing pod
kubectl delete pod load-generator

# Create a new one
kubectl run -i --tty load-generator --rm --image=busybox /bin/sh

# Or exec into the existing pod
kubectl exec -it load-generator -- /bin/sh

# Generate CPU load
while true; do :; done
```

### Monitor Autoscaling

Watch the HPA stats and pod count:

```bash
kubectl get hpa --watch
kubectl get pods --watch
```

## Troubleshooting

### 1. Pods Not Running

If pods are in a CrashLoopBackOff or Error state:

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

### 2. Services Not Accessible

Check if the services are properly exposed:

```bash
kubectl get svc
minikube ip
```

### 3. HPA Not Working

Verify metrics server is running and that the pods have resource requests defined:

```bash
kubectl get deployment metrics-server -n kube-system
kubectl describe hpa
```

### 4. Pod Network Issues

Check if pods can communicate with each other:

```bash
kubectl exec -it <pod-name> -- ping <service-name>
kubectl exec -it <pod-name> -- wget -O- <service-name>:<port>
```

## Conclusion

You've successfully deployed a multi-component Shopper application to a Kubernetes cluster with Horizontal Pod Autoscaling. The application will automatically scale based on CPU utilization, ensuring optimal resource usage and performance under varying loads.
