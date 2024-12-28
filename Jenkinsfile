@Library('jenkins-shared-library') _

pipeline {   
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        IMAGE_NAME = 'irschad/java-app:4.0'
    }
    
    stages {
        stage('increment version') {
            steps {
                script {
                    echo "Incrementing app version"
                    sh 'mvn build-helper:parse-version versions:set \
                       -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                       versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }
        stage('build app') {
            steps {
                echo 'building application jar...'
                buildJar()
            }
        }
        stage('build image') {
            steps {
                script {
                    echo 'building the docker image...'
                    buildImage(env.IMAGE_NAME)
                    dockerLogin()
                    dockerPush(env.IMAGE_NAME)
                }
            }
        } 
        stage("deploy") {
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                    def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}"
                    sshagent(['aws-ec2-server-key']) {
                        sh "scp server-cmds.sh ec2-user@3.81.55.173:/home/ec2-user"
                        sh "scp docker-compose.yaml ec2-user@3.81.55.173:/home/ec2-user" 
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@3.81.55.173 ${shellCmd}"
                    }
                }
            }               
        }
        stage('commit version update') {
              steps {
                  script {
                      withCredentials([usernamePassword(credentialsId: 'jenkinspush', passwordVariable: 'PAT' , usernameVariable: 'USER')]) {
                          sh "git remote set-url origin https://${PAT}@github.com/irschad/ci-cd-docker-compose-ec2-dynamic-versioning.git"
                          sh 'git config --global user.email "jenkins@example.com"'
                          sh 'git config --global user.name "jenkins"                  
                          sh 'git add .'
                          sh "git commit -m 'ci: version bump'"
                          sh 'git push origin HEAD:master'
                    }
                  }
              }
          }
    }
} 
