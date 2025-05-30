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
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage("Install Dependencies") {
            steps {
                withEnv(["PATH=/opt/gradle/gradle-8.5/bin:${env.PATH}"]) {
                    sh 'gradle build'
                }  // <-- added missing closing brace here for withEnv
            }      // <-- added missing closing brace here for steps
        }          // <-- added missing closing brace here for stage

        stage("Trivy - File System Scan") {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }

        stage("Build & Tag Docker Image") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker build -t saurabh021/adservice:latest .'
                    }
                }
            }
        }

        stage("Push Docker Image") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker push saurabh021/adservice:latest'
                    }
                }
            }
        }

        stage("Trivy - Image Scan") {
            steps {
                sh 'trivy image saurabh021/adservice:latest'
            }
        }
    }
}
