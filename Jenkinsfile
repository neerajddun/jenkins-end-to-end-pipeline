pipeline {

    agent any 

    tools { 

        maven 'Maven 3.8.7'
        jdk 'jdk 21'
    }

     environment {

        EKS_CLUSTER = "test-cluster"
        ECR_REGISTRY = "883999921903.dkr.ecr.ap-southeast-1.amazonaws.com"
        APP_NAME = "my-repo"
        IMAGE_TAG = "v1.${BUILD_NUMBER}"
    }

    stages {

        stage ('Checkout') {

            steps {

                checkout scm 
            }
        }

        stage ('Unit Test') {

            steps {

                sh 'mvn test'
            }
        }

        stage ('Integration Test') {

            steps {

                sh 'mvn clean verify -DskipTests -B'
            }
        }

        stage ('Build') {

            steps {

                sh 'mvn clean install -DskipTests'
            }
        }

        stage('SonarQube Scan') {
            
            steps {

                withSonarQubeEnv('SonarQube') {
                   
                   sh 'mvn sonar:sonar'

                }   
            }
        }

        stage('Quality Gate') {
            steps {
                sleep(time: 15, unit: 'SECONDS')

                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage ('Docker Image') {

            steps {

                sh "docker build -t ${APP_NAME}:${IMAGE_TAG} ."
                sh "docker tag ${APP_NAME}:${IMAGE_TAG} ${ECR_REGISTRY}/${APP_NAME}:${IMAGE_TAG}"
                sh "docker tag ${APP_NAME}:${IMAGE_TAG} ${ECR_REGISTRY}/${APP_NAME}:latest"
            }

        }

  /*      stage('OWASP Dependency-Check') {
          steps {
            sh 'mkdir -p ${WORKSPACE}/owasp-report'

             withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')]) {
             catchError(
                buildResult: 'SUCCESS',
                stageResult: 'UNSTABLE'
                )      {
                dependencyCheck(
                    odcInstallation: 'OWASP-DC',
                    additionalArguments:
                        '--scan ' + WORKSPACE +
                        ' --format HTML' +
                        ' --format XML' +
                        ' --out ' + WORKSPACE + '/owasp-report' +
                        ' --disableNodeAudit' +
                        ' --nvdApiKey ' + env.NVD_API_KEY
                    )
                }
            }

               dependencyCheckPublisher(
                 pattern: 'owasp-report/dependency-check-report.xml'
               )
            }
        }
*/

        stage('Trivy Image Scan') {
           steps {
              catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                 sh """
                    trivy image \
                      --exit-code 1 \
                      --severity CRITICAL \
                      --no-progress \
                      ${APP_NAME}:${IMAGE_TAG}
                 """
              }
           }

        post {
            always {
                sh """
                    trivy image \
                      --exit-code 0 \
                      --severity HIGH,CRITICAL \
                      --format json \
                      --output trivy-report.json \
                      ${APP_NAME}:${IMAGE_TAG}
                """

                  archiveArtifacts(
                      artifacts: 'trivy-report.json',
                      fingerprint: true
                  )
                }
            }
        }

        stage ('Docker push') {

            steps {

                script {
                     
                    withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-creds', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                     
                     sh """

                    aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    docker push ${ECR_REGISTRY}/${APP_NAME}:${IMAGE_TAG}   
                    docker push ${ECR_REGISTRY}/${APP_NAME}:latest
                     
                     """

                    }

                }
            }
        }


        stage('Deploy to EKS') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh """
                        aws eks update-kubeconfig --name test-cluster --region ap-southeast-1
                        envsubst < deployment.yaml | kubectl apply -f -
                        kubectl apply -f service.yaml
                        kubectl apply -f prometheusrule.yaml
                        kubectl apply -f service-monitor.yaml 
                        kubectl apply -f alertmanager-config.yaml
                    """
                }
            }
        }
    }

    post {
       
       always {

        cleanWs()
        }
     }

}