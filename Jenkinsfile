pipeline {
    agent any

    environment {
        ImageRegistry = 'professorweeb'
        EC2_IP = '54.171.233.251'
        DockerComposeFile = 'docker-compose.yml'
        DotEnvFile = '.env'
    }

    stages {

        stage("buildImage") {
            steps {
                script {
                    echo "Building Docker Image..."
                    bat "\"C:\\Users\\Danie_000\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe\" build -t ${ImageRegistry}/${JOB_NAME}:${BUILD_NUMBER} ."
                }
            }
        }

        stage("pushImage") {
            steps {
                script {
                    echo "Pushing Image to DockerHub..."
                    withCredentials([usernamePassword(credentialsId: 'docker-login', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push ${ImageRegistry}/${JOB_NAME}:${BUILD_NUMBER}"
                    }
                }
            }
        }

        stage("deployCompose") {
            steps {
                script {
                    echo "Deploying with Docker Compose..."
                    sshagent(['ec2']) {
                        // Upload files once to reduce redundant SCP commands
                        sh """
                        scp -o StrictHostKeyChecking=no ${DotEnvFile} ${DockerComposeFile} ubuntu@${EC2_IP}:/home/ubuntu
                        ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} "docker compose -f /home/ubuntu/${DockerComposeFile} --env-file /home/ubuntu/${DotEnvFile} down"
                        ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} "docker compose -f /home/ubuntu/${DockerComposeFile} --env-file /home/ubuntu/${DotEnvFile} up -d"
                        """
                    }
                }
            }
        }
    }
}
