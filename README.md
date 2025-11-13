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

![ArgoCDP2](./project_screenshots/ArgoCDP2.jpg) `ArgoCDP2.drawio`

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
Step 2: Create an Amazon EKS Cluster (via AWS Management Console)

Instead of using the CLI, the EKS cluster was created directly through the AWS Management Console for better visualization and control.

Navigate to the Amazon EKS service in your AWS Console.

Click “Add cluster” → “Create”.

Enter a cluster name (e.g., argocd-eks-cluster).
Creating the EKS Cluster (Console Overview)**  

![Creating EKS Cluster Step 1](project_screenshots/creatingEKSCluster1.png)

 Cluster Configuration Details**  
![Creating EKS Cluster Step 2](project_screenshots/creatingEKSCluster2.png)

Select the desired Kubernetes version (e.g., 1.29).

Choose an existing IAM role with EKS permissions or create a new one.
 Final Cluster Creation Confirmation**  
![Creating EKS Cluster Final Step](project_screenshots/creatingEKSCluster.png)

Configure networking:

Select your VPC and subnets.

Allow public access for the Kubernetes API endpoint (for testing purposes).

Leave the default settings for logging and tags.

Click “Create” and wait for the cluster status to show as Active.

Once the cluster is ready, note the cluster name — it will be used to connect your local kubectl client.


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

### Viewing All Namespaces in EKS

![All Namespaces in EKS](project_screenshots/allnamesapcesoneks.png)



Step 3: Create Argo CD Namespace
Commands
bash
Copy code
kubectl create namespace argocd
kubectl get ns

### ArgoCD Successfully Installed with Components

![ArgoCD Successfully Installed](project_screenshots/ArgoCDSuccessfullyInstalledWithComponents.png)

Step 4: Install Argo CD
Commands
bash
Copy code
# Install Argo CD into the argocd namespace
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

![Creating ArgoCD Namespace and Installing ArgoCD](project_screenshots/creatingargocdnamespace-and-installing-argocd.png)

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

![Checking Service IP and Type of ArgoCD Components](project_screenshots/CheckingServicenIPTypeofArgoCDComponenents.png)

### Port Forwarding ArgoCD Service

![Port Forwarding](project_screenshots/portforwarding.png)

Insert Screenshot: ArgoCD Login Page

Step 6: Retrieve and Decode Argo CD Admin Password
Commands
bash
Copy code
kubectl get secret argocd-initial-admin-secret -n argocd -o json

echo <password> | base64 --decode 

Use the following credentials to log in:

Username: admin

Password: (decoded value above)


### Retrieving ArgoCD Admin Password

![Retrieve ArgoCD Password](project_screenshots/retrieveargocdpassword.png)

### Getting the ArgoCD Admin Password and Checking the Service

![Get Password and Check Service](project_screenshots/getpassword-check-service.png)


Insert Screenshot: Successful Login to ArgoCD


### Logging into ArgoCD

![ArgoCD Login](project_screenshots/argocdlogin.png)


### ArgoCD Plain Dashboard View

![ArgoCD Plain Dashboard](project_screenshots/argocdplaindahsboard.png)


Step 7: Deploy Application Using Application Manifest
Once logged into Argo CD UI, deploy the application defined in application.yml.



Commands
bash
Copy code
kubectl apply -f application.yml

### Applying Kubernetes Manifests with kubectl

![kubectl Apply](project_screenshots/kubectl-apply.png)


### Application Created in ArgoCD

![Application Created](project_screenshots/application-created.png)

### ArgoCD Deployment Overview (Step 1)

![ArgoCD Deployment](project_screenshots/argocddeployment.png)

### ArgoCD Deployment Overview (Step 2)

![ArgoCD Deployment 2](project_screenshots/argocddeployment2.png)

Step 8: Test Argo CD Sync Policies
Argo CD supports Automated, Manual, SelfHeal, and Prune sync strategies.
We’ll test how Argo CD reacts to changes in the Git repo and cluster.

### Scaling Policies and Self-Healing (Synchronization Behavior)

This section demonstrates **ArgoCD's response to drift** when manual changes are made to the `myapp-deployment-6646856997` ReplicaSet in the cluster. ArgoCD continuously monitors the live state and automatically reconciles it to match the desired state defined in Git.

![Scaling Policies and Drift Detection](./project_screenshots/scaling-policies-in-relatioin-drift.png)

The events below summarize the reactions and responses of ArgoCD to manual changes in the `deployment.yml` applied directly in the cluster:

| Color | Timeframe | Manual Change (Drift) | ArgoCD's Response | Effect |
| :---: | :---: | :---: | :---: | :---: |
| **Orange** | 41m ago (12:35 PM) | Deployment/Application initiated using `kubectl apply -f application.yml` | Scaled up replica set `myapp-deployment-6646856997` from **0 to 2** | **Initial deployment** based on the configured desired state (2 replicas). |
| **Yellow** | 11m - 10m ago (1:05 PM - 1:06 PM) | **Manual change detected** | Scaled down replica set from **4 to 2** (followed by a scaled up from 2 to 4) | User manually scaled the live deployment **from 2 to 4**. ArgoCD detected the drift and scaled it **back down to 2** to match the Git configuration. *(The subsequent scale up from 2 to 4 suggests the user scaled it up again, and ArgoCD instantly responded.)* |
| **Blue** | 8m ago (1:09 PM) | **Manual change detected** | Scaled down replica set from **5 to 2** (followed by a scaled up from 2 to 5) | User manually scaled the live deployment **from 2 to 5**. ArgoCD detected the drift and scaled it **back down to 2** to match the Git configuration. *(The subsequent scale up from 2 to 5 suggests the user scaled it up again, and ArgoCD instantly responded.)* |

---

### Key Takeaways 

* **ArgoCD enforces Git as the source of truth.** Any manual changes in the live cluster (e.g., using `kubectl edit deployment`) will be reverted automatically to match the Git repository.
* The **initial deployment** of the application was done using `kubectl apply -f application.yml`, which ArgoCD then monitors for drift against the Git repository.
* To permanently update the number of replicas, you **must edit the `replicas:` field in the `deployment.yaml`** in Git and push the changes. ArgoCD will then synchronize the cluster accordingly.
* This demonstrates ArgoCD’s **self-healing capabilities**, ensuring consistency between Git and the cluster at all times.


## Cleaning Up Resources

After completing the ArgoCD and EKS project, it is important to clean up the resources to avoid unnecessary costs and maintain a tidy environment. This involves:

- **Deleting the EKS Cluster:** Removes all the nodes, pods, and associated AWS resources.
- **Deleting ArgoCD Components:** Ensures that ArgoCD namespaces, deployments, and services are removed.
- **Deleting Namespaces:** Any custom namespaces created for testing or deployments are deleted to avoid lingering resources.

Proper cleanup helps maintain cloud hygiene and prevents unexpected billing from leftover resources.


### Deleting EKS Cluster

![Delete Cluster](project_screenshots/deletecluster.png)

### Deleting ArgoCD Node

![Deleting ArgoCD Node](project_screenshots/deleteingargocdnode.png)

### Deleting Namespace and Verification

![Delete Namespace and Check](project_screenshots/deletenamespaceandcheck.png)

### Editing Deployment to Test ArgoCD

![Edit Deployment](project_screenshots/editdeployment.png)

### Editing Default ArgoCD Deployment Time

![Editing Default ArgoCD Deployment Time](project_screenshots/editingdefaultargocddeploymenttime.png)


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