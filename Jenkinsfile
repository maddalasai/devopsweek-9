pipeline {
    agent any

    environment {
        DOCKER_IMAGE = '141034durga/week9-registration:latest'
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out the Week 9 project'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE% .'
            }
        }

        stage('Docker Hub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDENTIALS_ID}",
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_PASSWORD'
                )]) {
                    // Diagnostic: shows what Jenkins actually holds (no secret printed)
                    powershell '''
                        $u = $env:DOCKERHUB_USERNAME
                        $p = $env:DOCKERHUB_PASSWORD
                        Write-Host "Username: [$u] length=$($u.Length)"
                        Write-Host "Token length: $($p.Length)"
                        Write-Host "Starts with dckr_pat_: $($p.StartsWith('dckr_pat_'))"
                        Write-Host "Has leading/trailing whitespace: $($p -ne $p.Trim())"
                    '''

                    // Actual login
                    powershell '''
                        $ErrorActionPreference = "Stop"

                        if ([string]::IsNullOrWhiteSpace($env:DOCKERHUB_USERNAME)) {
                            throw "Docker Hub username is empty"
                        }

                        if ([string]::IsNullOrWhiteSpace($env:DOCKERHUB_PASSWORD)) {
                            throw "Docker Hub token is empty"
                        }

                        $env:DOCKERHUB_PASSWORD |
                            docker login -u $env:DOCKERHUB_USERNAME --password-stdin

                        if ($LASTEXITCODE -ne 0) {
                            throw "Docker Hub login failed with exit code $LASTEXITCODE"
                        }

                        Write-Host "Docker Hub login successful."
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                bat 'docker push %DOCKER_IMAGE%'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat 'kubectl apply -f deployment.yaml'
                bat 'kubectl apply -f service.yaml'
            }
        }

        stage('Verify Deployment') {
            steps {
                bat 'kubectl get deployments'
                bat 'kubectl get pods'
                bat 'kubectl get services'
            }
        }
    }

    post {
        success {
            echo 'Week 9 pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check the Jenkins Console Output.'
        }

        always {
            bat 'docker logout'
        }
    }
}
