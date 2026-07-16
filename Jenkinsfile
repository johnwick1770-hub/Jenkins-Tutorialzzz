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
        echo 'Pushing Image to Docker Hub...'

        withCredentials([
            usernamePassword(
                credentialsId: 'dockerhub-pat',
                usernameVariable: 'DOCKER_USER',
                passwordVariable: 'DOCKER_TOKEN'
            )
        ]) {
            bat '''
                @echo off

                docker logout >nul 2>&1

                echo %DOCKER_TOKEN%| docker login --username %DOCKER_USER% --password-stdin

                if errorlevel 1 (
                    echo Docker Hub login failed.
                    exit /b 1
                )

                docker push shaahidgg/php-contactform:%BUILD_NUMBER%

                if errorlevel 1 (
                    echo Docker image push failed.
                    exit /b 1
                )
            '''
        }
    }
}

        stage("deployCompose") {
    steps {
        echo "Deploying with Docker Compose..."

        withCredentials([
            sshUserPrivateKey(
                credentialsId: 'ec2',
                keyFileVariable: 'EC2_KEY',
                usernameVariable: 'EC2_USER'
            )
        ]) {
            bat '''
                @echo off

                scp -i "%EC2_KEY%" ^
                    -o StrictHostKeyChecking=no ^
                    "%DotEnvFile%" "%DockerComposeFile%" ^
                    "%EC2_USER%@%EC2_IP%:/home/ubuntu/"

                if errorlevel 1 (
                    echo File upload failed.
                    exit /b 1
                )

                ssh -i "%EC2_KEY%" ^
                    -o StrictHostKeyChecking=no ^
                    "%EC2_USER%@%EC2_IP%" ^
                    "docker compose -f /home/ubuntu/%DockerComposeFile% --env-file /home/ubuntu/%DotEnvFile% down"

                if errorlevel 1 (
                    echo Docker Compose down failed.
                    exit /b 1
                )

                ssh -i "%EC2_KEY%" ^
                    -o StrictHostKeyChecking=no ^
                    "%EC2_USER%@%EC2_IP%" ^
                    "docker compose -f /home/ubuntu/%DockerComposeFile% --env-file /home/ubuntu/%DotEnvFile% pull && docker compose -f /home/ubuntu/%DockerComposeFile% --env-file /home/ubuntu/%DotEnvFile% up -d"

                if errorlevel 1 (
                    echo Docker Compose deployment failed.
                    exit /b 1
                )
            '''
        }
    }
}
    }
}