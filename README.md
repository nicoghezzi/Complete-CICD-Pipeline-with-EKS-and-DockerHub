# 🚀 CI/CD Pipeline: Java → Docker → ECR → EKS

![CI/CD: Jenkins](https://img.shields.io/badge/CI%2FCD-Jenkins-%23D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![Build: Maven](https://img.shields.io/badge/Build-Maven-%23C71A36?style=for-the-badge&logo=apache-maven&logoColor=white)
![Container: Docker](https://img.shields.io/badge/Container-Docker-%230db7ed?style=for-the-badge&logo=docker&logoColor=white)
![Registry: ECR](https://img.shields.io/badge/Registry-ECR-%23FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Orchestration: Kubernetes](https://img.shields.io/badge/Orchestration-Kubernetes-%23326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Cloud: AWS](https://img.shields.io/badge/Cloud-AWS-%23FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)

A production-style CI/CD pipeline that automates the deployment of a Java application from source code to a live environment on Amazon EKS.

---

## 📌 Overview

End-to-end pipeline that:

- Builds a Java (Spring Boot) application with Maven  
- Containerizes it using Docker  
- Pushes the image to Amazon ECR  
- Deploys to Amazon EKS using Kubernetes manifests  

---

## ⚙️ Pipeline Stages

### 1. Source Checkout
- Pulls `jenkins-jobs` branch from GitLab  
- Uses Jenkins credentials for authentication  

### 2. Build
- Runs `mvn clean package`  
- Executes unit tests  
- Produces JAR artifact  

### 3. Docker Build & Push
- Builds image using Amazon Corretto base image  
- Tags and pushes to ECR  
- Authenticates with AWS  

### 4. Deploy to EKS

- Configure kubeconfig:
  
  `aws eks update-kubeconfig --region us-east-1 --name demo-cluster`

- Deploy to cluster:
  
  `kubectl apply -f kubernetes/`

- Performs rolling updates  

---

## 🧰 Tech Stack

- Jenkins (CI/CD) = because  
- Maven (Build)  
- Docker (Containerization)  
- AWS ECR (Registry)  
- Kubernetes / EKS (Orchestration)  
- AWS (Cloud)  
- aws-cli, kubectl  
- GitLab  

---

## ⚠️ Challenges & Fixes

- **AWS CLI missing in Jenkins** → Installed CLI in execution environment  
- **kubectl could not connect** → Generated kubeconfig via AWS CLI  
- **Wrong EKS cluster name** → Verified directly in AWS Console  
- **Jenkins agent vs controller confusion** → Ensured tools run in agent  
- **Docker permission issues** → Fixed execution context and permissions  

> Pipeline required 23+ runs to stabilize (see `pipelines_failures/`)

---

## ✅ Result

- ✔ Application builds successfully  
- ✔ Image pushed to ECR  
- ✔ Kubernetes resources created  
- ✔ Application deployed to EKS  

---

## 💡 Key Learnings

- CI/CD failures are often **environment-related, not code-related**  
- Strong understanding of **Jenkins execution model (agent vs controller)**  
- Hands-on experience with:
  - AWS CLI in pipelines  
  - ECR authentication  
  - EKS kubeconfig setup  
  - Kubernetes deployments  

---

## 🧪 Takeaway

Go to proof folder on project root to find evidence.
This project demonstrates the ability to design, implement, and debug a real-world CI/CD pipeline across Jenkins, Docker, AWS, and Kubernetes — focusing on reliability, not just implementation.
