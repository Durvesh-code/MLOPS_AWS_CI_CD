# Student Performance Predictor with AWS MLOps Pipeline

This repository contains a complete Machine Learning Operations (MLOps) project demonstrating an automated Continuous Integration and Continuous Deployment (CI/CD) pipeline. The pipeline leverages GitHub Actions alongside Amazon Web Services (AWS) to automatically build, push, and deploy a containerized machine learning application.

## Architecture Overview

The deployment workflow integrates the following core components:

*   **GitHub Actions**: Serves as the CI/CD orchestrator. It listens for code changes, executes build steps, and triggers deployments.
*   **AWS IAM (Identity and Access Management)**: Provides secure, programmatic access for GitHub Actions to interact with AWS resources.
*   **AWS ECR (Elastic Container Registry)**: Acts as the secure repository for storing Docker images built during the continuous integration phase.
*   **AWS EC2 (Elastic Compute Cloud)**: Functions as the production server. It operates a self-hosted GitHub Actions runner to securely pull and execute the latest Docker containers.

## AWS Infrastructure Setup

To replicate this workflow, the following AWS resources must be configured:

### 1. AWS IAM Setup
1. Create a dedicated IAM User for GitHub Actions.
2. Attach the `AmazonEC2ContainerRegistryFullAccess` policy to allow the user to push and pull images.
3. Generate an Access Key ID and Secret Access Key.

### 2. AWS ECR Setup
1. Navigate to the Elastic Container Registry in the AWS Console.
2. Create a new private repository to store the project's Docker images.

### 3. GitHub Secrets Configuration
In the GitHub repository settings (Settings > Secrets and variables > Actions), add the following repository secrets:
*   `AWS_ACCESS_KEY_ID`
*   `AWS_SECRET_ACCESS_KEY`
*   `AWS_REGION`
*   `ECR_REPOSITORY_NAME`

### 4. AWS EC2 Setup (Self-Hosted Runner)
1. Launch an Ubuntu EC2 instance. Ensure the Security Group allows inbound traffic on port 8080.
2. Connect to the instance via SSH and install Docker.
3. Add the `ubuntu` user to the `docker` group to allow execution without `sudo`.
4. Navigate to the GitHub repository settings (Settings > Actions > Runners) and add a new Linux self-hosted runner.
5. Execute the provided download and configuration scripts on the EC2 instance.
6. Install and start the runner as a background system service (`sudo ./svc.sh install` and `sudo ./svc.sh start`).

## CI/CD Pipeline Workflow

The workflow is defined in `.github/workflows/main.yaml` and executes the following sequence upon pushing to the main branch:

1.  **Build and Push to ECR**:
    *   Checks out the latest source code.
    *   Authenticates with AWS using the IAM credentials stored in GitHub Secrets.
    *   Logs into Amazon ECR.
    *   Builds the Docker image using the provided `Dockerfile`.
    *   Tags and pushes the image to the ECR repository.

2.  **Continuous Deployment (EC2 Runner)**:
    *   Executes directly on the EC2 self-hosted runner.
    *   Cleans up unused Docker resources and dangling images to preserve disk space.
    *   Authenticates with AWS and pulls the latest Docker image from ECR.
    *   Gracefully stops and removes any previously running container.
    *   Starts the new Docker container, exposing the Flask application on port 8080.

## Accessing the Application

Once the deployment pipeline completes successfully, the machine learning application is accessible via a web browser at:
`http://<EC2-PUBLIC-IP>:8080/`
