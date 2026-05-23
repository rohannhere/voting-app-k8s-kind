# GitOps-Driven Multi-Tier Kubernetes Deployment

## Introduction

This repository contains a complete, end-to-end GitOps deployment project, built as part of an ongoing cloud and DevOps engineering journey. It demonstrates how to automate the deployment of a multi-tier application on a local Kubernetes cluster hosted on an AWS EC2 instance. By utilizing ArgoCD, the infrastructure automatically syncs with the manifests in this repository, ensuring a single source of truth, automated monitoring, and self-healing deployments.

---

## Architecture Diagram

```mermaid
graph TD
    %% Define External Entities
    Dev[Developer/Operator]
    Git["GitHub Repository <br/> Manifests"]
    User[End User]

    %% AWS Infrastructure
    subgraph "AWS Cloud (EC2 t2.medium)"
        subgraph "Docker Environment"
            subgraph "Kind Kubernetes Cluster (3-Node)"

                %% GitOps Tool
                Argo["ArgoCD <br/> GitOps Controller"]

                %% Application Microservices
                subgraph "Voting App Namespace"
                    Vote["Vote App <br/> Frontend (Python) <br/> Port: 5000"]
                    Redis[("Redis <br/> Message Queue")]
                    Worker["Worker <br/> Background Processor"]
                    DB[("PostgreSQL <br/> Database")]
                    Result["Result App <br/> Frontend (Node.js) <br/> Port: 5001"]
                end

                %% **Added Kubernetes Dashboard Monitoring**
                subgraph "Monitoring & Administration"
                    K8sDash["Kubernetes Dashboard <br/> Monitoring UI"]
                    DashSec["Service Account & <br/> ClusterRoleBinding <br/> (RBAC Security)"]
                end
            end
        end
    end

    %% GitOps Flow
    Dev -->|git push| Git
    Git -->|Monitors & Pulls| Argo
    Argo -.->|Syncs app resources| Vote
    Argo -.->|Syncs app resources| Redis
    Argo -.->|Syncs app resources| Worker
    Argo -.->|Syncs app resources| DB
    Argo -.->|Syncs app resources| Result

    %% App Traffic Flow
    Vote -->|1. Queues vote| Redis
    Worker -->|2. Fetches vote| Redis
    Worker -->|3. Commits vote| DB
    DB -->|4. Reads tally| Result

    %% External Access for End User
    User -->|Access via Port 5000| Vote
    User -->|Access via Port 5001| Result

    %% **Operator Access for Monitoring**
    Dev -->|Access via HTTPS & Token| K8sDash
    K8sDash -.->|Secured by| DashSec
    K8sDash -.->|Monitors Workloads| Vote
```

- **Traffic Flow:**

1. **Developer** pushes Kubernetes manifests to **GitHub**.
2. **ArgoCD** continuously monitors the repository for changes.
3. ArgoCD syncs and deploys resources to a **3-Node Kind Cluster** running on **AWS EC2**.
4. **Users** access the Voting App (Frontend) and Results App via exposed NodePorts.

---

## Tech Stack

* **Cloud Provider:** AWS (EC2 `c7i-flex.large (2vCPU, 4GiB Memory)` for multi-node capacity)
* **Containerization:** Docker
* **Kubernetes Environment:** Kind (Kubernetes in Docker, similar to minicube)
* **Continuous Deployment (GitOps):** ArgoCD
* **Monitoring:** Official Kubernetes Dashboard
* **Application Stack:** Python (Voting App), Node.js (Results App), Redis (Message Broker), PostgreSQL (Database)

---

## Repository Structure
All the necessary files to spin up and configure the cluster are organized as:
* `kind-k8s/` : Contains all required shell scripts (`.sh`), Kubernetes manifests (`.yml`), and command cheat sheets (`commands.md`).
* `images/` : Architecture diagrams and screenshots.
* `README.md` : Project documentation.
* `k8s-specifications/`: Contains all the Kubernetes manifests required to run the multi-tier application.

---

## Steps to Deploy

### Step 1. Provision Infrastructure
Launch an Ubuntu `c7i-flex.large` (2vCPU, 4GiB Memory) EC2 instance on AWS. Ensure ports `80`, `443`, `8080`, `5000`, `5001`, and `8443` are allowed in your Security Group's inbound rules.

### Step 2: Install Prerequisites
* Update System Packages: Update the package manager on your EC2 instance to ensure you are pulling the latest software versions.
* Install Docker: Download and install the Docker engine. Since we are using Kind (Kubernetes in Docker), Docker acts as the foundational layer that will host our cluster nodes.

![](<./screenshots/Screenshot 2026-05-22 100034.png>)

* Configure Permissions: Add your current system user to the Docker user group. This allows you to execute Docker commands seamlessly without needing to type sudo every time.
* Apply Group Changes: Refresh your user's group assignments so the new Docker permissions take effect immediately without requiring a logout.

### Step 3: Setup Kubernetes Cluster (Kind) & Kubectl
* Download Kind: Fetch the Kind binary file directly from the official release repository. (available in provided install-kind.sh file)
* Make Executable: Modify the file permissions of the .sh file so the system recognizes it as an executable program.

![](<./screenshots/Screenshot 2026-05-22 102159.png>)

* Create the Cluster: Instruct Kind to build a new Kubernetes cluster. Please refer given config.yml file that dictates the architecture—specifically, telling it to provision one control plane node and two worker nodes.

![](<./screenshots/Screenshot 2026-05-22 102711.png>)

* Verify Kubectl: Ensure the Kubernetes command-line tool (kubectl) is installed so you can communicate with and manage your newly created cluster. (refer install_kubectl.sh file)

![](<./screenshots/Screenshot 2026-05-22 103717.png>)


### Step 4: Install & Configure ArgoCD
* Create Namespace: Create a dedicated, isolated workspace (namespace) within your Kubernetes cluster specifically to hold all ArgoCD components.

![](<./screenshots/Screenshot 2026-05-22 105136.png>)

* Deploy ArgoCD: Pull the official installation manifests from the Argo Project repository and apply them to your cluster. This spins up the various ArgoCD services and controllers inside the namespace you just created. (you will find it in commands.md file)

![](<./screenshots/Screenshot 2026-05-22 105136.png>)

* Retrieve Credentials: By default, ArgoCD generates a secure, randomized admin password and stores it as a Kubernetes Secret. You will need to query the cluster for this specific secret and decode it to get your initial login password. (you will find it in commands.md file)

### Step 5: Deploy the Application via GitOps
* Access the UI: Make the ArgoCD dashboard accessible over the network, either by exposing its service as a NodePort or by setting up a port-forward.
* Log In: Open the ArgoCD web interface in your browser and authenticate using the default 'admin' username and the decoded password you retrieved in the previous step.

![](<./screenshots/Screenshot 2026-05-22 153037.png>)

* Create Application Link: Inside ArgoCD, configure a "New App." You will provide the URL to your GitHub repository and point it to the exact folder containing your Voting App Kubernetes manifests.

![](<./screenshots/Screenshot 2026-05-22 162419.png>)

![](<./screenshots/Screenshot 2026-05-22 162528.png>)

* Initiate Sync: Trigger the synchronization process. ArgoCD will read the manifests from GitHub and automatically orchestrate the creation of your Redis, Postgres, Voting, and Result pods.

![](<./screenshots/Screenshot 2026-05-22 163049.png>)

### Step 6: Expose the Application
* Forward Frontend Traffic: Map the internal Kubernetes port for the Voting application to a port on your EC2 instance, binding it to all network interfaces so it can accept external traffic.

![](<./screenshots/Screenshot 2026-05-22 163737.png>)

* Forward Results Traffic: Repeat the port-forwarding process for the Results application on a separate port.

![](<./screenshots/Screenshot 2026-05-22 163803.png>)

* Access the App: Open a web browser and navigate to your EC2 instance's public IP address (specifying the respective ports) to cast a vote and watch the results update in real-time.

![](<./screenshots/Screenshot 2026-05-22 164003.png>)
![](<./screenshots/Screenshot 2026-05-22 164319.png>)


### Step 7: Configure & Access the Kubernetes Dashboard
* Deploy the Dashboard: Apply the official Kubernetes Dashboard manifests to your cluster. This provisions all the necessary user interface services, core infrastructure pods, and networking components within a dedicated dashboard namespace.

![](<./screenshots/Screenshot 2026-05-22 164923.png>)

* Generate Authentication Token: Request a secure, long-lived bearer token specifically for your newly created administrative user. Because the Kubernetes Dashboard is highly secure by default, you will need to copy this token to authenticate and bypass the login screen.

![](<./screenshots/Screenshot 2026-05-22 165118.png>)

* Expose Dashboard Interface: Set up a network port-forwarding rule to map the dashboard's internal secure port (443) to an open port on your EC2 instance. Ensure it is bound to all network interfaces so it can receive inbound web traffic.

![](<./screenshots/Screenshot 2026-05-22 165734.png>)

* Log In and Monitor: Open a web browser, navigate to your EC2 instance's public IP using a secure https:// connection, paste your generated authentication token into the login prompt, and begin monitoring your cluster workloads, replica sets, and pod status in real-time.

![](<./screenshots/Screenshot 2026-05-22 165941.png>)
![](<./screenshots/Screenshot 2026-05-22 170144.png>)
![](<./screenshots/Screenshot 2026-05-22 170206.png>)
![](<./screenshots/Screenshot 2026-05-22 170222.png>)
![](<./screenshots/Screenshot 2026-05-22 170235.png>)

---

## Summary
This project successfully bridges core infrastructure setup with modern cloud-native deployment strategies. By containerizing a complex, multi-tier application and managing it strictly via GitOps principles, this setup provides a robust, scalable, and automated deployment pipeline without manual intervention in the cluster.

---

## Credits
Huge thanks to [TrainWithShubham](https://www.youtube.com/@TrainWithShubham) for the fantastic video tutorial that inspired and guided me for this project!
