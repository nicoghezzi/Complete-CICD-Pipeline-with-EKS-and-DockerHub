## Problem Statement

I wanted to design and implement a realistic CI/CD pipeline that takes a Java application from source code to a live deployment on Amazon EKS, using industry-standard DevOps tools.
The goal was not just success — but to understand and debug the real failures that occur when Jenkins, Docker, AWS, and Kubernetes interact.

## Solution Overview

I built a fully automated pipeline that:
1. Builds a Java Maven application
2. Containerizes it with Docker
3. Pushes the image to AWS ECR
4. Deploys it to Amazon EKS using Kubernetes manifests

## Tools & Technologies

- CI/CD: Jenkins (Declarative Pipeline)
- Build Tool: Maven
- Containerization: Docker
- Container Registry: AWS ECR
- Orchestration: Kubernetes (EKS)
- Cloud Provider: AWS
- CLI Tools: aws-cli, kubectl
- Source Control: GitLab

## Pipeline Breakdown (4 main bullet points)

1. Source Code Checkout

Jenkins pulls the jenkins-jobs branch from GitLab
Git authentication handled via Jenkins credentials

2. Build Java Application

mvn clean package
Runs unit tests
Produces a Spring Boot JAR

3. Docker Build & Push to ECR

docker build -t <ECR_REPO>:latest .
docker push <ECR_REPO>:latest
Uses Amazon Corretto base image
Authenticates securely with AWS ECR

4. Deploy to Amazon EKS

aws eks update-kubeconfig --region us-east-1 --name demo-cluster
kubectl apply -f kubernetes/
Dynamically configures kubeconfig
Deploys application with rolling updates

## Challenges Encountered (Real-World)

This pipeline required 23+ executions to stabilize. Key issues included:

❌ Jenkins Missing AWS CLI
Symptom: aws: not found
Fix: 
- Installed AWS CLI inside the Jenkins container
- Learned that Jenkins pipelines run where the agent executes

❌ kubectl Could Not Reach Cluster
Root Cause: kubeconfig was missing
Fix: aws eks update-kubeconfig

❌ Wrong EKS Cluster Name
Error: ResourceNotFoundException: No cluster found
Fix: Verified cluster name directly in AWS Console

❌ Jenkins Agent Misunderstanding
Confusion between Jenkins controller, container, and agent
Realized tooling must exist in the agent environment

❌ Docker Permission Issues

Jenkins user lacked required privileges
Resolved by correcting execution context

## Final Result
The pipeline now completes end-to-end successfully:

✔ Code builds 
✔ Image pushed to ECR
✔ Kubernetes resources created
✔ Application deployed to EKS

This project reflects real-world CI/CD practices for cloud-native applications, combining Jenkins, Docker, and Kubernetes to deliver software reliably across multiple cloud environments. It demonstrates the ability to design secure, scalable deployment pipelines—core skills for DevOps, SRE, and cloud engineering roles.

## Some Proof screenshots

1. Screenshot showing 23 runs that made me debugged for 2 days.
![Lighthouse Report](/images/23runs.png)

2. Screenshot showing success on the CICD pipeline
![Lighthouse Report](/images/pipeline_success.png)