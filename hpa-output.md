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

## Step 7: Accessing the Services in Web Browser

Let's walk through the exact process of accessing your services in a web browser with example outputs:

### 1. Get the Minikube IP address

```bash
$ minikube ip
```

Example output:
```
192.168.49.2
```

### 2. Get the NodePort for each service

```bash
$ kubectl get svc
```

Example output:
```
NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes                 ClusterIP   10.96.0.1        <none>        443/TCP          24h
shopper-admin-service      NodePort    10.106.253.20    <none>        5174:30818/TCP   1h
shopper-backend-service    ClusterIP   10.103.144.51    <none>        5050/TCP         1h
shopper-frontend-service   NodePort    10.111.130.50    <none>        5173:31588/TCP   1h
```

In this example output, note:
- Frontend service is exposed on port **31588** (NodePort)
- Admin service is exposed on port **30818** (NodePort)
- Backend service uses ClusterIP, so it's only accessible inside the cluster

### 3. Access the services in your web browser

Using the above example values, you would access:

#### Frontend (User Interface)
Open your browser and navigate to:
```
http://192.168.49.2:31588
```

#### Admin Dashboard
Open your browser and navigate to:
```
http://192.168.49.2:30818
```

### 4. Alternative: Use Minikube service command

This command will automatically open your default browser:

```bash
$ minikube service shopper-frontend-service
```

Example output:
```
|-----------|------------------------|-------------|---------------------------|
| NAMESPACE |          NAME          | TARGET PORT |            URL            |
|-----------|------------------------|-------------|---------------------------|
| default   | shopper-frontend-service |        5173 | http://192.168.49.2:31588 |
|-----------|------------------------|-------------|---------------------------|
🎉  Opening service default/shopper-frontend-service in default browser...
```

Similarly for the admin service:

```bash
$ minikube service shopper-admin-service
```

Example output:
```
|-----------|----------------------|-------------|---------------------------|
| NAMESPACE |         NAME         | TARGET PORT |            URL            |
|-----------|----------------------|-------------|---------------------------|
| default   | shopper-admin-service |        5174 | http://192.168.49.2:30818 |
|-----------|----------------------|-------------|---------------------------|
🎉  Opening service default/shopper-admin-service in default browser...
```

### Note on Backend Service

Since the backend service is defined with `type: ClusterIP` in our example, it's only accessible from inside the cluster. This is typically what you want for an API backend that should not be directly exposed to the internet.

If you need to access the backend API directly for testing:

1. Either change its service type to NodePort:
   ```yaml
   type: NodePort
   ```

2. Or use Kubernetes port-forwarding:
   ```bash
   $ kubectl port-forward svc/shopper-backend-service 5050:5050
   ```
   
   Example output:
   ```
   Forwarding from 127.0.0.1:5050 -> 5050
   Forwarding from [::1]:5050 -> 5050
   ```
   
   Then access via: http://localhost:5050

   #or

---

## 🧩 Step 1: Forward the Frontend Service (Port 5173)

```bash
kubectl port-forward service/shopper-frontend-service 5173:5173
````

🖥️ Now open in your browser: [http://localhost:5173](http://localhost:5173)

---

## 🧩 Step 2: Forward the Admin Service (Port 5174)

Open a **new terminal** and run:

```bash
kubectl port-forward service/shopper-admin-service 5174:5174
```

🖥️ Now open in your browser: [http://localhost:5174](http://localhost:5174)

---

## 🧩 Step 3: (Optional) Check Backend Service (Port 5050)

The backend is exposed as a `ClusterIP` and is only accessible inside the cluster by default.

To test it locally, run:

```bash
kubectl port-forward service/shopper-backend-service 5050:5050
```

🖥️ Test backend APIs at: [http://localhost:5050](http://localhost:5050)



```



## Step 8: Test the Autoscaler with Example Outputs

Let's test the Horizontal Pod Autoscaling with detailed examples:

### Option 1: Create a Load Generator Pod

```bash
$ kubectl run -i --tty load-generator --rm --image=busybox /bin/sh
```

Example output:
```
If you don't see a command prompt, try pressing enter.
/ #
```

Inside the pod, run:
```bash
/ # while true; do wget -q -O- http://shopper-backend-service:5050/; done
```

Example output (you'll see API responses repeating):
```
{"status":"success","message":"API is running"}
{"status":"success","message":"API is running"}
{"status":"success","message":"API is running"}
...
```

### Option 2: If Load Generator Pod Already Exists

If you get an error like:
```
Error from server (AlreadyExists): pods "load-generator" already exists
```

You can either delete it and recreate:
```bash
$ kubectl delete pod load-generator
```

Example output:
```
pod "load-generator" deleted
```

Or exec into the existing pod:
```bash
$ kubectl exec -it load-generator -- /bin/sh
```

Example output:
```
/ #
```

Then generate CPU load for admin service:
```bash
/ #   while true; do wget -q -O- http://shopper-admin-service:5174 > /dev/null; done 
```
This will run an infinite loop, consuming CPU resources.
Then generate CPU load for frontend service:
```bash
/ #  while true; do
  wget -q -O- http://shopper-frontend-service:5173 >/dev/null 2>&1
done   
```
This will run an infinite loop, consuming CPU resources.

### Monitor Autoscaling

In a separate terminal, watch the HPA stats:
```bash
$ kubectl get hpa --watch
```

Example output when load increases:
```
NAME           REFERENCE                 TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
hpa-backend    Deployment/shopper-backend  20%/50%    1         5         1          10m
hpa-backend    Deployment/shopper-backend  48%/50%    1         5         1          10m
hpa-backend    Deployment/shopper-backend  78%/50%    1         5         2          11m
hpa-backend    Deployment/shopper-backend  64%/50%    1         5         2          11m
hpa-backend    Deployment/shopper-backend  85%/50%    1         5         4          12m
```

Monitor pod scaling:
```bash
$ kubectl get pods --watch
```

Example output as pods scale up:
```
NAME                              READY   STATUS    RESTARTS   AGE
load-generator                    1/1     Running   0          5m23s
shopper-admin-74b887c6c-x8jhd     1/1     Running   0          30m
shopper-backend-d7f8554c4-2pngl   1/1     Running   0          30m
shopper-frontend-58bfd784f-qlpvz  1/1     Running   0          30m
shopper-backend-d7f8554c4-lr5pz   0/1     Pending   0          0s
shopper-backend-d7f8554c4-lr5pz   0/1     ContainerCreating   0          0s
shopper-backend-d7f8554c4-lr5pz   1/1     Running             0          3s
shopper-backend-d7f8554c4-6qknm   0/1     Pending             0          0s
shopper-backend-d7f8554c4-6qknm   0/1     ContainerCreating   0          0s
shopper-backend-d7f8554c4-6qknm   1/1     Running             0          3s
```

After stopping the load generator (by pressing Ctrl+C and exiting the pod), watch the pods scale back down after a few minutes:

```
NAME                              READY   STATUS    RESTARTS   AGE
shopper-admin-74b887c6c-x8jhd     1/1     Running   0          45m
shopper-backend-d7f8554c4-2pngl   1/1     Running   0          45m
shopper-backend-d7f8554c4-lr5pz   1/1     Running   0          15m
shopper-backend-d7f8554c4-6qknm   1/1     Running   0          14m
shopper-frontend-58bfd784f-qlpvz  1/1     Running   0          45m
shopper-backend-d7f8554c4-lr5pz   1/1     Terminating         0          18m
shopper-backend-d7f8554c4-6qknm   1/1     Terminating         0          17m
```

## Troubleshooting with Example Outputs

Here are common issues you might encounter and how to solve them:

### 1. Pods Not Running

If pods aren't starting correctly:

```bash
$ kubectl get pods
```

Example output with issues:
```
NAME                             READY   STATUS             RESTARTS      AGE
shopper-admin-74b887c6c-x8jhd    1/1     Running           0             30m
shopper-backend-d7f8554c4-2pngl  0/1     CrashLoopBackOff   5 (30s ago)   10m
shopper-frontend-58bfd784f-qlpvz 1/1     Running           0             30m
```

To diagnose the problem:

```bash
$ kubectl describe pod shopper-backend-d7f8554c4-2pngl
```

Example output:
```
Name:             shopper-backend-d7f8554c4-2pngl
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Wed, 14 May 2025 15:30:10 +0000
Labels:           app=shopper-backend
                  pod-template-hash=d7f8554c4
Status:           Running
IP:               172.17.0.5
...
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  10m                default-scheduler  Successfully assigned default/shopper-backend-d7f8554c4-2pngl to minikube
  Normal   Pulled     9m (x3 over 10m)   kubelet            Container image "yourusername/shopper-backend:latest" already present on machine
  Normal   Created    9m (x3 over 10m)   kubelet            Created container shopper-backend
  Normal   Started    9m (x3 over 10m)   kubelet            Started container shopper-backend
  Warning  BackOff    2m (x33 over 10m)  kubelet            Back-off restarting failed container
```

Check the container logs for error messages:

```bash
$ kubectl logs shopper-backend-d7f8554c4-2pngl
```

Example output:
```
Error: MONGO_URI environment variable is not set correctly
MongoDB connection failed: MongoNetworkError: failed to connect to server...
```

Solution: Fix the environment variables in the backend-deployment.yaml file and reapply.

### 2. Services Not Accessible

If you can't access your services in the browser:

First, check if the services are properly created:

```bash
$ kubectl get svc
```

Example output:
```
NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes                 ClusterIP   10.96.0.1        <none>        443/TCP          24h
shopper-admin-service      NodePort    10.106.253.20    <none>        5174:30818/TCP   1h
shopper-backend-service    ClusterIP   10.103.144.51    <none>        5050/TCP         1h
shopper-frontend-service   NodePort    10.111.130.50    <none>        5173:31588/TCP   1h
```

Verify the Minikube IP:

```bash
$ minikube ip
```

Example output:
```
192.168.49.2
```

Try accessing the service directly with curl:

```bash
$ curl http://192.168.49.2:31588
```

If this works but the browser doesn't, check for browser security settings or try a different browser.

### 3. HPA Not Working

If autoscaling doesn't work as expected:

First, check if metrics server is running:

```bash
$ kubectl get deployment metrics-server -n kube-system
```

Example output:
```
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
metrics-server   1/1     1            1           1h
```

Check if HPAs are getting metrics:

```bash
$ kubectl describe hpa hpa-backend
```

Example output with issues:
```
Name:                                                  hpa-backend
Namespace:                                             default
Reference:                                             Deployment/shopper-backend
Metrics:                                               ( current / target )
  resource cpu on pods  (as a percentage of request):  <unknown> / 50%
Min replicas:                                          1
Max replicas:                                          5
Deployment pods:                                       1 current / 0 desired
Conditions:
  Type            Status  Reason               Message
  ----            ------  ------               -------
  AbleToScale     True    SucceededGetScale    the HPA controller was able to get the target's current scale
  ScalingActive   False   FailedGetResourceMetric  the HPA was unable to compute the replica count: unable to get metrics for resource cpu: unable to fetch metrics from resource metrics API: the server could not find the requested resource (get pods.metrics.k8s.io)
Events:
  Type     Reason                Age                   From                       Message
  ----     ------                ----                  ----                       -------
  Warning  FailedGetResourceMetric  5m (x12 over 15m)  horizontal-pod-autoscaler  unable to get metrics for resource cpu: unable to fetch metrics from resource metrics API: the server could not find the requested resource (get pods.metrics.k8s.io)
```

Solution: Restart the metrics server:

```bash
$ kubectl rollout restart deployment metrics-server -n kube-system
```

Example output:
```
deployment.apps/metrics-server restarted
```

### 4. Pod Network Issues

If pods can't communicate with each other:

Test network connectivity from a pod:

```bash
$ kubectl exec -it shopper-frontend-58bfd784f-qlpvz -- wget -O- shopper-backend-service:5050
```

Example output with successful connection:
```
Connecting to shopper-backend-service:5050 (10.103.144.51:5050)
{"status":"success","message":"API is running"}
```

Example output with connection failure:
```
Connecting to shopper-backend-service:5050 (10.103.144.51:5050)
wget: can't connect to remote host (10.103.144.51): Connection refused
command terminated with exit code 1
```

In case of connection issues, check if the backend service and pods are running correctly:

```bash
$ kubectl get endpoints shopper-backend-service
```

Example output:
```
NAME                     ENDPOINTS           AGE
shopper-backend-service  172.17.0.5:5050     1h
```

If no endpoints are shown, the pods might not be labeled correctly or the service selector might be wrong.

## Conclusion

You've successfully deployed a multi-component Shopper application to a Kubernetes cluster with Horizontal Pod Autoscaling. The application will automatically scale based on CPU utilization, ensuring optimal resource usage and performance under varying loads.
