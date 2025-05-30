pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'mysonar'
    }

    stages {
        stage("Clean WS") {
            steps {
                cleanWs()
            }
        }

        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('mysonar') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=online \
                        -Dsonar.projectKey=online
                    '''
                }
            }
        }

        stage("Quality Gates") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }

        stage("Install Dependencies") {
            steps {
                sh 'npm install'
            }
        }

        stage("Trivy - File System Scan") {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }

        stage("Build & Tag Docker Image") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker build -t saurabh021/paymentservice:latest .'
                    }
                }
            }
        }

        stage("Push Docker Image") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker push saurabh021/paymentservice:latest'
                    }
                }
            }
        }

        stage("Trivy - Image Scan") {
            steps {
                sh 'trivy image saurabh021/paymentservice:latest'
            }
        }
    }
}
