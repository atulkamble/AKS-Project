Here’s a complete step-by-step project with **Azure Kubernetes Service (AKS)** using CLI commands, where we deploy a sample **Node.js web app** on AKS.

---

### 🚀 **AKS Project Structure**

✅ Provision AKS  
✅ Create Docker Image & Push to ACR  
✅ Create Kubernetes Manifests  
✅ Deploy App to AKS  
✅ Access the App via Public IP

---

### 🧰 **Pre-Requisites**
- Azure CLI logged in (`az login`)
- Azure Subscription
- `kubectl` and `docker` installed
- Basic Node.js app (`app.js`, `Dockerfile`, etc.)

---

### 📁 **1. Sample Node.js App Structure**
```bash
aks-nodejs-app/
│
├── Dockerfile
├── app.js
├── package.json
└── k8s/
    ├── deployment.yaml
    └── service.yaml
```

---

### 🛠️ **2. Azure CLI & AKS Commands**

#### 🌀 Step 1: Set Variables
```bash
RESOURCE_GROUP="aks-rg"
AKS_NAME="aks-demo-cluster"
ACR_NAME="aksdemoacr$RANDOM"
APP_NAME="nodewebapp"
IMAGE_NAME="${ACR_NAME}.azurecr.io/${APP_NAME}:v1"
```

---

#### 🏗️ Step 2: Create Resource Group
```bash
az group create --name $RESOURCE_GROUP --location eastus
```

---

#### ☸️ Step 3: Create AKS Cluster
```bash
az aks create --resource-group $RESOURCE_GROUP --name $AKS_NAME \
  --node-count 1 --enable-addons monitoring --generate-ssh-keys
```

---

#### 🧱 Step 4: Create Azure Container Registry
```bash
az acr create --resource-group $RESOURCE_GROUP --name $ACR_NAME \
  --sku Basic
```

---

#### 🔗 Step 5: Attach ACR to AKS
```bash
az aks update --name $AKS_NAME --resource-group $RESOURCE_GROUP \
  --attach-acr $ACR_NAME
```

---

#### 🐳 Step 6: Build & Push Docker Image
```bash
az acr login --name $ACR_NAME
docker build -t $IMAGE_NAME .
docker push $IMAGE_NAME
```

---

#### 📦 Step 7: Get AKS Credentials
```bash
az aks get-credentials --resource-group $RESOURCE_GROUP --name $AKS_NAME
```

---

### 📄 **3. Kubernetes YAML Files**

#### ✅ `k8s/deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nodejs
  template:
    metadata:
      labels:
        app: nodejs
    spec:
      containers:
      - name: nodejs
        image: <IMAGE_NAME>
        ports:
        - containerPort: 3000
```

Replace `<IMAGE_NAME>` with actual ACR image, e.g.,  
`aksdemoacr12345.azurecr.io/nodewebapp:v1`

---

#### 🌐 `k8s/service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodejs-service
spec:
  type: LoadBalancer
  selector:
    app: nodejs
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
```

---

### 🚀 **4. Deploy App on AKS**
```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

---

### 🌐 **5. Access App**
```bash
kubectl get service nodejs-service
```

Wait for `EXTERNAL-IP`, then open in browser:  
`http://<EXTERNAL-IP>`

---

### ✅ **6. Clean Up**
```bash
az group delete --name $RESOURCE_GROUP --yes --no-wait
```

---

Would you like me to create a GitHub-ready folder structure with `README.md`, `Dockerfile`, and manifest YAMLs?
