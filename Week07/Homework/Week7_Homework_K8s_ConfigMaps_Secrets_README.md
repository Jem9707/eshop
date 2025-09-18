# 🔐 Week 7 Homework: ConfigMaps & Secrets with eShopOnContainers

## 🧠 Objective

This week, you'll externalize application configuration using **Kubernetes ConfigMaps** and **Secrets**. You'll update existing eShop microservice deployments to consume values such as environment settings and database connection strings from these resources.

---

## 🔗 Reference

🔗 [Home Project GitHub](https://github.com/msalemcode/Container_Operating_Systems.git

---

## ✅ Tasks

### Step 1: Create a ConfigMap for webshoppingapigw

 a microservice (e.g., `webshoppingapigw`) depends on envoy configuration. You can find configuration under `ApiGateways/Envoy/config/webshopping/envoy.yaml`

Turn `envoy.yaml` into a Configmap. 

Apply:
```bash
kubectl apply -f configmap.yaml
```

---

### Step 2: Create a Secret for DB Connection String

Example `secret.yaml`:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret
type: Opaque
stringData:
  ConnectionStrings__CatalogDb: "Server=catalog-db;Database=CatalogDb;User=sa;Password=MyPass@word;"
```

Apply:
```bash
kubectl apply -f secret.yaml
```

---

### Step 3: Update Deployment to Use ConfigMap and Secret

Update the `catalog-api` deployment to use values  Secret:


Apply updated deployment:
```bash
kubectl apply -f catalog-deployment.yaml
```

---

### Step 4: Create a ConfigMap for external link for DB Connection String
Create ConfigMap for the following links

``` bash
  PURCHASE_URL: http://webshoppingapigw
  ESHOP_STORAGE_CATALOG_URL: http://webshoppingapigw/c/api/v1/catalog/items/[0]/pic/
```

After applying the configmap. Update `webmvc` and `catalog-api`

---


> [!WARNING]
> In this week we will created configmap for envoy however we did not use it yet. Will use it next week




## 📝 Deliverables

- YAML files:
  - ConfigMap
  - Secret
  - Updated Deployment
  - screnshot for catalog and webmvc pod description

---

## ⏱️ Deadline

Submit before next class.

---

## 💬 Tips

- Use `kubectl describe pod <pod>` to check applied environment variables.
- For secret testing, use a test endpoint or log output inside the container.