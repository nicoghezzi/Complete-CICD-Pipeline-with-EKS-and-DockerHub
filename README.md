# 🚀 CI/CD Pipeline: Java → Docker → ECR → EKS

![CI/CD: Jenkins](https://img.shields.io/badge/CI%2FCD-Jenkins-%23D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![Build: Maven](https://img.shields.io/badge/Build-Maven-%23C71A36?style=for-the-badge&logo=apache-maven&logoColor=white)
![Container: Docker](https://img.shields.io/badge/Container-Docker-%230db7ed?style=for-the-badge&logo=docker&logoColor=white)
![Registry: ECR](https://img.shields.io/badge/Registry-ECR-%23FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Orchestration: Kubernetes](https://img.shields.io/badge/Orchestration-Kubernetes-%23326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Cloud: AWS](https://img.shields.io/badge/Cloud-AWS-%23FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)

A production-style CI/CD pipeline that automates the deployment of a Java application from source code to a live environment on Amazon EKS.

---

# Java CI/CD Pipeline with Jenkins, Docker & Amazon EKS

## 📌 Overview

An end-to-end CI/CD pipeline that automates the build, containerization, and deployment of a Java Spring Boot application to Amazon EKS.

**Pipeline:** Jenkins → Maven → Docker → Amazon ECR → Amazon EKS

The original workplace application has been replaced with a Java baseline because the production source code cannot be shared for security reasons.

**Tech Stack:** Jenkins, Java, Maven, Docker, Amazon ECR, Amazon EKS, Kubernetes, AWS CLI, kubectl, GitLab.

## ⚙️ Pipeline Stages

1. **Source Checkout:** Retrieves source code from the `jenkins-jobs` branch using Jenkins credentials.
2. **Build & Test:** Runs `mvn clean package`, executes unit tests, and produces a JAR artifact.
3. **Containerization & Registry:** Builds a Docker image using an Amazon Corretto base image, tags it, authenticates with AWS, and pushes it to Amazon ECR.
4. **Deployment to EKS:** Configures cluster access using `aws eks update-kubeconfig` and deploys Kubernetes manifests with `kubectl apply -f kubernetes/`, enabling rolling updates.

## 🧠 Engineering Decisions & Trade-offs

| Decision | Rationale | Trade-off |
|---|---|---|
| **Jenkins for CI/CD** | Used the existing workplace-standard automation platform. | Requires agent configuration, plugin maintenance, and management of build-tool dependencies. GitHub Actions or GitLab CI could offer tighter repository integration in other environments. |
| **Maven for Java builds** | Standardizes dependency management, compilation, testing, and JAR packaging. | Build reliability depends on dependency availability, plugin versions, and build configuration. |
| **Docker and Amazon ECR** | Packages the application consistently and provides a centralized AWS container registry. | Requires image optimization, vulnerability scanning, registry permissions, and image lifecycle management. |
| **Amazon EKS for deployment** | Provides managed Kubernetes orchestration and declarative application deployment. | Adds cluster, networking, compute, and operational costs compared with simpler hosting options. |
| **AWS IAM and EKS access configuration** | Enables authenticated access to AWS resources and the Kubernetes API. | Requires correct coordination between IAM permissions, EKS access configuration, and Kubernetes RBAC where applicable. |
| **Kubernetes manifests and rolling updates** | Defines deployment resources declaratively and supports controlled application updates. | Requires appropriate readiness probes, resource settings, and rollback procedures to reduce deployment risk. |

## 🛠️ Challenges & Solutions

- **AWS CLI missing in Jenkins:** Installed the CLI in the execution environment.
- **kubectl could not connect:** Generated kubeconfig using the AWS CLI.
- **Incorrect EKS cluster name:** Verified the cluster name in the AWS Console.
- **Jenkins agent vs. controller confusion:** Ensured tools executed in the correct agent environment.
- **Docker permission issues:** Corrected execution context and permissions.

**Debugging evidence:** The pipeline required 23+ runs to stabilize. See `pipelines_failures/` for failure evidence and troubleshooting history.

## ✅ Results & Key Learnings

- Built and tested the Java application with Maven.
- Built and pushed the container image to Amazon ECR.
- Created Kubernetes resources and deployed the application to EKS.
- Improved understanding of Jenkins execution environments, AWS authentication, ECR image publishing, EKS connectivity, and Kubernetes deployments.

The primary lesson was that CI/CD failures often originate from execution environments, permissions, and infrastructure configuration—not application code alone.

## 🧪 Evidence

See the `evidence/` folder in the project root for screenshots documenting the implementation and deployment.


