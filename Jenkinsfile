pipeline {
    agent any

    environment {
        DOCKER_REPO_SERVER = '943066268094.dkr.ecr.us-east-1.amazonaws.com'
        DOCKER_REPO = "${DOCKER_REPO_SERVER}/java-maven-app"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Maven App') {
            steps {
                echo 'Building Maven app (no Docker, no tricks)'
                sh 'mvn clean package'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'ecr-credentials',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                    docker build -t ${DOCKER_REPO}:${IMAGE_TAG} .
                    echo $PASS | docker login -u $USER --password-stdin ${DOCKER_REPO_SERVER}
                    docker push ${DOCKER_REPO}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-access-key-id',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                    # Replace IMAGE_TAG in YAML before applying
                    envsubst < kubernetes/deployment.yaml | kubectl apply -f -
                    envsubst < kubernetes/service.yaml | kubectl apply -f -
                    '''
                }
            }
        }
    }
}
