# 🌐 Week 10 Homework: Ingress Controller & Routing with eShopOnContainers

## 🧠 Objective

This week, you will build on your previous Kubernetes work by exposing one or more eShop microservices and frontends using an **Ingress Controller**. You’ll configure routing rules that allow access to these services via user-friendly hostnames.

---

## 🔗 Context from Previous Weeks

You should already have the following from earlier weeks:

- Week 5: Kubernetes Deployments for eShop microservices
- Week 6: Kubernetes Services exposing the microservices
- Week 7: ConfigMaps & Secrets for microservice config
- Week 8: Volumes for persistence (optional for this task)
- Week 9: Liveness/Readiness probes

---

## ✅ Tasks

### Step 1: Install Ingress Controller

For Minikube:
```bash
minikube addons enable ingress
```

For other environments: [NGINX Ingress Docs](https://kubernetes.github.io/ingress-nginx/deploy/)

Verify:
```bash
kubectl get pods -n ingress-nginx
```

---

### Step 2: Choose eShop Frontend(s)

You must expose at least one of the following via ingress:

- `WebMVC`
- `WebSPA`
- `WebStatus`

Ensure the Deployment and Service are already applied and working.

---

### Step 3: Define Ingress Resource

Example `eshop-ingress.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: eshop-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: webmvc.eshop.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webmvc
            port:
              number: 80
```

You may also route multiple paths or hosts (e.g., `catalog`, `basket`,`webstatus`,`webshoppingagg`).

---

### Step 4: Update Your Hosts File

If using Minikube, run:
```bash
echo "$(minikube ip) webmvc.eshop.local" | sudo tee -a /etc/hosts
minikube tunnel
curl -H "Host: webmvc.eshop.local" http://127.0.0.1/

```

---

## 📝 Deliverables

- Ingress YAML file
- Screenshot of:
  - `kubectl get ingress`
  - Curl output

---

## ⏱️ Deadline

Submit before next class.