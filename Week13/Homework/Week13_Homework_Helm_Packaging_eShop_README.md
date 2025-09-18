# 📦 Week 13 Homework: Helm Packaging & Orchestration for eShopOnContainers

## 🧠 Objective

This week, you will learn **Helm fundamentals** and create your first **Helm Charts** for the eShopOnContainers application. You'll convert your existing Kubernetes YAML files into reusable, configurable Helm templates, and package the entire application for easy deployment and management.

---


## ✅ Tasks

### Step 1: Install and Setup Helm

First, install Helm 3 on your system:

```bash
# Install Helm (choose your platform)
# macOS
brew install helm

# Linux
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Windows (PowerShell)
choco install kubernetes-helm

# Verify installation
helm version
```

Add Helm repositories for dependencies:
```bash
# Add common repositories
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add stable https://charts.helm.sh/stable
helm repo update
```

---

### Step 2: Create Your First Helm Chart

Start by creating a simple chart for one microservice to learn Helm basics:

```bash
# Create your first chart for the Catalog API
helm create catalog-api-chart
cd catalog-api-chart

# Explore the generated structure
ls -la
cat Chart.yaml
cat values.yaml
```

#### Understanding Chart Structure:
```
catalog-api-chart/
├── Chart.yaml          # Chart metadata (name, version, description)
├── values.yaml          # Default configuration values
├── templates/           # Kubernetes YAML templates
│   ├── deployment.yaml  # Deployment template
│   ├── service.yaml     # Service template
│   ├── ingress.yaml     # Ingress template (optional)
│   ├── serviceaccount.yaml
│   ├── hpa.yaml         # HorizontalPodAutoscaler (from Week 12)
│   ├── NOTES.txt        # Post-installation notes
│   └── tests/
│       └── test-connection.yaml
└── .helmignore          # Files to ignore when packaging
```

**Key Files Explained:**
- **Chart.yaml**: Contains chart metadata (name, version, dependencies)
- **values.yaml**: Default values that can be overridden during installation
- **templates/**: Directory containing Kubernetes resource templates
- **NOTES.txt**: Instructions displayed after chart installation

---

### Step 3: Convert Kubernetes YAML to Helm Templates

Now you'll convert your existing Kubernetes YAML files into Helm templates. Start with the Catalog API:

#### Update Chart.yaml:
```yaml
# Chart.yaml
apiVersion: v2
name: catalog-api
fullname: catalog-api
description: A Helm chart for eShop Catalog API microservice
type: application
version: 0.1.0
appVersion: "1.0"


```

#### Create Your First Template:

Copy your existing Catalog API deployment YAML and convert it to a template. Replace hardcoded values with template variables:

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "catalog-api.fullname" . }}
  namespace: {{ .Values.namespace | default "default" }}
  labels:
    {{- include "catalog-api.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "catalog-api.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "catalog-api.selectorLabels" . | nindent 8 }}
    spec:
      serviceAccountName: {{ include "catalog-api.serviceAccountName" . }}
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
            - name: grpc
              containerPort: 81
              protocol: TCP
          env:
            - name: ASPNETCORE_ENVIRONMENT
              value: {{ .Values.environment }}
            - name: ConnectionString
              value: {{ .Values.connectionString }}
            - name: PicBaseUrl
              value: {{ .Values.PicBaseUrl }}  
            - name: EventBusConnection
              value: {{ .Values.EventBusConnection }}  
            - name: UseCustomizationData
              value: {{ printf "%v" .Values.UseCustomizationData  | quote }}
            - name: AzureServiceBusEnabled
              value: {{ printf "%v" .Values.AzureServiceBusEnabled | quote }}  
            - name: AzureStorageEnabled
              value: {{ printf "%v" .Values.AzureStorageEnabled | quote }}  
            - name: GRPC_PORT
              value: {{ printf "%v" .Values.GRPC_PORT | quote }}  
            - name: PORT
              value: {{ printf "%v" .Values.PORT | quote }}  
            - name: PATH_BASE
              value: {{ .Values.PATH_BASE | quote }}  
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          {{- if .Values.probes.liveness.enabled }}
          livenessProbe:
            httpGet:
              path: {{ .Values.probes.liveness.path }}
              port: http
            initialDelaySeconds: {{ .Values.probes.liveness.initialDelaySeconds }}
            periodSeconds: {{ .Values.probes.liveness.periodSeconds }}
          {{- end }}
          {{- if .Values.probes.readiness.enabled }}
          readinessProbe:
            httpGet:
              path: {{ .Values.probes.readiness.path }}
              port: http
            initialDelaySeconds: {{ .Values.probes.readiness.initialDelaySeconds }}
            periodSeconds: {{ .Values.probes.readiness.periodSeconds }}
          {{- end }}
```

---

### Step 4: Create Values File for Catalog API

Create a `values.yaml` file that defines all configurable parameters:

```yaml
# values.yaml
# Default values for catalog-api
replicaCount: 1

image:
  repository: magdysalem/catalog.api
  pullPolicy: IfNotPresent
  tag: "latest"

# Namespace to deploy to (from Week 11)
namespace: eshop-apis

# Service account (from Week 11)
serviceAccount:
  create: true
  name: ""

# Service configuration
service:
  type: ClusterIP
  port: 80
  grpcPort: 81

# Resource limits and requests
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 100m
    memory: 128Mi

# Autoscaling configuration (from Week 12)
autoscaling:
  enabled: true
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80

# Ingress configuration (from Week 10)
ingress:
  enabled: true
  className: nginx
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  hosts:
    - host: catalog-api.eshop-apis
      paths:
        - path: /
          pathType: Prefix
  tls: []

# Probes configuration (from Week 9)
probes:
  liveness:
    enabled: true
    path: /hc
    initialDelaySeconds: 30
    periodSeconds: 10
  readiness:
    enabled: true
    path: /hc
    initialDelaySeconds: 10
    periodSeconds: 10

# Application configuration
environment: Development
connectionString: "Server=sql-data.eshop-data;Database=CatalogDb;User=sa;Password=Pass@word"
PicBaseUrl: "http://webshoppingapigw.eshop-gateways/c/api/v1/catalog/items/[0]/pic/"
EventBusConnection: "rabbitmq.eshop-data"
UseCustomizationData: true
AzureServiceBusEnabled: false
AzureStorageEnabled: false
GRPC_PORT: 81
PORT: 80
PATH_BASE: /catalog-api

# Node selector, tolerations, and affinity
nodeSelector: {}
tolerations: []
affinity: {}

# Pod Disruption Budget
podDisruptionBudget:
  enabled: true
  minAvailable: 1

# Network Policy (Bonus)
networkPolicy:
  enabled: false
```

---

### Step 5: Create Additional Templates

Now create the remaining Kubernetes resource templates:

#### Service Template:
```yaml
# templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "catalog-api.fullname" . }}
  namespace: {{ .Values.namespace | default "default" }}
  labels:
    {{- include "catalog-api.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
    - port: {{ .Values.service.grpcPort }}
      targetPort: grpc
      protocol: TCP
      name: grpc
  selector:
    {{- include "catalog-api.selectorLabels" . | nindent 4 }}
```

#### HPA Template (from Week 12):
```yaml
# templates/hpa.yaml
{{- if .Values.autoscaling.enabled }}
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ include "catalog-api.fullname" . }}
  namespace: {{ .Values.namespace | default "default" }}
  labels:
    {{- include "catalog-api.labels" . | nindent 4 }}
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ include "catalog-api.fullname" . }}
  minReplicas: {{ .Values.autoscaling.minReplicas }}
  maxReplicas: {{ .Values.autoscaling.maxReplicas }}
  metrics:
    {{- if .Values.autoscaling.targetCPUUtilizationPercentage }}
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: {{ .Values.autoscaling.targetCPUUtilizationPercentage }}
    {{- end }}
    {{- if .Values.autoscaling.targetMemoryUtilizationPercentage }}
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: {{ .Values.autoscaling.targetMemoryUtilizationPercentage }}
    {{- end }}
{{- end }}
```

#### Ingress Template (from Week 10):
```yaml
# templates/ingress.yaml
{{- if .Values.ingress.enabled -}}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "catalog-api.fullname" . }}
  namespace: {{ .Values.namespace | default "default" }}
  labels:
    {{- include "catalog-api.labels" . | nindent 4 }}
  {{- with .Values.ingress.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
spec:
  {{- if .Values.ingress.className }}
  ingressClassName: {{ .Values.ingress.className }}
  {{- end }}
  {{- if .Values.ingress.tls }}
  tls:
    {{- range .Values.ingress.tls }}
    - hosts:
        {{- range .hosts }}
        - {{ . | quote }}
        {{- end }}
      secretName: {{ .secretName }}
    {{- end }}
  {{- end }}
  rules:
    {{- range .Values.ingress.hosts }}
    - host: {{ .host | quote }}
      http:
        paths:
          {{- range .paths }}
          - path: {{ .path }}
            pathType: {{ .pathType }}
            backend:
              service:
                name: {{ include "catalog-api.fullname" $ }}
                port:
                  number: {{ $.Values.service.port }}
          {{- end }}
    {{- end }}
{{- end }}
```

#### ServiceAccount Template:
```yaml
# templates/serviceaccount.yaml
{{- if .Values.serviceAccount.create }}
apiVersion: v1
kind: ServiceAccount
metadata:
  name: {{ include "catalog-api.serviceAccountName" . }}
  labels:
    {{- include "catalog-api.labels" . | nindent 4 }}
  {{- with .Values.serviceAccount.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
automountServiceAccountToken: {{ default true .Values.serviceAccount.automountServiceAccountToken }}
{{- end }}


```
---

### Step 6: Create Helper Templates

Create template helpers in `templates/_helpers.tpl`:

```yaml
{{/*
Expand the name of the chart.
*/}}
{{- define "catalog-api.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
*/}}
{{- define "catalog-api.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
Create chart name and version as used by the chart label.
*/}}
{{- define "catalog-api.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Common labels
*/}}
{{- define "catalog-api.labels" -}}
helm.sh/chart: {{ include "catalog-api.chart" . }}
{{ include "catalog-api.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "catalog-api.selectorLabels" -}}
app.kubernetes.io/name: {{ include "catalog-api.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}

{{/*
Create the name of the service account to use
*/}}
{{- define "catalog-api.serviceAccountName" -}}
{{- if .Values.serviceAccount.create }}
{{- default (include "catalog-api.fullname" .) .Values.serviceAccount.name }}
{{- else }}
{{- default "default" .Values.serviceAccount.name }}
{{- end }}
{{- end }}
```

---

### Step 7: Test and Validate Your Chart

Before deploying, validate your chart:

#### Lint Your Chart:
```bash
# Navigate to your chart directory
cd catalog-api-chart

# Lint the chart for errors
helm lint .

# Validate templates render correctly
helm template my-catalog ./
```

#### Test Template Rendering:
```bash
# Test with default values
helm template test-catalog . --debug

# Test with custom values
helm template test-catalog . \
  --set replicaCount=3 \
  --set image.tag=v2.0 \
  --set autoscaling.enabled=false

# Test namespace assignment
helm template test-catalog . \
  --set namespace=eshop-apis \
  --debug
```

#### Create Environment-Specific Values:

```yaml
# values-dev.yaml
namespace: eshop-dev
replicaCount: 1
autoscaling:
  enabled: false
resources:
  requests:
    cpu: 50m
    memory: 64Mi
  limits:
    cpu: 200m
    memory: 128Mi
environment: Development
connectionString: "Server=sqldata.eshop-data;Database=Microsoft.eShopOnContainers.Services.CatalogDb;User Id=sa;Password=Pass@word"
PicBaseUrl: "http://webshoppingapigw.eshop-gateways/c/api/v1/catalog/items/[0]/pic/"
EventBusConnection: "rabbitmq.eshop-data"
UseCustomizationData: true
AzureServiceBusEnabled: false
AzureStorageEnabled: false
GRPC_PORT: 81
PORT: 80
PATH_BASE: /catalog-api

```

```yaml
# values-prod.yaml  
namespace: eshop-prod
replicaCount: 3
autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 15
resources:
  requests:
    cpu: 200m
    memory: 256Mi
  limits:
    cpu: 1000m
    memory: 1Gi
environment: Production
connectionString: "Server=sql-prod.company.com;Database=CatalogDb;User=cataloguser;Password=${DB_PASSWORD}"
EventBusConnection: rabbitmq.eshop-data
UseCustomizationData: true
AzureServiceBusEnabled: true
AzureStorageEnabled: true
ingress:
  hosts:
    - host: catalog-api.company.com
      paths:
        - path: /
          pathType: Prefix
```

---

### Step 8: Deploy and Test Your Chart

Now deploy your chart to Kubernetes:

#### Deploy to Development:
```bash
# Create namespace first (from Week 11)
kubectl create namespace eshop-dev

# Install the chart
helm install catalog-dev ./catalog-api-chart \
  --namespace eshop-dev \
  --values values-dev.yaml

# Check deployment status
helm status catalog-dev --namespace eshop-dev
kubectl get pods -n eshop-dev
```

#### Test Your Deployment:
```bash
# Check if pods are running
kubectl get pods -n eshop-dev -l app.kubernetes.io/name=catalog-api

# Check HPA (if enabled)
kubectl get hpa -n eshop-dev

# Test the service
kubectl port-forward svc/catalog-dev-catalog-api 8080:80 -n eshop-dev
# Visit http://localhost:8080/hc
# Visit http://localhost:8080
```

---

### Step 9: Create Charts for More Microservices

Now that you understand Helm basics, create charts for additional eShop services:

#### Create Basket API Chart:
```bash
# Create second chart
helm create basket-api-chart
cd basket-api-chart

# Update Chart.yaml
# Copy and modify templates from catalog-api-chart
# Update values.yaml for basket-specific configuration
```

#### Create WebMVC Chart:
```bash
helm create webmvc-chart
cd webmvc-chart
```


#### Management Commands:
```bash
# Install multiple charts
helm install catalog-dev ./catalog-api-chart -n eshop-dev --values catalog-values-dev.yaml
helm install basket-dev ./basket-api-chart -n eshop-dev --values basket-values-dev.yaml
helm install webmvc-dev ./webmvc-chart -n eshop-frontend --values webmvc-values-dev.yaml

# List all releases
helm list --all-namespaces

# Upgrade a chart
helm upgrade catalog-dev ./catalog-api-chart -n eshop-dev

# Rollback if needed
helm rollback catalog-dev 1 -n eshop-dev
```

---

### Step 10: Package and Share Your Charts

Learn how to package your charts for distribution:

#### Package Individual Charts:
```bash
# Package your catalog-api chart
helm package ./catalog-api-chart

# This creates: catalog-api-0.1.0.tgz

# Package basket-api chart
helm package ./basket-api-chart

# Package webmvc chart
helm package ./webmvc-chart
```

#### Create NOTES.txt for User Instructions:
```yaml
# templates/NOTES.txt
1. Get the application URL by running these commands:
{{- if .Values.ingress.enabled }}
{{- range $host := .Values.ingress.hosts }}
  {{- range .paths }}
  http{{ if $.Values.ingress.tls }}s{{ end }}://{{ $host.host }}{{ .path }}
  {{- end }}
{{- end }}
{{- else if contains "NodePort" .Values.service.type }}
  export NODE_PORT=$(kubectl get --namespace {{ .Values.namespace }} -o jsonpath="{.spec.ports[0].nodePort}" services {{ include "catalog-api.fullname" . }})
  export NODE_IP=$(kubectl get nodes --namespace {{ .Values.namespace }} -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
{{- else if contains "LoadBalancer" .Values.service.type }}
     NOTE: It may take a few minutes for the LoadBalancer IP to be available.
           You can watch the status of by running 'kubectl get --namespace {{ .Values.namespace }} svc -w {{ include "catalog-api.fullname" . }}'
  export SERVICE_IP=$(kubectl get svc --namespace {{ .Values.namespace }} {{ include "catalog-api.fullname" . }} --template "{{"{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}"}}")
  echo http://$SERVICE_IP:{{ .Values.service.port }}
{{- else if contains "ClusterIP" .Values.service.type }}
  export POD_NAME=$(kubectl get pods --namespace {{ .Values.namespace }} -l "app.kubernetes.io/name={{ include "catalog-api.name" . }},app.kubernetes.io/instance={{ .Release.Name }}" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace {{ .Values.namespace }} $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace {{ .Values.namespace }} port-forward $POD_NAME 8080:$CONTAINER_PORT
{{- end }}

2. Test the health endpoint:
  curl http://{{ include "catalog-api.fullname" . }}.{{ .Values.namespace }}.svc.cluster.local:{{ .Values.service.port }}/hc
```

#### Final Validation:
```bash
# Lint all charts
helm lint ./catalog-api-chart
helm lint ./basket-api-chart
helm lint ./webmvc-chart

# Test template rendering with different values
helm template test ./catalog-api-chart --values values-dev.yaml --dry-run
helm template test ./catalog-api-chart --values values-prod.yaml --dry-run

# Test installation (dry run)
helm install test-catalog ./catalog-api-chart --namespace eshop-test --dry-run --debug
```

---

## 🧪 Testing Your Implementation

### Chart Validation:
```bash
# Validate individual charts
helm lint ./catalog-api-chart
helm lint ./basket-api-chart  
helm lint ./webmvc-chart

# Test template rendering
helm template catalog-test ./catalog-api-chart --values values-dev.yaml
helm template basket-test ./basket-api-chart --values values-dev.yaml

# Validate YAML output
helm template catalog-test ./catalog-api-chart --values values-dev.yaml | kubectl apply --dry-run=client -f -
```

### Deployment Testing:
```bash
# Test individual chart deployments
kubectl create namespace eshop-test

# Deploy catalog API
helm install catalog-test ./catalog-api-chart \
  --namespace eshop-test \
  --values values-dev.yaml

# Verify deployment
kubectl get pods -n eshop-test -l app.kubernetes.io/name=catalog-api
kubectl get svc -n eshop-test
kubectl get hpa -n eshop-test

# Test connectivity
kubectl run test-pod --image=curlimages/curl -it --rm --restart=Never -- /bin/sh
# Inside the pod: curl catalog-test-catalog-api.eshop-test.svc.cluster.local/health

# Run Helm tests
helm test catalog-test -n eshop-test
```

---

## 📝 Deliverables

Submit the following Helm charts and documentation:

### Required Chart Files:
Create at least **3 individual Helm charts** for eShop services:

1. **catalog-api-chart/** - Complete chart for Catalog API
   - Chart.yaml with proper metadata
   - values.yaml with all configuration options  
   - values-dev.yaml and values-prod.yaml
   - templates/ directory with all K8s resources
   - templates/_helpers.tpl with template functions
   - templates/tests/test-connection.yaml

2. **basket-api-chart/** - Complete chart for Basket API
   - Similar structure to catalog-api-chart
   - Customized for basket-specific requirements

3. **webmvc-chart/** - Complete chart for WebMVC frontend  
   - Frontend-specific configuration
   - Ingress configuration for web access

### Required Templates per Chart:
- **deployment.yaml** - With resource limits, probes, environment variables
- **service.yaml** - Service configuration
- **hpa.yaml** - HorizontalPodAutoscaler (from Week 12)
- **ingress.yaml** - Ingress configuration (from Week 10)
- **serviceaccount.yaml** - Service account (from Week 11)
- **NOTES.txt** - Post-installation instructions

### Packaged Charts:
- **catalog-api-0.1.0.tgz** - Packaged chart file
- **basket-api-0.1.0.tgz** - Packaged chart file  
- **webmvc-0.1.0.tgz** - Packaged chart file

### Screenshots:
- `helm lint` output for all charts showing no errors
- `helm list --all-namespaces` after deploying all charts
- `kubectl get pods --all-namespaces` showing all services running
- `helm test <chart-name>` showing successful connectivity tests


---
## ⏱️ Deadline

Submit all deliverables before next class. Start early - Helm templating takes time to master!


