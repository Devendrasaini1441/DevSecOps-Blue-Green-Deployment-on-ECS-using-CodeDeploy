# DevSecOps Blue-Green Deployment on AWS ECS using CodePipeline & CodeDeploy

## Project Overview
This project demonstrates a production-grade DevSecOps CI/CD pipeline that implements Blue-Green deployment on Amazon ECS (EC2 launch type) using AWS CodeDeploy, CodePipeline, and Application Load Balancer (ALB).
The pipeline integrates security at every stage (DevSecOps) and enables zero-downtime deployments with automated traffic shifting and rollback capability.

## Key Objectives

Implement zero-downtime Blue-Green deployment

Automate build, test, security scan, and deployment

Integrate DevSecOps tools (SonarQube, Trivy, OWASP Dependency Check)

Use AWS managed CI/CD services

Achieve production-ready deployment strategy

# Architecture Overview
![DevSecOps Architecture_page-0001](https://github.com/user-attachments/assets/84c8c001-7878-4776-92d6-6aeb66f1c707)

## Introduction
In today’s fast-paced software development landscape, DevSecOps has become a cornerstone for delivering applications that are not only fast and reliable but also secure by design. By embedding security into every stage of the software delivery lifecycle, teams can reduce risks without compromising deployment speed. One of the most effective deployment techniques that aligns perfectly with DevSecOps principles is Blue-Green deployment.
In this article, we explore the Blue-Green deployment strategy and walk through its implementation for a Swiggy-clone application deployed on AWS ECS (Elastic Container Service) using AWS CodePipeline.

## Understanding Blue-Green Deployment
Blue-Green deployment is a release management strategy designed to achieve zero downtime and safe rollouts. It works by maintaining two identical production environments—commonly referred to as Blue and Green.

At any given moment, one environment (for example, Blue) handles live production traffic, while the other (Green) remains idle. When a new application version is ready, it is deployed to the inactive environment. After validation and health checks are completed successfully, traffic is switched from the active environment to the updated one. This approach allows teams to instantly roll back to the previous version if any issues are detected, ensuring high availability and minimal risk.

## Deploying the Swiggy-Clone on AWS ECS
To demonstrate this strategy, we deploy a containerized Swiggy-clone application using AWS ECS, Amazon’s fully managed container orchestration service. ECS provides the scalability and reliability needed to manage container workloads efficiently while integrating seamlessly with other AWS services such as Application Load Balancer and CodeDeploy.

## Implementing Blue-Green Deployment Using AWS CodePipeline
AWS CodePipeline is a managed CI/CD service that automates the stages involved in releasing software—ranging from source code retrieval to deployment. Below is a high-level overview of how a Blue-Green deployment pipeline can be implemented using CodePipeline:
### 1. Source Stage
The pipeline begins by connecting to a source code repository, such as GitHub. Any change pushed to the repository automatically triggers the pipeline, ensuring continuous integration.
### 2. Build Stage
AWS CodeBuild is used to compile the application, build the Docker image, and execute any required tests or security checks. This stage ensures the application is ready for deployment.
### 3. Deploy Stage
AWS CodeDeploy manages the Blue-Green deployment process on ECS:

A. Two environments (Blue and Green) are maintained behind an Application Load Balancer.

B. The new version of the Swiggy-clone application is deployed to the inactive (Green) environment.

C. CodeDeploy shifts traffic from the Blue environment to the Green environment once health checks pass.

D. Continuous monitoring ensures that if any issues arise during deployment, traffic is automatically rolled back to the stable Blue environment.


This approach enables secure, automated, and zero-downtime deployments, making it an ideal strategy for modern cloud-native applications following DevSecOps best practices.
