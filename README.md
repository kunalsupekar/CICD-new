# Spring Boot Application with CI/CD Pipeline in AWS

## Overview
This project demonstrates how to set up a CI/CD pipeline in AWS for a Spring Boot application. The pipeline is designed to automate building, testing, and deploying your application using AWS services like CodeCommit, CodeBuild, CodeDeploy, and CodePipeline.

## Project Structure
- **src/**: Contains the Spring Boot application source code.
- **scripts/**: Contains deployment and lifecycle scripts used by CodeDeploy.
- **buildspec.yml**: Configuration file for CodeBuild.
- **appspec.yml**: Configuration file for CodeDeploy.

---

## Prerequisites
1. AWS account with sufficient permissions.
2. Installed:
   - AWS CLI
   - Git
   - Java (JDK 17 or compatible version)
   - Maven
3. Docker (optional, for containerized builds).
4. IAM roles and policies for AWS services:
   - CodePipelineRole
   - CodeBuildRole
   - CodeDeployRole

---

## AWS Services Used
- **CodeCommit**: Repository for source code.
- **CodeBuild**: To build and test the application.
- **CodeDeploy**: To deploy the application to EC2 or another deployment target.
- **CodePipeline**: Orchestrates the CI/CD process.

---

## Setup Instructions

### 1. Clone the Repository
```bash
git clone <your-codecommit-repo-url>
cd <repository-name>
```

### 2. Configure AWS CLI
Ensure AWS CLI is configured with credentials and the correct region:
```bash
aws configure
```

### 3. Build the Project Locally
Ensure the project builds without errors:
```bash
mvn clean install
```

### 4. Create the S3 Bucket for Artifacts
Create an S3 bucket to store build and deployment artifacts:
```bash
aws s3 mb s3://<your-artifact-bucket-name>
```

### 5. Configure CodePipeline
1. **Source**: Link the CodeCommit repository.
2. **Build**: Link CodeBuild and reference `buildspec.yml`.
3. **Deploy**: Link CodeDeploy and reference `appspec.yml`.

---

## Configuration Files

### `buildspec.yml`
This file defines the build steps for CodeBuild:
```yaml
version: 0.2
env:
  variables:
    SPRING_PROFILES_ACTIVE: prod
phases:
  install:
    commands:
      - echo Installing dependencies...
      - mvn clean install
  build:
    commands:
      - echo Building the project...
      - mvn package
artifacts:
  files:
    - target/*.jar
  discard-paths: yes
```

### `appspec.yml`
This file defines the deployment instructions for CodeDeploy:
```yaml
version: 0.0
os: linux
files:
  - source: /target
    destination: /home/ec2-user/app
hooks:
  ApplicationStop:
    - location: scripts/application_stop.sh
      timeout: 300
  BeforeInstall:
    - location: scripts/before_install.sh
      timeout: 300
  ApplicationStart:
    - location: scripts/application_start.sh
      timeout: 300
  ValidateService:
    - location: scripts/validate_service.sh
      timeout: 300
```

---

## Deployment Steps

1. **Push Code to Repository**:
   Commit and push your code to the CodeCommit repository.
   ```bash
   git add .
   git commit -m "Initial commit"
   git push
   ```

2. **Trigger the Pipeline**:
   Navigate to the AWS CodePipeline console and start the pipeline.

3. **Monitor the Deployment**:
   - Check build logs in CodeBuild.
   - Check deployment logs in CodeDeploy.

---

## Deployment Scripts

### `scripts/application_stop.sh`
Stops any running instance of the application.
```bash
#!/bin/bash
sudo systemctl stop spring-boot-app || true
```

### `scripts/before_install.sh`
Installs dependencies and prepares the environment.
```bash
#!/bin/bash
sudo yum update -y
sudo yum install -y java-17-amazon-corretto
```

### `scripts/application_start.sh`
Starts the Spring Boot application.
```bash
#!/bin/bash
sudo java -jar /home/ec2-user/app/<your-application-name>.jar &
```


---

## Notes
- Replace `<your-application-name>` and `<your-artifact-bucket-name>` with appropriate values.
- Ensure EC2 instances have the required IAM role for CodeDeploy.
- Use environment-specific configurations for better manageability.

---

## Troubleshooting
- **Pipeline Fails**: Check logs in CodePipeline, CodeBuild, and CodeDeploy.
- **Permission Errors**: Verify IAM roles and policies.
- **Application Fails**: Verify the logs at `/var/log/cloud-init-output.log` or application-specific logs.
