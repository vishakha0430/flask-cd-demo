pipeline {
    agent any

    stages {

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-cd-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    bat 'docker build -t %DOCKER_USER%/flask-demo:%BUILD_NUMBER% .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Pushing Docker image to Docker Hub...'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-cd-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    ),
                    string(
                        credentialsId: 'dockerhub-token',
                        variable: 'DOCKER_TOKEN_SAFE'
                    )
                ]) {
                    bat '''
                        setlocal DisableDelayedExpansion
                        <nul set /p="%DOCKER_TOKEN_SAFE%" | docker login -u "%DOCKER_USER%" --password-stdin
                        docker push %DOCKER_USER%/flask-demo:%BUILD_NUMBER%
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying application to Kubernetes...'

                withCredentials([
                    file(
                        credentialsId: 'kubeconfig-creds-new',,
                        variable: 'KUBECONFIG'
                    ),
                    usernamePassword(
                        credentialsId: 'dockerhub-cd-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    bat 'kubectl set image deployment/web-deploy flask-demo=%DOCKER_USER%/flask-demo:%BUILD_NUMBER%'
                    bat 'kubectl rollout status deployment/web-deploy'
                }
            }
        }
    }

    post {
        success {
            echo 'Continuous Deployment completed successfully!'
        }

        failure {
            echo 'Pipeline failed.'
        }
    }
}
