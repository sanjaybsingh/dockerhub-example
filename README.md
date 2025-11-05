# DockerHub Example - Jenkins Pipeline with Shell Scripts

This repository demonstrates how to build a Docker image and push it to DockerHub using Jenkins Pipeline with modular shell scripts. This approach separates the build logic into reusable shell scripts, making the pipeline cleaner and easier to maintain.

## Overview

This project contains a Jenkins pipeline that:
1. Builds a Docker image based on Alpine Linux 3.13.5 using a shell script
2. Logs into DockerHub using stored credentials via a shell script
3. Pushes the built image to DockerHub using a shell script
4. Logs out of DockerHub using a shell script

## Branch Information

**Current Branch**: `02-shell-scripts`

This branch demonstrates a modular approach where each pipeline stage executes a separate shell script from the `jenkins/` directory.

## Prerequisites

- Jenkins server with:
  - Docker installed and accessible to Jenkins agents
  - A Linux agent with the label `linux`
  - DockerHub credentials stored in Jenkins with ID: `darinpope-dockerhub`
- DockerHub account

## Project Structure

```
.
├── jenkins/
│   ├── build.sh      # Docker build script
│   ├── login.sh      # DockerHub login script
│   ├── push.sh       # Docker push script
│   └── logout.sh     # DockerHub logout script
├── Dockerfile        # Simple Dockerfile based on Alpine Linux
├── Jenkinsfile       # Jenkins Pipeline definition
└── README.md         # This file
```

## Shell Scripts

### build.sh
Builds the Docker image and tags it:
```bash
#!/bin/bash
docker build -t darinpope/dp-alpine:latest .
```

### login.sh
Authenticates with DockerHub using Jenkins credentials:
```bash
#!/bin/bash
echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
```

### push.sh
Pushes the Docker image to DockerHub:
```bash
#!/bin/bash
docker push darinpope/dp-alpine:latest
```

### logout.sh
Logs out from DockerHub:
```bash
#!/bin/bash
docker logout
```

## Setup Instructions

### 1. Configure DockerHub Credentials in Jenkins

1. Navigate to Jenkins → Manage Jenkins → Manage Credentials
2. Add new credentials:
   - **Kind**: Username with password
   - **Username**: Your DockerHub username
   - **Password**: Your DockerHub password or access token
   - **ID**: `darinpope-dockerhub` (or update the Jenkinsfile with your credential ID)

### 2. Update the Shell Scripts

Modify the DockerHub repository name in the shell scripts to match your account:

**jenkins/build.sh:**
```bash
docker build -t YOUR_USERNAME/dp-alpine:latest .
```

**jenkins/push.sh:**
```bash
docker push YOUR_USERNAME/dp-alpine:latest
```

### 3. Ensure Scripts are Executable

Make sure the shell scripts have execute permissions:
```bash
chmod +x jenkins/*.sh
```

### 4. Create Jenkins Pipeline Job

1. In Jenkins, create a new Pipeline job
2. Under Pipeline configuration, select:
   - **Definition**: Pipeline script from SCM
   - **SCM**: Git
   - **Repository URL**: `https://github.com/sanjaybsingh/dockerhub-example`
   - **Branch**: `02-shell-scripts`
3. Save the configuration

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

### 2. Install Docker CLI in Jenkins Container

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

**Option A - Use the master node:**
1. Go to Jenkins → Manage Jenkins → Nodes → Built-In Node → Configure
2. Set **Number of executors** to at least 1
3. Add label: `linux`
4. Save

**Option B - Modify the Jenkinsfile:**
Change `agent { label 'linux' }` to `agent any` in the Jenkinsfile.

### 4. Get Initial Admin Password

```powershell
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

## Pipeline Stages

### Build Stage
Executes `./jenkins/build.sh` to build the Docker image and tag it as `darinpope/dp-alpine:latest`.

### Login Stage
Executes `./jenkins/login.sh` to authenticate with DockerHub using credentials stored in Jenkins. The credentials are accessed via environment variables:
- `DOCKERHUB_CREDENTIALS_USR` - DockerHub username
- `DOCKERHUB_CREDENTIALS_PSW` - DockerHub password

### Push Stage
Executes `./jenkins/push.sh` to push the built Docker image to DockerHub.

### Post Actions
Always executes `./jenkins/logout.sh` to log out of DockerHub after the pipeline completes, regardless of success or failure.

## Advantages of Using Shell Scripts

1. **Modularity**: Each stage's logic is in a separate file, making it easier to test and maintain
2. **Reusability**: Scripts can be reused in other pipelines or run locally for testing
3. **Cleaner Jenkinsfile**: The pipeline definition focuses on orchestration rather than implementation details
4. **Version Control**: Scripts can be tracked and reviewed separately
5. **Local Testing**: Scripts can be tested locally before committing to the repository

## Usage

1. Push changes to your repository
2. Manually trigger the Jenkins job or configure a webhook for automatic builds
3. Monitor the build progress in Jenkins
4. Verify the image on DockerHub after successful completion

## Environment Variables

The pipeline uses the following environment variables:
- `DOCKERHUB_CREDENTIALS`: Jenkins credentials binding for DockerHub authentication
  - `DOCKERHUB_CREDENTIALS_USR`: DockerHub username
  - `DOCKERHUB_CREDENTIALS_PSW`: DockerHub password

## Features

- **Modular Design**: Separates build logic into individual shell scripts
- **No Additional Plugins Required**: Uses only core Jenkins functionality with Docker and credentials binding
- **Secure Credential Handling**: Credentials are never exposed in logs
- **Automatic Cleanup**: Always logs out of DockerHub after completion
- **Build History**: Keeps the last 5 builds for reference
- **Easy Testing**: Shell scripts can be tested independently

## Testing Locally

You can test the shell scripts locally before running them in Jenkins:

```bash
# Test the build script
./jenkins/build.sh

# Test login (requires DOCKERHUB_CREDENTIALS_USR and DOCKERHUB_CREDENTIALS_PSW environment variables)
export DOCKERHUB_CREDENTIALS_USR="your-username"
export DOCKERHUB_CREDENTIALS_PSW="your-password"
./jenkins/login.sh

# Test push
./jenkins/push.sh

# Test logout
./jenkins/logout.sh
```

## Troubleshooting

### Script Permission Denied
Ensure scripts are executable:
```bash
chmod +x jenkins/*.sh
```

### Docker Socket Not Accessible (Windows/Docker Desktop)
If you get "Cannot connect to the Docker daemon" errors:
1. Ensure Docker Desktop is running
2. Mount the Docker socket when starting Jenkins (included in the docker run command above)

### Permission Denied (Docker Desktop)
Fix permissions inside the Jenkins container:
```powershell
docker exec -it -u root jenkins chmod 666 /var/run/docker.sock
```

### Jenkins Cannot Find 'linux' Agent
Either:
- Configure the built-in node with the `linux` label
- OR change `agent { label 'linux' }` to `agent any` in the Jenkinsfile

### Docker Command Not Found
Ensure Docker CLI is installed in the Jenkins container (see setup instructions).

### Authentication Failed
Verify that:
- The credential ID in the Jenkinsfile matches the one configured in Jenkins
- Your DockerHub credentials are correct
- You're using an access token if 2FA is enabled on DockerHub

## Comparing with Other Branches

- **01-no-plugins-necessary**: Uses inline shell commands in the Jenkinsfile
- **02-shell-scripts** (current): Uses modular shell scripts for better organization

## License

This is an example project for demonstration purposes.

## Author

sanjaybsingh / darinpope