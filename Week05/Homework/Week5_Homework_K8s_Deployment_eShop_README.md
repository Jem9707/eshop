# 🚀 Week 5 Homework: Create Kubernetes Deployment for eShop Microservices

## 🧠 Objective

In this homework, you will create Kubernetes Deployment manifests for microservices from the eShopOnContainers project. You’ll focus on deploying the services — Services and networking will be handled in the next module.

---

## 🔗 Project Source

We are continuing with:

🔗 [Homework Project GitHub](https://github.com/msalemcode/Container_Operating_Systems.git

---

## ✅ Task Instructions

### Step 1: Create Deployment YAMLs

From the `/src/docker-compose.yml` , `/src/docker-compose.override.yml` and `.env` files, Create a `deployment.yaml` for following apps.

#### Core Services

- [ ] Catalog.API → `src/Services/Catalog/Catalog.API`
- [ ] Basket.API → `src/Services/Basket/Basket.API`
- [x] Ordering.API → `src/Services/Ordering/Ordering.API`
- [ ] Identity.API → `src/Services/Identity/Identity.API`
- [ ] Payment.API → `src/Services/Payment/Payment.API`

#### Frontend Services
- [ ] WebMVC → `src/Web/WebMVC`
- [ ] WebSPA → `src/Web/WebSPA`
- [ ] WebStatus → `src/Web/WebStatus`
- [ ] Gateway Mobil API → `src/ApiGateways/Mobile.Bff.Shopping/aggregator`
- [ ] Gateway Web API → `src/ApiGateways/Web.Bff.Shopping/aggregator`
- [ ] Ordering.SignalrHub → `src/ApiGateways/Web.Bff.Shopping/aggregator`
- [ ] Webhooks.API → `src/Web/WebhookClient`  
- [ ] Ordering.BackgroundTasks → `src/Services/Ordering/Ordering.BackgroundTasks`

## Open Source component
- [ ] mongo
- [ ] rabbitmq
- [ ] seq
- [ ] sqldata

Here is `ordering-api.yaml` deployment file
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ordering-api
  labels:
    app.kubernetes.io/name: ordering-api
    app.kubernetes.io/part-of: eshop
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: ordering-api
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ordering-api
        app.kubernetes.io/part-of: eshop
    spec:
      containers:
      - name: ordering-api
        image: <YOUR DOCKER HUB username>/ordering.api:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
        - containerPort: 81
        env:
        - name: ASPNETCORE_ENVIRONMENT
          value: Development
        - name: ASPNETCORE_URLS
          value: http://0.0.0.0:80
        - name: ConnectionString
          value: Server=sqldata;Database=Microsoft.eShopOnContainers.Services.OrderingDb;User
            Id=sa;Password=Pass@word
        - name: identityUrl
          value: http://identity-api
        - name: IdentityUrlExternal
          value: http://localhost:5105
        - name: EventBusConnection
          value: rabbitmq
        - name: UseCustomizationData
          value: 'True'
        - name: AzureServiceBusEnabled
          value: 'False'
        - name: CheckUpdateTime
          value: '30000'
        - name: UseLoadTest
          value: 'False'
        - name: Serilog__MinimumLevel__Override__Microsoft.eShopOnContainers.BuildingBlocks.EventBusRabbitMQ
          value: Verbose
        - name: Serilog__MinimumLevel__Override__ordering-api
          value: Verbose
        - name: PATH_BASE
          value: /ordering-api
        - name: GRPC_PORT
          value: '81'
        - name: PORT
          value: '80'

```
---
> [!WARNING]
> In this week we will deploy only Pods, Since Networking and communication via service is not covered yet.
> Expect Pod failures. The goal of this week is to write the deployment file and get Pod deploy even if it failed

---

### Step 4: Apply and Verify

```bash
kubectl apply -f ordering-api.yaml
kubectl get pods
kubectl describe deployment 
```




---

## 📝 Deliverables

- 1 Deployment YAML files
- Screenshots showing  pods running


---

## ⏱️ Deadline

Submit before next class.

---
