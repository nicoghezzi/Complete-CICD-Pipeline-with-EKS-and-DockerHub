#!/usr/bin/env groovy

pipeline {
    agent any

    environment {
        DOCKER_REPO_SERVER = '943066268094.dkr.ecr.us-east-1.amazonaws.com'
        DOCKER_REPO = "${DOCKER_REPO_SERVER}/java-maven-app"
        IMAGE_NAME = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Maven App') {
            steps {
                echo 'Building Maven app inside Docker...'
                sh '''
                docker run --rm \
                    -v $WORKSPACE:/workspace \
                    -v $HOME/.m2:/root/.m2 \
                    -w /workspace \
                    maven:3.9.4-eclipse-temurin-17 \
                    mvn clean package
                '''
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                echo 'Building and pushing Docker image...'
                withCredentials([usernamePassword(
                    credentialsId: 'ecr-credentials',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                    docker build -t ${DOCKER_REPO}:${IMAGE_NAME} .
                    echo $PASS | docker login -u $USER --password-stdin ${DOCKER_REPO_SERVER}
                    docker push ${DOCKER_REPO}:${IMAGE_NAME}
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying Docker image to Kubernetes...'
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-access-key-id',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                    docker run --rm \
                        -v $HOME/.kube:/root/.kube \
                        -v $WORKSPACE:/workspace \
                        -e AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID \
                        -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY \
                        bitnami/kubectl:latest \
                        sh -c "envsubst < /workspace/kubernetes/deployment.yaml | kubectl apply -f -"

                    docker run --rm \
                        -v $HOME/.kube:/root/.kube \
                        -v $WORKSPACE:/workspace \
                        -e AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID \
                        -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY \
                        bitnami/kubectl:latest \
                        sh -c "envsubst < /workspace/kubernetes/service.yaml | kubectl apply -f -"
                    '''
                }
            }
        }

        stage('Commit Version Update') {
            steps {
                echo 'Skipping Git commit step for now'
            }
        }
    }
}
