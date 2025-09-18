# 📈 Week 12 Homework: Kubernetes Autoscaling with eShopOnContainers

## 🧠 Objective

This week, you will implement **Horizontal Pod Autoscaling (HPA)** and **Vertical Pod Autoscaling (VPA)** for the eShopOnContainers microservices. You'll learn to automatically scale your applications based on resource utilization and demand, ensuring optimal performance and cost efficiency.

---


## ✅ Tasks

### Step 1: Prerequisites Setup

#### Install Metrics Server (Required for HPA):
```bash
# For most Kubernetes clusters
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# For Minikube
minikube addons enable metrics-server

# Verify metrics server is running
kubectl get pods -n kube-system | grep metrics-server
```

#### Install VPA (Optional but Recommended):
```bash
# (recommended for resource metrics)
minikube addons enable metrics-server

# Install VPA CRDs and components
kubectl apply -f https://github.com/kubernetes/autoscaler/blob/master/vertical-pod-autoscaler/deploy/vpa-v1-crd-gen.yaml

# Verify
kubectl get crd | findstr /I verticalpodautoscalers

```

---

### Step 2: Add Resource Requests/Limits to Deployments

Before implementing autoscaling, ensure your deployments have proper resource specifications:

```yaml
# Example: Updated catalog-api deployment with resources
apiVersion: apps/v1
kind: Deployment
metadata:
  name: catalog-api
  namespace: eshop-apis
spec:
  replicas: 2
  selector:
    matchLabels:
      app: catalog-api
  template:
    metadata:
      labels:
        app: catalog-api
    spec:
      containers:
      - name: catalog-api
        image: eshop/catalog.api:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

---

### Step 3: Implement Horizontal Pod Autoscaling (HPA)

#### HPA for Frontend Services (CPU-based):
```yaml
# hpa-frontend.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webmvc-hpa
  namespace: eshop-frontend
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webmvc
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
      - type: Pods
        value: 4
        periodSeconds: 15
      selectPolicy: Max
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webspa-hpa
  namespace: eshop-frontend
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webspa
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
```

#### HPA for API Services (Multi-metric):
```yaml
# hpa-apis.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: catalog-api-hpa
  namespace: eshop-apis
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: catalog-api
  minReplicas: 2
  maxReplicas: 15
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 75
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Pods
        value: 2
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Pods
        value: 1
        periodSeconds: 60
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: basket-api-hpa
  namespace: eshop-apis
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: basket-api
  minReplicas: 1
  maxReplicas: 8
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 65
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: ordering-api-hpa
  namespace: eshop-apis
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ordering-api
  minReplicas: 2
  maxReplicas: 12
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 70
```

---

### Step 4: Implement Vertical Pod Autoscaling (VPA)

#### VPA for Data Services:
```yaml
# vpa-data-services.yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: sql-data-vpa
  namespace: eshop-data
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: sql-data
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: sql-server
      maxAllowed:
        cpu: 2
        memory: 4Gi
      minAllowed:
        cpu: 100m
        memory: 128Mi
      controlledResources: ["cpu", "memory"]
---
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: rabbitmq-vpa
  namespace: eshop-data
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: rabbitmq
  updatePolicy:
    updateMode: "Initial"  # Only set resources for new pods
  resourcePolicy:
    containerPolicies:
    - containerName: rabbitmq
      maxAllowed:
        cpu: 1
        memory: 2Gi
      minAllowed:
        cpu: 50m
        memory: 64Mi
```

#### VPA for API Services (Recommendation Mode):
```yaml
# vpa-recommendations.yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: catalog-api-vpa
  namespace: eshop-apis
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: catalog-api
  updatePolicy:
    updateMode: "Off"  # Only provide recommendations
---
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: payment-api-vpa
  namespace: eshop-apis
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: payment-api
  updatePolicy:
    updateMode: "Off"
  resourcePolicy:
    containerPolicies:
    - containerName: payment-api
      controlledResources: ["memory"]  # Only manage memory, not CPU
```

---

### Step 5: Create Load Testing Scenarios

#### Install Load Testing Tools:
```bash
kubectl run load-test --image=busybox -it --rm --restart=Never -- /bin/sh
```

#### Load Test Scripts:
```bash
wget https://hey-release.s3.us-east-2.amazonaws.com/hey_linux_amd64
chmod +x hey_linux_amd64
mv hey_linux_amd64 /bin/hey

# load-test-webmvc.sh
#!/bin/bash
echo "Starting load test for WebMVC..."

# Get the external IP/URL of your ingress
WEBMVC_URL="http://webmvc.eshop.local"

# Start multiple concurrent load tests
echo "Running load test for 5 minutes with 50 concurrent connections..."
hey -z 5m -c 50 -q 10 $WEBMVC_URL &

echo "Running burst load test..."
hey -z 2m -c 100 -q 20 $WEBMVC_URL/catalog &

wait
echo "Load tests completed!"
```

```bash
# load-test-apis.sh
#!/bin/bash
echo "Starting API load tests..."

# Test Catalog API
CATALOG_URL="http://eshop.local/catalog-api"
echo "Testing Catalog API..."
hey -z 3m -c 30 -q 15 $CATALOG_URL/api/v1/catalog/items &

# Test Basket API  
BASKET_URL="http://eshop.local/basket-api"
echo "Testing Basket API..."
hey -z 3m -c 20 -q 10 $BASKET_URL/api/v1/basket &

# Test Ordering API
ORDERING_URL="http://eshop.local/ordering-api"
echo "Testing Ordering API..."
hey -z 3m -c 25 -q 12 $ORDERING_URL/api/v1/orders &

wait
echo "API load tests completed!"
```

---

### Step 6: Monitor Autoscaling Behavior

#### Monitor HPA Status:
```bash
# Watch HPA scaling in real-time
kubectl get hpa --all-namespaces -w

# Get detailed HPA information
kubectl describe hpa webmvc-hpa -n eshop-frontend
kubectl describe hpa catalog-api-hpa -n eshop-apis

# Check scaling events
kubectl get events --field-selector reason=SuccessfulRescale -A
```

#### Monitor VPA Recommendations:
```bash
# Get VPA status and recommendations
kubectl get vpa --all-namespaces
kubectl describe vpa catalog-api-vpa -n eshop-apis

# View VPA recommendations
kubectl get vpa catalog-api-vpa -n eshop-apis -o yaml
```

#### Monitor Resource Usage:
```bash
# Check current resource usage
kubectl top pods --all-namespaces
kubectl top nodes

# Monitor specific namespace
kubectl top pods -n eshop-apis
kubectl top pods -n eshop-frontend
```

## 📝 Deliverables

Submit the following files and documentation:

### Required Files:
1. **Updated Deployments** with proper resource requests/limits for at least 5 services
2. **hpa-frontend.yaml** - HPA for frontend services
3. **hpa-apis.yaml** - HPA for API services  
4. **vpa-data-services.yaml** - VPA for data layer services
5. **vpa-recommendations.yaml** - VPA in recommendation mode
7. **load-test scripts** - Shell scripts for load testing

### Screenshots:
- `kubectl get hpa --all-namespaces` (before and after load test)
- `kubectl top pods --all-namespaces` during load test
- `kubectl get events` showing scaling events
- Load test results (hey command output)
- `kubectl describe vpa <vpa-name>` showing recommendations



---

## ⏱️ Deadline

Submit all deliverables before next class. Plan for extended testing time!

