# Complete the CI/CD Pipeline with Docker-Compose and Dynamic Versioning

## Overview
This project demonstrates a CI/CD pipeline using Jenkins, Docker, and Docker Compose to build, version, deploy, and manage a Java Maven application. The pipeline incorporates dynamic versioning, ensuring that each build and deployment is uniquely identified and seamlessly managed.

## Key Features
- **Continuous Integration (CI):**
  - Increment application version dynamically in `pom.xml`.
  - Build Java Maven application with the new version.
  - Build and push Docker images to Docker Hub.
  
- **Continuous Deployment (CD):**
  - Deploy the updated application version to an AWS EC2 instance using Docker Compose.
  - Commit the version update back to the Git repository.

## Technologies Used
- AWS
- Jenkins
- Docker & Docker Hub
- Linux
- Git
- Java
- Maven

---

## Pipeline Workflow

### Steps Followed in the Pipeline:
1. **Increment Version in `pom.xml`:**  
   Dynamically updates the version in the `pom.xml` file based on the previous version and increments the patch number.

2. **Build Application with the New Version:**  
   Packages the Java Maven application with the updated version for deployment.

3. **Build and Push Docker Image:**  
   Builds a new Docker image with the updated application and pushes it to Docker Hub.

4. **Deploy New Version to EC2 Using Docker Compose:**  
   Deploys the updated application version on an EC2 instance using Docker Compose.

5. **Commit Version Update:**  
   Commits the version bump back to the Git repository, ensuring the updated `pom.xml` version is tracked.

---

## Jenkins Pipeline Stages

### Increment Version
This stage dynamically increments the version in `pom.xml` using Maven and sets the image name with the updated version.  
**Jenkinsfile Snippet:**
```groovy
stage('increment version') {
    steps {
        script {
            echo "Incrementing app version"
            sh 'mvn build-helper:parse-version versions:set \
               -DnewVersion=\\${parsedVersion.majorVersion}.\\${parsedVersion.minorVersion}.\\${parsedVersion.nextIncrementalVersion} \
               versions:commit'
            def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
            def version = matcher[0][1]
            env.IMAGE_NAME = "$version-$BUILD_NUMBER"
        }
    }
}
```

### Commit Version Update
This stage commits the updated `pom.xml` back to the Git repository without triggering an infinite pipeline loop.  
**Jenkinsfile Snippet:**
```groovy
stage('commit version update') {
    steps {
        script {
            withCredentials([usernamePassword(credentialsId: 'jenkinspush', passwordVariable: 'PAT' , usernameVariable: 'USER')]) {
                sh "git remote set-url origin https://${PAT}@github.com/irschad/ci-cd-docker-compose-ec2-dynamic-versioning.git"
                sh 'git config --global user.email "jenkins@example.com"'
                sh 'git config --global user.name "jenkins"'                  
                sh 'git add .'
                sh "git commit -m 'ci: version bump'"
                sh 'git push origin HEAD:master'
            }
        }
    }
}
```

---

## Deployment Notes
- **Docker Compose Deployment:**  
  The updated Docker image is deployed to an EC2 instance using a `docker-compose.yml` file. This ensures the environment is consistent across deployments.

- **Git Commit Strategy:**  
  To avoid triggering the pipeline by the commit itself, the email `jenkins@example.com` is used in the Git configuration.

---

## Prerequisites
- AWS EC2 instance configured for Docker and Docker Compose.
- Jenkins pipeline configured with:
  - Maven
  - Docker
  - Git credentials (`jenkinspush` for GitHub access)
- Docker Hub account for pushing the Docker images.

---

## How to Use
1. Clone the repository:
   ```bash
   git clone https://github.com/irschad/ci-cd-docker-compose-ec2-dynamic-versioning.git
   ```

2. Set up Jenkins pipeline using the `Jenkinsfile` provided in the repository.

3. Ensure credentials (`jenkinspush`) are configured in Jenkins for GitHub access.

4. Run the pipeline to build, push, and deploy the application automatically.

---


