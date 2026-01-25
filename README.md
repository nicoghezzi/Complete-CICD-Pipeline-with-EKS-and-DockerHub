## Project Overview

Built a production-style CI/CD pipeline for Kubernetes deployments using Jenkins, automating build, containerization, and delivery to AWS EKS and DigitalOcean LKE.
The project covered Maven-based Java applications, secure access to private Docker registries, and dynamic Kubernetes deployments across multiple environments.
Configured Jenkins and supporting tooling to enable fast, repeatable, and secure end-to-end application delivery.

## Stack and Tooling

- Jenkins
Designed multi-stage pipelines to automate build, test, and deployment workflows.

- AWS EKS and DigitalOcean LKE
Used as managed Kubernetes clusters to simulate staging and production environments across clouds.

- Docker
Containerized Java Maven applications for consistent Kubernetes deployments.

- kubectl
Applied manifests and managed rollouts directly from Jenkins pipelines.

- AWS CLI and GitLab
Handled cluster authentication, credential validation, and source control integration.

## What I Implemented

- End-to-End CI/CD Automation
Built pipelines that handle Maven builds, Docker image creation, and Kubernetes deployments across multiple cloud providers.

- Secure Pipeline Configuration
Managed cloud credentials, Docker registry access, and Kubernetes configurations using Jenkins credential management.

- Dynamic and Reusable Deployments
Templated Kubernetes manifests with environment variables to support environment-specific deployments without duplication.

- Version Control Automation
Integrated GitLab to automate version updates and commits as part of the deployment lifecycle.

## Challenges and Solutions

- Private Container Registry Access
Enabled secure image pulls in Kubernetes by configuring and referencing image pull secrets.

- Multi-Cloud Deployment Logic
Parameterized Jenkins pipelines and managed separate kubeconfigs to support both EKS and LKE from a single workflow.

- Credential Handling
Avoided exposing sensitive data by using Jenkins credential stores and secure injection patterns.

## Why This Project Is Relevant

This project reflects real-world CI/CD practices for cloud-native applications, combining Jenkins, Docker, and Kubernetes to deliver software reliably across multiple cloud environments. It demonstrates the ability to design secure, scalable deployment pipelines—core skills for DevOps, SRE, and cloud engineering roles.