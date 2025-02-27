# Containerized-Web-App-Deployment-on-AWS-EC2-Docker-ECR-ECS-Fargate-ALB
This project demonstrates a full deployment workflow for a web application using key AWS services. 

![Untitled Diagram drawio (1)](https://github.com/user-attachments/assets/5b5a3130-5624-4e2c-b8ba-a5e8f1afda93)

## Overview
This project demonstrates a full deployment workflow for a web application using key AWS services. It covers:
+ Launching an EC2 instance
+ Installing and configuring Docker
+ Building and pushing a Docker image to ECR
+ Setting up an Application Load Balancer (ALB)
+ Deploying a containerized app on Amazon ECS with Fargate

## Execution Flow

### Step 1: EC2 & Docker Setup
+ **Launch EC2 Instance**  
  Launch an EC2 instance (preferably Amazon Linux 2 or Ubuntu).
![Screenshot 2025-02-27 233214](https://github.com/user-attachments/assets/ac8c2063-954f-409d-9a21-b1649e6bd597)

+ **Install Docker**  
  Install Docker on the instance:
  ```sh
  sudo yum update -y
  sudo yum install docker -y
  sudo service docker start
  sudo usermod -aG docker ec2-user
  ```
  ![Screenshot 2025-02-27 085039](https://github.com/user-attachments/assets/e763fe2e-294b-4371-b8e5-aaeb1ab1d9b4)


+ **Build Docker Image**  
  Build your web app image:
  ```sh
  docker build -t my-web-app .
  ```
![Screenshot 2025-02-27 085016](https://github.com/user-attachments/assets/3ebc6ac6-4a01-4c14-93c0-7e88864567b4)

### Step 2: Push Image to Amazon ECR
+ **Create an ECR Repository**  
  Create a repository in Amazon ECR:
  ```sh
  aws ecr create-repository --repository-name my-web-app
  ```
![Screenshot 2025-02-27 233633](https://github.com/user-attachments/assets/b33192cf-380a-421a-91ef-993902fd7c54)

+ **Authenticate Docker to ECR**  
  Login to your ECR registry:
  ```sh
  aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
  ```

+ **Tag & Push Image**  
  Tag the image for your ECR repository and push it:
  ```sh
  docker tag my-web-app:latest <aws_account_id>.dkr.ecr.<region>.amazonaws.com/my-web-app:latest
  docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/my-web-app:latest
  ```
  ![Screenshot 2025-02-27 233751](https://github.com/user-attachments/assets/f952464e-f328-452f-8390-27e2f9300f8f)


### Step 3: Set Up Application Load Balancer (ALB)
+ **Create ALB**  
  Create an ALB using the AWS Management Console.
+ **Configure Listeners & Target Groups**  
  Set up the listener rules and target groups to forward traffic to ECS tasks.
![Screenshot 2025-02-27 233857](https://github.com/user-attachments/assets/0372bd11-7c0d-401a-8781-a60ba3d386c8)

### Step 4: Deploy on Amazon ECS using Fargate
+ **Create Task Definition**  
  Define your ECS task using the image from ECR.
+ **Create ECS Cluster**  
  Set up an ECS cluster:
  ```sh
  aws ecs create-cluster --cluster-name my-cluster
  ```
  ![Screenshot 2025-02-27 234108](https://github.com/user-attachments/assets/04ff1719-5315-41c3-95b3-9f517995e375)

+ **Deploy Service with Fargate**  
  Create an ECS service with Fargate launch type and link it with the ALB:
  ```sh
  aws ecs create-service \
    --cluster my-cluster \
    --service-name my-service \
    --task-definition my-task \
    --launch-type FARGATE \
    --desired-count 1 \
    --network-configuration 'awsvpcConfiguration={subnets=[subnet-xxx],securityGroups=[sg-xxx],assignPublicIp="ENABLED"}'
  ```
+ **Validation**  
  Access the ALB DNS to verify the deployment.
Attached below is the image of the deployed website


  ![Screenshot 2025-02-27 234203](https://github.com/user-attachments/assets/d2ca460a-5b3a-4b51-912c-d46b895da545)
