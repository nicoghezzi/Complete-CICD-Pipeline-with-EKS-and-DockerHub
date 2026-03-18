pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        DOCKER_REPO_SERVER = '943066268094.dkr.ecr.us-east-1.amazonaws.com'
        DOCKER_REPO = "${DOCKER_REPO_SERVER}/java-maven-app"
        IMAGE_TAG = "${BUILD_NUMBER}"   // Immutable tag
    }

    options {
        timestamps()
        disableConcurrentBuilds()
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
                sh 'mvn clean package -DskipTests=false'
            }
        }

        stage('Run Unit Tests') {
            steps {
                echo 'Running tests'
                sh 'mvn test'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image'
                sh '''
                docker build -t ${DOCKER_REPO}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Scan Docker Image') {
            steps {
                echo 'Scanning image for vulnerabilities'
                sh '''
                trivy image --exit-code 1 --severity HIGH,CRITICAL ${DOCKER_REPO}:${IMAGE_TAG}
                '''
            }
        }

        stage('Authenticate to AWS (IAM Role)') {
            steps {
                echo 'Using IAM role for authentication'
                // Assumes Jenkins agent has IAM role attached (no static credentials)
                sh '''
                aws sts get-caller-identity
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                echo 'Logging into ECR'
                sh '''
                aws ecr get-login-password --region ${AWS_REGION} \
                | docker login --username AWS --password-stdin ${DOCKER_REPO_SERVER}
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Pushing image to ECR'
                sh '''
                docker push ${DOCKER_REPO}:${IMAGE_TAG}
                '''
            }
        }

        stage('Manual Approval') {
            steps {
                input message: "Approve deployment to Kubernetes?", ok: "Deploy"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes cluster'

                withEnv(["IMAGE=${DOCKER_REPO}:${IMAGE_TAG}"]) {
                    sh '''
                    aws eks update-kubeconfig --region ${AWS_REGION} --name demo-cluster

                    # Replace image dynamically (avoid hardcoding in YAML)
                    sed -i "s|IMAGE_PLACEHOLDER|${IMAGE}|g" kubernetes/deployment.yaml

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
        always {
            cleanWs()
        }
    }
}
