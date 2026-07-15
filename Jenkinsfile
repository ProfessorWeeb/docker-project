pipeline {
    agent any

    environment {
        ImageRegistry = 'professorweeb'
        ImageName = 'a6-contactform'

        DockerExe = 'C:\\Users\\Danie_000\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'

        // Replace this after creating your EC2 instance.
        EC2_IP = 'REPLACE_WITH_YOUR_EC2_PUBLIC_IP'

        DockerComposeFile = 'docker-compose.yml'
        DotEnvFile = '.env'
    }

    stages {
        stage('buildImage') {
            steps {
                script {
                    echo 'Building Docker image...'

                    bat '''
                    "%DockerExe%" build ^
                    -t %ImageRegistry%/%ImageName%:%BUILD_NUMBER% ^
                    -t %ImageRegistry%/%ImageName%:latest .
                    '''
                }
            }
        }

        stage('pushImage') {
            steps {
                script {
                    echo 'Pushing image to Docker Hub...'

                    withCredentials([
                        usernamePassword(
                            credentialsId: 'jendoclog',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )
                    ]) {
                        bat '''
                        echo %DOCKER_PASS% | "%DockerExe%" login -u %DOCKER_USER% --password-stdin

                        "%DockerExe%" push %ImageRegistry%/%ImageName%:%BUILD_NUMBER%
                        "%DockerExe%" push %ImageRegistry%/%ImageName%:latest

                        "%DockerExe%" logout
                        '''
                    }
                }
            }
        }

        stage('deployCompose') {
            steps {
                script {
                    echo 'Deploying with Docker Compose...'

                    sshagent(credentials: ['ec2']) {
                        bat '''
                        scp -o StrictHostKeyChecking=no "%DotEnvFile%" "%DockerComposeFile%" ubuntu@%EC2_IP%:/home/ubuntu/

                        ssh -o StrictHostKeyChecking=no ubuntu@%EC2_IP% "docker compose -f /home/ubuntu/%DockerComposeFile% --env-file /home/ubuntu/%DotEnvFile% down"

                        ssh -o StrictHostKeyChecking=no ubuntu@%EC2_IP% "docker compose -f /home/ubuntu/%DockerComposeFile% --env-file /home/ubuntu/%DotEnvFile% pull"

                        ssh -o StrictHostKeyChecking=no ubuntu@%EC2_IP% "docker compose -f /home/ubuntu/%DockerComposeFile% --env-file /home/ubuntu/%DotEnvFile% up -d"
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'The image was built, pushed, and deployed successfully.'
        }

        failure {
            echo 'The pipeline failed. Check the Console Output for the first error.'
        }
    }
}
