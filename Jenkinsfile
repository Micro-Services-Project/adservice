pipeline {

    agent any

    environment {
        IMAGE_NAME = "obitomanu/adservice:${GIT_COMMIT}"
    }

    stages {

        stage("CleanWS") {
            steps {
                cleanWs()
            }
        }

        stage("Git-Checkout") {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Micro-Services-Project/adservice.git'
            }
        }

        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('Sonar') {
                    sh '''
                          -Dsonar.projectKey=adservice \
                          -Dsonar.projectName=adservice
                    '''
                }
            }
        }

        stage("Quality Gate") {
            steps {
                waitForQualityGate abortPipeline: true, credentialsId: 'Sonar'
            }
        }

        stage("Build") {
            steps {
                sh '''
                    printenv
                    docker build -t ${IMAGE_NAME} .
                '''
            }
        }

        stage("Scan") {
            steps {
                sh '''
                    trivy image ${IMAGE_NAME} > adservice-report.txt
                '''
            }
        }

        stage("Push Image") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker') {
                        sh '''
                            docker push ${IMAGE_NAME}
                        '''
                    }
                }
            }
        }
    }

    post {

        always {
            sh "docker rmi ${IMAGE_NAME} || true"
            sh "docker logout || true"
        }

        success {
            echo "Build and push successful: ${IMAGE_NAME}"
        }

        failure {
            echo "Pipeline failed. Check the logs above."
        }
    }
}