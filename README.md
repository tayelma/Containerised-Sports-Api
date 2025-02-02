# Sports API Containerised
This project provides a containerised infrastructure for running the **Sports API** using **FastAPI** as the backend framework and leveraging AWS services such as ECR, ECS, and API Gateway for hosting and managing the API.
## Project Overview
The **Sports API** project offers functionality backed by FastAPI and a PostgreSQL database. The focus of this folder is to containerise the application using Docker and deploy it to AWS infrastructure using ECS with Fargate, managed via an Application Load Balancer (ALB), and expose it through an API Gateway for external access.
## Prerequisites
Before starting, ensure you have the following tools installed:
- **Docker**: To build and manage container images
- **AWS CLI**: For managing AWS services via the command line
- **Python 3.8+**: Used to define services within the codebase
- **PostgreSQL**: Database dependency for the application
- **Sports API Key**: Ensure you have an API key from [Sports Data IO]()

## Setup Instructions
### 1. Docker & Local Setup
1. **Navigate to the Project Directory**:
``` sh
   cd sports-api-containerised
```
1. **Build Docker Image**:
``` sh
   docker build --platform linux/amd64 -t sports-api .
```
This will create a Docker image for the Sports API project that is compatible with AWS ECS Fargate.
1. **Test Locally**:
    - After building the image, you can run it locally with:
``` sh
     docker run -p 8080:8080 sports-api
```
1. Test the API by visiting:
``` 
   http://localhost:8080/sports
```
### 2. AWS Deployment
#### **Step 1: Create and Push Docker Image with ECR (Elastic Container Registry)**
AWS Elastic Container Registry (ECR) is a fully managed Docker container registry enabling you to securely store container images.
Steps to create and publish the Docker image in AWS ECR:
1. **Create an ECR Repository**:
``` sh
   aws ecr create-repository --repository-name sports-api --region us-east-1
```
1. **Authenticate Docker CLI with ECR**:
``` sh
   aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com
```
1. **Tag and Push the Image to ECR**:
    - Tag the image:
``` sh
     docker tag sports-api:latest <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/sports-api:sports-api-latest
```
- Push to ECR:
``` sh
     docker push <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/sports-api:sports-api-latest
```
#### **Step 2: Set Up an ECS (Elastic Container Service) Cluster with Fargate**
AWS ECS manages containers and services on the cloud. Here, we're using AWS Fargate, a serverless compute engine, to eliminate the need for provisioning and managing servers.
**Steps to Configure ECS:**
1. **Create a Fargate ECS Cluster**:
    - Go to the AWS ECS Console.
    - Create a new cluster and choose **Fargate** as the infrastructure type.
    - Name the cluster **sports-api-cluster**.

2. **Define a Task**:
    - Go to **Task Definitions** → Create New Task Definition.
    - Name the task **sports-api-task**.
    - Select **Fargate** as the infrastructure option.
    - Add a container:
        - **Container Name**: sports-api-container
        - **Image URI**: `<AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/sports-api:sports-api-latest`
        - **Port**: 8080

    - Add an environment variable to your container:
        - **Key**: `SPORTS_API_KEY`
        - **Value**: `<YOUR_SPORTSDATA.IO_API_KEY>`

3. **Create the Service**:
    - Go to the created cluster → Services → Create Service → Select **Fargate**.
    - Fill in the name as **sports-api-service**.
    - Set **Desired tasks** to 2 for high availability.
    - Set networking configurations and attach a **new security group**:
        - Open all TCP ports for testing.

#### **Step 3: Connect ECS with ALB (Application Load Balancer)**
AWS ALB distributes incoming traffic to your service tasks and ensures high availability.
1. Configure ALB while setting up the service:
    - Create a new ALB.
        - **Name**: sports-api-alb.
        - **Target Group Health Check Path**: `/sports`

    - Register the service with the ALB.

2. After deployment, fetch the ALB DNS name (e.g., sports-api-alb-<AWS_ACCOUNT_ID>.us-east-1.elb.amazonaws.com), and access the API via:
``` 
   http://sports-api-alb-<AWS_ACCOUNT_ID>.us-east-1.elb.amazonaws.com/sports
```
#### **Step 4: API Gateway Configuration**
AWS API Gateway allows you to expose your API securely with an HTTP endpoint.
1. **Create API Gateway**:
    - Go to API Gateway → **Create REST API** → Provide a name such as **Sports API Gateway**.

2. **Create Resource and Method**:
    - Add a **GET method** to the resource `/sports`.
    - Select **HTTP Proxy** integration and enter the ALB DNS (e.g., [http://sports-api-alb]()-<AWS_ACCOUNT_ID>.us-east-1.elb.amazonaws.com/sports).

3. **Deploy the API**:
    - Deploy the API and note the deployment URL.

4. Test the endpoint URL to verify the system.

## Summary of AWS Services
### **Amazon ECR**:
- Fully managed Docker container registry for storing and deploying images.
- Why? Ensures security and seamless integration with ECS.

### **Amazon ECS**:
- Manages container workloads.
- Why? It abstracts server management when combined with Fargate.

### **AWS Fargate**:
- Serverless compute engine for running containers.
- Why? No manual provisioning of servers is needed, reducing operational overhead.

### **Amazon ALB**:
- Balances incoming traffic across ECS tasks.
- Why? Provides health checks and increases availability.

### **API Gateway**:
- Exposes the ECS service to external clients securely and efficiently.
- Why? Publicly exposes the API for consumption.

## Testing & Validation
- Access the API through API Gateway endpoint or directly via ALB DNS.
- Testing involves making a GET request to `/sports` to retrieve API data.
- URL in your browser should be **yourALBDNS/sports**

This README provides a complete guide to containerising the **Sports API**, setting up AWS infrastructure, and exposing the API. Make sure to replace placeholders such as `<AWS_ACCOUNT_ID>` or `<YOUR_SPORTSDATA.IO_API_KEY>` while setting up.
