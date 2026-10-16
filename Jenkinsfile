
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

        stage('Check Docker Access') {
            steps {
                bat 'whoami'
                bat 'docker version'
                bat 'docker info'
            }
        }

        stage('Docker Hub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDENTIALS_ID}",
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_PASSWORD'
                )]) {
                    powershell '''
                        $ErrorActionPreference = "Stop"

                        $username = $env:DOCKERHUB_USERNAME
                        $token = $env:DOCKERHUB_PASSWORD

                        if ([string]::IsNullOrWhiteSpace($username)) {
                            throw "Docker Hub username is missing."
                        }

                        if ([string]::IsNullOrWhiteSpace($token)) {
                            throw "Docker Hub token is missing."
                        }

                        Write-Host "Docker Hub username is present."
                        Write-Host "Docker Hub token is present."

                        $psi = New-Object System.Diagnostics.ProcessStartInfo
                        $psi.FileName = "docker.exe"
                        $psi.Arguments = "login -u $username --password-stdin"
                        $psi.UseShellExecute = $false
                        $psi.RedirectStandardInput = $true
                        $psi.RedirectStandardOutput = $true
                        $psi.RedirectStandardError = $true
                        $psi.CreateNoWindow = $true

                        $process = New-Object System.Diagnostics.Process
                        $process.StartInfo = $psi

                        [void]$process.Start()

                        $process.StandardInput.Write($token)
                        $process.StandardInput.WriteLine()
                        $process.StandardInput.Close()

                        $stdout = $process.StandardOutput.ReadToEnd()
                        $stderr = $process.StandardError.ReadToEnd()
                        $process.WaitForExit()

                        if ($stdout) {
                            Write-Host $stdout
                        }

                        if ($process.ExitCode -ne 0) {
                            throw "Docker Hub login failed. $stderr"
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
