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
GitHub Actions triggers on every push to the `Main` branch. It builds Docker images for the microservices, pushes them to Amazon ECR, and automatically updates the Kubernetes manifests in the `gitops/` directory with the new image tags.

### 2. Continuous Deployment (CD) & GitOps
Argo CD monitors the `gitops/` folder in the GitHub repository. As soon as the CI pipeline updates the manifests, Argo CD synchronizes the changes, ensuring the EKS cluster state always matches the desired state in Git.

### 3. Observability & Log Shipping
- **Fluent Bit** runs as a DaemonSet on EKS, collecting logs from every container and streaming them to **AWS CloudWatch Logs**.
- **Prometheus** collects real-time performance metrics (CPU, Memory, Latency) from the microservices.

### 4. AIOps Diagnostics (The "Kira" Assistant)
When an issue occurs (e.g., 503 errors or OOM kills), a human operator can ask the **Bedrock Agent** (Kira) for help. 
- The agent uses **Lambda functions** to "tool-call" into CloudWatch and Prometheus.
- It analyzes the raw data using LLMs and provides a human-readable summary of the root cause and a suggested fix.

---

## 🚀 Getting Started

Detailed setup instructions for the AIOps component can be found in the [AIOps Assistant Sub-directory](./projects/aiops-assistant/README.md).

### Quick Deployment:
1. **Scale your EKS cluster**: Ensure you have at least 2 nodes (`m7i-flex.large` or `t3.medium`).
2. **Setup Logging**: Install the `aws-for-fluent-bit` helm chart and ensure IAM policies are attached to the node role.
3. **Deploy App**: Apply the Argo CD application manifest:
   ```bash
   kubectl apply -f gitops/argo-cd.yml
   ```
4. **Run Kira**: Start the Streamlit UI in the `projects/aiops-assistant` folder.

---

## 💌 Special Thanks

A huge thank you to **Vishakha Sadhwani** for creating such an incredible and high-quality project series. This series provide a perfect bridge between traditional DevOps and the future of AIOps.

- **Part 1: DevOps & GitOps** — [Watch on YouTube](https://youtu.be/abkv1MSk1lU)
- **Part 2: AIOps & Bedrock Integration** — [Watch on YouTube](https://youtu.be/WyLWj5MjzCE)

Thank you, Vishakha, for your dedication to the community and for providing these resources for free!

---
