#!/usr/bin/env groovy

pipeline {
    agent any

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
                      mvn build-helper:parse-version \
                      versions:set -DnewVersion=1.0.${BUILD_NUMBER} \
                      versions:commit
                    '''

                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "${version}-${BUILD_NUMBER}"
                }
            }
        }

        stage('build app') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('build image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'ecr-credentials',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh "docker build -t ${DOCKER_REPO}:${IMAGE_NAME} ."
                    sh "echo ${PASS} | docker login -u ${USER} --password-stdin ${DOCKER_REPO_SERVER}"
                    sh "docker push ${DOCKER_REPO}:${IMAGE_NAME}"
                }
            }
        }

        stage('deploy') {
            environment {
                AWS_ACCESS_KEY_ID     = credentials('jenkins_aws_access_key_id')
                AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws_secret_access_key')
            }
            steps {
                sh 'envsubst < kubernetes/deployment.yaml | kubectl apply -f -'
                sh 'envsubst < kubernetes/service.yaml | kubectl apply -f -'
            }
        }

        stage('commit version update') {
            steps {
                echo 'skipping git commit step for now'
            }
        }
    }
}