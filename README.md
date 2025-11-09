---

# DevOpsify a Go Web App Project

This repository demonstrates **DevOps practices** on a simple Go web application. You’ll learn how to **containerize the app, deploy it to Kubernetes on AWS EKS, manage deployments via Helm charts, and implement GitOps using ArgoCD**. By the end of this guide, you’ll have a full **CI/CD pipeline** from code commit to live application updates.

---

## **1. Understanding and Running the Application Locally**

Before containerizing anything, it’s crucial to understand the application and run it locally.

### Step 1.1 – Explore the Project Structure

Our project folder looks like this:

```
go-web-app/
│
├── main.go
├── go.mod
├── main_test.go
└── static/
    └── home.html
    └── courses.html
    └── contact.html
    └── about.html
```

* `main.go` – Entry point of the Go web app.
* `static/` – Contains static HTML pages.
* `go.mod` – Go module dependencies.
* `main_test.go` – Unit tests for the application.

---

### Step 1.2 – Build the Application

Compile the code into an executable binary:

```bash
go build -o main .
```

* `go build` – Compiles Go code.
* `-o main` – Output binary name.
* `.` – Build from the current directory.

---

### Step 1.3 – Run the Application

```bash
./main
```

You should see:

```
Server is running on port 8080
```

Open a browser and visit:

```
http://localhost:8080/courses

```

<img width="2940" height="1690" alt="image" src="https://github.com/user-attachments/assets/3eac502a-2df8-43c0-9a50-2f217ffecc55" />
<img width="2940" height="1690" alt="image" src="https://github.com/user-attachments/assets/847ad140-017e-4e39-9d5c-f08f3215fdb7" />
<img width="2940" height="1690" alt="image" src="https://github.com/user-attachments/assets/c072f01c-b119-4b0b-9c51-394bd94ff590" />
<img width="2940" height="1690" alt="image" src="https://github.com/user-attachments/assets/8defb964-cdb7-44d5-a88c-d1d35a7159f1" />


You’ve successfully run the app locally!

---

## **2. Dockerizing the Go Web App**

Containerization allows us to package the app with all dependencies.

### Step 2.1 – Build Docker Image

```bash
docker build -t arnablogs/go-web-app:v1 .
```

* `-t` – Name and tag the image.
* `.` – Build context (current folder).

<img width="2940" height="836" alt="image" src="https://github.com/user-attachments/assets/a1ed10fe-c585-4067-9767-d3754ff53e9e" />

### Step 2.2 – Run Docker Container

```bash
docker run -p 8080:8080 -it arnablogs/go-web-app:v1
```

Verify in the browser:

```
http://localhost:8080/home
```

<img width="2940" height="1690" alt="image" src="https://github.com/user-attachments/assets/678d7c20-8e5d-4f67-85d7-05a961bf1495" />

Containerization is now verified.

### Step 2.3 – Push to Docker Hub

Before deploying to Kubernetes, push the image:

```bash
docker push arnablogs/go-web-app:v1
```

<img width="2940" height="946" alt="image" src="https://github.com/user-attachments/assets/42ac7857-388b-45b6-90b1-bf233dbd9401" />

<img width="2940" height="756" alt="image" src="https://github.com/user-attachments/assets/870b1a04-2a13-448d-8a50-c109f3502109" />

---

## **3. Kubernetes Manifests**

We will define the app’s deployment, service, and ingress.

1. Create folder: `k8s/manifests`.
2. Add three files:

* `deployment.yaml` – Defines Pods and Deployment strategy.
* `service.yaml` – Exposes app internally or externally.
* `ingress.yaml` – Maps domain names to services.

---

## **4. AWS EKS Cluster Setup**

We’ll deploy our app on AWS EKS (managed Kubernetes).

### Step 4.1 – Prerequisites

* Install **kubectl**, **eksctl**, and **AWS CLI**.
* Configure AWS CLI with credentials:

```bash
aws configure
```

---

### Step 4.2 – Create Cluster

```bash
eksctl create cluster \
  --name three-tier-cluster \
  --region ap-south-1 \
  --node-type c7i-flex.large \
  --nodes-min 2 --nodes-max 2
```

<img width="2940" height="756" alt="image" src="https://github.com/user-attachments/assets/ee289514-b665-494f-8fd8-0cd32d11a99b" />

<img width="2940" height="482" alt="image" src="https://github.com/user-attachments/assets/4ae02cb7-3ac9-4c18-8309-411b297f8a49" />

* Cluster creation typically takes 15–20 minutes.
* Verify cluster:

```bash
kubectl get nodes
```

---

## **5. Deploy Application to EKS**

### Step 5.1 – Apply Kubernetes Manifests

```bash
kubectl apply -f k8s/manifests/deployment.yaml
kubectl apply -f k8s/manifests/service.yaml
kubectl apply -f k8s/manifests/ingress.yaml
```

* Verify pods:

```bash
kubectl get pods
kubectl get svc
kubectl get ingress
```

### Step 5.2 – Configure Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/aws/deploy.yaml
```

<img width="2940" height="1034" alt="image" src="https://github.com/user-attachments/assets/e52b4744-b7c1-459a-8e1d-e8a3ac8e6589" />

* The Ingress controller assigns an **FQDN/IP** for external access.

  <img width="2940" height="208" alt="image" src="https://github.com/user-attachments/assets/3ee6896b-5c66-4191-9651-df0cf9416020" />

* Map this address to `go-web-app.local` in `/etc/hosts` for testing.

---

## **6. Helm Charts for Multi-Environment Deployment**

1. Create a Helm chart:

```bash
helm create go-web-app-chart
```

<img width="2940" height="426" alt="image" src="https://github.com/user-attachments/assets/9fcdba7d-4f16-4a73-ad5b-6c837d4b9705" />

2. Update `values.yaml` with Docker image:

```yaml
image:
  repository: arnablogs/go-web-app
  tag: v1
```

3. Deploy using Helm:

```bash
helm install go-web-app go-web-app-chart
```

<img width="2940" height="426" alt="image" src="https://github.com/user-attachments/assets/ec06f141-9208-46e9-8225-201fac252e9c" />

Verify deployments:

```bash
kubectl get deployments
kubectl get svc
kubectl get ingress
```

---

## **7. ArgoCD GitOps Deployment**

### Step 7.1 – Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

<img width="2940" height="666" alt="image" src="https://github.com/user-attachments/assets/89ad0a1d-d605-4f71-8e84-14b08866b767" />

### Step 7.2 – Access ArgoCD UI

1. Expose service (LoadBalancer or NodePort).
2. Get initial password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode
```

3. Login at `https://<ARGOCD_LOAD_BALANCER>/`.

<img width="2940" height="666" alt="image" src="https://github.com/user-attachments/assets/4c1fd387-e544-4cde-86ff-7076599f191b" />

### Step 7.3 – Create ArgoCD App

* Repository: your GitHub repo
* Path: `helm/go-web-app-chart`
* Cluster: default
* Sync Policy: Automatic + Self-Heal

<img width="2940" height="1358" alt="image" src="https://github.com/user-attachments/assets/2a76d1f9-ceca-4409-8e12-e8e5a46d1d85" />

<img width="2940" height="1526" alt="image" src="https://github.com/user-attachments/assets/12289aa9-e2c7-4990-82c4-1c25c2c67576" />

<img width="2940" height="1220" alt="image" src="https://github.com/user-attachments/assets/d98f42bb-0da7-4dac-b74f-e22417fbe351" />

ArgoCD will now **monitor Helm charts** and **automatically deploy changes** to EKS.

---

## **8. End-to-End CI/CD Testing**

Now we verify the full pipeline from **Git commit → Docker build → Helm → ArgoCD → live app**.

### Step 8.1 – Make a Change

1. Edit `static/ourses.yaml`:

```html
<h1>Arnab’s DevOps Learning Journey</h1>
```

<img width="2900" height="1636" alt="image" src="https://github.com/user-attachments/assets/b3a3eb80-7220-42ea-b3d0-03d234a1c778" />

2. Commit and push:

```bash
git add static/home.html
git commit -m "Update home page title"
git push
```

---

### Step 8.2 – Observe CI/CD Pipeline

* GitHub Actions triggers automatically.

<img width="2900" height="1636" alt="image" src="https://github.com/user-attachments/assets/179aecc5-c510-4da3-b06b-b46d60d041a7" />

* Steps executed:

  1. Build & Test Go app.
  2. Build Docker image & push to Docker Hub.
  3. Update Helm chart with new Docker tag.
  4. ArgoCD detects changes and deploys automatically.

<img width="2900" height="654" alt="image" src="https://github.com/user-attachments/assets/d6b8241c-04d5-4155-a636-75f277f13304" />

Monitor progress:

```bash
kubectl get pods
kubectl get deployments
```

---

### Step 8.3 – Verify the Live Application

1. Visit:

```
http://go-web-app.local/home
```

2. Confirm the updated title is visible:

```
Arnab’s DevOps Learning Journey
```

<img width="2900" height="1704" alt="image" src="https://github.com/user-attachments/assets/2b6c907e-3d5a-4936-bf31-8fd9488a9839" />

---

### Step 8.4 – Practice

* Update other static pages (courses.html, contact.html).
* Observe **Docker image creation → Helm update → ArgoCD deployment**.
* This hands-on practice demonstrates **true GitOps workflow**.

---

## ✅ Key Takeaways

* CI/CD pipeline enables **automated, reliable deployments**.
* Docker + Helm + EKS + ArgoCD ensures **consistent environments**.
* Every commit triggers the **entire workflow**, eliminating manual steps.
* GitOps with ArgoCD enforces **desired state and self-healing**.

---
