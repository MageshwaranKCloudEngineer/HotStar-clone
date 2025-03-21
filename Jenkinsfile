pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'Sonar-Scanner'
    }

    tools {
        jdk 'jdk'
        nodejs 'Nodejs'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/gashok13193/HotStar-clone.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('Sonarqube') {
                    sh '''
                    "${SCANNER_HOME}/bin/sonar-scanner" \
                    -Dsonar.projectName=Hotstar \
                    -Dsonar.projectKey=Hotstar \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=.
                    '''
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }

        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Trivy FS Scan') {
            steps {
                script {
                    sh 'trivy fs --severity HIGH,CRITICAL ./ --format table --output trivy-fs-report.txt'
                }
            }
        }
        
        stage('Docker Build') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '4b0b35be-a68f-4791-9733-34193a997ad7', toolName: 'Docker') {
                        sh "docker build -t hotstar ."
                        sh "docker tag hotstar magesh506/hotstar:latest"
                        sh "docker push magesh506/hotstar:latest"
                    }
                }
            }
        }
        
        stage('Trivy Image Scan') {
            steps {
                script {
                    sh 'trivy image --severity HIGH,CRITICAL magesh506/hotstar:latest --format table --output trivy-image-report.txt'
                }
            }
        }

        stage('Deploy Docker') {
            steps {
                sh "docker run --rm -d --name hotstar -p 3000:3000 magesh506/hotstar:latest"
            }
        }
    }
}
