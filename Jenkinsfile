pipeline {
    agent any

    environment {
        IMAGE_NAME = 'vivekpn6/flask-cicd-app'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh '''
                    python3 -m venv .venv
                    .venv/bin/pip install -r requirements.txt pytest
                    .venv/bin/pytest -v
                '''
            }
        }

        stage('Build Image') {
            steps {
                sh '''
                    docker build \
                      -t ${IMAGE_NAME}:${GIT_COMMIT} \
                      -t ${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    sh '''
                        set +x
                        echo "$DOCKER_TOKEN" | docker login \
                          --username "$DOCKER_USER" --password-stdin

                        docker push ${IMAGE_NAME}:${GIT_COMMIT}
                        docker push ${IMAGE_NAME}:latest

                        docker logout
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'CI pipeline completed. Check the stage results above.'
        }
    }
}
