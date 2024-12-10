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


## 2. Configure Jenkins Pipeline

### Steps to Set Up the Jenkins Pipeline
1. **Add the `Jenkinsfile`** to your Git repository.
2. **Create a new pipeline job** in Jenkins:
   - Navigate to Jenkins Dashboard > `New Item`.
   - Choose `Pipeline` and name it appropriately.
   - Configure the job by linking it to the repository where the `Jenkinsfile` is located.

### Set Up Jenkins Credentials
Ensure the following Jenkins credentials are configured:
- **Docker Hub**: Create a credential with the ID `docker-hub-credentials` for Docker image pushes.
- **GitHub**: Create a credential with the ID `github` for source code access and Git operations.
- **SonarQube**: Set up a credential with the ID `sonar-scanner` for SonarQube analysis.
- **Git Username/Password**: Add credentials for pushing changes to the repository if needed.

## 3. Build and Deploy Pipeline

### Running the Jenkins Pipeline
1. **Commit and Push the Code**:
   - Push your code to the repository, including the `Jenkinsfile` and deployment configuration files.
2. **Run the Jenkins Pipeline**:
   - Trigger the pipeline manually or set up automated triggers (e.g., webhook for GitHub).
   - The pipeline will proceed through the following stages:
     - **Code Checkout**: Pulls the latest code from the repository.
     - **Installation and Testing**: Runs `npm install` and `npm test` to ensure the application is working correctly.
     - **Build and Push Docker Image**: Builds the Docker image using the `Dockerfile` and pushes it to Docker Hub.
     - **Image Scanning**: Uses `Trivy` to scan the Docker image for vulnerabilities.
     - **Deployment Manifest Update and Commit**: Updates the `deployment.yaml` file with the new image tag and commits it back to the repository.
     - **Deployment to Kubernetes**: Applies the updated deployment using `kubectl` to deploy the changes to the Kubernetes cluster.

## 4. Verify Deployment
To check the status of your deployment, run the following command in your terminal:
```bash
kubectl get deployments
