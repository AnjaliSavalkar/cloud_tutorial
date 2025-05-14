# 🛍 Shopper App Deployment on Minikube

This guide documents how to deploy the Shopper app (with frontend, admin, and backend) on a Minikube Kubernetes cluster. It includes load testing with HPA (Horizontal Pod Autoscaler), handling .env, using Docker images, resolving common issues, and accessing the app.

---

## 📁 Project Structure

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



---



---

## 🚀 Start Minikube

bash
minikube start


---
docker login
cd frontend
docker build -t <your-dockerhub-username>/shopper-frontend:latest .
docker push <your-dockerhub-username>/shopper-frontend:latest
cd ../backend
docker build -t <your-dockerhub-username>/shopper-backend:latest .
docker push <your-dockerhub-username>/shopper-backend:latest


## 🐳 Build Docker Images

bash
# Backend
docker build -t shopper-backend ./backend

# Frontend
docker build -t shopper-frontend ./frontend

# Admin
docker build -t shopper-admin ./admin


---

## 🔄 Load Images into Minikube (if not using Docker Hub)

bash
minikube image load shopper-backend
minikube image load shopper-frontend
minikube image load shopper-admin


---

## 📦 Apply Deployments and Services

bash
kubectl apply -f k8s/backend-deployment.yaml
kubectl apply -f k8s/backend-service.yaml

kubectl apply -f k8s/frontend-deployment.yaml
kubectl apply -f k8s/frontend-service.yaml

kubectl apply -f k8s/admin-deployment.yaml
kubectl apply -f k8s/admin-service.yaml


---

## 🔍 Verify Everything Is Running

bash
kubectl get deployments
kubectl get pods
kubectl get svc


Example output:

| Name                    | Type       | Port      | NodePort  |
|-------------------------|------------|-----------|-----------|
| shopper-frontend-service | NodePort   | 5173/TCP  | 31588     |
| shopper-admin-service    | NodePort   | 5174/TCP  | 30818     |
| shopper-backend-service  | ClusterIP  | 5050/TCP  | —         |

---

## 🌐 Access the Application

First, get the Minikube IP:

bash
minikube ip


Assume the IP is 192.168.49.2. Access using:

- Frontend → http://192.168.49.2:31588
- Admin    → http://192.168.49.2:30818

> ❗Use *http*, not https. If the browser warns you, click Advanced → Continue.

---

## 📈 Add HPA (Horizontal Pod Autoscaler)

1. *Enable metrics server*:

bash
minikube addons enable metrics-server


2. *Apply HPA YAML*:

bash
kubectl apply -f k8s/hpa.yaml


3. *Check HPA*:

bash
kubectl get hpa


---

## 🧪 Load Testing (BusyBox)

Run a temporary load generator:

bash
kubectl run -i --tty load-generator --image=busybox /bin/sh


Then run inside:

sh
while true; do wget -q -O- http://shopper-backend-service:5050; done


Exit with Ctrl+C.

To delete it:

bash
kubectl delete pod load-generator


---

## 🔁 Reuse Existing Images & Deployments

- To restart a deployment:

bash
kubectl rollout restart deployment shopper-backend


- To update the image:

bash
kubectl set image deployment/shopper-backend backend=shopper-backend:latest


- To scale down or stop:

bash
kubectl scale deployment shopper-backend --replicas=0


---

## 🧯 Delete Services & Deployments

bash
kubectl delete -f k8s/


Or manually:

bash
kubectl delete deployment shopper-backend
kubectl delete service shopper-backend-service


---

## ❗ Common Errors

| Error                                               | Fix                                                      |
|-----------------------------------------------------|-----------------------------------------------------------|
| ERR_CONNECTION_REFUSED                            | Use http://<minikube-ip>:<NodePort> not https          |
| Error from server (AlreadyExists)                 | Delete pod: kubectl delete pod <name>                  |
| LoadBalancer IP <pending>                         | Use NodePort in Minikube                                 |
| kubectl stop error                                | Use kubectl delete or scale instead                  |
| HPA not scaling                                     | Check metrics server: kubectl top pods                 |

---

## ✅ Done!

Now your full-stack Shopper app runs on Kubernetes with autoscaling support. Happy deploying! 🚀
