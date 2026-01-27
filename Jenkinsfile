pipeline {
    agent any
    environment {
        DOCKER_REPO_SERVER = '943066268094.dkr.ecr.us-east-1.amazonaws.com'
        DOCKER_REPO = "${DOCKER_REPO_SERVER}/java-maven-app"
        IMAGE_TAG = "latest"  // can replace with ${BUILD_NUMBER} if you want versioning
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code'
                checkout scm
            }
        }

        stage('Build Maven App') {
            steps {
                echo 'Building Maven app'
                sh 'mvn clean package'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                echo 'Building Docker image and pushing to ECR'
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
                echo 'Deploying to Kubernetes cluster'
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-access-key-id',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                    aws eks update-kubeconfig --region us-east-1 --name my-eks-cluster
                    kubectl apply -f kubernetes/deployment.yaml
                    kubectl apply -f kubernetes/service.yaml
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline finished successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs!"
        }
    }
}

