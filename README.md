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

## Step 1: Creating a SonarQube Server (Code Quality & Security Analysis)
To perform Static Code Analysis in our DevSecOps pipeline, we first need to set up a SonarQube Server. SonarQube helps in identifying code smells, bugs, and security vulnerabilities before deploying the application.

i. Open the AWS Management Console, navigate to EC2 → Key Pairs, and click Create key pair.
ii. Enter a suitable name for the key pair, choose the key format as .pem, and click Create.
The .pem file will be downloaded to your local machine.

Next, go to the EC2 Dashboard and click on Launch Instance.
Provide a meaningful name for the instance (for example, sonar-server)
Select Ubuntu as the Amazon Machine Image (AMI) and choose t2.medium as the instance type.
In the Key Pair section, select the key pair that you created earlier.
<img width="940" height="444" alt="image" src="https://github.com/user-attachments/assets/fefb3e73-5f41-4cc1-a7f2-19bf6c42bd2d" />

Leave the remaining settings as default and click Launch Instance.
Once the instance reaches the Running state, select it and click Connect.
<img width="940" height="427" alt="image" src="https://github.com/user-attachments/assets/e2594011-596e-4d65-bdec-cf83fd814468" />

You can connect using EC2 Instance Connect or via SSH using the downloaded .pem file.
After connecting to the instance, install Docker, which will be used to run SonarQube.

```go
# Installing Docker 
sudo apt update
sudo apt install docker.io -y
sudo usermod -aG docker ubuntu
sudo systemctl restart docker
sudo chmod 777 /var/run/docker.sock
```

Start SonarQube as a Docker container using the following command:

```go
# Run SonarQube container
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```
Make sure that port 9000 is allowed in the security group attached to the EC2 instance.
<img width="940" height="449" alt="image" src="https://github.com/user-attachments/assets/7ebf05c3-6a73-4282-8739-ed0166c42c43" />

Finally, access the SonarQube dashboard using the following URL:

```go
http://<public_ip>:9000
```
<img width="940" height="452" alt="image" src="https://github.com/user-attachments/assets/e36cfc3f-a5dd-4a27-b08b-6cdd539a912a" />

Default Credentials:

Username: admin

Password: admin


## Step-2: SonarQube Configuration and Token Setup

After successfully launching the SonarQube server, the next step is to configure SonarQube and generate an authentication token that will be used by AWS CodeBuild to perform static code analysis securely.


<img width="940" height="452" alt="image" src="https://github.com/user-attachments/assets/64b9f579-c203-4e30-88d4-dd6b6002a778" />


### 1. Change Default Password

Log in to the SonarQube dashboard.

You will be prompted to change the default password.

Set a strong custom password and proceed.


<img width="1001" height="515" alt="image" src="https://github.com/user-attachments/assets/44e13af0-825b-4122-a0db-4f1cfcad282b" />


### 2. Create a New Project Manually

From the SonarQube dashboard, click on Create Project.

Choose the Manual option (<> icon).

<img width="940" height="452" alt="image" src="https://github.com/user-attachments/assets/2ec27a96-1bdd-4b3e-9ab6-33848b09d5d1" />

### 3. Provide Project Details

Enter a Project Name (for example, Swiggy).

The project key will be generated automatically.

<img width="940" height="476" alt="image" src="https://github.com/user-attachments/assets/63972375-bf90-4271-9ed9-97615772ece9" />

### 4. Select Local Analysis

Select Locally as the project setup method.

<img width="940" height="481" alt="image" src="https://github.com/user-attachments/assets/36e9f5fe-9f39-4052-bd72-d48308a948a4" />

### 5. Complete Project Setup

Click on Set Up to finalize the project configuration.

<img width="940" height="475" alt="image" src="https://github.com/user-attachments/assets/a528c967-b860-44ff-a2c1-1a48a4ce55a2" />


### 6. Generate SonarQube Token

Click on Generate to create a new authentication token.

Copy the generated token and store it securely.

<img width="940" height="454" alt="image" src="https://github.com/user-attachments/assets/60b24cbd-e460-4f91-a4eb-834a2b914cdd" />


### 7. Select Analysis Configuration

Under the Run analysis section:

Choose Other as the build tool.

Select Linux as the operating system.

Copy the Sonar token shown on the screen.

### 8. Store Secrets in AWS Systems Manager Parameter Store

Open the AWS Console.

Search for Systems Manager.

Navigate to Parameter Store.
<img width="940" height="450" alt="image" src="https://github.com/user-attachments/assets/1e529181-a1c6-429a-af8f-7e11da2779c2" />


### 9. Create a New Parameter

Click on Create parameter.

### 10. Add SonarQube Token

Provide a parameter name for the Sonar token.

Paste the copied Sonar token in the Value field.

Set the type as SecureString.
<img width="940" height="448" alt="image" src="https://github.com/user-attachments/assets/ec41ee09-c7a7-4850-9810-0d0268f75c02" />


⚠️ The parameter name must match the reference used in buildspec.yaml.

### 11. Store Docker Credentials in Parameter Store

Similarly, create additional parameters for Docker credentials and registry details:
<img width="940" height="447" alt="image" src="https://github.com/user-attachments/assets/c722eada-298c-4850-b1ce-b498afd7d3be" />


```go
Parameter name: /cicd/sonar/sonar-token
Value: <sonar_token>

Parameter name: /cicd/docker-credentials/username
Value: <docker_username>

Parameter name: /cicd/docker-credentials/password
Value: <docker_password>

Parameter name: /cicd/docker-registry/url
Value: docker.io
```

## Step-3: Create an AWS CodeBuild Project

In this step, we configure AWS CodeBuild to build the application, run security scans, and generate artifacts required for deployment.

Open the AWS Management Console, navigate to CodeBuild, and click on Create build project.
<img width="940" height="450" alt="image" src="https://github.com/user-attachments/assets/107bb28d-d61d-4b74-9aa6-63e351836493" />


Enter a suitable Project name for the build project.
<img width="940" height="448" alt="image" src="https://github.com/user-attachments/assets/b55df0fe-951d-4a33-851e-34b0612c7ade" />


In the Source section, select GitHub as the source provider.
<img width="940" height="448" alt="image" src="https://github.com/user-attachments/assets/65d4e8ae-f902-4654-a11a-ed5581478158" />


Choose Connect using OAuth to authenticate with GitHub.
<img width="1007" height="439" alt="image" src="https://github.com/user-attachments/assets/8f781a4c-7553-4c17-b876-63dfd8594119" />


Complete the GitHub authorization process by granting the required permissions and logging in.

From the list of repositories, select the GitHub repository that contains your application source code.

In the Environment section, keep all settings as default.
<img width="999" height="438" alt="image" src="https://github.com/user-attachments/assets/108dafa4-79e0-4151-88ba-1138d3d78588" />


In the Buildspec section:

Select Use a buildspec file
<img width="1002" height="440" alt="image" src="https://github.com/user-attachments/assets/be1108e1-738a-42cb-b9e6-fb63160a6d85" />


Enter the file name as buildspec.yaml

In the Artifacts section, choose an existing S3 bucket to store the build artifacts.

Click on Update project to save the configuration.
<img width="940" height="452" alt="image" src="https://github.com/user-attachments/assets/eba67bc4-b27c-4922-ae54-064685004c69" />


Navigate to IAM → Roles and open the IAM role automatically created by CodeBuild for this project.

Attach the following permissions to the role:

AmazonSSMFullAccess (to access secrets from Parameter Store)

AmazonS3FullAccess (to upload build artifacts)
<img width="940" height="449" alt="image" src="https://github.com/user-attachments/assets/235293c1-61fc-4335-91bc-b31dd77f567d" />


Return to the CodeBuild console and click Start build.

Once the build completes successfully, you should observe the following results in the logs and reports:
<img width="940" height="451" alt="image" src="https://github.com/user-attachments/assets/61e54057-59a5-4f6f-ae01-143d94bd3dfe" />


SonarQube analysis executed successfully
<img width="940" height="452" alt="image" src="https://github.com/user-attachments/assets/7d4d6266-2cdb-48c6-b0ad-cacd37d83379" />

<img width="940" height="454" alt="image" src="https://github.com/user-attachments/assets/207de374-ad50-4f34-a73a-bdadaff86f61" />


OWASP Dependency-Check reports generated

Trivy filesystem scan completed
<img width="940" height="450" alt="image" src="https://github.com/user-attachments/assets/1cc38e02-a482-42cf-9532-86f6b32b82cf" />


Trivy image scan completed



<img width="940" height="441" alt="image" src="https://github.com/user-attachments/assets/fc7e2b07-f919-4f22-80f8-ee5eaed29938" />
<img width="940" height="450" alt="image" src="https://github.com/user-attachments/assets/fbb39584-f6a2-4e94-ba67-1e1fd06f3139" />
<img width="940" height="448" alt="image" src="https://github.com/user-attachments/assets/46d81e40-0e97-4869-b9f2-26257d378d9a" />
<img width="940" height="446" alt="image" src="https://github.com/user-attachments/assets/d27ec344-5796-4acd-819a-0a4a32a9ec21" />
<img width="940" height="445" alt="image" src="https://github.com/user-attachments/assets/a84e3687-feae-4244-970c-5776dd4c3f97" />
<img width="940" height="449" alt="image" src="https://github.com/user-attachments/assets/39ba75a7-da4b-43e7-8237-cd1b27267f0a" />
<img width="940" height="446" alt="image" src="https://github.com/user-attachments/assets/fe67ad68-644b-46d4-a60b-7150e8057875" />
<img width="940" height="446" alt="image" src="https://github.com/user-attachments/assets/9221e529-d3f9-498c-9891-baf72366e674" />
<img width="940" height="448" alt="image" src="https://github.com/user-attachments/assets/31a4097f-a009-4cf8-a66d-81cef17bc8a2" />
<img width="940" height="448" alt="image" src="https://github.com/user-attachments/assets/b59295fb-3ab6-41ab-b209-e0de2449fcef" />
<img width="940" height="447" alt="image" src="https://github.com/user-attachments/assets/75e81536-d993-4ebf-9830-1248b465d699" />
<img width="940" height="450" alt="image" src="https://github.com/user-attachments/assets/51e38a01-d7c2-4b8d-baff-2f96b1583e6f" />
<img width="940" height="445" alt="image" src="https://github.com/user-attachments/assets/effc5c4d-ac9a-4c82-a6f2-01ca3488ce11" />

<img width="940" height="453" alt="image" src="https://github.com/user-attachments/assets/0ec8844d-6f7f-4173-bbc6-2ea52f5585c0" />
<img width="940" height="453" alt="image" src="https://github.com/user-attachments/assets/88c76acc-b695-4516-802b-1a95f4dfb926" />


















