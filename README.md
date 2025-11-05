# DockerHub Example - Jenkins Pipeline

This repository demonstrates how to build a Docker image and push it to DockerHub using Jenkins Pipeline without requiring additional plugins beyond the core Jenkins Docker and credentials functionality.

## Overview

This project contains a simple Jenkins pipeline that:
1. Builds a Docker image based on Alpine Linux 3.13.5
2. Logs into DockerHub using stored credentials
3. Pushes the built image to DockerHub
4. Logs out of DockerHub

## Prerequisites

- Jenkins server with:
  - Docker installed and accessible to Jenkins agents
  - A Linux agent with the label `linux`
  - DockerHub credentials stored in Jenkins with ID: `sanjaybsingh-dockerhub`
- DockerHub account

## Running on Jenkins with Docker Desktop (Windows)

If you're running Jenkins on Docker Desktop for Windows, follow these additional steps:

### 1. Start Jenkins in Docker

Run Jenkins as a Docker container with access to Docker Desktop:

```powershell
docker run -d `
  --name jenkins `
  -p 8080:8080 -p 50000:50000 `
  -v jenkins_home:/var/jenkins_home `
  -v /var/run/docker.sock:/var/run/docker.sock `
  jenkins/jenkins:lts
```

**Note**: On Windows with Docker Desktop, you may need to enable the Docker socket in Docker Desktop settings.

### 2. Install Docker CLI in Jenkins Container

Jenkins needs Docker CLI to build and push images:

```powershell
# Access the Jenkins container
docker exec -it -u root jenkins bash

# Inside the container, install Docker CLI
apt-get update
apt-get install -y docker.io

# Give Jenkins user permission to use Docker
chmod 666 /var/run/docker.sock

# Exit the container
exit
```

### 3. Configure Jenkins Agent

Since the Jenkinsfile requires a `linux` agent label:

**Option A - Use the master node:**
1. Go to Jenkins → Manage Jenkins → Nodes → Built-In Node → Configure
2. Set **Number of executors** to at least 1
3. Add label: `linux`
4. Save

**Option B - Run Jenkins without agent label:**
Modify the Jenkinsfile to use `any` instead of `linux`:
```groovy
agent any
```

### 4. Get Initial Admin Password

```powershell
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### 5. Complete Jenkins Setup

1. Open browser to `http://localhost:8080`
2. Enter the initial admin password
3. Install suggested plugins
4. Create your first admin user
5. Start using Jenkins

### 6. Verify Docker Access

Create a test pipeline to verify Docker is accessible:

```groovy
pipeline {
    agent any
    stages {
        stage('Test Docker') {
            steps {
                sh 'docker --version'
                sh 'docker ps'
            }
        }
    }
}
```

## Project Structure

```
.
├── Dockerfile        # Simple Dockerfile based on Alpine Linux
├── Jenkinsfile       # Jenkins Pipeline definition
└── README.md         # This file
```

## Setup Instructions

### 1. Configure DockerHub Credentials in Jenkins

1. Navigate to Jenkins → Manage Jenkins → Manage Credentials
2. Add new credentials:
   - **Kind**: Username with password
   - **Username**: Your DockerHub username
   - **Password**: Your DockerHub password or access token
   - **ID**: `sanjaybsingh-dockerhub` (or update the Jenkinsfile with your credential ID)

### 2. Update the Jenkinsfile

Modify the DockerHub repository name in the Jenkinsfile to match your account:

```groovy
// Change 'sanjaybsingh' to your DockerHub username
sh 'docker build -t YOUR_USERNAME/dp-alpine:latest .'
sh 'docker push YOUR_USERNAME/dp-alpine:latest'
```

### 3. Create Jenkins Pipeline Job

1. In Jenkins, create a new Pipeline job
2. Under Pipeline configuration, select:
   - **Definition**: Pipeline script from SCM
   - **SCM**: Git
   - **Repository URL**: Your repository URL
   - **Branch**: `01-no-plugins-necessary` (or your target branch)
3. Save the configuration

## Pipeline Stages

### Build Stage
Builds the Docker image using the provided Dockerfile and tags it as `sanjaybsingh/dp-alpine:latest`.

### Login Stage
Authenticates with DockerHub using credentials stored in Jenkins. The credentials are accessed via environment variables:
- `DOCKERHUB_CREDENTIALS_USR` - DockerHub username
- `DOCKERHUB_CREDENTIALS_PSW` - DockerHub password

### Push Stage
Pushes the built Docker image to DockerHub.

### Post Actions
Always logs out of DockerHub after the pipeline completes, regardless of success or failure.

## Usage

1. Push changes to your repository
2. Manually trigger the Jenkins job or configure a webhook for automatic builds
3. Monitor the build progress in Jenkins
4. Verify the image on DockerHub after successful completion

## Environment Variables

The pipeline uses the following environment variables:
- `DOCKERHUB_CREDENTIALS`: Jenkins credentials binding for DockerHub authentication

## Features

- **No Additional Plugins Required**: Uses only core Jenkins functionality with Docker and credentials binding
- **Secure Credential Handling**: Credentials are never exposed in logs
- **Automatic Cleanup**: Always logs out of DockerHub after completion
- **Build History**: Keeps the last 5 builds for reference

## Troubleshooting

### Docker Socket Not Accessible (Windows/Docker Desktop)
If you get "Cannot connect to the Docker daemon" errors:
1. Ensure Docker Desktop is running
2. In Docker Desktop, go to Settings → Advanced → Enable "Expose daemon on tcp://localhost:2375 without TLS"
3. Or mount the Docker socket when starting Jenkins (already included in the docker run command above)

### Permission Denied (Docker Desktop)
If Jenkins cannot access Docker:
```powershell
# Fix permissions inside the Jenkins container
docker exec -it -u root jenkins chmod 666 /var/run/docker.sock
```

### Jenkins Cannot Find 'linux' Agent
Either:
- Configure the built-in node with the `linux` label (see setup instructions above)
- OR change `agent { label 'linux' }` to `agent any` in the Jenkinsfile

### Docker Command Not Found
Ensure Docker is installed on the Jenkins agent and the Jenkins user has permission to run Docker commands.

If running Jenkins in Docker, make sure you installed Docker CLI inside the container (see setup instructions).

### Authentication Failed
Verify that:
- The credential ID in the Jenkinsfile matches the one configured in Jenkins
- Your DockerHub credentials are correct
- You're using an access token if 2FA is enabled on DockerHub

### Permission Denied
Add the Jenkins user to the docker group:
```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

## License

This is an example project for demonstration purposes.

## Author

sanjaybsingh