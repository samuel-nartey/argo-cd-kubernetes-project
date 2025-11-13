# ArgoCDP2 – GitOps Continuous Deployment on Amazon EKS

## Project Overview

ArgoCDP2 demonstrates a GitOps-driven Continuous Deployment pipeline built on Amazon Elastic Kubernetes Service (EKS) using Argo CD.  
This project showcases infrastructure automation, version-controlled deployments, and declarative GitOps workflows — aligning with DevOps and DevSecOps best practices.

### Objective

To automate application deployments and configuration management on Kubernetes using Argo CD, ensuring:

- Single source of truth: Git repository governs cluster state  
- Secure delivery: Least-privilege Kubernetes RBAC and secret management  
- Observability and rollback: Continuous sync, self-healing, and versioned rollbacks  

---

## Architecture Explanation

### Architecture Diagram

**Insert Screenshot:** `ArgoCDP2.drawio`

### Workflow Breakdown

1. **Developer Commit:**  
   The developer pushes Kubernetes manifests or Helm charts to the Git manifest repository.

2. **Argo CD Pull/Sync:**  
   Argo CD continuously monitors the Git repository for changes and automatically syncs them to the EKS cluster.

3. **Continuous Deployment:**  
   Updated configurations are applied to the workload namespace, automatically rolling out new pods or rolling back failed deployments.

This approach enforces a declarative deployment model and maintains auditability, consistency, and security by treating Git as the single source of truth.

---

## Step 1: Create and Configure the Amazon EKS Cluster

### Concept

EKS (Elastic Kubernetes Service) provides a managed Kubernetes control plane, allowing DevOps teams to focus on deploying workloads instead of managing cluster infrastructure.

### Commands

```bash
# Create EKS cluster using eksctl
eksctl create cluster \
  --name argocd-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --managed
Insert Screenshot: EKS Cluster Creation Output

Step 2: Connect kubectl to the EKS Cluster
Commands
bash
Copy code
# Update kubeconfig to connect kubectl to the new cluster
aws eks --region us-east-1 update-kubeconfig --name argocd-cluster

# Verify cluster connectivity
kubectl get nodes
kubectl cluster-info
Insert Screenshot: kubectl Connected to EKS Cluster

Step 3: Create Argo CD Namespace
Commands
bash
Copy code
kubectl create namespace argocd
kubectl get ns
Insert Screenshot: ArgoCD Namespace Created

Step 4: Install Argo CD
Commands
bash
Copy code
# Install Argo CD into the argocd namespace
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Verify installation
kubectl get all -n argocd
Insert Screenshot: ArgoCD Pods Running

Step 5: Expose Argo CD UI via Port-Forwarding
Commands
bash
Copy code
kubectl port-forward svc/argocd-server -n argocd 8080:443
Access the UI at:
https://localhost:8080

Insert Screenshot: ArgoCD Login Page

Step 6: Retrieve and Decode Argo CD Admin Password
Commands
bash
Copy code
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode; echo
Use the following credentials to log in:

Username: admin

Password: (decoded value above)

Insert Screenshot: Successful Login to ArgoCD

Step 7: Deploy Application Using Application Manifest
Once logged into Argo CD UI, deploy the application defined in application.yml.

Commands
bash
Copy code
kubectl apply -f application.yml
Insert Screenshot: Application Synced and Healthy

Step 8: Test Argo CD Sync Policies
Argo CD supports Automated, Manual, SelfHeal, and Prune sync strategies.
We’ll test how Argo CD reacts to changes in the Git repo and cluster.

Manual Sync
Commit manifest updates to Git and manually trigger sync in Argo CD UI.

Insert Screenshot: Manual Sync Operation

Automatic Sync
Enable automated sync in application.yml:

yaml
Copy code
syncPolicy:
  automated:
    prune: true
    selfHeal: true
Insert Screenshot: Auto Sync Enabled

Step 9: Validate GitOps Behavior
Scenario 1 – Image Version Change
Change image tag in manifest from v2 → v1

Push to Git

Observe Argo CD automatically rolling back to reflect the Git version

Insert Screenshot: Image Version Rollback Event

Scenario 2 – Replica Count Update
Modify replicas in the deployment manifest from 2 → 5

Commit and push

Observe Argo CD syncing changes to the cluster

Insert Screenshot: Replica Update Reflected in ArgoCD

Scenario 3 – CLI Manual Modification
Edit resources directly via CLI

Argo CD detects drift and reverts unauthorized changes, maintaining Git as source of truth

Insert Screenshot: Drift Detection and Self-Healing

Key Learnings and DevSecOps Perspective
Git as Source of Truth: Ensures all deployments are version-controlled, traceable, and auditable.

Drift Management: Argo CD automatically restores desired state when manual changes are detected.

Security:

Principle of least privilege enforced through Kubernetes RBAC.

Secrets managed through Kubernetes Secret objects and optionally sealed-secrets for Git-safe encryption.

Scalability: Easily extended to multi-cluster (hub-and-spoke) GitOps architectures.

Tools and Technologies
Category	Tool/Service
Cloud Provider	AWS (EKS, IAM)
CI/CD	Argo CD
Version Control	GitHub
IaC	eksctl, kubectl
Security	AWS IAM, RBAC, Base64 Secrets
Observability	Argo CD Dashboard, kubectl logs

Summary
This project illustrates a standalone EKS cluster GitOps workflow leveraging Argo CD’s capabilities to achieve secure, automated, and versioned deployments — aligning with DevOps and DevSecOps principles.

The exercise proves Argo CD enforces Git as the single source of truth, automates deployments, and self-heals deviations — forming the foundation of modern, secure, and scalable Kubernetes delivery pipelines.

Copy code






You said:
i have added the name of the argocd architecture so insert how it can be generated as an image



ChatGPT said:
Here’s your updated README.md — now including a clear section on how to generate the architecture image (for example, from your ArgoCDP2.drawio file) using Draw.io / diagrams.net or VS Code Draw.io extension, so it fits professional DevOps documentation standards.

markdown
Copy code
# ArgoCDP2 – GitOps Continuous Deployment on Amazon EKS

## Project Overview

ArgoCDP2 demonstrates a GitOps-driven Continuous Deployment pipeline built on Amazon Elastic Kubernetes Service (EKS) using Argo CD.  
This project showcases infrastructure automation, version-controlled deployments, and declarative GitOps workflows — aligning with DevOps and DevSecOps best practices.

### Objective

To automate application deployments and configuration management on Kubernetes using Argo CD, ensuring:

- Single source of truth: Git repository governs cluster state  
- Secure delivery: Least-privilege Kubernetes RBAC and secret management  
- Observability and rollback: Continuous sync, self-healing, and versioned rollbacks  

---

## Architecture Explanation

### Architecture Diagram

**File Name:** `ArgoCDP2.drawio`

This diagram visually explains the end-to-end GitOps workflow involving the Developer, Git manifest repository, Argo CD, and the EKS workload namespace.

**Insert Screenshot:** Architecture Diagram Output (after export)

---

### How to Generate the Architecture Image

If you have created the architecture using **Draw.io (diagrams.net)** or **VS Code Draw.io extension**, you can export it as an image (PNG or JPEG) using one of the following methods:

#### Option 1: Using the Draw.io (Web App)
1. Open your `.drawio` file (`ArgoCDP2.drawio`) in [https://app.diagrams.net](https://app.diagrams.net)
2. Go to **File → Export As → PNG**
3. Select the option **"Include a copy of my diagram"**
4. Click **Export** and save the image as `ArgoCDP2.png`
5. Add the exported image to your repository and reference it in your README:

```markdown
![ArgoCD Architecture](./ArgoCDP2.png)
Option 2: Using the VS Code Draw.io Extension
Install the Draw.io Integration extension in VS Code

Open ArgoCDP2.drawio

Right-click anywhere on the canvas → Select Export → PNG

Save it as ArgoCDP2.png in your repo root

Reference it in your README as shown above.

Workflow Breakdown
Developer Commit:
The developer pushes Kubernetes manifests or Helm charts to the Git manifest repository.

Argo CD Pull/Sync:
Argo CD continuously monitors the Git repository for changes and automatically syncs them to the EKS cluster.

Continuous Deployment:
Updated configurations are applied to the workload namespace, automatically rolling out new pods or rolling back failed deployments.

This approach enforces a declarative deployment model and maintains auditability, consistency, and security by treating Git as the single source of truth.

Step 1: Create and Configure the Amazon EKS Cluster
Concept
EKS (Elastic Kubernetes Service) provides a managed Kubernetes control plane, allowing DevOps teams to focus on deploying workloads instead of managing cluster infrastructure.

Commands
bash
Copy code
# Create EKS cluster using eksctl
eksctl create cluster \
  --name argocd-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --managed
Insert Screenshot: EKS Cluster Creation Output

Step 2: Connect kubectl to the EKS Cluster
Commands
bash
Copy code
# Update kubeconfig to connect kubectl to the new cluster
aws eks --region us-east-1 update-kubeconfig --name argocd-cluster

# Verify cluster connectivity
kubectl get nodes
kubectl cluster-info
Insert Screenshot: kubectl Connected to EKS Cluster

Step 3: Create Argo CD Namespace
Commands
bash
Copy code
kubectl create namespace argocd
kubectl get ns
Insert Screenshot: ArgoCD Namespace Created

Step 4: Install Argo CD
Commands
bash
Copy code
# Install Argo CD into the argocd namespace
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Verify installation
kubectl get all -n argocd
Insert Screenshot: ArgoCD Pods Running

Step 5: Expose Argo CD UI via Port-Forwarding
Commands
bash
Copy code
kubectl port-forward svc/argocd-server -n argocd 8080:443
Access the UI at:
https://localhost:8080

Insert Screenshot: ArgoCD Login Page

Step 6: Retrieve and Decode Argo CD Admin Password
Commands
bash
Copy code
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode; echo
Use the following credentials to log in:

Username: admin

Password: (decoded value above)

Insert Screenshot: Successful Login to ArgoCD

Step 7: Deploy Application Using Application Manifest
Once logged into Argo CD UI, deploy the application defined in application.yml.

Commands
bash
Copy code
kubectl apply -f application.yml
Insert Screenshot: Application Synced and Healthy

Step 8: Test Argo CD Sync Policies
Argo CD supports Automated, Manual, SelfHeal, and Prune sync strategies.
We’ll test how Argo CD reacts to changes in the Git repo and cluster.

Manual Sync
Commit manifest updates to Git and manually trigger sync in Argo CD UI.

Insert Screenshot: Manual Sync Operation

Automatic Sync
Enable automated sync in application.yml:

yaml
Copy code
syncPolicy:
  automated:
    prune: true
    selfHeal: true
Insert Screenshot: Auto Sync Enabled

Step 9: Validate GitOps Behavior
Scenario 1 – Image Version Change
Change image tag in manifest from v2 → v1

Push to Git

Observe Argo CD automatically rolling back to reflect the Git version

Insert Screenshot: Image Version Rollback Event

Scenario 2 – Replica Count Update
Modify replicas in the deployment manifest from 2 → 5

Commit and push

Observe Argo CD syncing changes to the cluster

Insert Screenshot: Replica Update Reflected in ArgoCD

Scenario 3 – CLI Manual Modification
Edit resources directly via CLI

Argo CD detects drift and reverts unauthorized changes, maintaining Git as source of truth

Insert Screenshot: Drift Detection and Self-Healing

Key Learnings and DevSecOps Perspective
Git as Source of Truth: Ensures all deployments are version-controlled, traceable, and auditable.

Drift Management: Argo CD automatically restores desired state when manual changes are detected.

Security:

Principle of least privilege enforced through Kubernetes RBAC.

Secrets managed through Kubernetes Secret objects and optionally sealed-secrets for Git-safe encryption.

Scalability: Easily extended to multi-cluster (hub-and-spoke) GitOps architectures.

Tools and Technologies
Category	Tool/Service
Cloud Provider	AWS (EKS, IAM)
CI/CD	Argo CD
Version Control	GitHub
IaC	eksctl, kubectl
Security	AWS IAM, RBAC, Base64 Secrets
Observability	Argo CD Dashboard, kubectl logs

Summary
This project illustrates a standalone EKS cluster GitOps workflow leveraging Argo CD’s capabilities to achieve secure, automated, and versioned deployments — aligning with DevOps and DevSecOps principles.

The exercise proves Argo CD enforces Git as the single source of truth, automates deployments, and self-heals deviations — forming the foundation of modern, secure, and scalable Kubernetes delivery pipelines.

yaml
Copy code

---

Would you like me to add a **section for GitHub repository structure** (for example, showing `application.yml`, manifests, and image folder layout)? That would make your README even more complete for recruiters and open-source standards.






You said:
since it is a readme make it a markdown so that everything will appear as it should,so for instance step 8 or the steps use the ** and when to use `



ChatGPT said:
Perfect — below is your final README.md version, professionally formatted in full Markdown syntax (no emojis, proper use of **bold** for headings/emphasis and code blocks for commands).
This version will render perfectly on GitHub or any Markdown viewer — making it both recruiter-ready and technically professional.

markdown
Copy code
# ArgoCDP2 – GitOps Continuous Deployment on Amazon EKS

## Project Overview

**ArgoCDP2** demonstrates a **GitOps-driven Continuous Deployment pipeline** built on **Amazon Elastic Kubernetes Service (EKS)** using **Argo CD**.  
This project showcases **infrastructure automation**, **version-controlled deployments**, and **declarative GitOps workflows** — aligning with **DevOps** and **DevSecOps** best practices.

### Objective

To automate application deployments and configuration management on Kubernetes using Argo CD, ensuring:

- **Single source of truth:** Git repository governs cluster state  
- **Secure delivery:** Least-privilege Kubernetes RBAC and secret management  
- **Observability and rollback:** Continuous sync, self-healing, and versioned rollbacks  

---

## Architecture Explanation

### Architecture Diagram

**File Name:** `ArgoCDP2.drawio`

This diagram visually explains the end-to-end GitOps workflow involving the Developer, Git manifest repository, Argo CD, and the EKS workload namespace.

**Insert Screenshot:** Architecture Diagram Output (after export)

---

### How to Generate the Architecture Image

If you created the architecture using **Draw.io (diagrams.net)** or the **VS Code Draw.io extension**, you can export it as an image (PNG or JPEG) using one of the following methods:

#### Option 1: Using Draw.io (Web App)
1. Open your `.drawio` file (`ArgoCDP2.drawio`) in [https://app.diagrams.net](https://app.diagrams.net)
2. Go to **File → Export As → PNG**
3. Select the option **"Include a copy of my diagram"**
4. Click **Export** and save the image as `ArgoCDP2.png`
5. Add the exported image to your repository and reference it in this README:

```markdown
![ArgoCD Architecture](./ArgoCDP2.png)
Option 2: Using the VS Code Draw.io Extension
Install the Draw.io Integration extension in VS Code

Open ArgoCDP2.drawio

Right-click on the canvas → Select Export → PNG

Save it as ArgoCDP2.png in your repo root

Reference it as shown above.

Workflow Breakdown
Developer Commit:
The developer pushes Kubernetes manifests or Helm charts to the Git manifest repository.

Argo CD Pull/Sync:
Argo CD continuously monitors the Git repository for changes and automatically syncs them to the EKS cluster.

Continuous Deployment:
Updated configurations are applied to the workload namespace, automatically rolling out new pods or rolling back failed deployments.

This approach enforces a declarative deployment model and maintains auditability, consistency, and security by treating Git as the single source of truth.

Step 1: Create and Configure the Amazon EKS Cluster
Concept
EKS (Elastic Kubernetes Service) provides a managed Kubernetes control plane, allowing DevOps teams to focus on deploying workloads instead of managing cluster infrastructure.

Commands
bash
Copy code
# Create EKS cluster using eksctl
eksctl create cluster \
  --name argocd-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --managed
Insert Screenshot: EKS Cluster Creation Output

Step 2: Connect kubectl to the EKS Cluster
Commands
bash
Copy code
# Update kubeconfig to connect kubectl to the new cluster
aws eks --region us-east-1 update-kubeconfig --name argocd-cluster

# Verify cluster connectivity
kubectl get nodes
kubectl cluster-info
Insert Screenshot: kubectl Connected to EKS Cluster

Step 3: Create Argo CD Namespace
Commands
bash
Copy code
kubectl create namespace argocd
kubectl get ns
Insert Screenshot: ArgoCD Namespace Created

Step 4: Install Argo CD
Commands
bash
Copy code
# Install Argo CD into the argocd namespace
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Verify installation
kubectl get all -n argocd
Insert Screenshot: ArgoCD Pods Running

Step 5: Expose Argo CD UI via Port-Forwarding
Commands
bash
Copy code
kubectl port-forward svc/argocd-server -n argocd 8080:443
Access the UI at:
https://localhost:8080

Insert Screenshot: ArgoCD Login Page

Step 6: Retrieve and Decode Argo CD Admin Password
Commands
bash
Copy code
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode; echo
Use the following credentials to log in:

Username: admin

Password: (decoded value above)

Insert Screenshot: Successful Login to ArgoCD

Step 7: Deploy Application Using Application Manifest
Once logged into Argo CD UI, deploy the application defined in application.yml.

Commands
bash
Copy code
kubectl apply -f application.yml
Insert Screenshot: Application Synced and Healthy

Step 8: Test Argo CD Sync Policies
Argo CD supports Automated, Manual, SelfHeal, and Prune sync strategies.
This section demonstrates how Argo CD reacts to configuration drift and Git changes.

Manual Sync
Commit manifest updates to Git and manually trigger sync in Argo CD UI.

Insert Screenshot: Manual Sync Operation

Automatic Sync
Enable automated sync in application.yml by defining the following:

yaml
Copy code
syncPolicy:
  automated:
    prune: true
    selfHeal: true
Insert Screenshot: Auto Sync Enabled

Step 9: Validate GitOps Behavior
Scenario 1 – Image Version Change
Change image tag in manifest from v2 → v1

Push to Git

Observe Argo CD automatically rolling back to reflect the Git version

Insert Screenshot: Image Version Rollback Event

Scenario 2 – Replica Count Update
Modify replicas in the deployment manifest from 2 → 5

Commit and push

Observe Argo CD syncing changes to the cluster

Insert Screenshot: Replica Update Reflected in ArgoCD

Scenario 3 – CLI Manual Modification
Edit resources directly via CLI

Argo CD detects drift and reverts unauthorized changes, maintaining Git as source of truth

Insert Screenshot: Drift Detection and Self-Healing

Key Learnings and DevSecOps Perspective
Git as Source of Truth: Ensures all deployments are version-controlled, traceable, and auditable.

Drift Management: Argo CD automatically restores desired state when manual changes are detected.

Security:

Principle of least privilege enforced through Kubernetes RBAC.

Secrets managed through Kubernetes Secret objects and optionally sealed-secrets for Git-safe encryption.

Scalability: Easily extended to multi-cluster (hub-and-spoke) GitOps architectures.

Tools and Technologies
Category	Tool/Service
Cloud Provider	AWS (EKS, IAM)
CI/CD	Argo CD
Version Control	GitHub
IaC	eksctl, kubectl
Security	AWS IAM, RBAC, Base64 Secrets
Observability	Argo CD Dashboard, kubectl logs

Summary
This project illustrates a standalone EKS cluster GitOps workflow leveraging Argo CD’s capabilities to achieve secure, automated, and versioned deployments — aligning with DevOps and DevSecOps principles.

The exercise proves that Argo CD enforces Git as the single source of truth, automates deployments, and self-heals deviations — forming the foundation of modern, secure, and scalable Kubernetes delivery pipelines.

yaml
Copy code

---

Would you like me to add a **"Repository Structure"** section next (showing how your files like `application.yml`, manifests, and images are organized)?  
It makes the README appear more professional and complete for GitHub recruiters.