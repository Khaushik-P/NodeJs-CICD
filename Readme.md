# Node.js CI/CD Pipeline for Kubernetes Deployment

## Overview
This repository demonstrates how to set up a continuous integration and deployment (CI/CD) pipeline for a Node.js application using Jenkins, ArgoCD, and Kubernetes. The pipeline automates the entire process of code build, test, security scan, Docker image creation, and deployment to a Kubernetes cluster.

## Project Structure
The repository contains the following key files:


### 1. ArgoCD Application Manifest (`deployment.yaml`)
- **Purpose**: Deploys the Node.js application on Kubernetes and integrates with ArgoCD for continuous delivery.
- **Configuration**:
  - **repoURL**: Points to the Git repository containing the `Jenkinsfile` and deployment manifests.
  - **path**: Specifies the path to the deployment manifest within the repository.
  - **syncPolicy**: Configures automated deployment with `prune` and `selfHeal` to keep the environment consistent.

### 2. Kubernetes Deployment Manifest (`deployment.yaml`)
- **Purpose**: Defines how the Node.js application is deployed in the Kubernetes cluster.
- **Configuration**:
  - **replicas**: Number of instances of the application running.
  - **containerPort**: Exposes port 3000 for the Node.js application.
  - **image**: Docker image for the application (tagged with `latest` or a version).

### 3. Jenkins Pipeline (`Jenkinsfile`)
- **Purpose**: Automates the CI/CD process including building, testing, scanning, pushing Docker images, and deploying to Kubernetes.
- **Stages**:
  - **Clean Workspace**: Cleans up the workspace if it’s not a pull request build.
  - **Checkout Code**: Clones the source code from the GitHub repository.
  - **Install Dependencies**: Runs `npm install` to install Node.js dependencies.
  - **Run Tests**: Executes `npm test` to validate the code.
  - **SonarQube Analysis**: Analyzes code quality using SonarQube.
  - **Quality Gate**: Waits for SonarQube's feedback to ensure the code passes quality checks.
  - **Build and Push to Docker Hub**: Builds the Docker image and pushes it to Docker Hub.
  - **Trivy Image Scan**: Scans the Docker image for vulnerabilities.
  - **Update Deployment Tags**: Updates the `deployment.yaml` with the new Docker image tag.
  - **Push Changed Deployment File to Git**: Commits the updated `deployment.yaml` back to the repository.
  - **Deploy to Kubernetes**: Applies the updated deployment manifest using `kubectl`.

### 4. Docker Configuration (`Dockerfile`)
- **Purpose**: Defines how to build a Docker image for the Node.js application.
- **Instructions**:
  - **Base Image**: Uses `node:16`.
  - **Working Directory**: Sets the working directory to `/usr/src/app`.
  - **Copy Package Files**: Copies `package.json` and `package-lock.json`.
  - **Install Dependencies**: Runs `npm install` to set up the app.
  - **Copy Application Code**: Copies the application code into the container.
  - **Expose Port**: Exposes port 3000 for the app.
  - **Run Command**: Specifies `node app.js` as the container's entry point.

## Setup Instructions

### Prerequisites
- **Jenkins**: Installed with necessary plugins for pipeline execution, GitHub integration, Docker, and Slack notifications.
- **Kubernetes Cluster**: Set up and accessible with `kubectl` configured.
- **ArgoCD**: Installed and configured on the Kubernetes cluster for managing the deployment.
- **Docker**: Installed and configured to build and push images.
- **SonarQube**: Set up for code analysis and connected to Jenkins.
- **Slack**: Configured for sending build notifications.

### Step-by-Step Setup

#### 1. Deploy ArgoCD Application
- Ensure ArgoCD is running on your Kubernetes cluster.
- Apply the ArgoCD `Application` manifest:
  ```bash
  kubectl apply -f deployment.yaml


2. Configure Jenkins Pipeline
Add the Jenkinsfile to your Jenkins job.
Create and configure the pipeline job in Jenkins.
Set up the following Jenkins credentials:
Docker Hub: docker-hub-credentials
GitHub: github
SonarQube: sonar-scanner
Git Username/Password for pushing changes.
3. Build and Deploy Pipeline
Commit and Push the code to the repository.
Run the Jenkins Pipeline. This triggers the following stages:
Code checkout
Installation and testing
Build and push Docker image
Image scanning
Deployment manifest update and commit
Deployment to Kubernetes
4. Verify Deployment
Use the following command to check the status of the deployment:
bash
Copy code
kubectl get deployments
Notifications
The pipeline sends notifications to a configured Slack channel using the slackSend function based on build status, providing real-time updates.

Security Considerations
Image Scanning: The Trivy stage scans the Docker image for vulnerabilities before deployment.
Secrets Management: Ensure credentials used in Jenkins are stored securely.
Requirements
Jenkins Plugins: Docker, GitHub, Slack, SonarQube.
Kubernetes Cluster: Access configured with kubectl.
ArgoCD: Deployed and connected to Kubernetes.
Docker Hub: Account for storing images.
License
This project is licensed under the MIT License. See the LICENSE file for details.

Notes
Update the placeholders like github-username and github-email in the Jenkinsfile with actual values.
Ensure your Docker Hub repository exists and you have the required permissions.
vbnet
Copy code

This `README.md` explains the components, setup steps, and detailed instructions to deploy your N