# 🌐 Week 6 Homework: Add Kubernetes Services for eShop Web Apps

## 🧠 Objective

In this assignment, you’ll build on your Week 5 Deployments by adding Kubernetes **Service** objects to expose your microservices inside the cluster. You’ll also deploy web applications from the eShopOnContainers project.

---

## 🔗 Reference

🔗 [Home Project GitHub](https://github.com/msalemcode/Container_Operating_Systems.git

---

## 🧰 Prerequisites

- Week 5 Deployments created and applied (e.g., `Catalog.API`, `Basket.API`)
- A running Kubernetes cluster (Minikube, KIND, etc.)
- Docker images already built and pushed to your registry

---

## ✅ Tasks

### Step 1: Add a Service for Each Microservice

For each microservice you deployed in Week 5, create a `Service` of type `ClusterIP`.

Example `catalog-api-service.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: catalog-api
  labels:
    app.kubernetes.io/name: catalog-api
    app.kubernetes.io/part-of: eshop
spec:
  type: ClusterIP
  selector:
    app.kubernetes.io/name: catalog-api
  ports:
  - name: p80
    port: 80
    targetPort: 80
  - name: p81
    port: 81
    targetPort: 81

```

Apply with:
```bash
kubectl apply -f catalog-api-service.yaml
```

---

### Step 2: Deploy a Services Application

Deploy all the services and make sure you can access the following  resources
- `basket-api`
- `catalog-api`
- `webwtatus`

```bash
kubectl.exe port-forward svc/basket-api 5500:80

kubectl.exe port-forward svc/catalog-api 5000:80

kubectl.exe port-forward svc/webstatus 5000:80
```


---

## 📝 Deliverables

- Deployment and Service YAMLs for both microservices (Week 5) and the Web App (Week 6)
- Screenshot showing pod and service status:
```bash
kubectl get pods
kubectl get svc
```
- Screenshot for Basket, Catalog Swagger API and webstatus


## ⏱️ Deadline

Submit before the next class session.

---
