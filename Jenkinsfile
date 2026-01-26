#!/usr/bin/env groovy

pipeline {
    agent {
        docker {
            image 'maven:3.9.4-openjdk-17'  // Maven + Java pre-installed
            args '-v $HOME/.m2:/root/.m2'   // persist Maven cache
        }
    }

    environment {
        DOCKER_REPO_SERVER = '943066268094.dkr.ecr.us-east-1.amazonaws.com'
        DOCKER_REPO = "${DOCKER_REPO_SERVER}/java-maven-app"
    }

    stages {
        stage('increment version') {
            steps {
                script {
                    echo 'incrementing app version...'
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

        stage('build app') {
            steps {
                script {
                    echo 'building the application...'
                    sh 'mvn clean package'
                }
            }
        }

        stage('build image') {
            steps {
                script {
                    echo "building the docker image..."
                    withCredentials([usernamePassword(
                        credentialsId: 'ecr-credentials',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )]){
                        sh "docker build -t ${DOCKER_REPO}:${IMAGE_NAME} ."
                        sh 'echo $PASS | docker login -u $USER --password-stdin ${DOCKER_REPO_SERVER}'
                        sh "docker push ${DOCKER_REPO}:${IMAGE_NAME}"
                    }
                }
            }
        }

        stage('deploy') {
            environment {
                AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
                AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
                APP_NAME = 'java-maven-app'
            }
            steps {
                script {
                    echo 'deploying docker image...'
                    sh 'envsubst < kubernetes/deployment.yaml | kubectl apply -f -'
                    sh 'envsubst < kubernetes/service.yaml | kubectl apply -f -'
                }
            }
        }

        stage('commit version update') {
            steps {
                script {
                    echo 'skipping git commit step for now'
                }
            }        
        }
    }
}