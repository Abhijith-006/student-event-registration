pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'abhijith006/student-event-registration'
    }

    stages {

        stage('Clone Code') {
            steps {
                echo 'Code is already checked out by Jenkins.'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:%BUILD_NUMBER% .'
                bat 'docker tag %DOCKER_IMAGE%:%BUILD_NUMBER% %DOCKER_IMAGE%:latest'
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'dockerhub-token',
                        variable: 'DOCKER_TOKEN'
                    )
                ]) {

                    bat '''
                        powershell -NoProfile -Command "$env:DOCKER_TOKEN | docker login -u 'abhijith006' --password-stdin"
                    '''

                    bat 'docker push %DOCKER_IMAGE%:%BUILD_NUMBER%'
                    bat 'docker push %DOCKER_IMAGE%:latest'

                    bat 'docker logout'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kubeconfig',
                        variable: 'KUBECONFIG'
                    )
                ]) {

                    bat 'kubectl apply -f deployment.yaml'

                    bat 'kubectl set image deployment/student-event student-event=%DOCKER_IMAGE%:%BUILD_NUMBER%'

                    bat 'kubectl rollout status deployment/student-event'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kubeconfig',
                        variable: 'KUBECONFIG'
                    )
                ]) {

                    bat 'kubectl get deployments'
                    bat 'kubectl get pods -o wide'
                    bat 'kubectl get services'
                }
            }
        }
    }
}
