# ❤️‍🩹 Week 9 Homework: Liveness and Readiness Probes

## 🧠 Objective

You’ll learn how Kubernetes keeps apps healthy using **liveness** and **readiness** probes. You will add probes to existing eShopOnContainers microservice deployments.

---

## ✅ Tasks

### Step 1: Review Docker Compose from Week3

Review the docker compose override files and note containers with '/hc' path

---

### Step 2: Add Probes to Deployment

Update `containers` in your deployment YAML to include probes:

```yaml
livenessProbe:
  httpGet:
    path: /hc
    port: 80
  initialDelaySeconds: 10
  periodSeconds: 5

readinessProbe:
  httpGet:
    path: /hc
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 3
```



---

### Step 3: Apply and Observe

```bash
kubectl apply -f deployment.yaml
kubectl describe pod <pod-name>
```

Watch how Kubernetes restarts containers or keeps them out of service based on probe results.

---

### Step 4: Check Web Status site
using Port forward access webstatus site and confirm all apps are up and running.


Watch how Kubernetes restarts containers or keeps them out of service based on probe results.

---




## 📝 Deliverables

- Updated Deployment YAML with probes
- Screenshot for 'webstatus' site


---