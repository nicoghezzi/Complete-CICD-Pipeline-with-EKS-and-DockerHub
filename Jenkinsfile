#!/usr/bin/env groovy

pipeline {
    agent {
        docker {
            image 'maven:3.9.4-openjdk-17'
            args '-v /var/run/docker.sock:/var/run/docker.sock -v $HOME/.m2:/root/.m2 -v $WORKSPACE:/workspace'
        }
    }

    environment {
        DOCKER_REPO_SERVER = '943066268094.dkr.ecr.us-east-1.amazonaws.com'
        DOCKER_REPO = "${DOCKER_REPO_SERVER}/java-maven-app"
    }

    stages {
        stage('Increment Version') {
            steps {
                script {
                    echo 'Incrementing app version...'
                    // Make sure parsedVersion exists or wrap in try/catch
                    sh '''
                    mvn build-helper:parse-version versions:set \
                        -DnewVersion=${parsedVersion.majorVersion}.${parsedVersion.minorVersion}.${parsedVersion.nextIncrementalVersion} \
                        versions:commit
                    '''
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }

        stage('Build App') {
            steps {
                script {
                    echo 'Building the application...'
                    sh 'mvn clean package'
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    withCredentials([usernamePassword(
                        credentialsId: 'ecr-credentials',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )]) {
                        sh "docker build -t ${DOCKER_REPO}:${IMAGE_NAME} ."
                        sh 'echo $PASS | docker login -u $USER --password-stdin ${DOCKER_REPO_SERVER}'
                        sh "docker push ${DOCKER_REPO}:${IMAGE_NAME}"
                    }
                }
            }
        }

 stage('Deploy') {
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