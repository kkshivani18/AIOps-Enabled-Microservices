# AIOps-Enabled Microservices: Boutique Web App on AWS EKS

An end-to-end DevOps and AIOps project featuring a polyglot microservices application ("Boutique") deployed on Amazon EKS, integrated with a GitOps workflow, and monitored by an AI-powered SRE assistant (Kira).

---

## 🌟 Overview

This project demonstrates a production-grade cloud-native architecture. It moves beyond traditional DevOps by implementing **AIOps**—using Generative AI to diagnose incidents, analyze logs, and monitor cluster health. The application is a multi-tier microservices web store (Google's Online Boutique) running on a highly available EKS cluster.

### Key Pillars:
- **Infrastructure as Code**: EKS Cluster management and scaling.
- **GitOps**: Automated deployments using **Argo CD**.
- **Observability**: **Prometheus** for metrics and **Fluent Bit** for log shipping to **CloudWatch**.
- **AIOps**: **AWS Bedrock Agent** (Kira) acting as an automated SRE to troubleshoot production issues.

---

## Architecture

![full architecture](/images/architecture%20and%20workflow/Screenshot%202026-05-01%20094458.png)

### 1. Microservices Layer

The application follows a decoupled microservices architecture where each service is responsible for a specific business domain.

![API Gateway](/images/architecture%20and%20workflow/Screenshot%202026-05-01%20194814.png)

*   **API Gateway**: Acts as the single entry point for all client requests, routing them to the appropriate backend services and handling security.
*   **Core Services**: Includes the **Auth Service** (identity), **Product Service** (catalog), and **Order Service** (checkout).
*   **Frontend**: A web interface that communicates with the backend via the Gateway.
*   **Database Integration**: Each service connects to its own persistent storage (PostgreSQL).

### 2. Infrastructure & Kubernetes Layer

![Infrastructure & Kubernetes Layer](/images/architecture%20and%20workflow/kuber%20and%20docker.png)

The entire stack is hosted on **Amazon EKS (Elastic Kubernetes Service)** across multiple worker nodes to ensure high availability and fault tolerance.


![cluster](/images/architecture%20and%20workflow/kuber%20and%20docker.png)

*   **Cluster Design**: The application is deployed across a multi-node cluster (Node 1, Node 2, Node 3) connected via a common internal network.
*   **Containerization**: Every service is containerized using **Docker** and orchestrated by Kubernetes.

![Observability Stack](/images/architecture%20and%20workflow/Screenshot%202026-05-01%20202537.png)

*   **Observability Stack**:
    *   **Prometheus**: Scrapes metrics from the backend services and system nodes.
    *   **Grafana**: Visualizes those metrics through dynamic dashboards.
    *   **Fluent Bit**: Collects container logs and ships them to **AWS CloudWatch** for centralized logging.
*   **AIOps Integration**: An AI-powered assistant (Kira) uses AWS Bedrock and Lambda functions to analyze these logs and metrics to diagnose issues in real-time.

---

## 🛠️ Tech Stack

- **Cloud**: Amazon Web Services (EKS, ECR, CloudWatch, Bedrock, Lambda)
- **Containerization**: Docker
- **Orchestration**: Kubernetes (EKS)
- **GitOps**: Argo CD
- **CI/CD**: GitHub Actions
- **Logging**: AWS For Fluent Bit
- **Monitoring**: Prometheus & Grafana
- **AI/ML**: AWS Bedrock (Claude 3 / Titan)
- **Frontend**: Streamlit (Python)

---

## 🔄 Project Workflow

### 1. Continuous Integration (CI)

![ci workflow](/images/architecture%20and%20workflow/Screenshot%202026-05-01%20202416.png)

GitHub Actions triggers on every push to the `Main` branch. It builds Docker images for the microservices, pushes them to Amazon ECR, and automatically updates the Kubernetes manifests in the `gitops/` directory with the new image tags.

### 2. Continuous Deployment (CD) & GitOps

![cd workflow](/images/architecture%20and%20workflow/Screenshot%202026-05-01%20202759.png)

Argo CD monitors the `gitops/` folder in the GitHub repository. As soon as the CI pipeline updates the manifests, Argo CD synchronizes the changes, ensuring the EKS cluster state always matches the desired state in Git.

### 3. Observability & Log Shipping

![Observability Stack](/images/architecture%20and%20workflow/Screenshot%202026-05-01%20202537.png)

- **Fluent Bit** runs as a DaemonSet on EKS, collecting logs from every container and streaming them to **AWS CloudWatch Logs**.
- **Prometheus** collects real-time performance metrics (CPU, Memory, Latency) from the microservices.

### 4. AIOps Diagnostics (The "Kira" Assistant)

![aiops workflow](/images/architecture%20and%20workflow/Screenshot%202026-05-01%20202840.png)

When an issue occurs (e.g., 503 errors or OOM kills), a human operator can ask the **Bedrock Agent** (Kira) for help. 
- The agent uses **Lambda functions** to "tool-call" into CloudWatch and Prometheus.
- It analyzes the raw data using LLMs and provides a human-readable summary of the root cause and a suggested fix.

---

## 🚀 For Getting Started

Detailed setup instructions for the AIOps component can be found in the [AIOps Assistant Sub-directory](./projects/aiops-assistant/README.md).

### Quick Deployment:

1. **Scale your EKS cluster**: Ensure you have at least 2 nodes (`m7i-flex.large` or `t3.medium`).

![aiops workflow](/images/helm%20upgrade/Screenshot%202026-05-03%20150331.png)

![aiops workflow](/images/fluentBit/Screenshot%202026-05-03%20155342.png)

2. **Setup Logging**: Install the `aws-for-fluent-bit` helm chart and ensure IAM policies are attached to the node role.

3. **Deploy App**: Apply the Argo CD application manifest:
   ```bash
   kubectl apply -f gitops/argo-cd.yml
   ```

![streamlit UI](/images/streamlit/Screenshot%202026-05-03%20200536.png)

4. **Run Kira**: Start the Streamlit UI in the `projects/aiops-assistant` folder.

---

## 🤖 AIOps in Action: Incident Diagnosis Scenario

One of the most powerful features demonstrated in this project is Kira's ability to diagnose a "silent failure" caused by manual scaling errors.

### The Scenario:

![aiops working 1](/images/aiops%20working/Screenshot%202026-05-03%20201734.png)

1.  **The Incident**: A developer accidentally scales the `orders` deployment down to **0 replicas** (or a node failure occurs).

2.  **The Symptom**: The frontend starts showing "Service Unavailable" errors.

![aiops working 2](/images/aiops%20working/Screenshot%202026-05-03%20201702.png)

3.  **The Diagnosis**:
    *   The user asks Kira: *"The store is down, can you check why?"*
    *   **Kira** automatically calls the `fetch_service_health` tool and identifies that the `orders` has **0/1 pods running**.
    *   **Kira** then cross-references this with `fetch_logs` to see if there were any crashes or if it was a manual scale-down.

![aiops working 3](/images/aiops%20working/Screenshot%202026-05-03%20203458.png)

![aiops working 4](/images/aiops%20working/Screenshot%202026-05-03%20203448.png)

4.  **The Resolution**: Kira reports the exact deployment name that is missing and provides the `kubectl` command to scale it back up to 1 replica.

---

## 💌 Special Thanks

A huge thank you to **Vishakha Sadhwani** for creating such an incredible and high-quality project series. This series provide a perfect bridge between traditional DevOps and the future of AIOps.

- **Part 1: DevOps & GitOps** — [Watch on YouTube](https://youtu.be/abkv1MSk1lU)
- **Part 2: AIOps & Bedrock Integration** — [Watch on YouTube](https://youtu.be/WyLWj5MjzCE)

Thank you, Vishakha, for your dedication to the community and for providing these resources for free!

---

## 📖 Blog Post
Check out the detailed article on this project on Medium: 
[Architecting Intelligent Microservices: Bridging the Gap Between AI and DevOps](https://medium.com/@kkrishnashivani18/architecting-intelligent-microservices-bridging-the-gap-between-ai-and-devops-d981826a6446)

---