# VClipper Pipelines

This repository contains the GitHub Actions pipeline responsible for provisioning the entire infrastructure and deploying the VClipper application to AWS and MongoDB Atlas.

## 💡 Overview

This multi-stage CI/CD pipeline automates the provisioning and deployment of a distributed video processing application, covering cloud resources such as:

- **S3 Buckets** (terraform state, video storage, frontend hosting)
- **VPC, NAT Gateway, and Security Groups**
- **EKS Cluster**
- **MongoDB Atlas Cluster**
- **Amazon Cognito**
- **Amazon SQS and SNS**
- **Helm Charts**
- **Monitoring Stack (Prometheus + Grafana)**
- **Java Applications (Maven build and test)**
- **React Frontend (Build and deploy)**

---

## 📦 Repository Structure

This repository contains a single GitHub Actions workflow:

```yaml
.vclipper_pipeline/.github/workflows/multi-stage-pipeline.yaml
```
The workflow is designed to be triggered remotely by other repositories, using `workflow_dispatch`, and is divided into several jobs executed in a specific order.

---

## ⚙️ Pipeline Stages

### 1. **Bucket Initialization**
Creates the main S3 bucket used to store Helm charts and Terraform state files.

- `setup-bucket`

---

### 2. **Helm Chart Repository Setup**
Installs the S3 Helm plugin and pushes packaged charts.

- `install-helm-chart-repo`

---

### 3. **Global Infrastructure**
Initializes shared Terraform backend configuration and sets up global dependencies.

- `global-setup`

---

### 4. **VPC + Networking**
Creates the VPC, subnets, and routing via Terraform.

- `vpc-setup`
- `securitygroup-setup`

---

### 5. **EKS Cluster**
Provisions an EKS cluster where backend services are deployed.

- `eks-setup`

---

### 6. **MongoDB Atlas**
Creates and configures a MongoDB cluster using Atlas, extracting the connection string for backend usage.

- `database-mongodb-setup`

---

### 7. **Cognito Setup**
Creates user pool and client credentials for frontend authentication.

- `coginito-setup`

---

### 8. **Storage + Hosting**
Creates and configures:

- S3 bucket for **video file storage**
- S3 bucket for **static frontend hosting**

- `video-storage-setup`
- `frontend-hosting-setup`

---

### 9. **Queues and Notifications**
Provisions:

- **SQS queues** (processing, result)
- **SNS topics** (success notification)

- `queue-setup`
- `sns-notification-setup`

---

### 10. **Unit Tests**
Clones Java backend repositories and runs unit tests using Maven.

- `unit-tests`

---

### 11. **Static Code Analysis**
Analyzes code quality using Maven + SonarCloud.

- `code-analysis`

---

### 12. **Docker Build & Push**
Builds Docker images for backend services and pushes them to Docker Hub.

- `build-and-push`

---

### 13. **Backend Deployment**
Deploys backend apps (`vclipper-processing`, `vclipping`) using Helm to EKS. Also installs Prometheus and Grafana.

- `deploy-be`

---

### 14. **Frontend Deployment**
Builds the React application and deploys it to the hosting S3 bucket.

- `deploy-fe`

---

### 15. **App Ready**
Final step – prints the website public URL once the entire stack is available.

- `app_ready`

---

## 🔐 Secrets and Variables

The pipeline uses GitHub Secrets and Environment Variables extensively. These include:

### Secrets:
- `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`
- `GH_TOKEN` – GitHub token to clone private repos
- `DOCKER_USERNAME`, `DOCKER_PASSWORD`
- `MONGODB_USER`, `MONGODB_PASSWORD`
- `MONGO_APP_DB`, `JWT_SECRET`, `JWT_EXPIRES`
- `MONGODBATLAS_*` and `ORG_ID`
- `SONAR_TOKEN`

### Environment Variables:
- `AWS_DEFAULT_REGION`
- `BUCKET_S3`
- `MONGO_HOST`
- `WEBSITE_URL`, `HTTPS_URL`
- `MOCK` (mock flags for auth and video)

---

## 🚀 How to Trigger

This pipeline is manually triggered using the **"Run workflow"** button on GitHub Actions (`workflow_dispatch` event). It can also receive a `branch` input to deploy from different branches (e.g., `main`, `homolog`). Also, its can be trigged by others repos, like backend e frontend. 

---

## 📝 Notes

- The workflow is **multi-repo aware**, cloning various repositories under the `fiap-9soat-snackbar` organization.
- Helm charts are pushed to an **S3-backed Helm repo**.
- Terraform state is managed remotely in the S3 bucket.
- Application monitoring is handled by **Kube-Prometheus-Stack** (Prometheus + Grafana).
- DNS of the backend load balancer is used to configure the frontend.

---

## 📎 Related Repositories

- [`vclipper_infra`](https://github.com/fiap-9soat-snackbar/vclipper_infra) – Terraform code and Helm charts
- [`vclipper_processing`](https://github.com/fiap-9soat-snackbar/vclipper_processing) - Backend Application
- [`vclipping`](https://github.com/fiap-9soat-snackbar/vclipping) - Backend Application
- [`vclipper_fe`](https://github.com/fiap-9soat-snackbar/vclipper_fe) - Frontend Application

---

## ✅ Final Output

Once the pipeline completes successfully, your VClipper application will be fully deployed, including:

- Backend APIs running in EKS
- Frontend served from S3
- MongoDB Atlas cluster connected
- Authentication via Cognito
- Full monitoring stack
- Notifications and queue processing setup

---
