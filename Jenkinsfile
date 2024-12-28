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
                    def shellCmd = "bash ./server-cmds.sh"
                    sshagent(['aws-ec2-server-key']) {
                        sh "scp server-cmds.sh ec2-user@3.81.55.173:/home/ec2-user"
                        sh "scp docker-compose.yaml ec2-user@3.81.55.173:/home/ec2-user" 
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@3.81.55.173 ${shellCmd}"
                    }
                }
            }               
        }
    }
} 
