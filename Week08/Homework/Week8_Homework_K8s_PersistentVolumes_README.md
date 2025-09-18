# 💾 Week 8 Homework: Volumes & Persistent Storage

## 🧠 Objective

In this assignment, you’ll use **Kubernetes Volumes** and **PersistentVolumeClaims (PVCs)** to persist data for services in the eShopOnContainers project.

---

## ✅ Tasks

### Step 1: Create a PersistentVolume and PersistentVolumeClaim

Create basketdata PVC 

Example `basketdata-pvc.yaml`:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: basketdata-pvc

  labels:
    app.kubernetes.io/name: basketdata-pvc
    app.kubernetes.io/part-of: eshop
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

---

### Step 2: Attach Volume to a Microservice

Edit the `basketdata`  deployment and add pvc

```yaml
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: basketdata-pvc
```

---

### Step 3: Create PVC and  Update the following deployment
1- nosqldata
2- sqldata
3- webshoppingapigw: We created configmap, we need to map the configmap as volume

---


### Step 4: Access MVC web site

```bash
kubectl.exe port-forward svc/webmvc 5000:80
```


## 📝 Deliverables

- YAMLs:  PVC, updated Deployment
- Screenshot for MVC site


---