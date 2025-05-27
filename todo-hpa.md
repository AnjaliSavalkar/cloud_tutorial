# Horizontal Pod Autoscaling for Todo App in Kubernetes

This guide explains how to set up Horizontal Pod Autoscaling (HPA) for a Todo application in a Kubernetes environment (specifically Minikube).

## Prerequisites

- Minikube installed
- kubectl installed
- Basic Todo app files (index.html, styles.css, app.js)

## Step 1: Start Minikube with Metrics Server

The Metrics Server is required for HPA to function properly.

```bash
# Start Minikube
minikube start

# Enable the metrics-server addon
minikube addons enable metrics-server

# Verify metrics-server is running
kubectl get pods -n kube-system | grep metrics-server
```

## Step 2: Create ConfigMap for Todo App Files

Since we're using a public Nginx image and injecting our app files, create a ConfigMap:

```bash
# Create ConfigMap from your todo app files
kubectl create configmap todo-app-files --from-file=index.html --from-file=styles.css --from-file=app.js
```

## Step 3: Create Deployment

Create a file named `todo-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: todo-app
  labels:
    app: todo-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: todo-app
  template:
    metadata:
      labels:
        app: todo-app
    spec:
      containers:
      - name: todo-app
        image: nginx:alpine
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 256Mi
        volumeMounts:
        - name: config-volume
          mountPath: /usr/share/nginx/html
      volumes:
      - name: config-volume
        configMap:
          name: todo-app-files
```

Apply the deployment:

```bash
kubectl apply -f todo-deployment.yaml
```

## Step 4: Create Service

Create a file named `todo-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: todo-app-service
spec:
  selector:
    app: todo-app
  ports:
  - port: 80
    targetPort: 80
  type: NodePort
```

Apply the service:

```bash
kubectl apply -f todo-service.yaml
```

## Step 5: Create Horizontal Pod Autoscaler

Create a file named `todo-hpa.yaml`:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: todo-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: todo-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

Apply the HPA:

```bash
kubectl apply -f todo-hpa.yaml
```

## Step 6: Verify and Monitor the HPA

Check the status of the HPA:

```bash
kubectl get hpa todo-app-hpa
```

You should see output similar to:
```
NAME           REFERENCE                 TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
todo-app-hpa   Deployment/todo-app       0%/70%    2         10        2          X
```

## Step 7: Access Your Todo App

Get the URL to access your Todo app:

```bash
minikube service todo-app-service --url   or
kubectl port-forward service/todo-service 8080:80

```

Or open it directly in your browser:

```bash
minikube service todo-app-service
```

## Step 8: Generate Load to Test Autoscaling

Create a file named `load-generator.yaml` to simulate traffic:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: load-generator
spec:
  replicas: 1
  selector:
    matchLabels:
      app: load-generator
  template:
    metadata:
      labels:
        app: load-generator
    spec:
      containers:
      - name: load-generator
        image: busybox
        command: ["/bin/sh", "-c"]
        args:
        - while true; do
            wget -q -O- http://todo-app-service;
            sleep 0.01;
          done
```

Apply the load generator:

```bash
kubectl apply -f load-generator.yaml
```

## Step 9: Monitor Autoscaling in Action

Watch the HPA metrics and how it scales your deployment:

```bash
kubectl get hpa todo-app-hpa --watch
```

In another terminal window, you can watch the pods being created:

```bash
kubectl get pods -w
```

You should see additional pods being created as the CPU utilization increases.

## Step 10: Clean Up When Done

Remove all the resources:

```bash
kubectl delete -f load-generator.yaml
kubectl delete -f todo-hpa.yaml
kubectl delete -f todo-service.yaml
kubectl delete -f todo-deployment.yaml
kubectl delete configmap todo-app-files
```

## Troubleshooting

### Metrics Not Showing

If HPA shows `<unknown>/70%` instead of actual metrics:

```bash
# Restart the metrics-server
minikube addons disable metrics-server
minikube addons enable metrics-server

# Wait a few minutes for metrics collection
```

### Pods Not Scaling

Check if your load is actually generating CPU usage:

```bash
kubectl top pods
```

### Other Issues

Check events and logs:

```bash
# Check events
kubectl get events --sort-by='.lastTimestamp'

# Check HPA description
kubectl describe hpa todo-app-hpa
```

## Understanding HPA Behavior

- HPA checks metrics every 15 seconds by default
- It avoids rapid scaling by considering average usage over a window
- Scale-down has a longer cooldown period than scale-up
- It may take several minutes to see the autoscaling take effect

## Alternative: Using Custom Docker Image

If you prefer to use your own Docker image instead of ConfigMaps:

1. Build and push your image to a registry like Docker Hub
2. Update the deployment to use your image:

```yaml
spec:
  containers:
  - name: todo-app
    image: yourusername/your-todo-app:latest
    # Remove the configMap volume mounts
```

This approach is recommended for production environments.
