pipeline {
    agent any
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage ("Clean workspace") {
            steps {
                cleanWs()
            }
        }
        stage ("Git checkout") {
            steps {
                git branch: 'main', url: 'https://github.com/am2308/background-remover-python.git'
            }
        }
        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=background-remover-python-app \
                    -Dsonar.projectKey=background-remover-python-app '''
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('OWASP FS Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage ("Trivy File Scan") {
            steps {
                sh "trivy fs . > trivy.txt"
            }
        }
        stage ("Build Docker Image") {
            steps {
                sh "docker build -t background-remover-python-app ."
            }
        }
        stage ("Tag & Push to DockerHub") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker') {
                        sh "docker tag background-remover-python-app akhilmittal510/background-remover-python-app:latest"
                        sh "docker push akhilmittal510/background-remover-python-app:latest"
                    }
                }
            }
        }
        stage('Docker Scout Image') {
            steps {
                script {
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                       sh 'docker-scout quickview akhilmittal510/background-remover-python-app:latest'
                       sh 'docker-scout cves akhilmittal510/background-remover-python-app:latest'
                       sh 'docker-scout recommendations akhilmittal510/background-remover-python-app:latest'
                   }
                }
            }
        }
        stage ("Deploy to Container") {
            steps {
                sh 'docker run -d --name background-remover-python-app -p 5100:5100 akhilmittal510/background-remover-python-app:latest'
            }
        }
    }
}
