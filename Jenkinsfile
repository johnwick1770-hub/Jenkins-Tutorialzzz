pipeline {
    agent any

    environment {
        ImageRegistry = 'shaahidgg'
        ImageRepository = 'php-contactform' 
        EC2_IP = '50.17.126.209'
        DockerComposeFile = 'docker-compose.yml'
        DotEnvFile = '.env'
    }

    stages {

        stage("buildImage") {
            steps {
                script {
                    echo "Building Docker Image..."
                    bat "docker build -t ${ImageRegistry}/${ImageRepository}:${BUILD_NUMBER} ."
                }
            }
        }

        stage('pushImage') {
    steps {
        echo 'Pushing Image to DockerHub...'

        withCredentials([
            usernamePassword(
                credentialsId: 'dockerhub-pat',
                usernameVariable: 'DOCKER_USER',
                passwordVariable: 'DOCKER_TOKEN'
            )
        ]) {
            powershell '''
                docker logout

                $env:DOCKER_TOKEN | docker login `
                    --username $env:DOCKER_USER `
                    --password-stdin

                if ($LASTEXITCODE -ne 0) {
                    throw "Docker Hub login failed."
                }

                docker push shaahidgg/php-contactform:${env:BUILD_NUMBER}

                if ($LASTEXITCODE -ne 0) {
                    throw "Docker image push failed."
                }
            '''
        }
    }
}

        stage("deployCompose") {
            steps {
                script {
                    echo "Deploying with Docker Compose..."
                    sshagent(['ec2']) {
                        // Upload files once to reduce redundant SCP commands
                        bat """
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