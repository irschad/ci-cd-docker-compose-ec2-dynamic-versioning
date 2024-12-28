# Deploy Application from Jenkins Pipeline on EC2 Instance (with Docker Compose)

## Project Overview
This project demonstrates continuous deployment of a web application using a Jenkins pipeline to an AWS EC2 instance. The deployment leverages Docker Compose for container orchestration and includes optimizations by utilizing a shell script for remote server commands.

### Technologies Used
- **AWS**: EC2 Instance for deployment
- **Jenkins**: Pipeline automation
- **Docker & Docker Compose**: Containerization and orchestration
- **Linux**: EC2 instance environment
- **Git**: Source code management
- **Java & Maven**: Application development and build tools
- **Docker Hub**: Hosting Docker images

---

## Project Description
The project involves the following steps:

1. **Install Docker Compose on EC2 Instance**:
   - Install Docker Compose on an AWS EC2 instance to orchestrate containerized services.

2. **Create `docker-compose.yaml`**:
   - Define a `docker-compose.yaml` file to deploy a Java web application and a PostgreSQL database.

3. **Configure Jenkins Pipeline**:
   - Create a Jenkins pipeline to build and push the Docker image to Docker Hub and deploy the application using Docker Compose on the EC2 server.

4. **Optimizations**:
   - Extract Linux commands for remote execution into a separate shell script and execute the script from the Jenkinsfile.

---

## Setup Instructions

### Step 1: Install Docker Compose on EC2 Instance
1. SSH into the EC2 instance:
   ```bash
   ssh -i ~/.ssh/aws-ec2.pem ec2-user@<EC2-PUBLIC-IP>
   ```
2. Install Docker Compose:
   ```bash
   sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose
   docker-compose --version
   ```

### Step 2: Create `docker-compose.yaml`
Create a `docker-compose.yaml` file with the following content:
```yaml
docker-compose.yaml
version: '3.8'
services:
  java-maven-app:
    image: irschad/java-app:4.0
    ports:
      - 8000:8080
  postgres:
    image: postgres:15
    ports:
      - 5432:5432
    environment:
      - POSTGRES_PASSWORD=my-pwd
```

### Step 3: Configure Jenkins Pipeline
#### Initial Jenkinsfile:
```groovy
pipeline {   
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        IMAGE_NAME = 'irschad/java-app:4.0'
    }
    
    stages {
        stage('Build App') {
            steps {
                echo 'Building application jar...'
                buildJar()
            }
        }
        stage('Build Image') {
            steps {
                script {
                    echo 'Building the Docker image...'
                    buildImage(env.IMAGE_NAME)
                    dockerLogin()
                    dockerPush(env.IMAGE_NAME)
                }
            }
        } 
        stage('Deploy') {
            steps {
                script {
                    echo 'Deploying Docker image to EC2...'
                    def dockerComposeCmd = "docker-compose -f docker-compose.yaml up --detach"
                    sshagent(['aws-ec2-server-key']) {
                        sh "scp docker-compose.yaml ec2-user@<EC2-PUBLIC-IP>:/home/ec2-user"
                        sh "ssh ec2-user@<EC2-PUBLIC-IP> ${dockerComposeCmd}"
                    }
                }
            }               
        }
    }
}
```

#### Optimized Jenkinsfile (Using Shell Script):
```groovy
pipeline {   
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        IMAGE_NAME = 'irschad/java-app:4.0'
    }
    
    stages {
        stage('Build App') {
            steps {
                echo 'Building application jar...'
                buildJar()
            }
        }
        stage('Build Image') {
            steps {
                script {
                    echo 'Building the Docker image...'
                    buildImage(env.IMAGE_NAME)
                    dockerLogin()
                    dockerPush(env.IMAGE_NAME)
                }
            }
        } 
        stage('Deploy') {
            steps {
                script {
                    echo 'Deploying Docker image to EC2...'
                    def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}"
                    def ec2Instance = "ec2-user@<EC2-PUBLIC-IP>"
                    sshagent(['aws-ec2-server-key']) {
                        sh "scp server-cmds.sh ${ec2Instance}"
                        sh "scp docker-compose.yaml ${ec2Instance}:/home/ec2-user" 
                        sh "ssh ${ec2Instance} ${shellCmd}"
                    }
                }
            }               
        }
    }
}
```

### Step 4: Shell Script for Deployment
Create a `server-cmds.sh` script:
```bash
#!/usr/bin/env bash

export IMAGE=$1
docker-compose -f docker-compose.yaml up --detach
echo "Deployment successful!"
```

---

## Key Features
- Automates the deployment process for containerized applications using Jenkins pipelines.
- Leverages Docker Compose to simplify multi-container application deployments.
- Optimizes remote server commands by consolidating them into a reusable shell script.

---

## Benefits
- **Scalability**: Easily deploy additional services by updating the `docker-compose.yaml` file.
- **Reusability**: Shell script and pipeline can be reused for similar deployment scenarios.
- **Automation**: End-to-end deployment is triggered by Jenkins, reducing manual intervention.

---

