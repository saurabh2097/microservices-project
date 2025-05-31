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

        stage("Checkout") {
            steps {
                checkout scm
            }
        }

        stage("Install Dependencies") {
            steps {
                script {
                    if (fileExists('build.gradle') || fileExists('gradlew')) {
                        echo "Gradle project detected"
                        sh 'chmod +x ./gradlew'
                        sh './gradlew build --no-daemon'
                    } else if (fileExists('pom.xml')) {
                        echo "Maven project detected"
                        sh 'mvn clean package'
                    } else if (fileExists('requirements.in')) {
                        echo "Python project detected (requirements.in found)"
                        sh '''
                            pip install pip-tools   # install pip-compile tool
                            pip-compile requirements.in  # generate requirements.txt
                            pip install -r requirements.txt
                            '''
                    } else if (fileExists('requirements.txt')) {
                            echo "Python project detected (requirements.txt found)"
                            sh 'pip install -r requirements.txt'
                    } else if (fileExists('package.json')) {
                        echo "Node.js project detected"
                        sh 'npm install'
                        sh 'npm ci'
                    } else if (fileExists('go.mod')) {
                        echo "Go project detected"
                        sh '''
                            go mod tidy
                            go build -v ./...
                            go test ./...
                        '''
                    } else if (fileExists('cartservice.sln') || fileExists('cartservice.csproj')) {
                        echo ".NET project detected"
                        sh '''
                            dotnet restore
                            dotnet build --configuration Release
                        '''
                    } else {
                        error "Unknown project type. Cannot install dependencies."
                    }
                }
            }
        }

        stage("Generate Protobuf Code") {
            when {
                expression { fileExists('genproto.sh') }
            }
            steps {
                sh 'chmod +x genproto.sh && ./genproto.sh'
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
                sh 'pip install -r requirements.txt'
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
                        sh 'docker build -t saurabh021/loadgenerator:latest .'
                    }
                }
            }
        }

        stage("Push Docker Image") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker push saurabh021/loadgenerator:latest'
                    }
                }
            }
        }

        stage("Trivy - Image Scan") {
            steps {
                sh 'trivy image saurabh021/loadgenerator:latest'
            }
        }
    }
}
