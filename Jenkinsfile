pipeline {

    agent any 

    tools {

        jdk 'jdk 17'
        maven 'Maven 3.8.7'
    }

    environment {

        ECR_REGISTRY = "883999921903.dkr.ecr.ap-southeast-1.amazonaws.com"
        IMAGE_TAG = "v1.${BUILD_NUMBER}"
        APP_NAME = "my-repo"
    }

    stages {

        stage ('Checkout') {
            
            steps {

                sh 'checkout scm'
            }
        }

        stage ('Unit test') {

            steps {

                sh 'mvn test'
            }
        }

        stage ('Integration Test') {

            steps {

                sh 'mvn verify -DskipTests -B'
            }
        }

        stage ('Build') {

            steps {

                sh 'mvn clean install'
            }
        }

        stage ('Docker Image') {

            steps {

                sh '''

                docker build -t ${APP_NAME}.${IMAGE_TAG} .

                docker tag ${APP_NAME}:${IMAGE_TAG} ${ECR_REGISTRY}/${APP_NAME}:${IMAGE_TAG}

                docker tag ${APP_NAME}:${IMAGE_TAG} ${ECR_REGISTRY}}/${APP_NAME}:latest

                '''
            }
        }
    }
}