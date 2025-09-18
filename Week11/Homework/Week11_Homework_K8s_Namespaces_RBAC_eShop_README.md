# 🔐 Week 11 Homework: Kubernetes Namespaces & RBAC with eShopOnContainers

## 🧠 Objective

This week, you will implement **Kubernetes Namespaces** and **Role-Based Access Control (RBAC)** for the eShopOnContainers microservices application. You'll organize services into logical namespaces and create proper access controls for different user roles, following Kubernetes security best practices.

---


## ✅ Tasks

### Step 1: Analyze Current eShop Architecture 

First, examine the current microservices structure:

```bash
# List all current services (if deployed)
kubectl get services --all-namespaces


```

**eShop Services Overview:**
- **Frontend Layer**: webmvc, webspa
- **API Gateways**: webshoppingapigw, mobileshoppingapigw ,webshoppingagg, mobileshoppingagg 
- **Business APIs**: catalog-api, basket-api, ordering-api, payment-api, identity-api,webhooks-api
- **Background Services**: ordering-backgroundtasks, ordering-signalrhub, webhooks-client
- **Monitor**:  webstatus
- **Data Layer**: sql-data, nosql-data, basket-data, rabbitmq

---

### Step 2: Design Namespace Strategy

Create a logical namespace organization for the eShop application:

**Recommended Namespace Structure:**

```yaml
# namespaces.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: eshop-frontend
  labels:
    tier: frontend
    app: eshop
---
apiVersion: v1
kind: Namespace
metadata:
  name: eshop-gateways
  labels:
    tier: gateway
    app: eshop
---
apiVersion: v1
kind: Namespace
metadata:
  name: eshop-apis
  labels:
    tier: api
    app: eshop
---
apiVersion: v1
kind: Namespace
metadata:
  name: eshop-background
  labels:
    tier: background
    app: eshop
---
apiVersion: v1
kind: Namespace
metadata:
  name: eshop-data
  labels:
    tier: data
    app: eshop
---
apiVersion: v1
kind: Namespace
metadata:
  name: eshop-monitoring
  labels:
    tier: monitoring
    app: eshop
```

Apply the namespaces:
```bash
kubectl apply -f namespaces.yaml
```

---

### Step 3: Create Service Accounts for Different Roles

Create service accounts for different user types:

```yaml
# service-accounts.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: eshop-developer
  namespace: eshop-apis
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: eshop-devops
  namespace: default
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: eshop-readonly
  namespace: default
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: eshop-frontend-deployer
  namespace: eshop-frontend
```

Add Secret for eshop-developer
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: eshop-sa-token
  namespace: eshop-apis
  annotations:
    kubernetes.io/service-account.name: eshop-developer
type: kubernetes.io/service-account-token
```

---

### Step 4: Define RBAC Roles

Create different roles with specific permissions:

```yaml
# rbac-roles.yaml
# Role for developers - can manage API services
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: eshop-apis
  name: api-developer
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps", "secrets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["pods/log", "pods/exec"]
  verbs: ["get", "list"]
---
# Role for frontend developers - limited to frontend namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: eshop-frontend
  name: frontend-developer
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get", "list"]
---
# ClusterRole for DevOps - can manage all namespaces
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: eshop-devops
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["apps"]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["networking.k8s.io"]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["rbac.authorization.k8s.io"]
  resources: ["*"]
  verbs: ["*"]
---
# ClusterRole for read-only access
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: eshop-readonly
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps", "namespaces"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get", "list"]
```

---

### Step 5: Create Role Bindings

Bind the roles to service accounts:

```yaml
# rbac-bindings.yaml
# Bind API developer role
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: api-developer-binding
  namespace: eshop-apis
subjects:
- kind: ServiceAccount
  name: eshop-developer
  namespace: eshop-apis
roleRef:
  kind: Role
  name: api-developer
  apiGroup: rbac.authorization.k8s.io
---
# Bind frontend developer role
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: frontend-developer-binding
  namespace: eshop-frontend
subjects:
- kind: ServiceAccount
  name: eshop-frontend-deployer
  namespace: eshop-frontend
roleRef:
  kind: Role
  name: frontend-developer
  apiGroup: rbac.authorization.k8s.io
---
# Bind DevOps cluster role
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: eshop-devops-binding
subjects:
- kind: ServiceAccount
  name: eshop-devops
  namespace: default
roleRef:
  kind: ClusterRole
  name: eshop-devops
  apiGroup: rbac.authorization.k8s.io
---
# Bind readonly cluster role
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: eshop-readonly-binding
subjects:
- kind: ServiceAccount
  name: eshop-readonly
  namespace: default
roleRef:
  kind: ClusterRole
  name: eshop-readonly
  apiGroup: rbac.authorization.k8s.io
```

---

### Step 6: Update Existing Deployments

Modify existing helm charts or YAML files to use namespaces. For example, update catalog-api:

```yaml
# Example: catalog-api deployment with namespace
apiVersion: apps/v1
kind: Deployment
metadata:
  name: catalog-api
  namespace: eshop-apis  # Add namespace
  labels:
    app: catalog-api
    tier: api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: catalog-api
  template:
    metadata:
      labels:
        app: catalog-api
    spec:
      serviceAccountName: eshop-developer  # Add service account
      containers:
      - name: catalog-api
        image: eshop/catalog.api:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: catalog-api
  namespace: eshop-apis  # Add namespace
spec:
  selector:
    app: catalog-api
  ports:
  - port: 80
    targetPort: 80
```

---


### Step 8: Test RBAC Permissions

Test different access levels:

```bash
# Get service account tokens
kubectl get secret -n eshop-apis | grep eshop-developer
kubectl describe secret <token-name> -n eshop-apis

# Test API developer permissions
kubectl auth can-i create deployments --namespace=eshop-apis --as=system:serviceaccount:eshop-apis:eshop-developer

# Test readonly permissions
kubectl auth can-i delete pods --as=system:serviceaccount:default:eshop-readonly

# Test cross-namespace access
kubectl auth can-i get pods --namespace=eshop-frontend --as=system:serviceaccount:eshop-apis:eshop-developer
```

---


## 🧪 Testing Your Implementation

### Test Namespace Organization:
```bash
# List all namespaces
kubectl get namespaces -l app=eshop

# Check pods per namespace
kubectl get pods --all-namespaces -l app.kubernetes.io/part-of=eshop

```

### Test RBAC:
```bash
# Test different permissions
kubectl auth can-i get pods --namespace=eshop-apis --as=system:serviceaccount:eshop-apis:eshop-developer
kubectl auth can-i delete deployments --namespace=eshop-frontend --as=system:serviceaccount:eshop-apis:eshop-developer
kubectl auth can-i create namespaces --as=system:serviceaccount:default:eshop-readonly

# View effective permissions
kubectl describe clusterrole eshop-devops
kubectl describe role api-developer -n eshop-apis
```

---

## 📝 Deliverables

Submit the following files and documentation:

### Required Files:
1. **namespaces.yaml** - Namespace definitions
2. **service-accounts.yaml** - Service account definitions  
3. **rbac-roles.yaml** - Role and ClusterRole definitions
4. **rbac-bindings.yaml** - RoleBinding and ClusterRoleBinding definitions


### Updated Deployments:
- All eShop service deployments updated with:
  - Proper namespace assignment
  - Service account references
  - Appropriate labels

### Screenshots:
- `kubectl get namespaces -l app.kubernetes.io/part-of=eshop`
- `kubectl get pods --all-namespaces -l app.kubernetes.io/part-of=eshop`  
- `kubectl get serviceaccounts --all-namespaces`
- RBAC testing commands showing different permission levels



## ⏱️ Deadline

Submit all deliverables before next class. Start early as RBAC can be tricky to get right!


