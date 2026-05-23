# GitOps-Driven Multi-Tier Kubernetes Deployment

## Introduction

This repository contains a complete, end-to-end GitOps deployment project, built as part of an ongoing self-taught cloud and DevOps engineering journey. It demonstrates how to automate the deployment of a multi-tier application on a local Kubernetes cluster hosted on an AWS EC2 instance. By utilizing ArgoCD, the infrastructure automatically syncs with the manifests in this repository, ensuring a single source of truth, automated monitoring, and self-healing deployments.

---

## Architecture Diagram


- **Traffic Flow:**

1. **Developer** pushes Kubernetes manifests to **GitHub**.
2. **ArgoCD** continuously monitors the repository for changes.
3. ArgoCD syncs and deploys resources to a **3-Node Kind Cluster** running on **AWS EC2**.
4. **Users** access the Voting App (Frontend) and Results App via exposed NodePorts.

---

## Tech Stack

* **Cloud Provider:** AWS (EC2 `t2.medium` for multi-node capacity)
* **Containerization:** Docker
* **Kubernetes Environment:** Kind (Kubernetes in Docker)
* **Continuous Deployment (GitOps):** ArgoCD
* **Monitoring:** Official Kubernetes Dashboard
* **Application Stack:** Python (Voting App), Node.js (Results App), Redis (Message Broker), PostgreSQL (Database)

---

## Steps to Deploy

### 1. Provision Infrastructure
Launch an Ubuntu `t2.medium` (2vCPU, 4GiB Memory) EC2 instance on AWS. Ensure ports `80`, `443`, `8080`, `5000`, `5001`, and `8443` are allowed in your Security Group's inbound rules.

### 2. Install Prerequisites
Update the system and install Docker to serve as the foundation for the Kind cluster.

### 3. Setup Kubernetes Cluster (Kind) & Kubectl
Install Kind and spin up a 3-node cluster (1 Control Plane, 2 Workers) using provided configuration file.

Note: Make sure to install kubectl to interact with the newly created cluster.

### 4. Install & Configure ArgoCD
Create an isolated namespace and install the ArgoCD provided manifests.

Retrieve the initial admin password to log into the ArgoCD UI:

### 5. Deploy the Application via GitOps
* Access the ArgoCD Web UI by port-forwarding the argocd-server service or modifying the service to a NodePort.
* Log in using admin and the password retrieved above.
* Create a New App in ArgoCD.
* Connect your GitHub repository URL and set the path to your Kubernetes manifests directory.
* Click Sync to automatically provision the Redis, Postgres, Voting, and Result pods.

### 6. Expose the Application
Use port-forwarding to access the applications externally through your EC2 public IP:

Visit http://<EC2-PUBLIC-IP>:5000 to vote, and http://<EC2-PUBLIC-IP>:5001 to view results.

---

## Summary 

This project successfully bridges core infrastructure setup with modern cloud-native deployment strategies. By containerizing a complex, multi-tier application and managing it strictly via GitOps principles, this setup provides a robust, scalable, and automated deployment pipeline without manual intervention in the cluster.
