## Cloud-Native Web Application on GKE - GitOps CI/CD & Disaster Recovery
---
## Project Overview
This project demonstrates a production-grade CI/CD and Disaster Recovery (DR) pipeline for deploying a cloud-native web application on Google Kubernetes Engine (GKE) using GitOps principles.
The solution automates the full application lifecycle, including infrastructure provisioning, container image delivery, Kubernetes deployment, and full cluster backup/recovery. It is designed to simulate real-world DevOps challenges—such as infrastructure failures, human deployment errors, and data loss—while ensuring the system can be rapidly restored with minimal downtime.

---
##  Problem Statement
- Modern application deployment often faces significant operational risks when managed manually:
- Complexity & Drift: Inconsistent environments between development and production.
- Human Error: Manual configuration changes leading to outages.
- Recovery RTO/RPO: Slow and unreliable recovery from system or namespace failures due to a lack of backups.
- Standardization: Difficulty in achieving fast iteration cycles while maintaining high stability.
---
## Project Goals
- This project provides a comprehensive DevOps framework to bridge the gap between development speed and operational reliability:
- Infrastructure as Code (IaC): Using Terraform to eliminate manual setup and configuration drift.
- Immutable Infrastructure: Containerization with Docker and TypeScript for consistent execution.
- GitOps: Leveraging Argo CD to ensure the cluster state always matches the version-controlled manifests.
- Resilience: Implementing Velero for automated Kubernetes backups, enabling rapid restoration in disaster scenarios.
---

## Key Features
- Full-Stack Cloud-Native Architecture: Built with a React (Vite) frontend and Node.js backend for high performance.
- IaC Provisioning: 100% of the GKE environment is managed via Terraform.
- Automated CI/CD: GitHub Actions triggers builds and image pushes to Google Artifact Registry.
- Declarative GitOps: Argo CD manages the "Desired State" of the cluster via GitHub.
- Disaster Recovery Ready: Integrated with Velero and GCS for full-state snapshots and recovery.
- Type-Safe Development: Developed using TypeScript to reduce runtime errors and improve maintainability.
---
##  Architecture Overview
**Infrastructure Layer**
- GKE cluster provisioned with Terraform
- Google Artifact Registry for container images
- Google Cloud Storage (GCS) bucket for Velero backups
**CI/CD & GitOps Layer**
- GitHub Actions pipeline builds and pushes Docker images
-  Kubernetes manifests stored in GitHub
-  Argo CD continuously syncs manifests to GKE

**Disaster Recovery Layer**
- Velero configured with GCS
- Namespace-scoped backups
- Restore tested after deletion
---

## Technologies Used
| Technology | Purpose | Key Benefits |
| :--- | :--- | :--- |
| **TypeScript / Node.js** | App Development | Strong type safety and scalable runtime. |
| **React / Vite** | Frontend | Rapid UI rendering and modern build tooling. |
| **Docker** | Containerization | "Build once, run anywhere" consistency. |
| **Terraform** | IaC | Automated, versioned infrastructure. |
| **Argo CD** | GitOps Delivery | Automated synchronization and drift detection. |
| **Velero** | Disaster Recovery | Protects against cluster or namespace loss. |
| **GKE** | Orchestration | High availability and self-healing workloads. |
---
##  Workflow
- Terraform provisions GKE infrastructure
- Code push triggers GitHub Actions workflow
- Docker image is built and pushed to Artifact Registry
- Kubernetes manifests are updated in GitHub
- Argo CD syncs manifests to GKE
- Application runs in a dedicated namespace
- Velero backs up namespace resources to GCS
- Namespace or cluster is deleted (failure simulation)
- Velero restores workloads and data successfully
---
##  Step-by-Step Implementation Guide
Before deploying the infrastructure, ensure the following tools are installed and properly configured:
- Google Cloud SDK (gcloud)
- Terraform
- Docker
- kubectl
- Git
- A GitHub account
---
## GCP Authentication
Authenticate your local environment or container with Google Cloud:

```
gcloud auth login
gcloud config set project <PROJECT_ID>
gcloud  auth  application-deafult login
```
---

### Step 2: Provision GKE with Terraform

1. Navigate to the Terraform directory:
![terraform deployment](https://github.com/smogalloyubio/GoogleCloud-Disaster-Recovery-project-/blob/main/picture/Screenshot%202026-01-24%20at%2011.47.27.png)
```
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
wget -O- https://apt.releases.hashicorp.com/gpg | \
gpg --dearmor | \
sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt-get update && sudo apt-get install terraform
cd terraform
terraform init
terraform apply

```
--- 
Terraform provisions:
- GKE cluster
- Networking resources
- Required IAM roles and service accounts

**![Terraform  apply command](https://github.com/smogalloyubio/02-Devops-project-NetflixClone-app/blob/main/picture/Screenshot%202026-01-24%20at%2012.18.12.png):**
---
## What Terraform Provisions
Once applied, Terraform automatically creates the following cloud resources:
- Google Kubernetes Engine (GKE) cluster
- Networking components (VPC, subnets, routing)
- IAM roles and service accounts required for cluster operation
---
## CI Pipeline – Build and Push Docker Image to Artifact Registry
In this step, a CI/CD pipeline is implemented using GitHub Actions to automate the process of building the application container image and pushing it to Google Artifact Registry.
This ensures that every code change is validated, containerized, and versioned in a consistent and repeatable way before deployment.
The GitHub Actions pipeline is triggered automatically on every push to the main branch.
It performs the following tasks:
- Checks out the latest application source code from GitHub
- Builds a Docker image for the web application
- Tags the image with a version for traceability
- Authenticates securely with Google Cloud
- Pushes the image to Google Artifact Registry

**![Gitaction pipeline](https://github.com/smogalloyubio/02-Devops-project-NetflixClone-app/blob/main/picture/Screenshot%202026-01-24%20at%2011.12.05.png)**


google cloud artifact registry 
![artifact registry](https://github.com/smogalloyubio/02-Devops-project-NetflixClone-app/blob/main/picture/Screenshot%202026-01-24%20at%2011.11.50.png)
---

## Step 4: Kubernetes Cluster Configuration and Manifests
After provisioning the Kubernetes cluster using Terraform, the next step was to configure the application workloads using Kubernetes manifests.

![Google kubernstes cluster](https://github.com/smogalloyubio/02-Devops-project-NetflixClone-app/blob/main/picture/Screenshot%202026-01-24%20at%2012.33.10.png)
---
## Step 5: Install and Configure Argo CD (GitOps Controller)
In this step, Argo CD is installed on the Kubernetes cluster to enable GitOps-based continuous delivery. It acts as the reconciliation engine that continuously syncs the desired state stored in GitHub with the actual state of the Kubernetes cluster.
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl get pods - n argocd
# check the  service of the argocd
kubectl  get svc -n argocd
# change the  service orgocd to loadbalancer
kubectl  patch  svc/argocd-server -n argocd -p
# connect the git repo to argocd
Create Argo CD Application pointing to GitHub repo
Enable auto-sync
kubectl get secret argocd-initial-admin-secret -n argocd \
-o jsonpath="{.data.password}" | base64 -d

kubectl patch svc argocd-server -n argocd \
-p '{"spec": {"type": "LoadBalancer"}}'
argocd login <ARGOCD_SERVER_IP> --username admin --password <PASSWORD> --insecure
argocd repo add https://github.com/smogalloyubio/GoogleCloud-Disaster-Recovery-project.git

```
![argocd cd](https://github.com/smogalloyubio/02-Devops-project-NetflixClone-app/blob/main/picture/Screenshot%202026-01-23%20at%2023.38.26.png)

---

### Step 6: Deploy Application via GitOps
- Push Kubernetes manifest changes to GitHub
- Argo CD automatically deploys to GKE
  
```
kubectl get pods -n namespace
kuebctl get all --all-namespace
kubectl get pods -n dev
#check  deployment of the  namespace
kubectl get pods -n  dev 
argocd app create webapp \
--repo  https://github.com/smogalloyubio/GoogleCloud-Disaster-Recovery-project.git
--path apps \
--dest-server https://kubernetes.default.svc \
--dest-namespace dev
argocd app sync apps 

```
 **![ argocd  deployment](https://github.com/smogalloyubio/02-Devops-project-NetflixClone-app/blob/main/picture/Screenshot%202026-01-23%20at%2023.45.03.png):**

---

### Step 7: Install and Configure Velero

Install Velero with GCS backend:

```bash

# Download the latest Linux release
wget https://github.com/vmware-tanzu/velero/releases/download/v1.15.0/velero-v1.15.0-linux-amd64.tar.gz

# Extract the file
tar -xvf velero-v1.15.0-linux-amd64.tar.gz

# Move the binary to your path
sudo mv velero-v1.15.0-linux-amd64/velero /usr/local/bin/

# Verify installation
velero version --client-only
velero install \
  --provider gcp \
  --plugins velero/velero-plugin-for-gcp:v1.8.0 \
  --bucket <GCS_BUCKET_NAME> \
  --secret-file ./credentials-velero

```
**![velero backup](https://github.com/smogalloyubio/02-Devops-project-NetflixClone-app/blob/main/picture/Screenshot%202026-01-24%20at%2013.22.31.png):**


---

### Step 8: Backup Application Namespace

Create a namespace-scoped backup:

```bash
velero backup create netflix-backup --include-namespaces all-namespace
velero  backup  dscribe netflix-backup
velero  backup get

```

Check backup status:

```bash
velero backup get
```

![velero backup](https://github.com/smogalloyubio/02-Devops-project-NetflixClone-app/blob/main/picture/Screenshot%202026-01-24%20at%2013.21.54.png)



---

### Step 9: Disaster Simulation

* Delete the application namespace or entire cluster

```bash
kubectl delete namespace -all-namespace  dev  canary argocd
kubectl get namespace 
```

**![check  namespace](https://github.com/smogalloyubio/02-Devops-project-NetflixClone-app/blob/main/picture/Screenshot%202026-01-24%20at%2013.26.08.png):**

> Add screenshot showing namespace deletion

---

### Step 10: Restore from Backup

Restore the backup:

```bash
velero restore create --from-backup netflix-backup
```

Verify restore:

```bash
kubectl get all -n <APP_NAMESPACE>
```

**![retore namespace](https://github.com/smogalloyubio/02-Devops-project-NetflixClone-app/blob/main/picture/Screenshot%202026-01-24%20at%2013.36.05.png):**



---

## 📂 Repository Structure

```
.
├── terraform/          
├── argocd/              
├── .github/workflows/  
├── manifest/      
├── argocd/         
├── velero/             
└── README.md
```

---

## Disaster Recovery Test

* Created a Velero backup for the application namespace
* Deleted the namespace / cluster
* Restored from backup stored in GCS
* Verified:

  * Pods recreated successfully
  * Services accessible
  * Application functional after restore

---

## 🔐 Security Considerations

* IAM roles follow least-privilege principle
* Service accounts scoped per component
* No secrets stored in Git
* GitOps provides full audit trail

---

## 📈 Future Improvements

* Automated Velero backup schedules
* Terraform remote state in GCS
* Workload Identity for GKE
* Sealed Secrets for secret management
* Canary or blue/green deployments
* Monitoring with Prometheus and Grafana

---

## skills Demonstrated

* Kubernetes operations on GKE
* Infrastructure as Code with Terraform
* CI/CD pipeline design
* GitOps with Argo CD
* Disaster recovery planning and testing
* Cloud IAM and security best practices

---


**ubioworo rukevwe**
DevOps / Cloud Engineer

