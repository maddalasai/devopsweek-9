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
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_PASSWORD'
                )]) {
                    bat 'echo %DOCKERHUB_PASSWORD% | docker login -u %DOCKERHUB_USERNAME% --password-stdin'
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
