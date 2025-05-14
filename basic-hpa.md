To **push your Shopper app** (with frontend and backend) to Docker Hub and **deploy it with Horizontal Pod Autoscaling (HPA)** in a Minikube Kubernetes cluster, follow these steps:

---

## ✅ Step 1: Structure Your App

Make sure your project directory looks like this:

```
shopper-app/
├── frontend/
│   ├── Dockerfile
│   └── (frontend source code)
├── backend/
│   ├── Dockerfile
│   └── (backend source code)
├── k8s/
│   ├── frontend-deployment.yaml
│   ├── backend-deployment.yaml
│   ├── frontend-service.yaml
│   ├── backend-service.yaml
│   ├── hpa-backend.yaml
│   └── hpa-frontend.yaml
```

---

## ✅ Step 2: Build and Push Docker Images to Docker Hub

1. **Login to Docker Hub**

   ```bash
   docker login
   ```

2. **Tag and push frontend**

   ```bash
   cd frontend
   docker build -t <your-dockerhub-username>/shopper-frontend:latest .
   docker push <your-dockerhub-username>/shopper-frontend:latest
   ```

3. **Tag and push backend**

   ```bash
   cd ../backend
   docker build -t <your-dockerhub-username>/shopper-backend:latest .
   docker push <your-dockerhub-username>/shopper-backend:latest
   ```

---

## ✅ Step 3: Write Kubernetes Manifests

**Example: `frontend-deployment.yaml`**

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
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
          limits:
            cpu: "500m"
```

**Example: `frontend-service.yaml`**

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
      port: 80
      targetPort: 80
```

Repeat similarly for backend.

---

## ✅ Step 4: Deploy to Minikube

```bash
kubectl apply -f k8s/frontend-deployment.yaml
kubectl apply -f k8s/frontend-service.yaml

kubectl apply -f k8s/backend-deployment.yaml
kubectl apply -f k8s/backend-service.yaml
```

---

## ✅ Step 5: Enable Metrics Server for HPA

Make sure the **metrics-server** is running:

```bash
minikube addons enable metrics-server
```

Check with:

```bash
kubectl get deployment metrics-server -n kube-system
```

---

## ✅ Step 6: Create HPA for Frontend/Backend

**Example: `hpa-frontend.yaml`**

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

Then apply it:

```bash
kubectl apply -f k8s/hpa-frontend.yaml
kubectl apply -f k8s/hpa-backend.yaml
```

---

## ✅ Step 7: Test the Autoscaler

Use this to load test:

```bash
kubectl run -i --tty load-generator --rm --image=busybox /bin/sh

# In the shell:
while true; do wget -q -O- http://shopper-backend-service:5000/; done
```

Then check:

```bash
kubectl get hpa
kubectl get pods
```

---

If you'd like, I can generate all YAML files for you—just confirm the ports and image names.
